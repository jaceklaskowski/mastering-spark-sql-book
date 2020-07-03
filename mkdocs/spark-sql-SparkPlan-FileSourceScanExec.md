title: FileSourceScanExec

# FileSourceScanExec Leaf Physical Operator

`FileSourceScanExec` is a <<spark-sql-SparkPlan.adoc#LeafExecNode, leaf physical operator>> (being a <<spark-sql-SparkPlan-DataSourceScanExec.adoc#, DataSourceScanExec>>) that represents a scan over collections of files (incl. Hive tables).

`FileSourceScanExec` is <<creating-instance, created>> exclusively for a link:spark-sql-LogicalPlan-LogicalRelation.adoc[LogicalRelation] logical operator with a link:spark-sql-BaseRelation-HadoopFsRelation.adoc[HadoopFsRelation] when <<spark-sql-SparkStrategy-FileSourceStrategy.adoc#, FileSourceStrategy>> execution planning strategy is executed.

[source, scala]
----
// Create a bucketed data source table
// It is one of the most complex examples of a LogicalRelation with a HadoopFsRelation
val tableName = "bucketed_4_id"
spark
  .range(100)
  .withColumn("part", $"id" % 2)
  .write
  .partitionBy("part")
  .bucketBy(4, "id")
  .sortBy("id")
  .mode("overwrite")
  .saveAsTable(tableName)
val q = spark.table(tableName)

val sparkPlan = q.queryExecution.executedPlan
scala> :type sparkPlan
org.apache.spark.sql.execution.SparkPlan

