# Build End-to-End Streaming Application using Flink SQL、Kafka、MySQL、Elasticsearch and Kibana.

<img width="451" alt="图片2" src="https://user-images.githubusercontent.com/5378924/79943461-838bdc80-849b-11ea-81f4-b28b31e03176.png">

You can download the data here: https://drive.google.com/file/d/1P4BAZnpCY-knMt5HFfcx3onVjykYt3et/view?usp=sharing

This is a repository to build the dockers which will be used in the tuturial.

Blog: https://flink.apache.org/2020/07/28/flink-sql-demo-building-e2e-streaming-application.html

## Steps to test

Build the SQL Client instance:

```bash
docker build sql-client
```

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
    'properties.bootstrap.servers' = 'kafka:9094',  -- kafka broker address
    'format' = 'json'  -- the data format is json
);
```

You should see:

```
[INFO] Table has been created.
```

Let's try running a query:

```sql
SELECT * FROM user_behavior;
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