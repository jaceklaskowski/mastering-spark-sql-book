title: KafkaWriter

# KafkaWriter Helper Object -- Writing Structured Queries to Kafka

`KafkaWriter` is a Scala object that is used to <<write, write>> the rows of a batch (or a streaming) structured query to Apache Kafka.

.KafkaWriter (write) in web UI
image::images/spark-sql-KafkaWriter-write-webui.png[align="center"]

`KafkaWriter` <<validateQuery, validates the schema of a structured query>> that it contains the following columns (<<spark-sql-catalyst-QueryPlan.adoc#output, output schema attributes>>):

* Either *topic* of type `StringType` or the <<spark-sql-kafka-options.adoc#topic, topic>> option are defined

* Optional *key* of type `StringType` or `BinaryType`

* Required *value* of type `StringType` or `BinaryType`

[source, scala]
----
// KafkaWriter is a private `kafka010` package object
// and so the code to use it should also be in the same package
// BEGIN: Use `:paste -raw` in spark-shell
package org.apache.spark.sql.kafka010

object PublicKafkaWriter {
  import org.apache.spark.sql.execution.QueryExecution
  def validateQuery(
      queryExecution: QueryExecution,
      kafkaParameters: Map[String, Object],
      topic: Option[String] = None): Unit = {
    import scala.collection.JavaConversions.mapAsJavaMap
    KafkaWriter.validateQuery(queryExecution, kafkaParameters, topic)
  }
}
// END

import org.apache.spark.sql.kafka010.{PublicKafkaWriter => PKW}

val spark: SparkSession = ...
val q = spark.range(1).select('id)
scala> PKW.validateQuery(
  queryExecution = q.queryExecution,
  kafkaParameters = Map.empty[String, Object])
org.apache.spark.sql.AnalysisException: topic option required when no 'topic' attribute is present. Use the topic option for setting a topic.;
  at org.apache.spark.sql.kafka010.KafkaWriter$$anonfun$2.apply(KafkaWriter.scala:53)
  at org.apache.spark.sql.kafka010.KafkaWriter$$anonfun$2.apply(KafkaWriter.scala:52)
  at scala.Option.getOrElse(Option.scala:121)
  at org.apache.spark.sql.kafka010.KafkaWriter$.validateQuery(KafkaWriter.scala:51)
  at org.apache.spark.sql.kafka010.PublicKafkaWriter$.validateQuery(<pastie>:10)
  ... 50 elided
----

=== [[write]] Writing Rows of Structured Query to Kafka Topic -- `write` Method

[source, scala]
----
write(
  sparkSession: SparkSession,
  queryExecution: QueryExecution,
  kafkaParameters: ju.Map[String, Object],
  topic: Option[String] = None): Unit
----

`write` gets the <<spark-sql-catalyst-QueryPlan.adoc#output, output schema>> of the <<spark-sql-QueryExecution.adoc#analyzed, analyzed logical plan>> of the input <<spark-sql-QueryExecution.adoc#, QueryExecution>>.

`write` then <<validateQuery, validates the schema of a structured query>>.

In the end, `write` requests the `QueryExecution` for <<spark-sql-QueryExecution.adoc#toRdd, RDD[InternalRow]>> (that represents the structured query as an RDD) and executes the following function on every partition of the RDD (using `RDD.foreachPartition` operation):

. Creates a <<spark-sql-KafkaWriteTask.adoc#creating-instance, KafkaWriteTask>> (for the input `kafkaParameters`, the schema and the input `topic`)

. Requests the `KafkaWriteTask` to <<spark-sql-KafkaWriteTask.adoc#execute, write the rows (of the partition) to Kafka topic>>

. Requests the `KafkaWriteTask` to <<spark-sql-KafkaWriteTask.adoc#close, close>>

[NOTE]
====
`write` is used when:

* `KafkaSourceProvider` is requested to <<spark-sql-KafkaSourceProvider.adoc#createRelation-CreatableRelationProvider, write a DataFrame to a Kafka topic>>

* (Spark Structured Streaming) `KafkaSink` is requested to `addBatch`
====

=== [[validateQuery]] Validating Schema (Attributes) of Structured Query and Topic Option Availability -- `validateQuery` Method

[source, scala]
----
validateQuery(
  schema: Seq[Attribute],
  kafkaParameters: ju.Map[String, Object],
  topic: Option[String] = None): Unit
----

`validateQuery` makes sure that the following attributes are in the input schema (or their alternatives) and of the right data types:

* Either `topic` attribute of type `StringType` or the <<spark-sql-kafka-options.adoc#topic, topic>> option are defined

* If `key` attribute is defined it is of type `StringType` or `BinaryType`

* `value` attribute is of type `StringType` or `BinaryType`

If any of the requirements are not met, `validateQuery` throws an `AnalysisException`.

[NOTE]
====
`validateQuery` is used when:

* `KafkaWriter` object is requested to <<write, write the rows of a structured query to a Kafka topic>>

* (Spark Structured Streaming) `KafkaStreamWriter` is created

* (Spark Structured Streaming) `KafkaSourceProvider` is requested to `createStreamWriter`
====
