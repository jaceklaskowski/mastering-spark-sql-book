title: Options

# Kafka Data Source Options

[[options]]
.Kafka Data Source Options
[cols="1m,1,2",options="header",width="100%"]
|===
| Option
| Default
| Description

| assign
|
| [[assign]] One of the three subscription strategy options (with <<spark-sql-kafka-options.adoc#subscribe, subscribe>> and <<spark-sql-kafka-options.adoc#subscribepattern, subscribepattern>>)

See <<spark-sql-KafkaSourceProvider.adoc#strategy, KafkaSourceProvider.strategy>>

| endingoffsets
|
| [[endingoffsets]]

| failondataloss
|
| [[failondataloss]]

| kafkaConsumer.pollTimeoutMs
|
| [[kafkaConsumer.pollTimeoutMs]] See <<spark-sql-KafkaRelation.adoc#pollTimeoutMs, kafkaConsumer.pollTimeoutMs>>

| startingoffsets
|
| [[startingoffsets]]

| subscribe
|
| [[subscribe]] One of the three subscription strategy options (with <<spark-sql-kafka-options.adoc#subscribepattern, subscribepattern>> and <<spark-sql-kafka-options.adoc#assign, assign>>)

See <<spark-sql-KafkaSourceProvider.adoc#strategy, KafkaSourceProvider.strategy>>

| subscribepattern
|
| [[subscribepattern]] One of the three subscription strategy options (with <<spark-sql-kafka-options.adoc#subscribe, subscribe>> and <<spark-sql-kafka-options.adoc#assign, assign>>)

See <<spark-sql-KafkaSourceProvider.adoc#strategy, KafkaSourceProvider.strategy>>

| topic
|
a| [[topic]] *Required* for writing a DataFrame to Kafka

Used when:

* `KafkaSourceProvider` is requested to <<spark-sql-KafkaSourceProvider.adoc#createRelation-CreatableRelationProvider, write a DataFrame to a Kafka topic and create a BaseRelation afterwards>>

* (Spark Structured Streaming) `KafkaSourceProvider` is requested to `createStreamWriter` and `createSink`
|===
