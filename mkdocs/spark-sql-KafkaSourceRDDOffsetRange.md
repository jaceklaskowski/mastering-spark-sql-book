# KafkaSourceRDDOffsetRange

`KafkaSourceRDDOffsetRange` is an <<spark-sql-KafkaSourceRDDPartition.adoc#offsetRange, offset range>> that one  `KafkaSourceRDDPartition` partition of a <<spark-sql-KafkaSourceRDD.adoc#getPartitions, KafkaSourceRDD>> has to read.

`KafkaSourceRDDOffsetRange` is <<creating-instance, created>> when:

* `KafkaRelation` is requested to <<spark-sql-KafkaRelation.adoc#buildScan, build a distributed data scan with column pruning>> (as a <<spark-sql-TableScan.adoc#, TableScan>>) (and creates a <<spark-sql-KafkaSourceRDD.adoc#offsetRanges, KafkaSourceRDD>>)

* `KafkaSourceRDD` is requested to <<spark-sql-KafkaSourceRDD.adoc#resolveRange, resolveRange>>

* (Spark Structured Streaming) `KafkaSource` is requested to `getBatch`

[[creating-instance]]
`KafkaSourceRDDOffsetRange` takes the following when created:

* [[topicPartition]] Kafka https://kafka.apache.org/20/javadoc/org/apache/kafka/common/TopicPartition.html[TopicPartition]
* [[fromOffset]] `fromOffset`
* [[untilOffset]] `untilOffset`
* [[preferredLoc]] Preferred location

NOTE: https://kafka.apache.org/20/javadoc/org/apache/kafka/common/TopicPartition.html[TopicPartition] is a topic name and partition number.
