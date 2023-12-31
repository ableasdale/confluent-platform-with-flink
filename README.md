# Confluent Platform with Apache Flink

This project will set up the following components:

- Confluent Platform 7.5.1
  - broker
  - control-center
  - restproxy
  - connect

- Apache Flink 1.18
  - sql-client
  - jobmanager
  - taskmanager

When the project is running, you'll be able to access the following URLs in your browser:

- Confluent Control Center: <http://localhost:9021/>
- Apache Flink Dashboard: <http://localhost:8081/>

And you have access to a number of ReST APIs:

- ReST Proxy <http://localhost:8082>
- Kafka Connect: <http://localhost:8083>
- Schema Registry <http://localhost:8084>
- Broker <http://localhost:8090>

## Getting Started

Build the container instances and start them:

```bash
docker-compose build --pull sql-client
docker-compose build --pull jobmanager
docker-compose up
```

Confirm that the various endpoints are responding:

ReST Proxy:

```bash
curl -XGET http://localhost:8082/v3/clusters | jq
```

Connect:

```bash
curl -XGET http://localhost:8083/ | jq
```

Schema Registry:

```bash
curl -XGET http://localhost:8084/config | jq
```

Broker Metadata:

```bash
curl -XGET http://localhost:8090/v1/metadata/id | jq
```

### Generate some data

After the containers have started, we can start by getting data loaded into Kafka; to do this, let's use datagen to create some pageviews:

```bash
curl -i -X PUT http://localhost:8083/connectors/datagen_local_01/config \
     -H "Content-Type: application/json" \
     -d '{
            "connector.class": "io.confluent.kafka.connect.datagen.DatagenConnector",
            "key.converter": "org.apache.kafka.connect.storage.StringConverter",
            "kafka.topic": "pageviews",
            "quickstart": "pageviews",
            "max.interval": 1000,
            "iterations": 10000000,
            "tasks.max": "1"
        }'
```

As soon as that is done, let's run the consumer to inspect the topic:

```bash
docker-compose exec broker kafka-console-consumer --topic pageviews --bootstrap-server broker:29092 --from-beginning --max-messages 10
```

The messages are not looking quite right; let's use the `kafka-avro-console-consumer` instead:

```bash
docker-compose exec connect kafka-avro-console-consumer \
 --bootstrap-server broker:29092 \
 --property schema.registry.url=http://schemaregistry:8084 \
 --topic pageviews \
 --property print.key=true \
 --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer \
 --property key.separator=" : " \
 --max-messages 10
```

That works nicely... Let's see how Flink handles Avro:

```bash
docker-compose exec sql-client sql-client.sh
```

To create the `pageviews` table, you can use this:

```sql
CREATE TABLE pageviews (
    viewtime BIGINT,
    userid STRING,
    pageid STRING,
    `ts` TIMESTAMP(3) METADATA FROM 'timestamp',
    `proc_time` AS PROCTIME(),
    WATERMARK FOR `ts` AS `ts` 
) WITH (
    'connector' = 'kafka', 
    'topic' = 'pageviews', 
    'scan.startup.mode' = 'earliest-offset', 
    'properties.bootstrap.servers' = 'broker:29092', 
    'value.format' = 'avro-confluent',
    'value.avro-confluent.schema-registry.url' = 'http://schemaregistry:8084'
);
```

Now we can create a `SELECT` statement using the `pageviews` topic as the source:

```sql
SELECT * FROM pageviews;
```

We get some output! :)


### Set up a `join`

We're going to use user_id for this join

```bash
curl -i -X PUT http://localhost:8083/connectors/datagen_users/config \
     -H "Content-Type: application/json" \
     -d '{
            "connector.class": "io.confluent.kafka.connect.datagen.DatagenConnector",
            "kafka.topic": "users",
            "quickstart": "users",
            "key.converter": "org.apache.kafka.connect.storage.StringConverter",
            "max.interval": 1000,
            "iterations": 10000000,
            "tasks.max": "1"
        }'
```

docker-compose exec sql-client sql-client.sh

```sql
CREATE TABLE users (
    registertime BIGINT,
    userid STRING,
    regionid STRING,
    gender STRING,
    `ts` TIMESTAMP(3) METADATA FROM 'timestamp',
    `proc_time` AS PROCTIME(),
    WATERMARK FOR `ts` AS `ts` 
) WITH (
    'connector' = 'kafka', 
    'topic' = 'users', 
    'scan.startup.mode' = 'earliest-offset', 
    'properties.bootstrap.servers' = 'broker:29092', 
    'value.format' = 'avro-confluent',
    'value.avro-confluent.schema-registry.url' = 'http://schemaregistry:8084'
);
```

You should see:

```
[INFO] Execute statement succeed.
```

```sql
SELECT * FROM users;
```

Let's create the `stock-trades` topic and load some data into it:

```bash
curl -i -X PUT http://localhost:8083/connectors/datagen_stock/config \
     -H "Content-Type: application/json" \
     -d '{
            "connector.class": "io.confluent.kafka.connect.datagen.DatagenConnector",
            "kafka.topic": "stock-trades",
            "quickstart": "Stock_Trades",
            "key.converter": "org.apache.kafka.connect.storage.StringConverter",
            "max.interval": 100,
            "iterations": 10000000,
            "tasks.max": "1"
        }'
```

docker-compose exec sql-client sql-client.sh

```sql
CREATE TABLE stock_trades (
    side STRING,
    quantity INT,
    symbol STRING,
    price INT,
    account STRING,
    userid STRING,
    regionid STRING,
    `ts` TIMESTAMP(3) METADATA FROM 'timestamp',
    `proc_time` AS PROCTIME(),
    WATERMARK FOR `ts` AS `ts` 
) WITH (
    'connector' = 'kafka', 
    'topic' = 'stock-trades', 
    'scan.startup.mode' = 'earliest-offset', 
    'properties.bootstrap.servers' = 'broker:29092', 
    'value.format' = 'avro-confluent',
    'value.avro-confluent.schema-registry.url' = 'http://schemaregistry:8084'
);
```

```sql
SELECT * FROM stock_trades;
```

Let's now attempt a join!

```sql
SELECT * FROM stock_trades
  INNER JOIN users
  ON stock_trades.userid = users.userid;
```

Seems to be working as expected.

-----------------------------
### Notes below



OR:

```bash
docker-compose build --pull sql-client
```

Start the containers:

```bash
docker-compose up
```

Make sure there are some messages in the `user_behavior` topic:

```bash
docker-compose exec kafka bash -c 'kafka-console-consumer.sh --topic user_behavior --bootstrap-server kafka:9094 --from-beginning --max-messages 10'
```

If you see messages in the Kafka topic, the next thing to do is to access the topic from within Flink:

```bash
docker-compose exec sql-client ./sql-client.sh
```

You should see:

```
Reading default environment from: file:/opt/flink/conf/sql-client-conf.yaml
```



You'll start to see results paging through as a result of the `SELECT` statement.


docker-compose exec sql-client bash



```bash
cd $SQL_CLIENT_HOME/lib
cp flink-sql-connector-kafka-3.0.0-1.17.jar ../../flink/lib/
cd ../../flink/lib/
chown flink:flink flink-sql-connector-kafka-3.0.0-1.17.jar
exit
```
```bash
docker-compose exec broker kafka-console-consumer --topic user_behavior --bootstrap-server broker:29092 --from-beginning --max-messages 10
```
Now let's connect to the Flink SQL Client:

```bash
docker-compose exec sql-client sql-client.sh
```