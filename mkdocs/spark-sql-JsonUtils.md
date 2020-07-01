title: JsonUtils

# JsonUtils Helper Object

`JsonUtils` is a Scala object with <<methods, methods>> for serializing and deserializing Kafka https://kafka.apache.org/20/javadoc/org/apache/kafka/common/TopicPartition.html[TopicPartitions] to and from a single JSON text.

`JsonUtils` uses http://json4s.org/[json4s] library that provides a single AST with the Jackson parser for parsing to the AST (using `json4s-jackson` module).

[[methods]]
.JsonUtils API
[cols="1,2",options="header",width="100%"]
|===
| Name
| Description

| <<partitionOffsets-String-Map, partitionOffsets>>
a| Deserializing partition offsets (i.e. offsets per Kafka `TopicPartition`) from JSON, e.g. `{"topicA":{"0":23,"1":-1},"topicB":{"0":-2}}`

[source, scala]
----
partitionOffsets(str: String): Map[TopicPartition, Long]
----

| <<partitionOffsets-Map-String, partitionOffsets>>
a| Serializing partition offsets (i.e. offsets per Kafka `TopicPartition`) to JSON

[source, scala]
----
partitionOffsets(partitionOffsets: Map[TopicPartition, Long]): String
----

| <<partitions-String-Array, partitions>>
a| Deserializing `TopicPartitions` from JSON, e.g. `{"topicA":[0,1],"topicB":[0,1]}`

[source, scala]
----
partitions(str: String): Array[TopicPartition]
----

| <<partitions-Iterable-String, partitions>>
a| Serializing `TopicPartitions` to JSON

[source, scala]
----
partitions(partitions: Iterable[TopicPartition]): String
----
|===

=== [[partitionOffsets-String-Map]] Deserializing Partition Offsets From JSON -- `partitionOffsets` Method

[source, scala]
----
partitionOffsets(str: String): Map[TopicPartition, Long]
----

`partitionOffsets`...FIXME

[NOTE]
====
`partitionOffsets` is used when:

* `KafkaSourceProvider` is requested to <<spark-sql-KafkaSourceProvider.adoc#getKafkaOffsetRangeLimit, get the desired KafkaOffsetRangeLimit (for offset option)>>

* (Spark Structured Streaming) `KafkaContinuousReader` is requested to `deserializeOffset`

* (Spark Structured Streaming) `KafkaSourceOffset` is created (from a `SerializedOffset`)
====

=== [[partitionOffsets-Map-String]] Serializing Partition Offsets to JSON -- `partitionOffsets` Method

[source, scala]
----
partitionOffsets(partitionOffsets: Map[TopicPartition, Long]): String
----

`partitionOffsets`...FIXME

NOTE: `partitionOffsets` is used when...FIXME

=== [[partitions-Iterable-String]] Serializing TopicPartitions to JSON -- `partitions` Method

[source, scala]
----
partitions(partitions: Iterable[TopicPartition]): String
----

`partitions`...FIXME

NOTE: `partitions` seems not to be used.

=== [[partitions-String-Array]] Deserializing TopicPartitions from JSON -- `partitions` Method

[source, scala]
----
partitions(str: String): Array[TopicPartition]
----

`partitions` uses json4s-jakson's `Serialization` object to read a `Map[String, Seq[Int]` from the input string that represents a `Map` of topics and partition numbers, e.g. `{"topicA":[0,1],"topicB":[0,1]}`.

For every pair of topic and partition number, `partitions` creates a new Kafka https://kafka.apache.org/20/javadoc/org/apache/kafka/common/TopicPartition.html[TopicPartition].

In case of any parsing issues, `partitions` throws a new `IllegalArgumentException`:

```
Expected e.g. {"topicA":[0,1],"topicB":[0,1]}, got [str]
```

NOTE: `partitions` is used exclusively when `KafkaSourceProvider` is requested for a <<spark-sql-KafkaSourceProvider.adoc#strategy, ConsumerStrategy>> (given <<spark-sql-kafka-options.adoc#assign, assign>> option).
