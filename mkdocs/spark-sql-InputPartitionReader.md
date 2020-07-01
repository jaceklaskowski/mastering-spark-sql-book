# InputPartitionReader

`InputPartitionReader` is the <<contract, abstraction>> of <<implementations, input partition readers>> in <<spark-sql-data-source-api-v2.adoc#, Data Source API V2>> that can <<next, proceed to the next record>> and <<get, get the current record>>.

`InputPartitionReader` is also a Java https://docs.oracle.com/javase/8/docs/api/java/io/Closeable.html[Closeable].

`InputPartitionReader` is associated with two other abstractions: <<spark-sql-InputPartition.adoc#, InputPartition>> and `ContinuousInputPartition` that are responsible for creating <<implementations, concrete InputPartitionReaders>>.

NOTE: It _appears_ that all concrete <<implementations, input partition readers>> are used in Spark Structured Streaming only.

[[contract]]
.InputPartitionReader Contract
[cols="30m,70",options="header",width="100%"]
|===
| Method
| Description

| get
a| [[get]]

[source, java]
----
T get()
----

Gets the current record

Used when:

* `DataSourceRDD` is requested to <<spark-sql-DataSourceRDD.adoc#compute, compute a partition>>

* `DataReaderThread` is requested to run (_start up_)

| next
a| [[next]]

[source, java]
----
boolean next()
  throws IOException
----

Proceeds to the next record if available (`true`)

Used when:

* `DataSourceRDD` is requested to <<spark-sql-DataSourceRDD.adoc#compute, compute a partition>>

* `DataReaderThread` is requested to run (_start up_)

|===

[[implementations]]
[[extensions]]
.InputPartitionReaders (Direct Implementations and Extensions Only)
[cols="30m,70",options="header",width="100%"]
|===
| InputPartitionReader
| Description

| ContinuousInputPartitionReader
| [[ContinuousInputPartitionReader]] Extension that is used in Spark Structured Streaming for Continuous Stream Processing

| KafkaMicroBatchInputPartitionReader
| [[KafkaMicroBatchInputPartitionReader]] Used in Spark Structured Streaming for Kafka Data Source

| Anonymous
| [[MemoryStreamInputPartition]] Used in Spark Structured Streaming for Memory Data Source

| RateStreamMicroBatchInputPartitionReader
| [[RateStreamMicroBatchInputPartitionReader]] Used in Spark Structured Streaming for Rate Data Source

| Anonymous
| [[TextSocketMicroBatchReader]] Used in Spark Structured Streaming for Text Socket Data Source

|===