scala> println(sparkPlan.numberedTreeString)
00 *(1) FileScan parquet default.bucketed_4_id[id#7L,part#8L] Batched: true, Format: Parquet, Location: CatalogFileIndex[file:/Users/jacek/dev/oss/spark/spark-warehouse/bucketed_4_id], PartitionCount: 2, PartitionFilters: [], PushedFilters: [], ReadSchema: struct<id:bigint>, SelectedBucketsCount: 4 out of 4

import org.apache.spark.sql.execution.FileSourceScanExec
val scan = sparkPlan.collectFirst { case exec: FileSourceScanExec => exec }.get

scala> :type scan
org.apache.spark.sql.execution.FileSourceScanExec

scala> scan.metadata.toSeq.sortBy(_._1).map { case (k, v) => s"$k -> $v" }.foreach(println)
Batched -> true
Format -> Parquet
Location -> CatalogFileIndex[file:/Users/jacek/dev/oss/spark/spark-warehouse/bucketed_4_id]
PartitionCount -> 2
PartitionFilters -> []
PushedFilters -> []
ReadSchema -> struct<id:bigint>
SelectedBucketsCount -> 4 out of 4
----

[[inputRDDs]]
`FileSourceScanExec` uses the single <<inputRDD, input RDD>> as the link:spark-sql-CodegenSupport.adoc#inputRDDs[input RDDs] (in <<spark-sql-whole-stage-codegen.adoc#, Whole-Stage Java Code Generation>>).

When <<doExecute, executed>>, `FileSourceScanExec` operator creates a <<spark-sql-FileScanRDD.adoc#, FileScanRDD>> (for <<createBucketedReadRDD, bucketed>> and <<createNonBucketedReadRDD, non-bucketed reads>>).

[source, scala]
----
scala> :type scan
org.apache.spark.sql.execution.FileSourceScanExec

val rdd = scan.execute
scala> println(rdd.toDebugString)
(6) MapPartitionsRDD[7] at execute at <console>:28 []
 |  FileScanRDD[2] at execute at <console>:27 []

import org.apache.spark.sql.execution.datasources.FileScanRDD
assert(rdd.dependencies.head.rdd.isInstanceOf[FileScanRDD])
----

`FileSourceScanExec` supports <<spark-sql-bucketing.adoc#bucket-pruning, bucket pruning>> so it only scans the bucket files required for a query.

[source, scala]
----
scala> :type scan
org.apache.spark.sql.execution.FileSourceScanExec

import org.apache.spark.sql.execution.datasources.FileScanRDD
val rdd = scan.inputRDDs.head.asInstanceOf[FileScanRDD]

import org.apache.spark.sql.execution.datasources.FilePartition
val bucketFiles = for {
  FilePartition(bucketId, files) <- rdd.filePartitions
  f <- files
} yield s"Bucket $bucketId => $f"

scala> println(bucketFiles.size)
51

scala> bucketFiles.foreach(println)
Bucket 0 => path: file:///Users/jacek/dev/oss/spark/spark-warehouse/bucketed_4_id/part=0/part-00004-5301d371-01c3-47d4-bb6b-76c3c94f3699_00000.c000.snappy.parquet, range: 0-423, partition values: [0]
Bucket 0 => path: file:///Users/jacek/dev/oss/spark/spark-warehouse/bucketed_4_id/part=0/part-00001-5301d371-01c3-47d4-bb6b-76c3c94f3699_00000.c000.snappy.parquet, range: 0-423, partition values: [0]
...
Bucket 3 => path: file:///Users/jacek/dev/oss/spark/spark-warehouse/bucketed_4_id/part=1/part-00005-5301d371-01c3-47d4-bb6b-76c3c94f3699_00003.c000.snappy.parquet, range: 0-423, partition values: [1]
Bucket 3 => path: file:///Users/jacek/dev/oss/spark/spark-warehouse/bucketed_4_id/part=1/part-00000-5301d371-01c3-47d4-bb6b-76c3c94f3699_00003.c000.snappy.parquet, range: 0-431, partition values: [1]
Bucket 3 => path: file:///Users/jacek/dev/oss/spark/spark-warehouse/bucketed_4_id/part=1/part-00007-5301d371-01c3-47d4-bb6b-76c3c94f3699_00003.c000.snappy.parquet, range: 0-423, partition values: [1]
----

`FileSourceScanExec` uses a `HashPartitioning` or the default `UnknownPartitioning` as the <<outputPartitioning, output partitioning scheme>>.

`FileSourceScanExec` is a <<ColumnarBatchScan, ColumnarBatchScan>> and <<supportsBatch, supports batch decoding>> only when the link:spark-sql-BaseRelation-HadoopFsRelation.adoc#fileFormat[FileFormat] (of the <<relation, HadoopFsRelation>>) link:spark-sql-FileFormat.adoc#supportBatch[supports it].

`FileSourceScanExec` supports <<pushedDownFilters, data source filters>> that are printed out to the console (at <<logging, INFO>> logging level) and available as <<metadata, metadata>> (e.g. in web UI or link:spark-sql-dataset-operators.adoc#explain[explain]).

```
Pushed Filters: [pushedDownFilters]
```

[[metrics]]
.FileSourceScanExec's Performance Metrics
[cols="1m,2,2",options="header",width="100%"]
|===
| Key
| Name (in web UI)
| Description

| metadataTime
| metadata time (ms)
| [[metadataTime]]

| numFiles
| number of files
| [[numFiles]]

| numOutputRows
| number of output rows
| [[numOutputRows]]

| scanTime
| scan time
| [[scanTime]]
|===

As a link:spark-sql-SparkPlan-DataSourceScanExec.adoc[DataSourceScanExec], `FileSourceScanExec` uses *Scan* for the prefix of the link:spark-sql-SparkPlan-DataSourceScanExec.adoc#nodeName[node name].

[source, scala]
----
val fileScanExec: FileSourceScanExec = ... // see the example earlier
assert(fileScanExec.nodeName startsWith "Scan")
----

.FileSourceScanExec in web UI (Details for Query)
image::images/spark-sql-FileSourceScanExec-webui-query-details.png[align="center"]

[[nodeNamePrefix]]
`FileSourceScanExec` uses *File* for link:spark-sql-SparkPlan-DataSourceScanExec.adoc#nodeNamePrefix[nodeNamePrefix] (that is used for the link:spark-sql-SparkPlan-DataSourceScanExec.adoc#simpleString[simple node description] in query plans).

[source, scala]
----
val fileScanExec: FileSourceScanExec = ... // see the example earlier
assert(fileScanExec.nodeNamePrefix == "File")

scala> println(fileScanExec.simpleString)
FileScan csv [id#20,name#21,city#22] Batched: false, Format: CSV, Location: InMemoryFileIndex[file:/Users/jacek/dev/oss/datasets/people.csv], PartitionFilters: [], PushedFilters: [], ReadSchema: struct<id:string,name:string,city:string>
----

[[internal-registries]]
.FileSourceScanExec's Internal Properties (e.g. Registries, Counters and Flags)
[cols="1m,2",options="header",width="100%"]
|===
| Name
| Description

| metadata
a| [[metadata]]

[source, scala]
----
metadata: Map[String, String]
----

Metadata

NOTE: `metadata` is part of link:spark-sql-SparkPlan-DataSourceScanExec.adoc#metadata[DataSourceScanExec] contract.

| pushedDownFilters
a| [[pushedDownFilters]] link:spark-sql-Filter.adoc[Data source filters] that are <<dataFilters, dataFilters>> expressions link:spark-sql-SparkStrategy-DataSourceStrategy.adoc#translateFilter[converted to their respective filters]

[TIP]
====
Enable <<logging, INFO>> logging level to see <<pushedDownFilters, pushedDownFilters>> printed out to the console.

```
Pushed Filters: [pushedDownFilters]
```
====

Used when `FileSourceScanExec` is requested for the <<metadata, metadata>> and <<inputRDD, input RDD>>
|===

[[logging]]
[TIP]
====
Enable `ALL` logging level for `org.apache.spark.sql.execution.FileSourceScanExec` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.execution.FileSourceScanExec=ALL
```

Refer to link:spark-logging.adoc[Logging].
====

=== [[createNonBucketedReadRDD]] Creating RDD for Non-Bucketed Reads -- `createNonBucketedReadRDD` Internal Method

[source, scala]
----
createNonBucketedReadRDD(
  readFile: (PartitionedFile) => Iterator[InternalRow],
  selectedPartitions: Seq[PartitionDirectory],
  fsRelation: HadoopFsRelation): RDD[InternalRow]
----

`createNonBucketedReadRDD` calculates the maximum size of partitions (`maxSplitBytes`) based on the following properties:

* <<spark-sql-properties.adoc#spark.sql.files.maxPartitionBytes, spark.sql.files.maxPartitionBytes>> (default: `128m`)

* <<spark-sql-properties.adoc#spark.sql.files.openCostInBytes, spark.sql.files.openCostInBytes>> (default: `4m`)

`createNonBucketedReadRDD` sums up the size of all the files (with the extra <<spark-sql-properties.adoc#spark.sql.files.openCostInBytes, spark.sql.files.openCostInBytes>>) for the given `selectedPartitions` and divides the sum by the "default parallelism" (i.e. number of CPU cores assigned to a Spark application) that gives `bytesPerCore`.

The maximum size of partitions is then the minimum of <<spark-sql-properties.adoc#spark.sql.files.maxPartitionBytes, spark.sql.files.maxPartitionBytes>> and the bigger of <<spark-sql-properties.adoc#spark.sql.files.openCostInBytes, spark.sql.files.openCostInBytes>> and the `bytesPerCore`.

`createNonBucketedReadRDD` prints out the following INFO message to the logs:

```
Planning scan with bin packing, max size: [maxSplitBytes] bytes, open cost is considered as scanning [openCostInBytes] bytes.
```

For every file (as Hadoop's `FileStatus`) in every partition (as `PartitionDirectory` in the given `selectedPartitions`), `createNonBucketedReadRDD` <<getBlockLocations, gets the HDFS block locations>> to create <<spark-sql-PartitionedFile.adoc#, PartitionedFiles>> (possibly split per the maximum size of partitions if the <<spark-sql-BaseRelation-HadoopFsRelation.adoc#fileFormat, FileFormat>> of the <<fsRelation, HadoopFsRelation>> is <<spark-sql-FileFormat.adoc#isSplitable, splittable>>). The partitioned files are then sorted by number of bytes to read (aka _split size_) in decreasing order (from the largest to the smallest).

`createNonBucketedReadRDD` "compresses" multiple splits per partition if together they are smaller than the `maxSplitBytes` ("Next Fit Decreasing") that gives the necessary partitions (file blocks as <<spark-sql-FileScanRDD.adoc#FilePartition, FilePartitions>>).

In the end, `createNonBucketedReadRDD` creates a <<spark-sql-FileScanRDD.adoc#, FileScanRDD>> (with the given `(PartitionedFile) => Iterator[InternalRow]` read function and the partitions).

NOTE: `createNonBucketedReadRDD` is used exclusively when `FileSourceScanExec` physical operator is requested for the <<inputRDD, input RDD>> (and neither the optional <<spark-sql-BaseRelation-HadoopFsRelation.adoc#bucketSpec, bucketing specification>> of the <<relation, HadoopFsRelation>> is defined nor <<spark-sql-SQLConf.adoc#bucketingEnabled, bucketing is enabled>>).

=== [[selectedPartitions]] `selectedPartitions` Internal Lazily-Initialized Property

[source, scala]
----
selectedPartitions: Seq[PartitionDirectory]
----

`selectedPartitions`...FIXME

[NOTE]
====
`selectedPartitions` is used when `FileSourceScanExec` is requested for the following:

* <<outputPartitioning, outputPartitioning>> and <<outputOrdering, outputOrdering>> when <<spark-sql-SQLConf.adoc#bucketingEnabled, bucketing is enabled>> and the optional <<spark-sql-BaseRelation-HadoopFsRelation.adoc#bucketSpec, bucketing specification>> of the <<relation, HadoopFsRelation>> is defined

* <<metadata, metadata>>

* <<inputRDD, inputRDD>>
====

=== [[creating-instance]] Creating FileSourceScanExec Instance

`FileSourceScanExec` takes the following when created:

* [[relation]] <<spark-sql-BaseRelation-HadoopFsRelation.adoc#, HadoopFsRelation>>
* [[output]] Output schema <<spark-sql-Expression-Attribute.adoc#, attributes>>
* [[requiredSchema]] <<spark-sql-StructType.adoc#, Schema>>
* [[partitionFilters]] `partitionFilters` <<spark-sql-Expression.adoc#, expressions>>
* [[optionalBucketSet]] Bucket IDs for bucket pruning (`Option[BitSet]`)
* [[dataFilters]] `dataFilters` <<spark-sql-Expression.adoc#, expressions>>
* [[tableIdentifier]] Optional `TableIdentifier`

`FileSourceScanExec` initializes the <<internal-registries, internal registries and counters>>.

=== [[outputPartitioning]] Output Partitioning Scheme -- `outputPartitioning` Attribute

[source, scala]
----
outputPartitioning: Partitioning
----

NOTE: `outputPartitioning` is part of the <<spark-sql-SparkPlan.adoc#outputPartitioning, SparkPlan Contract>> to specify output data partitioning.

`outputPartitioning` can be one of the following:

* <<spark-sql-SparkPlan-Partitioning.adoc#HashPartitioning, HashPartitioning>> (with the <<spark-sql-BucketSpec.adoc#bucketColumnNames, bucket column names>> and the <<spark-sql-BucketSpec.adoc#numBuckets, number of buckets>> of the <<spark-sql-BaseRelation-HadoopFsRelation.adoc#bucketSpec, bucketing specification>> of the <<relation, HadoopFsRelation>>) when <<spark-sql-SQLConf.adoc#bucketingEnabled, bucketing is enabled>> and the <<relation, HadoopFsRelation>> has a <<spark-sql-BaseRelation-HadoopFsRelation.adoc#bucketSpec, bucketing specification>> defined

* <<spark-sql-SparkPlan-Partitioning.adoc#UnknownPartitioning, UnknownPartitioning>> (with `0` partitions) otherwise

=== [[createBucketedReadRDD]] Creating FileScanRDD with Bucketing Support -- `createBucketedReadRDD` Internal Method

[source, scala]
----
createBucketedReadRDD(
  bucketSpec: BucketSpec,
  readFile: (PartitionedFile) => Iterator[InternalRow],
  selectedPartitions: Seq[PartitionDirectory],
  fsRelation: HadoopFsRelation): RDD[InternalRow]
----

`createBucketedReadRDD` prints the following INFO message to the logs:

```
Planning with [numBuckets] buckets
```

`createBucketedReadRDD` maps the available files of the input `selectedPartitions` into link:spark-sql-PartitionedFile.adoc[PartitionedFiles]. For every file, `createBucketedReadRDD` <<getBlockLocations, getBlockLocations>> and <<getBlockHosts, getBlockHosts>>.

`createBucketedReadRDD` then groups the `PartitionedFiles` by bucket ID.

NOTE: Bucket ID is of the format *_0000n*, i.e. the bucket ID prefixed with up to four ``0``s.

`createBucketedReadRDD` prunes (filters out) the bucket files for the bucket IDs that are not listed in the <<optionalBucketSet, bucket IDs for bucket pruning>>.

`createBucketedReadRDD` creates a <<spark-sql-FileScanRDD.adoc#FilePartition, FilePartition>> (_file block_) for every bucket ID and the (pruned) bucket `PartitionedFiles`.

In the end, `createBucketedReadRDD` creates a link:spark-sql-FileScanRDD.adoc#creating-instance[FileScanRDD] (with the input `readFile` for the link:spark-sql-FileScanRDD.adoc#readFunction[read function] and the file blocks (`FilePartitions`) for every bucket ID for link:spark-sql-FileScanRDD.adoc#filePartitions[partitions])

[TIP]
====
Use `RDD.toDebugString` to see `FileScanRDD` in the RDD execution plan (aka RDD lineage).

[source, scala]
----
// Create a bucketed table
spark.range(8).write.bucketBy(4, "id").saveAsTable("b1")

scala> sql("desc extended b1").where($"col_name" like "%Bucket%").show
+--------------+---------+-------+
|      col_name|data_type|comment|
+--------------+---------+-------+
|   Num Buckets|        4|       |
|Bucket Columns|   [`id`]|       |
+--------------+---------+-------+

val bucketedTable = spark.table("b1")

val lineage = bucketedTable.queryExecution.toRdd.toDebugString
scala> println(lineage)
(4) MapPartitionsRDD[26] at toRdd at <console>:26 []
 |  FileScanRDD[25] at toRdd at <console>:26 []
----
====

NOTE: `createBucketedReadRDD` is used exclusively when `FileSourceScanExec` physical operator is requested for the <<inputRDD, inputRDD>> (and the optional <<spark-sql-BaseRelation-HadoopFsRelation.adoc#bucketSpec, bucketing specification>> of the <<relation, HadoopFsRelation>> is defined and <<spark-sql-SQLConf.adoc#bucketingEnabled, bucketing is enabled>>).

=== [[supportsBatch]] `supportsBatch` Attribute

[source, scala]
----
supportsBatch: Boolean
----

NOTE: `supportsBatch` is part of the link:spark-sql-ColumnarBatchScan.adoc#supportsBatch[ColumnarBatchScan Contract] to enable link:spark-sql-vectorized-parquet-reader.adoc[vectorized decoding].

`supportsBatch` is enabled (`true`) only when the link:spark-sql-BaseRelation-HadoopFsRelation.adoc#fileFormat[FileFormat] (of the <<relation, HadoopFsRelation>>) link:spark-sql-FileFormat.adoc#supportBatch[supports vectorized decoding]. Otherwise, `supportsBatch` is disabled (i.e. `false`).

NOTE: <<spark-sql-FileFormat.adoc#, FileFormat>> does not support vectorized decoding by default (i.e. <<spark-sql-FileFormat.adoc#supportBatch, supportBatch>> flag is disabled). Only <<spark-sql-ParquetFileFormat.adoc#, ParquetFileFormat>> and <<spark-sql-OrcFileFormat.adoc#, OrcFileFormat>> have support for it under certain conditions.

=== [[ColumnarBatchScan]] FileSourceScanExec As ColumnarBatchScan

`FileSourceScanExec` is a link:spark-sql-ColumnarBatchScan.adoc[ColumnarBatchScan] and <<supportsBatch, supports batch decoding>> only when the link:spark-sql-BaseRelation-HadoopFsRelation.adoc#fileFormat[FileFormat] (of the <<relation, HadoopFsRelation>>) link:spark-sql-FileFormat.adoc#supportBatch[supports it].

`FileSourceScanExec` has <<needsUnsafeRowConversion, needsUnsafeRowConversion>> flag enabled for `ParquetFileFormat` data sources exclusively.

`FileSourceScanExec` has <<vectorTypes, vectorTypes>>...FIXME

==== [[needsUnsafeRowConversion]] `needsUnsafeRowConversion` Flag

[source, scala]
----
needsUnsafeRowConversion: Boolean
----

NOTE: `needsUnsafeRowConversion` is part of link:spark-sql-ColumnarBatchScan.adoc#needsUnsafeRowConversion[ColumnarBatchScan Contract] to control the name of the variable for an input row while link:spark-sql-CodegenSupport.adoc#consume[generating the Java source code to consume generated columns or row from a physical operator].

`needsUnsafeRowConversion` is enabled (i.e. `true`) when the following conditions all hold:

. link:spark-sql-BaseRelation-HadoopFsRelation.adoc#fileFormat[FileFormat] of the <<relation, HadoopFsRelation>> is link:spark-sql-ParquetFileFormat.adoc[ParquetFileFormat]

. link:spark-sql-properties.adoc#spark.sql.parquet.enableVectorizedReader[spark.sql.parquet.enableVectorizedReader] configuration property is enabled (default: `true`)

Otherwise, `needsUnsafeRowConversion` is disabled (i.e. `false`).

NOTE: `needsUnsafeRowConversion` is used when `FileSourceScanExec` is <<doExecute, executed>> (and <<supportsBatch, supportsBatch>> flag is off).

==== [[vectorTypes]] Fully-Qualified Class Names (Types) of Concrete ColumnVectors -- `vectorTypes` Method

[source, scala]
----
vectorTypes: Option[Seq[String]]
----

NOTE: `vectorTypes` is part of link:spark-sql-ColumnarBatchScan.adoc#vectorTypes[ColumnarBatchScan Contract] to..FIXME.

`vectorTypes` simply requests the link:spark-sql-BaseRelation-HadoopFsRelation.adoc#fileFormat[FileFormat] of the <<relation, HadoopFsRelation>> for link:spark-sql-FileFormat.adoc#vectorTypes[vectorTypes].

=== [[doExecute]] Executing Physical Operator (Generating RDD[InternalRow]) -- `doExecute` Method

[source, scala]
----
doExecute(): RDD[InternalRow]
----

NOTE: `doExecute` is part of the <<spark-sql-SparkPlan.adoc#doExecute, SparkPlan Contract>> to generate the runtime representation of a structured query as a distributed computation over <<spark-sql-InternalRow.adoc#, internal binary rows>> on Apache Spark (i.e. `RDD[InternalRow]`).

`doExecute` branches off per <<supportsBatch, supportsBatch>> flag.

NOTE: <<supportsBatch, supportsBatch>> flag can be enabled for <<spark-sql-ParquetFileFormat.adoc#, ParquetFileFormat>> and <<spark-sql-OrcFileFormat.adoc#, OrcFileFormat>> built-in file formats (under certain conditions).

With <<supportsBatch, supportsBatch>> flag enabled, `doExecute` creates a <<spark-sql-SparkPlan-WholeStageCodegenExec.adoc#, WholeStageCodegenExec>> physical operator (with the `FileSourceScanExec` for the <<spark-sql-SparkPlan-WholeStageCodegenExec.adoc#child, child physical operator>> and link:spark-sql-SparkPlan-WholeStageCodegenExec.adoc#codegenStageId[codegenStageId] as `0`) and link:spark-sql-SparkPlan.adoc#execute[executes] it right after.

With <<supportsBatch, supportsBatch>> flag disabled, `doExecute` creates an `unsafeRows` RDD to scan over which is different per <<needsUnsafeRowConversion, needsUnsafeRowConversion>> flag.

If <<needsUnsafeRowConversion, needsUnsafeRowConversion>> flag is on, `doExecute` takes the <<inputRDD, inputRDD>> and creates a new RDD by applying a function to each partition (using `RDD.mapPartitionsWithIndexInternal`):

. Creates a link:spark-sql-UnsafeProjection.adoc#create[UnsafeProjection] for the link:spark-sql-catalyst-QueryPlan.adoc#schema[schema]

. Initializes the link:spark-sql-Projection.adoc#initialize[UnsafeProjection]

. Maps over the rows in a partition iterator using the `UnsafeProjection` projection

Otherwise, `doExecute` simply takes the <<inputRDD, inputRDD>> as the `unsafeRows` RDD (with no changes).

`doExecute` takes the link:spark-sql-ColumnarBatchScan.adoc#numOutputRows[numOutputRows] metric and creates a new RDD by mapping every element in the `unsafeRows` and incrementing the `numOutputRows` metric.

[TIP]
====
Use `RDD.toDebugString` to review the RDD lineage and "reverse-engineer" the values of the <<supportsBatch, supportsBatch>> and <<needsUnsafeRowConversion, needsUnsafeRowConversion>> flags given the number of RDDs.

With <<supportsBatch, supportsBatch>> off and <<needsUnsafeRowConversion, needsUnsafeRowConversion>> on you should see two more RDDs in the RDD lineage.
====

=== [[inputRDD]] Creating Input RDD of Internal Rows -- `inputRDD` Internal Property

[source, scala]
----
inputRDD: RDD[InternalRow]
----

NOTE: `inputRDD` is a Scala lazy value which is computed once when accessed and cached afterwards.

`inputRDD` is an input `RDD` of link:spark-sql-InternalRow.adoc[internal binary rows] (i.e. `InternalRow`) that is used when `FileSourceScanExec` physical operator is requested for <<inputRDDs, inputRDDs>> and <<doExecute, execution>>.

When created, `inputRDD` requests <<relation, HadoopFsRelation>> to get the underlying link:spark-sql-BaseRelation-HadoopFsRelation.adoc#fileFormat[FileFormat] that is in turn requested to link:spark-sql-FileFormat.adoc#buildReaderWithPartitionValues[build a data reader with partition column values appended] (with the input parameters from the properties of <<relation, HadoopFsRelation>> and <<pushedDownFilters, pushedDownFilters>>).

In case <<relation, HadoopFsRelation>> has link:spark-sql-BaseRelation-HadoopFsRelation.adoc#bucketSpec[bucketing specification] defined and link:spark-sql-bucketing.adoc#spark.sql.sources.bucketing.enabled[bucketing support is enabled], `inputRDD` <<createBucketedReadRDD, creates a FileScanRDD with bucketing>> (with the bucketing specification, the reader, <<selectedPartitions, selectedPartitions>> and the <<relation, HadoopFsRelation>> itself). Otherwise, `inputRDD` <<createNonBucketedReadRDD, createNonBucketedReadRDD>>.

NOTE: <<createBucketedReadRDD, createBucketedReadRDD>> accepts a bucketing specification while <<createNonBucketedReadRDD, createNonBucketedReadRDD>> does not.

=== [[outputOrdering]] Output Data Ordering -- `outputOrdering` Attribute

[source, scala]
----
outputOrdering: Seq[SortOrder]
----

NOTE: `outputOrdering` is part of the <<spark-sql-SparkPlan.adoc#outputOrdering, SparkPlan Contract>> to specify output data ordering.

`outputOrdering` is a `SortOrder` expression for every <<spark-sql-BucketSpec.adoc#sortColumnNames, sort column>> in `Ascending` order only when all the following hold:

* <<spark-sql-SQLConf.adoc#bucketingEnabled, bucketing is enabled>>

* <<relation, HadoopFsRelation>> has a <<spark-sql-BaseRelation-HadoopFsRelation.adoc#bucketSpec, bucketing specification>> defined

* All the buckets have a single file in it

Otherwise, `outputOrdering` is simply empty (`Nil`).

=== [[updateDriverMetrics]] `updateDriverMetrics` Internal Method

[source, scala]
----
updateDriverMetrics(): Unit
----

`updateDriverMetrics` updates the following <<metrics, performance metrics>>:

* <<numFiles, numFiles>> metric with the total of all the sizes of the files in the <<selectedPartitions, selectedPartitions>>

* <<metadataTime, metadataTime>> metric with the time spent in the <<selectedPartitions, selectedPartitions>>

In the end, `updateDriverMetrics` requests the `SQLMetrics` object to link:spark-sql-SQLMetric.adoc#postDriverMetricUpdates[posts the metric updates].

NOTE: `updateDriverMetrics` is used exclusively when `FileSourceScanExec` physical operator is requested for the <<inputRDD, input RDD>> (the very first time).

=== [[getBlockLocations]] `getBlockLocations` Internal Method

[source, scala]
----
getBlockLocations(file: FileStatus): Array[BlockLocation]
----

`getBlockLocations` simply requests the given Hadoop https://hadoop.apache.org/docs/r2.7.3/api/index.html?org/apache/hadoop/fs/LocatedFileStatus.html[FileStatus] for the block locations (`getBlockLocations`) if it is a Hadoop https://hadoop.apache.org/docs/r2.7.3/api/index.html?org/apache/hadoop/fs/LocatedFileStatus.html[LocatedFileStatus]. Otherwise, `getBlockLocations` returns an empty array.

NOTE: `getBlockLocations` is used when `FileSourceScanExec` physical operator is requested to <<createBucketedReadRDD, createBucketedReadRDD>> and <<createNonBucketedReadRDD, createNonBucketedReadRDD>>.
