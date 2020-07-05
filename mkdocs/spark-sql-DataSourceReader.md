# DataSourceReader

`DataSourceReader` is the <<contract, abstraction>> of <<implementations, data source readers>> in <<spark-sql-data-source-api-v2.adoc#, Data Source API V2>> that can <<planInputPartitions, plan InputPartitions>> and know the <<readSchema, schema for reading>>.

`DataSourceReader` is created to scan the data from a data source when:

* `DataSourceV2Relation` is requested to <<spark-sql-LogicalPlan-DataSourceV2Relation.adoc#newReader, create a new reader>>

* `ReadSupport` is requested to <<spark-sql-ReadSupport.adoc#createReader, create a reader>>

`DataSourceReader` is used to create `StreamingDataSourceV2Relation` and <<spark-sql-SparkPlan-DataSourceV2ScanExec.adoc#, DataSourceV2ScanExec>> physical operator

NOTE: It _appears_ that all concrete <<implementations, data source readers>> are used in Spark Structured Streaming only.

[[contract]]
.DataSourceReader Contract
[cols="30m,70",options="header",width="100%"]
|===
| Method
| Description

| planInputPartitions
a| [[planInputPartitions]]

[source, java]
----
List<InputPartition<InternalRow>> planInputPartitions()
----

<<spark-sql-InputPartition.adoc#, InputPartitions>>

Used exclusively when `DataSourceV2ScanExec` leaf physical operator is requested for the <<spark-sql-SparkPlan-DataSourceV2ScanExec.adoc#partitions, input partitions>> (and simply delegates to the underlying <<spark-sql-SparkPlan-DataSourceV2ScanExec.adoc#reader, DataSourceReader>>) to create the input `RDD[InternalRow]` (`inputRDD`)

| readSchema
a| [[readSchema]]

[source, java]
----
StructType readSchema()
----

<<spark-sql-StructType.adoc#, Schema>> for reading (loading) data from a data source

Used when:

* `DataSourceV2Relation` factory object is requested to <<spark-sql-LogicalPlan-DataSourceV2Relation.adoc#create, create a DataSourceV2Relation>> (when `DataFrameReader` is requested to ["load" data (as a DataFrame)](DataFrameReader.md#load) from a data source with [ReadSupport](spark-sql-ReadSupport.md))

* `DataSourceV2Strategy` execution planning strategy is requested to <<spark-sql-SparkStrategy-DataSourceV2Strategy.adoc#pruneColumns, apply column pruning optimization>>

* Spark Structured Streaming's `MicroBatchExecution` stream execution is requested to run a single streaming batch

* Spark Structured Streaming's `ContinuousExecution` stream execution is requested to run a streaming query in continuous mode

* Spark Structured Streaming's `DataStreamReader` is requested to "load" data (as a DataFrame)

|===

[NOTE]
====
`DataSourceReader` is an `Evolving` contract that is evolving towards becoming a stable API, but is not a stable API yet and can change from one feature release to another release.

In other words, using the contract is as _"treading on thin ice"_.
====

[[implementations]]
[[extensions]]
.DataSourceReaders (Direct Implementations and Extensions Only)
[cols="30,70",options="header",width="100%"]
|===
| DataSourceReader
| Description

| ContinuousReader
| [[ContinuousReader]] `DataSourceReaders` for *Continuous Stream Processing* in Spark Structured Streaming

Consult https://jaceklaskowski.gitbooks.io/spark-structured-streaming/spark-sql-streaming-ContinuousReader.html[The Internals of Spark Structured Streaming]

| MicroBatchReader
| [[MicroBatchReader]] `DataSourceReaders` for *Micro-Batch Stream Processing* in Spark Structured Streaming

Consult https://jaceklaskowski.gitbooks.io/spark-structured-streaming/spark-sql-streaming-MicroBatchReader.html[The Internals of Spark Structured Streaming]

| <<spark-sql-SupportsPushDownFilters.adoc#, SupportsPushDownFilters>>
| [[SupportsPushDownFilters]] `DataSourceReaders` that can push down filters to the data source and reduce the size of the data to be read

| <<spark-sql-SupportsPushDownRequiredColumns.adoc#, SupportsPushDownRequiredColumns>>
| [[SupportsPushDownRequiredColumns]] `DataSourceReaders` that can push down required columns to the data source and only read these columns during scan to reduce the size of the data to be read

| <<spark-sql-SupportsReportPartitioning.adoc#, SupportsReportPartitioning>>
| [[SupportsReportPartitioning]] `DataSourceReaders` that can report data partitioning and try to avoid shuffle at Spark side

| <<spark-sql-SupportsReportStatistics.adoc#, SupportsReportStatistics>>
| [[SupportsReportStatistics]] `DataSourceReaders` that can report statistics to Spark

| <<spark-sql-SupportsScanColumnarBatch.adoc#, SupportsScanColumnarBatch>>
| [[SupportsScanColumnarBatch]] `DataSourceReaders` that can output <<spark-sql-ColumnarBatch.adoc#, ColumnarBatch>> and make the scan faster

|===
