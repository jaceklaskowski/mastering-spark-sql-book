# Kafka Data Source

Spark SQL supports <<reading, reading>> data from or <<writing, writing>> data to one or more topics in Apache Kafka.

[NOTE]
====
*Apache Kafka* is a storage of records in a format-independent and fault-tolerant durable way.

Read up on Apache Kafka in the http://kafka.apache.org/documentation/[official documentation] or in my other gitbook https://bit.ly/mastering-apache-kafka[Mastering Apache Kafka].
====

Kafka Data Source supports <<spark-sql-kafka-options.adoc#options, options>> to get better performance of structured queries that use it.

=== [[reading]] Reading Data from Kafka Topics

As a Spark developer, you use <<spark-sql-DataFrameReader.adoc#format, DataFrameReader.format>> method to specify Apache Kafka as the external data source to load data from.

You use <<spark-sql-KafkaSourceProvider.adoc#shortName, kafka>> (or `org.apache.spark.sql.kafka010.KafkaSourceProvider`) as the input data source format.

[source, scala]
----
val kafka = spark.read.format("kafka").load

// Alternatively
val kafka = spark.read.format("org.apache.spark.sql.kafka010.KafkaSourceProvider").load
----

These one-liners create a <<spark-sql-DataFrame.adoc#, DataFrame>> that represents the distributed process of loading data from one or many Kafka topics (with additional properties).

=== [[writing]] Writing Data to Kafka Topics

As a Spark developer,...FIXME
