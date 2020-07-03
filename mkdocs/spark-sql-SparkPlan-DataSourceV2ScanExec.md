title: DataSourceV2ScanExec

# DataSourceV2ScanExec Leaf Physical Operator

`DataSourceV2ScanExec` is a <<spark-sql-SparkPlan.adoc#LeafExecNode, leaf physical operator>> that represents a <<spark-sql-LogicalPlan-DataSourceV2Relation.adoc#, DataSourceV2Relation>> logical operator at execution time.

NOTE: A <<spark-sql-LogicalPlan-DataSourceV2Relation.adoc#, DataSourceV2Relation>> logical operator is created exclusively when `DataFrameReader` is requested to <<spark-sql-DataFrameReader.adoc#load, "load" data (as a DataFrame)>> (from a data source with <<spark-sql-ReadSupport.adoc#, ReadSupport>>).

`DataSourceV2ScanExec` supports <<spark-sql-ColumnarBatchScan.adoc#, ColumnarBatchScan>> with <<supportsBatch, vectorized batch decoding>> (when <<creating-instance, created>> for a <<reader, DataSourceReader>> that supports it, i.e. the `DataSourceReader` is a link:spark-sql-SupportsScanColumnarBatch.adoc[SupportsScanColumnarBatch] with the link:spark-sql-SupportsScanColumnarBatch.adoc#enableBatchRead[enableBatchRead] flag enabled).

`DataSourceV2ScanExec` is also a <<spark-sql-DataSourceV2StringFormat.adoc#, DataSourceV2StringFormat>>, i.e....FIXME

`DataSourceV2ScanExec` is <<creating-instance, created>> exclusively when <<spark-sql-SparkStrategy-DataSourceV2Strategy.adoc#, DataSourceV2Strategy>> execution planning strategy is executed (i.e. applied to a logical plan) and finds a <<spark-sql-LogicalPlan-DataSourceV2Relation.adoc#, DataSourceV2Relation>> logical operator.

[[inputRDDs]]
`DataSourceV2ScanExec` gives the single <<inputRDD, input RDD>> as the link:spark-sql-CodegenSupport.adoc#inputRDDs[only input RDD of internal rows] (when `WholeStageCodegenExec` physical operator is link:spark-sql-SparkPlan-WholeStageCodegenExec.adoc#doExecute[executed]).

=== [[doExecute]] Executing Physical Operator (Generating RDD[InternalRow]) -- `doExecute` Method

[source, scala]
----
doExecute(): RDD[InternalRow]
----

NOTE: `doExecute` is part of <<spark-sql-SparkPlan.adoc#doExecute, SparkPlan Contract>> to generate the runtime representation of a structured query as a distributed computation over <<spark-sql-InternalRow.adoc#, internal binary rows>> on Apache Spark (i.e. `RDD[InternalRow]`).

`doExecute`...FIXME

=== [[supportsBatch]] `supportsBatch` Property

[source, scala]
----
supportsBatch: Boolean
----

NOTE: `supportsBatch` is part of link:spark-sql-ColumnarBatchScan.adoc#supportsBatch[ColumnarBatchScan Contract] to control whether the physical operator supports link:spark-sql-vectorized-parquet-reader.adoc[vectorized decoding] or not.

`supportsBatch` is enabled (`true`) only when the <<reader, DataSourceReader>> is a link:spark-sql-SupportsScanColumnarBatch.adoc[SupportsScanColumnarBatch] with the link:spark-sql-SupportsScanColumnarBatch.adoc#enableBatchRead[enableBatchRead] flag enabled.

NOTE: link:spark-sql-SupportsScanColumnarBatch.adoc#enableBatchRead[enableBatchRead] flag is enabled by default.

`supportsBatch` is disabled (i.e. `false`) otherwise.

=== [[creating-instance]] Creating DataSourceV2ScanExec Instance

`DataSourceV2ScanExec` takes the following when created:

* [[output]] Output schema (as a collection of `AttributeReferences`)
* [[reader]] link:spark-sql-DataSourceReader.adoc[DataSourceReader]

`DataSourceV2ScanExec` initializes the <<internal-properties, internal properties>>.

=== [[inputRDD]] Creating Input RDD of Internal Rows -- `inputRDD` Internal Property

[source, scala]
----
inputRDD: RDD[InternalRow]
----

NOTE: `inputRDD` is a Scala lazy value which is computed once when accessed and cached afterwards.

`inputRDD` branches off per the type of the <<reader, DataSourceReader>>:

. For a `ContinuousReader` in Spark Structured Streaming, `inputRDD` is a `ContinuousDataSourceRDD` that...FIXME

. For a <<spark-sql-SupportsScanColumnarBatch.adoc#, SupportsScanColumnarBatch>> with the <<spark-sql-SupportsScanColumnarBatch.adoc#enableBatchRead, enableBatchRead>> flag enabled, `inputRDD` is a <<spark-sql-DataSourceRDD.adoc#, DataSourceRDD>> with the <<batchPartitions, batchPartitions>>

. For all other types of the <<reader, DataSourceReader>>, `inputRDD` is a <<spark-sql-DataSourceRDD.adoc#, DataSourceRDD>> with the <<partitions, partitions>>.

NOTE: `inputRDD` is used when `DataSourceV2ScanExec` physical operator is requested for the <<inputRDDs, input RDDs>> and to <<doExecute, execute>>.

=== [[internal-properties]] Internal Properties

[cols="30m,70",options="header",width="100%"]
|===
| Name
| Description

| batchPartitions
a| [[batchPartitions]] Input partitions of <<spark-sql-ColumnarBatch.adoc#, ColumnarBatches>> (`Seq[InputPartition[ColumnarBatch]]`)

| partitions
a| [[partitions]] Input partitions of <<spark-sql-InternalRow.adoc#, InternalRows>> (`Seq[InputPartition[InternalRow]]`)

|===
