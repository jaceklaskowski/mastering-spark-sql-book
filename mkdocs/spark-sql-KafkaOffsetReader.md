# KafkaOffsetReader

`KafkaOffsetReader` is used to query a Kafka cluster for partition offsets.

`KafkaOffsetReader` is <<creating-instance, created>> when:

* `KafkaRelation` is requested to <<spark-sql-KafkaRelation.adoc#buildScan, build a distributed data scan with column pruning>> (as a <<spark-sql-TableScan.adoc#, TableScan>>) (to <<spark-sql-KafkaRelation.adoc#getPartitionOffsets, get the initial partition offsets>>)

* (Spark Structured Streaming) `KafkaSourceProvider` is requested to `createSource` and `createContinuousReader`

[[toString]]
When requested for the human-readable text representation (aka `toString`), `KafkaOffsetReader` simply requests the <<consumerStrategy, ConsumerStrategy>> for one.

[[options]]
.KafkaOffsetReader's Options
[cols="1,1,2",options="header",width="100%"]
|===
| Name
| Default Value
| Description

| [[fetchOffset.numRetries]] `fetchOffset.numRetries`
| `3`
|

| [[fetchOffset.retryIntervalMs]] `fetchOffset.retryIntervalMs`
| `1000`
| How long to wait before retries.
|===

[[internal-registries]]
.KafkaOffsetReader's Internal Registries and Counters
[cols="1,2",options="header",width="100%"]
|===
| Name
| Description

| `consumer`
a| [[consumer]] Kafka's https://kafka.apache.org/0110/javadoc/org/apache/kafka/clients/consumer/Consumer.html[Consumer] (with keys and values of `Array[Byte]` type)

<<createConsumer, Initialized>> when `KafkaOffsetReader` is <<creating-instance, created>>.

Used when `KafkaOffsetReader`:

* <<fetchTopicPartitions, fetchTopicPartitions>>
* <<fetchEarliestOffsets, fetchEarliestOffsets>>
* <<fetchLatestOffsets, fetchLatestOffsets>>
* <<resetConsumer, resetConsumer>>
* <<close, is closed>>

| `execContext`
| [[execContext]]

| `groupId`
| [[groupId]]

| `kafkaReaderThread`
| [[kafkaReaderThread]]

| `maxOffsetFetchAttempts`
| [[maxOffsetFetchAttempts]]

| `nextId`
| [[nextId]]

| `offsetFetchAttemptIntervalMs`
| [[offsetFetchAttemptIntervalMs]]
|===

[TIP]
====
Enable `INFO` or `DEBUG` logging levels for `org.apache.spark.sql.kafka010.KafkaOffsetReader` to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.kafka010.KafkaOffsetReader=DEBUG
```

Refer to link:spark-sql-streaming-logging.adoc[Logging].
====

=== [[createConsumer]] Creating Kafka Consumer -- `createConsumer` Internal Method

[source, scala]
----
createConsumer(): Consumer[Array[Byte], Array[Byte]]
----

`createConsumer` requests the <<consumerStrategy, ConsumerStrategy>> to <<spark-sql-ConsumerStrategy.adoc#createConsumer, create a Kafka Consumer>> with <<driverKafkaParams, driverKafkaParams>> and <<nextGroupId, new generated group.id Kafka property>>.

NOTE: `createConsumer` is used when `KafkaOffsetReader` is <<creating-instance, created>> (and initializes <<consumer, consumer>>) and <<resetConsumer, resetConsumer>>

=== [[creating-instance]] Creating KafkaOffsetReader Instance

`KafkaOffsetReader` takes the following when created:

* [[consumerStrategy]] <<spark-sql-ConsumerStrategy.adoc#, ConsumerStrategy>>
* [[driverKafkaParams]] Kafka parameters (as `Map[String, Object]`)
* [[readerOptions]] Reader options (as `Map[String, String]`)
* [[driverGroupIdPrefix]] Prefix for the group id

`KafkaOffsetReader` initializes the <<internal-registries, internal registries and counters>>.

=== [[close]] `close` Method

[source, scala]
----
close(): Unit
----

`close`...FIXME

NOTE: `close` is used when...FIXME

=== [[fetchEarliestOffsets]] `fetchEarliestOffsets` Method

[source, scala]
----
fetchEarliestOffsets(): Map[TopicPartition, Long]
----

`fetchEarliestOffsets`...FIXME

NOTE: `fetchEarliestOffsets` is used when...FIXME

=== [[fetchEarliestOffsets-newPartitions]] `fetchEarliestOffsets` Method

[source, scala]
----
fetchEarliestOffsets(newPartitions: Seq[TopicPartition]): Map[TopicPartition, Long]
----

`fetchEarliestOffsets`...FIXME

NOTE: `fetchEarliestOffsets` is used when...FIXME

=== [[fetchLatestOffsets]] `fetchLatestOffsets` Method

[source, scala]
----
fetchLatestOffsets(): Map[TopicPartition, Long]
----

`fetchLatestOffsets`...FIXME

NOTE: `fetchLatestOffsets` is used when...FIXME

=== [[fetchTopicPartitions]] Fetching (and Pausing) Assigned Kafka TopicPartitions -- `fetchTopicPartitions` Method

[source, scala]
----
fetchTopicPartitions(): Set[TopicPartition]
----

`fetchTopicPartitions` <<runUninterruptibly, uses an UninterruptibleThread thread>> to do the following:

. Requests the <<consumer, Kafka Consumer>> to link:++https://kafka.apache.org/20/javadoc/org/apache/kafka/clients/consumer/Consumer.html#poll-long-++[poll] (fetch data) for the topics and partitions (with `0` timeout)

. Requests the <<consumer, Kafka Consumer>> to link:++https://kafka.apache.org/20/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#assignment--++[get the set of partitions currently assigned]

. Requests the <<consumer, Kafka Consumer>> to link:++https://kafka.apache.org/20/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#pause-java.util.Collection-++[suspend fetching from the partitions assigned]

In the end, `fetchTopicPartitions` returns the `TopicPartitions` assigned (and paused).

NOTE: `fetchTopicPartitions` is used exclusively when `KafkaRelation` is requested to <<buildScan, build a distributed data scan with column pruning>> (as a <<spark-sql-TableScan.adoc#, TableScan>>) through <<spark-sql-KafkaRelation.adoc#getPartitionOffsets, getPartitionOffsets>>.

=== [[nextGroupId]] `nextGroupId` Internal Method

[source, scala]
----
nextGroupId(): String
----

`nextGroupId`...FIXME

NOTE: `nextGroupId` is used when...FIXME

=== [[resetConsumer]] `resetConsumer` Internal Method

[source, scala]
----
resetConsumer(): Unit
----

`resetConsumer`...FIXME

NOTE: `resetConsumer` is used when...FIXME

=== [[runUninterruptibly]] `runUninterruptibly` Internal Method

[source, scala]
----
runUninterruptibly[T](body: => T): T
----

`runUninterruptibly`...FIXME

NOTE: `runUninterruptibly` is used when...FIXME

=== [[withRetriesWithoutInterrupt]] `withRetriesWithoutInterrupt` Internal Method

[source, scala]
----
withRetriesWithoutInterrupt(body: => Map[TopicPartition, Long]): Map[TopicPartition, Long]
----

`withRetriesWithoutInterrupt`...FIXME

NOTE: `withRetriesWithoutInterrupt` is used when...FIXME
