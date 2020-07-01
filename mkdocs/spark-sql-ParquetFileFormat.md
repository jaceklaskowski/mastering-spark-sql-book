# ParquetFileFormat

[[shortName]]
`ParquetFileFormat` is the link:spark-sql-FileFormat.adoc[FileFormat] for *parquet* data source (i.e. link:spark-sql-DataSourceRegister.adoc#shortName[registers itself to handle files in parquet format] and converts them to Spark SQL rows).

NOTE: `parquet` is the link:spark-sql-DataFrameReader.adoc#source[default data source format] in Spark SQL.

NOTE: http://parquet.apache.org/[Apache Parquet] is a columnar storage format for the Apache Hadoop ecosystem with support for efficient storage and encoding of data.

[source, scala]
----
// All the following queries are equivalent
// schema has to be specified manually
import org.apache.spark.sql.types.StructType
val schema = StructType($"id".int :: Nil)

spark.read.schema(schema).format("parquet").load("parquet-datasets")

// The above is equivalent to the following shortcut
// Implicitly does format("parquet").load
spark.read.schema(schema).parquet("parquet-datasets")

// parquet is the default data source format
spark.read.schema(schema).load("parquet-datasets")
----

[[isSplitable]]
`ParquetFileFormat` is <<spark-sql-FileFormat.adoc#isSplitable, splitable>>, i.e. FIXME

[[supportBatch]]
`ParquetFileFormat` link:spark-sql-FileFormat.adoc#supportBatch[supports vectorized parquet decoding in whole-stage code generation] when all of the following hold:

. link:spark-sql-properties.adoc#spark.sql.parquet.enableVectorizedReader[spark.sql.parquet.enableVectorizedReader] configuration property is enabled

. link:spark-sql-properties.adoc#spark.sql.codegen.wholeStage[spark.sql.codegen.wholeStage] internal configuration property is enabled

. The number of fields in the schema is at most link:spark-sql-properties.adoc#spark.sql.codegen.maxFields[spark.sql.codegen.maxFields] internal configuration property

. All the fields in the output schema are of link:spark-sql-DataType.adoc#AtomicType[AtomicType]

`ParquetFileFormat` supports *filter predicate push-down optimization* (via <<createFilter, createFilter>>) as per the following <<ParquetFilters, table>>.

[[ParquetFilters]]
.Spark Data Source Filters to Parquet Filter Predicates Conversions (aka `ParquetFilters.createFilter`)
[cols="1m,2",options="header",width="100%"]
|===
| Data Source Filter
| Parquet FilterPredicate

| IsNull
| [[IsNull]] `FilterApi.eq`

| IsNotNull
| [[IsNotNull]] `FilterApi.notEq`

| EqualTo
| [[EqualTo]] `FilterApi.eq`

| Not EqualTo
| [[NotEqualTo]] `FilterApi.notEq`

| EqualNullSafe
| [[EqualNullSafe]] `FilterApi.eq`

| Not EqualNullSafe
| [[NotEqualNullSafe]] `FilterApi.notEq`

| LessThan
| [[LessThan]] `FilterApi.lt`

| LessThanOrEqual
| [[LessThanOrEqual]] `FilterApi.ltEq`

| GreaterThan
| [[GreaterThan]] `FilterApi.gt`

| GreaterThanOrEqual
| [[GreaterThanOrEqual]] `FilterApi.gtEq`

| And
| [[And]] `FilterApi.and`

| Or
| [[Or]] `FilterApi.or`

| No
| [[Not]] `FilterApi.not`
|===

[[logging]]
[TIP]
====
Enable `ALL` logging level for `org.apache.spark.sql.execution.datasources.parquet.ParquetFileFormat` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.execution.datasources.parquet.ParquetFileFormat=ALL
```

Refer to <<spark-logging.adoc#, Logging>>.
====

=== [[prepareWrite]] Preparing Write Job -- `prepareWrite` Method

[source, scala]
----
prepareWrite(
  sparkSession: SparkSession,
  job: Job,
  options: Map[String, String],
  dataSchema: StructType): OutputWriterFactory
----

NOTE: `prepareWrite` is part of the <<spark-sql-FileFormat.adoc#prepareWrite, FileFormat Contract>> to prepare a write job.

`prepareWrite`...FIXME

=== [[inferSchema]] `inferSchema` Method

[source, scala]
----
inferSchema(
  sparkSession: SparkSession,
  parameters: Map[String, String],
  files: Seq[FileStatus]): Option[StructType]
----

NOTE: `inferSchema` is part of link:spark-sql-FileFormat.adoc#inferSchema[FileFormat Contract] to...FIXME.

`inferSchema`...FIXME

=== [[vectorTypes]] `vectorTypes` Method

[source, scala]
----
vectorTypes(
  requiredSchema: StructType,
  partitionSchema: StructType,
  sqlConf: SQLConf): Option[Seq[String]]
----

NOTE: `vectorTypes` is part of link:spark-sql-FileFormat.adoc#vectorTypes[FileFormat Contract] to define the concrete column vector class names for each column used in a columnar batch when <<supportBatch, enabled>>.

`vectorTypes` creates a collection of the names of <<spark-sql-OffHeapColumnVector.adoc#, OffHeapColumnVector>> or <<spark-sql-OnHeapColumnVector.adoc#, OnHeapColumnVector>> when <<spark-sql-properties.adoc#spark.sql.columnVector.offheap.enabled, spark.sql.columnVector.offheap.enabled>> property is enabled or disabled, respectively.

NOTE: <<spark-sql-properties.adoc#spark.sql.columnVector.offheap.enabled, spark.sql.columnVector.offheap.enabled>> property is disabled (`false`) by default.

The size of the collection are all the fields of the given `requiredSchema` and `partitionSchema` schemas.

=== [[buildReaderWithPartitionValues]] Building Data Reader With Partition Column Values Appended -- `buildReaderWithPartitionValues` Method

[source, scala]
----
buildReaderWithPartitionValues(
  sparkSession: SparkSession,
  dataSchema: StructType,
  partitionSchema: StructType,
  requiredSchema: StructType,
  filters: Seq[Filter],
  options: Map[String, String],
  hadoopConf: Configuration): (PartitionedFile) => Iterator[InternalRow]
----

NOTE: `buildReaderWithPartitionValues` is part of link:spark-sql-FileFormat.adoc#buildReaderWithPartitionValues[FileFormat Contract] to build a data reader with the partition column values appended.

`buildReaderWithPartitionValues` sets the <<options, configuration options>> in the input `hadoopConf`.

[[options]]
.Hadoop Configuration Options
[cols="1m,3",options="header",width="100%"]
|===
| Name
| Value

| parquet.read.support.class
| [[parquet.read.support.class]] <<spark-sql-ParquetReadSupport.adoc#, ParquetReadSupport>>

| org.apache.spark.sql.parquet.row.requested_schema
| [[org.apache.spark.sql.parquet.row.requested_schema]] link:spark-sql-DataType.adoc#json[JSON] representation of `requiredSchema`

| org.apache.spark.sql.parquet.row.attributes
| [[org.apache.spark.sql.parquet.row.attributes]] link:spark-sql-DataType.adoc#json[JSON] representation of `requiredSchema`

| spark.sql.session.timeZone
| [[spark.sql.session.timeZone]] <<spark-sql-properties.adoc#spark.sql.session.timeZone, spark.sql.session.timeZone>>

| spark.sql.parquet.binaryAsString
| [[spark.sql.parquet.binaryAsString]] <<spark-sql-properties.adoc#spark.sql.parquet.binaryAsString, spark.sql.parquet.binaryAsString>>

| spark.sql.parquet.int96AsTimestamp
| [[spark.sql.parquet.int96AsTimestamp]] <<spark-sql-properties.adoc#spark.sql.parquet.int96AsTimestamp, spark.sql.parquet.int96AsTimestamp>>

|===

`buildReaderWithPartitionValues` requests `ParquetWriteSupport` to `setSchema`.

`buildReaderWithPartitionValues` tries to push filters down to create a Parquet `FilterPredicate` (aka `pushed`).

NOTE: Filter predicate push-down optimization for parquet data sources uses link:spark-sql-properties.adoc#spark.sql.parquet.filterPushdown[spark.sql.parquet.filterPushdown] configuration property (default: enabled).

With link:spark-sql-properties.adoc#spark.sql.parquet.filterPushdown[spark.sql.parquet.filterPushdown] configuration property enabled, `buildReaderWithPartitionValues` takes the input Spark data source `filters` and converts them to Parquet filter predicates if possible (as described in the <<ParquetFilters, table>>). Otherwise, the Parquet filter predicate is not specified.

NOTE: `buildReaderWithPartitionValues` creates filter predicates for the following types: link:spark-sql-DataType.adoc#BooleanType[BooleanType], link:spark-sql-DataType.adoc#IntegerType[IntegerType], link:spark-sql-DataType.adoc#LongType[LongType], link:spark-sql-DataType.adoc#FloatType[FloatType], link:spark-sql-DataType.adoc#DoubleType[DoubleType], link:spark-sql-DataType.adoc#StringType[StringType], link:spark-sql-DataType.adoc#BinaryType[BinaryType].

`buildReaderWithPartitionValues` broadcasts the input `hadoopConf` Hadoop `Configuration`.

In the end, `buildReaderWithPartitionValues` gives a function that takes a link:spark-sql-PartitionedFile.adoc[PartitionedFile] and does the following:

. Creates a Hadoop `FileSplit` for the input `PartitionedFile`

. Creates a Parquet `ParquetInputSplit` for the Hadoop `FileSplit` created

. Gets the broadcast Hadoop `Configuration`

. Creates a flag that says whether to apply timezone conversions to int96 timestamps or not (aka `convertTz`)

. Creates a Hadoop `TaskAttemptContextImpl` (with the broadcast Hadoop `Configuration` and a Hadoop `TaskAttemptID` for a map task)

. Sets the Parquet `FilterPredicate` (only when link:spark-sql-properties.adoc#spark.sql.parquet.filterPushdown[spark.sql.parquet.filterPushdown] configuration property is enabled and it is by default)

The function then branches off on whether link:spark-sql-VectorizedParquetRecordReader.adoc[Parquet vectorized reader] is enabled or not.

NOTE: link:spark-sql-VectorizedParquetRecordReader.adoc[Parquet vectorized reader] is enabled by default.

With link:spark-sql-VectorizedParquetRecordReader.adoc[Parquet vectorized reader] enabled, the function does the following:

. Creates a link:spark-sql-VectorizedParquetRecordReader.adoc#creating-instance[VectorizedParquetRecordReader] and a <<spark-sql-RecordReaderIterator.adoc#, RecordReaderIterator>>

. Requests `VectorizedParquetRecordReader` to link:spark-sql-VectorizedParquetRecordReader.adoc#initialize[initialize] (with the Parquet `ParquetInputSplit` and the Hadoop `TaskAttemptContextImpl`)

. Prints out the following DEBUG message to the logs:
+
```
Appending [partitionSchema] [partitionValues]
```

. Requests `VectorizedParquetRecordReader` to link:spark-sql-VectorizedParquetRecordReader.adoc#initBatch[initBatch]

. (only with <<supportBatch, supportBatch>> enabled) Requests `VectorizedParquetRecordReader` to link:spark-sql-VectorizedParquetRecordReader.adoc#enableReturningBatches[enableReturningBatches]

. In the end, the function gives the <<spark-sql-RecordReaderIterator.adoc#, RecordReaderIterator>> (over the `VectorizedParquetRecordReader`) as the `Iterator[InternalRow]`

With link:spark-sql-VectorizedParquetRecordReader.adoc[Parquet vectorized reader] disabled, the function does the following:

. FIXME (since Parquet vectorized reader is enabled by default it's of less interest currently)

=== [[mergeSchemasInParallel]] `mergeSchemasInParallel` Method

[source, scala]
----
mergeSchemasInParallel(
  filesToTouch: Seq[FileStatus],
  sparkSession: SparkSession): Option[StructType]
----

`mergeSchemasInParallel`...FIXME

NOTE: `mergeSchemasInParallel` is used when...FIXME
