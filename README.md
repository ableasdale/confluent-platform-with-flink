# Confluent Platform with Apache Flink

- Confluent Platform 7.5.1 (TODO)
- Apache Flink 1.17.1

## Getting Started

Build the container instances and start them:

```bash
docker-compose build --pull sql-client
docker-compose build --pull jobmanager
docker-compose up
```

After the containers have started, data should be getting loaded into Kafka; to confirm this, run:

```bash
docker-compose exec kafka bash -c 'kafka-console-consumer.sh --topic user_behavior --bootstrap-server kafka:9094 --from-beginning --max-messages 10'
```
docker-compose exec broker bash -c 'kafka-console-consumer --topic user_behavior --bootstrap-server broker:29092 --from-beginning --max-messages 10'


Now let's connect to the Flink SQL Client:

```bash
docker-compose exec sql-client sql-client.sh
```

Let's create a TABLE for the topic data:

```sql
CREATE TABLE user_behavior (
    user_id BIGINT,
    item_id BIGINT,
    category_id BIGINT,
    behavior STRING,
    ts TIMESTAMP(3),
    proctime AS PROCTIME(),   -- generates processing-time attribute using computed column
    WATERMARK FOR ts AS ts - INTERVAL '5' SECOND  -- defines watermark on ts column, marks ts as event-time attribute
) WITH (
    'connector' = 'kafka',  -- using kafka connector
    'topic' = 'user_behavior',  -- kafka topic
    'scan.startup.mode' = 'earliest-offset',  -- reading from the beginning
    'properties.bootstrap.servers' = 'broker:29092',  -- kafka broker address
    'format' = 'json'  -- the data format is json
);
```

You should see:

```
[INFO] Table has been created.
```

Let's try running a SQL `SELECT` query:

```sql
SELECT * FROM user_behavior;
```



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

```
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
