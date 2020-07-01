# InputPartition

`InputPartition` is the <<contract, abstraction>> of <<implementations, input partitions>> in <<spark-sql-data-source-api-v2.adoc#, Data Source API V2>> that can <<createPartitionReader, create an InputPartitionReader>> and optionally <<preferredLocations, specify preferred locations>>.

`InputPartition` is also a Java https://docs.oracle.com/javase/8/docs/api/java/io/Serializable.html[Serializable].

`InputPartition` is associated with <<spark-sql-DataSourceReader.adoc#, DataSourceReader>> abstraction and its extension <<spark-sql-SupportsScanColumnarBatch.adoc#, SupportsScanColumnarBatch>>.

`InputPartition` is used for the following:

* Creating <<spark-sql-DataSourceRDD.adoc#, DataSourceRDD>>, <<spark-sql-DataSourceRDDPartition.adoc#, DataSourceRDDPartition>>, `ContinuousDataSourceRDD` and `ContinuousDataSourceRDDPartition`

* Requesting <<spark-sql-SparkPlan-DataSourceV2ScanExec.adoc#, DataSourceV2ScanExec>> physical operator for the <<spark-sql-SparkPlan-DataSourceV2ScanExec.adoc#partitions, partitions>> or <<spark-sql-SparkPlan-DataSourceV2ScanExec.adoc#batchPartitions, batchPartitions>> (of the input RDD)

NOTE: It _appears_ that all concrete <<implementations, input partitions>> are used in Spark Structured Streaming only.

[[contract]]
.InputPartition Contract
[cols="30m,70",options="header",width="100%"]
|===
| Method
| Description

| createPartitionReader
a| [[createPartitionReader]]

[source, java]
----
InputPartitionReader<T> createPartitionReader()
----

Creates an <<spark-sql-InputPartitionReader.adoc#, InputPartitionReader>>

Used when:

* `DataSourceRDD` is requested to <<spark-sql-DataSourceRDD.adoc#compute, compute a partition>>

* `ContinuousQueuedDataReader` is created

| preferredLocations
a| [[preferredLocations]]

[source, java]
----
String[] preferredLocations()
----

Specifies the preferred locations (executor hosts)

Default: `(empty)`

Used when:

* `DataSourceRDD` is requested for the <<spark-sql-DataSourceRDD.adoc#getPreferredLocations, preferred locations>>

* `ContinuousDataSourceRDD` is requested for the preferred locations

|===

[[implementations]]
[[extensions]]
.InputPartitions (Direct Implementations and Extensions Only)
[cols="30m,70",options="header",width="100%"]
|===
| InputPartition
| Description

| ContinuousInputPartition
| [[ContinuousInputPartition]] InputPartitions for *Continuous Stream Processing* in Spark Structured Streaming

Consult https://jaceklaskowski.gitbooks.io/spark-structured-streaming/spark-sql-streaming-ContinuousReader.html[The Internals of Spark Structured Streaming]

| ContinuousMemoryStreamInputPartition
| [[ContinuousMemoryStreamInputPartition]] Used in Spark Structured Streaming

| KafkaMicroBatchInputPartition
| [[KafkaMicroBatchInputPartition]] Used in Spark Structured Streaming

| MemoryStreamInputPartition
| [[MemoryStreamInputPartition]] Used in Spark Structured Streaming

| RateStreamMicroBatchInputPartition
| [[RateStreamMicroBatchInputPartition]] Used in Spark Structured Streaming

| TextSocketContinuousInputPartition
| [[TextSocketContinuousInputPartition]] Used in Spark Structured Streaming

| Anonymous
| [[TextSocketMicroBatchReader]] Used in Spark Structured Streaming

|===
