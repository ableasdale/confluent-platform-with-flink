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
    'properties.bootstrap.servers' = 'kafka:9094',  -- kafka broker address
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



```

jobmanager                                     | 2023-10-24 21:40:01,955 INFO  org.apache.flink.runtime.dispatcher.StandaloneDispatcher     [] - Received JobGraph submission 'collect' (017e9a549c97cffce6b98fd75369092a).
jobmanager                                     | 2023-10-24 21:40:01,958 INFO  org.apache.flink.runtime.dispatcher.StandaloneDispatcher     [] - Submitting job 'collect' (017e9a549c97cffce6b98fd75369092a).
jobmanager                                     | 2023-10-24 21:40:01,977 INFO  org.apache.flink.runtime.jobmaster.JobMasterServiceLeadershipRunner [] - JobMasterServiceLeadershipRunner for job 017e9a549c97cffce6b98fd75369092a was granted leadership with leader id 00000000-0000-0000-0000-000000000000. Creating new JobMasterServiceProcess.
jobmanager                                     | 2023-10-24 21:40:01,996 INFO  org.apache.flink.runtime.rpc.akka.AkkaRpcService             [] - Starting RPC endpoint for org.apache.flink.runtime.jobmaster.JobMaster at akka://flink/user/rpc/jobmanager_2 .
jobmanager                                     | 2023-10-24 21:40:02,012 INFO  org.apache.flink.runtime.jobmaster.JobMaster                 [] - Initializing job 'collect' (017e9a549c97cffce6b98fd75369092a).
jobmanager                                     | 2023-10-24 21:40:02,054 INFO  org.apache.flink.runtime.jobmaster.JobMaster                 [] - Using restart back off time strategy NoRestartBackoffTimeStrategy for collect (017e9a549c97cffce6b98fd75369092a).
jobmanager                                     | 2023-10-24 21:40:02,088 INFO  org.apache.flink.runtime.executiongraph.ExecutionGraph       [] - Created execution graph 349d06dcb8d7f424187e544304e6e6b0 for job 017e9a549c97cffce6b98fd75369092a.
jobmanager                                     | 2023-10-24 21:40:02,104 INFO  org.apache.flink.runtime.jobmaster.JobMaster                 [] - Running initialization on master for job collect (017e9a549c97cffce6b98fd75369092a).
jobmanager                                     | 2023-10-24 21:40:02,106 INFO  org.apache.flink.runtime.jobmaster.JobMaster                 [] - Successfully ran initialization on master in 1 ms.
jobmanager                                     | 2023-10-24 21:40:02,315 INFO  org.apache.flink.runtime.dispatcher.StandaloneDispatcher     [] - Job 017e9a549c97cffce6b98fd75369092a reached terminal state FAILED.
jobmanager                                     | org.apache.flink.runtime.client.JobInitializationException: Could not start the JobMaster.
jobmanager                                     | 	at org.apache.flink.runtime.jobmaster.DefaultJobMasterServiceProcess.lambda$new$0(DefaultJobMasterServiceProcess.java:97)
jobmanager                                     | 	at java.base/java.util.concurrent.CompletableFuture.uniWhenComplete(Unknown Source)
jobmanager                                     | 	at java.base/java.util.concurrent.CompletableFuture$UniWhenComplete.tryFire(Unknown Source)
jobmanager                                     | 	at java.base/java.util.concurrent.CompletableFuture.postComplete(Unknown Source)
jobmanager                                     | 	at java.base/java.util.concurrent.CompletableFuture$AsyncSupply.run(Unknown Source)
jobmanager                                     | 	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
jobmanager                                     | 	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
jobmanager                                     | 	at java.base/java.lang.Thread.run(Unknown Source)
jobmanager                                     | Caused by: java.util.concurrent.CompletionException: java.lang.RuntimeException: org.apache.flink.runtime.JobException: Cannot instantiate the coordinator for operator Source: user_behavior[1] -> Calc[2] -> ConstraintEnforcer[3] -> StreamRecordTimestampInserter[3] -> Sink: Collect table sink
jobmanager                                     | 	at java.base/java.util.concurrent.CompletableFuture.encodeThrowable(Unknown Source)
jobmanager                                     | 	at java.base/java.util.concurrent.CompletableFuture.completeThrowable(Unknown Source)
jobmanager                                     | 	... 4 more
jobmanager                                     | Caused by: java.lang.RuntimeException: org.apache.flink.runtime.JobException: Cannot instantiate the coordinator for operator Source: user_behavior[1] -> Calc[2] -> ConstraintEnforcer[3] -> StreamRecordTimestampInserter[3] -> Sink: Collect table sink
jobmanager                                     | 	at org.apache.flink.util.ExceptionUtils.rethrow(ExceptionUtils.java:321)
jobmanager                                     | 	at org.apache.flink.util.function.FunctionUtils.lambda$uncheckedSupplier$4(FunctionUtils.java:114)
jobmanager                                     | 	... 4 more
jobmanager                                     | Caused by: org.apache.flink.runtime.JobException: Cannot instantiate the coordinator for operator Source: user_behavior[1] -> Calc[2] -> ConstraintEnforcer[3] -> StreamRecordTimestampInserter[3] -> Sink: Collect table sink
jobmanager                                     | 	at org.apache.flink.runtime.executiongraph.ExecutionJobVertex.initialize(ExecutionJobVertex.java:229)
jobmanager                                     | 	at org.apache.flink.runtime.executiongraph.DefaultExecutionGraph.initializeJobVertex(DefaultExecutionGraph.java:914)
jobmanager                                     | 	at org.apache.flink.runtime.executiongraph.ExecutionGraph.initializeJobVertex(ExecutionGraph.java:218)
jobmanager                                     | 	at org.apache.flink.runtime.executiongraph.DefaultExecutionGraph.initializeJobVertices(DefaultExecutionGraph.java:896)
jobmanager                                     | 	at org.apache.flink.runtime.executiongraph.DefaultExecutionGraph.attachJobGraph(DefaultExecutionGraph.java:852)
jobmanager                                     | 	at org.apache.flink.runtime.executiongraph.DefaultExecutionGraphBuilder.buildGraph(DefaultExecutionGraphBuilder.java:207)
jobmanager                                     | 	at org.apache.flink.runtime.scheduler.DefaultExecutionGraphFactory.createAndRestoreExecutionGraph(DefaultExecutionGraphFactory.java:163)
jobmanager                                     | 	at org.apache.flink.runtime.scheduler.SchedulerBase.createAndRestoreExecutionGraph(SchedulerBase.java:365)
jobmanager                                     | 	at org.apache.flink.runtime.scheduler.SchedulerBase.<init>(SchedulerBase.java:210)
jobmanager                                     | 	at org.apache.flink.runtime.scheduler.DefaultScheduler.<init>(DefaultScheduler.java:136)
jobmanager                                     | 	at org.apache.flink.runtime.scheduler.DefaultSchedulerFactory.createInstance(DefaultSchedulerFactory.java:152)
jobmanager                                     | 	at org.apache.flink.runtime.jobmaster.DefaultSlotPoolServiceSchedulerFactory.createScheduler(DefaultSlotPoolServiceSchedulerFactory.java:119)
jobmanager                                     | 	at org.apache.flink.runtime.jobmaster.JobMaster.createScheduler(JobMaster.java:371)
jobmanager                                     | 	at org.apache.flink.runtime.jobmaster.JobMaster.<init>(JobMaster.java:348)
jobmanager                                     | 	at org.apache.flink.runtime.jobmaster.factories.DefaultJobMasterServiceFactory.internalCreateJobMasterService(DefaultJobMasterServiceFactory.java:123)
jobmanager                                     | 	at org.apache.flink.runtime.jobmaster.factories.DefaultJobMasterServiceFactory.lambda$createJobMasterService$0(DefaultJobMasterServiceFactory.java:95)
jobmanager                                     | 	at org.apache.flink.util.function.FunctionUtils.lambda$uncheckedSupplier$4(FunctionUtils.java:112)
jobmanager                                     | 	... 4 more
jobmanager                                     | Caused by: java.lang.ClassNotFoundException: org.apache.flink.connector.kafka.source.KafkaSource
jobmanager                                     | 	at java.base/java.net.URLClassLoader.findClass(Unknown Source)
jobmanager                                     | 	at java.base/java.lang.ClassLoader.loadClass(Unknown Source)
jobmanager                                     | 	at org.apache.flink.util.FlinkUserCodeClassLoader.loadClassWithoutExceptionHandling(FlinkUserCodeClassLoader.java:67)
jobmanager                                     | 	at org.apache.flink.util.ChildFirstClassLoader.loadClassWithoutExceptionHandling(ChildFirstClassLoader.java:65)
jobmanager                                     | 	at org.apache.flink.util.FlinkUserCodeClassLoader.loadClass(FlinkUserCodeClassLoader.java:51)
jobmanager                                     | 	at java.base/java.lang.ClassLoader.loadClass(Unknown Source)
jobmanager                                     | 	at org.apache.flink.util.FlinkUserCodeClassLoaders$SafetyNetWrapperClassLoader.loadClass(FlinkUserCodeClassLoaders.java:192)
jobmanager                                     | 	at java.base/java.lang.Class.forName0(Native Method)
jobmanager                                     | 	at java.base/java.lang.Class.forName(Unknown Source)
jobmanager                                     | 	at org.apache.flink.util.InstantiationUtil$ClassLoaderObjectInputStream.resolveClass(InstantiationUtil.java:76)
jobmanager                                     | 	at java.base/java.io.ObjectInputStream.readNonProxyDesc(Unknown Source)
jobmanager                                     | 	at java.base/java.io.ObjectInputStream.readClassDesc(Unknown Source)
jobmanager                                     | 	at java.base/java.io.ObjectInputStream.readOrdinaryObject(Unknown Source)
jobmanager                                     | 	at java.base/java.io.ObjectInputStream.readObject0(Unknown Source)
jobmanager                                     | 	at java.base/java.io.ObjectInputStream.defaultReadFields(Unknown Source)
jobmanager                                     | 	at java.base/java.io.ObjectInputStream.readSerialData(Unknown Source)
jobmanager                                     | 	at java.base/java.io.ObjectInputStream.readOrdinaryObject(Unknown Source)
jobmanager                                     | 	at java.base/java.io.ObjectInputStream.readObject0(Unknown Source)
jobmanager                                     | 	at java.base/java.io.ObjectInputStream.readObject(Unknown Source)
jobmanager                                     | 	at java.base/java.io.ObjectInputStream.readObject(Unknown Source)
jobmanager                                     | 	at org.apache.flink.util.InstantiationUtil.deserializeObject(InstantiationUtil.java:534)
jobmanager                                     | 	at org.apache.flink.util.InstantiationUtil.deserializeObject(InstantiationUtil.java:522)
jobmanager                                     | 	at org.apache.flink.util.SerializedValue.deserializeValue(SerializedValue.java:67)
jobmanager                                     | 	at org.apache.flink.runtime.operators.coordination.OperatorCoordinatorHolder.create(OperatorCoordinatorHolder.java:471)
jobmanager                                     | 	at org.apache.flink.runtime.executiongraph.ExecutionJobVertex.createOperatorCoordinatorHolder(ExecutionJobVertex.java:286)
jobmanager                                     | 	at org.apache.flink.runtime.executiongraph.ExecutionJobVertex.initialize(ExecutionJobVertex.java:223)
jobmanager                                     | 	... 20 more
jobmanager                                     | 2023-10-24 21:40:02,339 INFO  org.apache.flink.runtime.dispatcher.StandaloneDispatcher     [] - Job 017e9a549c97cffce6b98fd75369092a has been registered for cleanup in the JobResultStore after reaching a terminal state.



jobmanager                                     | 2023-10-24 21:45:29,556 INFO  org.apache.flink.runtime.dispatcher.StandaloneDispatcher     [] - Received JobGraph submission 'collect' (157624e65b58f1a75f822b8e7c6c4e39).
jobmanager                                     | 2023-10-24 21:45:29,558 INFO  org.apache.flink.runtime.dispatcher.StandaloneDispatcher     [] - Submitting job 'collect' (157624e65b58f1a75f822b8e7c6c4e39).
jobmanager                                     | 2023-10-24 21:45:29,562 INFO  org.apache.flink.runtime.jobmaster.JobMasterServiceLeadershipRunner [] - JobMasterServiceLeadershipRunner for job 157624e65b58f1a75f822b8e7c6c4e39 was granted leadership with leader id 00000000-0000-0000-0000-000000000000. Creating new JobMasterServiceProcess.
jobmanager                                     | 2023-10-24 21:45:29,574 INFO  org.apache.flink.runtime.rpc.akka.AkkaRpcService             [] - Starting RPC endpoint for org.apache.flink.runtime.jobmaster.JobMaster at akka://flink/user/rpc/jobmanager_3 .
jobmanager                                     | 2023-10-24 21:45:29,575 INFO  org.apache.flink.runtime.jobmaster.JobMaster                 [] - Initializing job 'collect' (157624e65b58f1a75f822b8e7c6c4e39).
jobmanager                                     | 2023-10-24 21:45:29,578 INFO  org.apache.flink.runtime.jobmaster.JobMaster                 [] - Using restart back off time strategy NoRestartBackoffTimeStrategy for collect (157624e65b58f1a75f822b8e7c6c4e39).
jobmanager                                     | 2023-10-24 21:45:29,581 INFO  org.apache.flink.runtime.executiongraph.ExecutionGraph       [] - Created execution graph 55afca8efa53af4c2e820031c4dbae45 for job 157624e65b58f1a75f822b8e7c6c4e39.
jobmanager                                     | 2023-10-24 21:45:29,582 INFO  org.apache.flink.runtime.jobmaster.JobMaster                 [] - Running initialization on master for job collect (157624e65b58f1a75f822b8e7c6c4e39).
jobmanager                                     | 2023-10-24 21:45:29,582 INFO  org.apache.flink.runtime.jobmaster.JobMaster                 [] - Successfully ran initialization on master in 0 ms.
jobmanager                                     | 2023-10-24 21:45:29,642 INFO  org.apache.flink.runtime.dispatcher.StandaloneDispatcher     [] - Job 157624e65b58f1a75f822b8e7c6c4e39 reached terminal state FAILED.
jobmanager                                     | org.apache.flink.runtime.client.JobInitializationException: Could not start the JobMaster.
jobmanager                                     | 	at org.apache.flink.runtime.jobmaster.DefaultJobMasterServiceProcess.lambda$new$0(DefaultJobMasterServiceProcess.java:97)
jobmanager                                     | 	at java.base/java.util.concurrent.CompletableFuture.uniWhenComplete(Unknown Source)
jobmanager                                     | 	at java.base/java.util.concurrent.CompletableFuture$UniWhenComplete.tryFire(Unknown Source)
jobmanager                                     | 	at java.base/java.util.concurrent.CompletableFuture.postComplete(Unknown Source)
jobmanager                                     | 	at java.base/java.util.concurrent.CompletableFuture$AsyncSupply.run(Unknown Source)
jobmanager                                     | 	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
jobmanager                                     | 	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
jobmanager                                     | 	at java.base/java.lang.Thread.run(Unknown Source)
jobmanager                                     | Caused by: java.util.concurrent.CompletionException: java.lang.RuntimeException: org.apache.flink.runtime.JobException: Cannot instantiate the coordinator for operator Source: user_behavior[1] -> Calc[2] -> ConstraintEnforcer[3] -> StreamRecordTimestampInserter[3] -> Sink: Collect table sink
jobmanager                                     | 	at java.base/java.util.concurrent.CompletableFuture.encodeThrowable(Unknown Source)
jobmanager                                     | 	at java.base/java.util.concurrent.CompletableFuture.completeThrowable(Unknown Source)
jobmanager                                     | 	... 4 more
jobmanager                                     | Caused by: java.lang.RuntimeException: org.apache.flink.runtime.JobException: Cannot instantiate the coordinator for operator Source: user_behavior[1] -> Calc[2] -> ConstraintEnforcer[3] -> StreamRecordTimestampInserter[3] -> Sink: Collect table sink
jobmanager                                     | 	at org.apache.flink.util.ExceptionUtils.rethrow(ExceptionUtils.java:321)
jobmanager                                     | 	at org.apache.flink.util.function.FunctionUtils.lambda$uncheckedSupplier$4(FunctionUtils.java:114)
jobmanager                                     | 	... 4 more
jobmanager                                     | Caused by: org.apache.flink.runtime.JobException: Cannot instantiate the coordinator for operator Source: user_behavior[1] -> Calc[2] -> ConstraintEnforcer[3] -> StreamRecordTimestampInserter[3] -> Sink: Collect table sink
jobmanager                                     | 	at org.apache.flink.runtime.executiongraph.ExecutionJobVertex.initialize(ExecutionJobVertex.java:229)
jobmanager                                     | 	at org.apache.flink.runtime.executiongraph.DefaultExecutionGraph.initializeJobVertex(DefaultExecutionGraph.java:914)
jobmanager                                     | 	at org.apache.flink.runtime.executiongraph.ExecutionGraph.initializeJobVertex(ExecutionGraph.java:218)
jobmanager                                     | 	at org.apache.flink.runtime.executiongraph.DefaultExecutionGraph.initializeJobVertices(DefaultExecutionGraph.java:896)
jobmanager                                     | 	at org.apache.flink.runtime.executiongraph.DefaultExecutionGraph.attachJobGraph(DefaultExecutionGraph.java:852)
jobmanager                                     | 	at org.apache.flink.runtime.executiongraph.DefaultExecutionGraphBuilder.buildGraph(DefaultExecutionGraphBuilder.java:207)
jobmanager                                     | 	at org.apache.flink.runtime.scheduler.DefaultExecutionGraphFactory.createAndRestoreExecutionGraph(DefaultExecutionGraphFactory.java:163)
jobmanager                                     | 	at org.apache.flink.runtime.scheduler.SchedulerBase.createAndRestoreExecutionGraph(SchedulerBase.java:365)
jobmanager                                     | 	at org.apache.flink.runtime.scheduler.SchedulerBase.<init>(SchedulerBase.java:210)
jobmanager                                     | 	at org.apache.flink.runtime.scheduler.DefaultScheduler.<init>(DefaultScheduler.java:136)
jobmanager                                     | 	at org.apache.flink.runtime.scheduler.DefaultSchedulerFactory.createInstance(DefaultSchedulerFactory.java:152)
jobmanager                                     | 	at org.apache.flink.runtime.jobmaster.DefaultSlotPoolServiceSchedulerFactory.createScheduler(DefaultSlotPoolServiceSchedulerFactory.java:119)
jobmanager                                     | 	at org.apache.flink.runtime.jobmaster.JobMaster.createScheduler(JobMaster.java:371)
jobmanager                                     | 	at org.apache.flink.runtime.jobmaster.JobMaster.<init>(JobMaster.java:348)
jobmanager                                     | 	at org.apache.flink.runtime.jobmaster.factories.DefaultJobMasterServiceFactory.internalCreateJobMasterService(DefaultJobMasterServiceFactory.java:123)
jobmanager                                     | 	at org.apache.flink.runtime.jobmaster.factories.DefaultJobMasterServiceFactory.lambda$createJobMasterService$0(DefaultJobMasterServiceFactory.java:95)
jobmanager                                     | 	at org.apache.flink.util.function.FunctionUtils.lambda$uncheckedSupplier$4(FunctionUtils.java:112)
jobmanager                                     | 	... 4 more
jobmanager                                     | Caused by: java.lang.ClassNotFoundException: org.apache.flink.connector.kafka.source.KafkaSource
jobmanager                                     | 	at java.base/java.net.URLClassLoader.findClass(Unknown Source)
jobmanager                                     | 	at java.base/java.lang.ClassLoader.loadClass(Unknown Source)
jobmanager                                     | 	at org.apache.flink.util.FlinkUserCodeClassLoader.loadClassWithoutExceptionHandling(FlinkUserCodeClassLoader.java:67)
jobmanager                                     | 	at org.apache.flink.util.ChildFirstClassLoader.loadClassWithoutExceptionHandling(ChildFirstClassLoader.java:65)
jobmanager                                     | 	at org.apache.flink.util.FlinkUserCodeClassLoader.loadClass(FlinkUserCodeClassLoader.java:51)
jobmanager                                     | 	at java.base/java.lang.ClassLoader.loadClass(Unknown Source)
jobmanager                                     | 	at org.apache.flink.util.FlinkUserCodeClassLoaders$SafetyNetWrapperClassLoader.loadClass(FlinkUserCodeClassLoaders.java:192)
jobmanager                                     | 	at java.base/java.lang.Class.forName0(Native Method)
jobmanager                                     | 	at java.base/java.lang.Class.forName(Unknown Source)
jobmanager                                     | 	at org.apache.flink.util.InstantiationUtil$ClassLoaderObjectInputStream.resolveClass(InstantiationUtil.java:76)
jobmanager                                     | 	at java.base/java.io.ObjectInputStream.readNonProxyDesc(Unknown Source)
jobmanager                                     | 	at java.base/java.io.ObjectInputStream.readClassDesc(Unknown Source)
jobmanager                                     | 	at java.base/java.io.ObjectInputStream.readOrdinaryObject(Unknown Source)
jobmanager                                     | 	at java.base/java.io.ObjectInputStream.readObject0(Unknown Source)
jobmanager                                     | 	at java.base/java.io.ObjectInputStream.defaultReadFields(Unknown Source)
jobmanager                                     | 	at java.base/java.io.ObjectInputStream.readSerialData(Unknown Source)
jobmanager                                     | 	at java.base/java.io.ObjectInputStream.readOrdinaryObject(Unknown Source)
jobmanager                                     | 	at java.base/java.io.ObjectInputStream.readObject0(Unknown Source)
jobmanager                                     | 	at java.base/java.io.ObjectInputStream.readObject(Unknown Source)
jobmanager                                     | 	at java.base/java.io.ObjectInputStream.readObject(Unknown Source)
jobmanager                                     | 	at org.apache.flink.util.InstantiationUtil.deserializeObject(InstantiationUtil.java:534)
jobmanager                                     | 	at org.apache.flink.util.InstantiationUtil.deserializeObject(InstantiationUtil.java:522)
jobmanager                                     | 	at org.apache.flink.util.SerializedValue.deserializeValue(SerializedValue.java:67)
jobmanager                                     | 	at org.apache.flink.runtime.operators.coordination.OperatorCoordinatorHolder.create(OperatorCoordinatorHolder.java:471)
jobmanager                                     | 	at org.apache.flink.runtime.executiongraph.ExecutionJobVertex.createOperatorCoordinatorHolder(ExecutionJobVertex.java:286)
jobmanager                                     | 	at org.apache.flink.runtime.executiongraph.ExecutionJobVertex.initialize(ExecutionJobVertex.java:223)
jobmanager                                     | 	... 20 more
jobmanager                                     | 2023-10-24 21:45:29,655 INFO  org.apache.flink.runtime.dispatcher.StandaloneDispatcher     [] - Job 157624e65b58f1a75f822b8e7c6c4e39 has been registered for cleanup in the JobResultStore after reaching a terminal state.


```