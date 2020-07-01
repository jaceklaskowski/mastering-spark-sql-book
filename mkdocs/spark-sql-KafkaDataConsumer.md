# KafkaDataConsumer

`KafkaDataConsumer` is the <<contract, contract>> for <<implementations, KafkaDataConsumers>> that use an <<internalConsumer, InternalKafkaConsumer>> for the following:

* <<get, Getting a single Kafka ConsumerRecord>>

* <<getAvailableOffsetRange, Getting a single AvailableOffsetRange>>

`KafkaDataConsumer` has to be <<release, released>> explicitly.

[[contract]]
[source, scala]
----
package org.apache.spark.sql.kafka010

sealed trait KafkaDataConsumer {
  // only required properties (vals and methods) that have no implementation
  // the others follow
  def internalConsumer: InternalKafkaConsumer
  def release(): Unit
}
----

.KafkaDataConsumer Contract
[cols="1m,2",options="header",width="100%"]
|===
| Property
| Description

| internalConsumer
a| [[internalConsumer]] Used when:

* `KafkaDataConsumer` is requested to <<get, get a single Kafka ConsumerRecord>> and <<getAvailableOffsetRange, get a single AvailableOffsetRange>>

* `CachedKafkaDataConsumer` and `NonCachedKafkaDataConsumer` are requested to <<release, release>> the `InternalKafkaConsumer`

| release
a| [[release]] Used when:

* `KafkaSourceRDD` is requested to <<spark-sql-KafkaSourceRDD.adoc#compute, compute a partition>>

* (Spark Structured Streaming) `KafkaContinuousDataReader` is requested to `close`
|===

[[implementations]]
.KafkaDataConsumers
[cols="1,2",options="header",width="100%"]
|===
| KafkaDataConsumer
| Description

| `CachedKafkaDataConsumer`
| [[CachedKafkaDataConsumer]]

| `NonCachedKafkaDataConsumer`
| [[NonCachedKafkaDataConsumer]]
|===

NOTE: `KafkaDataConsumer` is a Scala *sealed trait* which means that all the <<implementations, implementations>> are in the same compilation unit (a single file).

=== [[get]] Getting Single Kafka ConsumerRecord -- `get` Method

[source, scala]
----
get(
  offset: Long,
  untilOffset: Long,
  pollTimeoutMs: Long,
  failOnDataLoss: Boolean): ConsumerRecord[Array[Byte], Array[Byte]]
----

`get` simply requests the <<internalConsumer, InternalKafkaConsumer>> to <<spark-sql-InternalKafkaConsumer.adoc#get, get a single Kafka ConsumerRecord>>.

[NOTE]
====
`get` is used when:

* `KafkaSourceRDD` is requested to <<spark-sql-KafkaSourceRDD.adoc#compute, compute a partition>>

* (Spark Structured Streaming) `KafkaContinuousDataReader` is requested to `next`
====

=== [[getAvailableOffsetRange]] Getting Single AvailableOffsetRange -- `getAvailableOffsetRange` Method

[source, scala]
----
getAvailableOffsetRange(): AvailableOffsetRange
----

`getAvailableOffsetRange` simply requests the <<internalConsumer, InternalKafkaConsumer>> to <<spark-sql-InternalKafkaConsumer.adoc#getAvailableOffsetRange, get a single AvailableOffsetRange>>.

[NOTE]
====
`getAvailableOffsetRange` is used when:

* `KafkaSourceRDD` is requested to <<spark-sql-KafkaSourceRDD.adoc#compute, compute a partition>> (through <<spark-sql-KafkaSourceRDD.adoc#resolveRange, resolveRange>>)

* (Spark Structured Streaming) `KafkaContinuousDataReader` is requested to `next`
====
