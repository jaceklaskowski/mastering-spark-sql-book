title: FileFormat

# FileFormat -- Data Sources to Read and Write Data In Files

`FileFormat` is the <<contract, contract>> for <<implementations, data sources>> that <<buildReader, read>> and <<prepareWrite, write>> data stored in files.

[[contract]]
.FileFormat Contract
[cols="1m,2",options="header",width="100%"]
|===
| Method
| Description

| buildReader
a| [[buildReader]]

[source, scala]
----
buildReader(
  sparkSession: SparkSession,
  dataSchema: StructType,
  partitionSchema: StructType,
  requiredSchema: StructType,
  filters: Seq[Filter],
  options: Map[String, String],
  hadoopConf: Configuration): PartitionedFile => Iterator[InternalRow]
----

Builds a Catalyst data reader, i.e. a function that reads a <<spark-sql-PartitionedFile.adoc#, PartitionedFile>> file as <<spark-sql-InternalRow.adoc#, InternalRows>>.

`buildReader` throws an `UnsupportedOperationException` by default (and should therefore be overriden to work):

```
buildReader is not supported for [this]
```

Used exclusively when `FileFormat` is requested to <<buildReaderWithPartitionValues, buildReaderWithPartitionValues>>

| <<buildReaderWithPartitionValues-internals, buildReaderWithPartitionValues>>
a| [[buildReaderWithPartitionValues]]

[source, scala]
----
buildReaderWithPartitionValues(
  sparkSession: SparkSession,
  dataSchema: StructType,
  partitionSchema: StructType,
  requiredSchema: StructType,
  filters: Seq[Filter],
  options: Map[String, String],
  hadoopConf: Configuration): PartitionedFile => Iterator[InternalRow]
----

`buildReaderWithPartitionValues` builds a data reader with partition column values appended, i.e. a function that is used to read a single file in (as a <<spark-sql-PartitionedFile.adoc#, PartitionedFile>>) as an `Iterator` of <<spark-sql-InternalRow.adoc#, InternalRows>> (like <<buildReader, buildReader>>) with the partition values appended.

Used exclusively when `FileSourceScanExec` physical operator is requested for the <<spark-sql-SparkPlan-FileSourceScanExec.adoc#inputRDD, inputRDD>> (when requested for the <<spark-sql-SparkPlan-FileSourceScanExec.adoc#inputRDDs, inputRDDs>> and <<spark-sql-SparkPlan-FileSourceScanExec.adoc#doExecute, execution>>)

| inferSchema
a| [[inferSchema]]

[source, scala]
----
inferSchema(
  sparkSession: SparkSession,
  options: Map[String, String],
  files: Seq[FileStatus]): Option[StructType]
----

Infers (returns) the <<spark-sql-StructType.adoc#, schema>> of the given files (as Hadoop's https://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/fs/FileStatus.html[FileStatuses]) if supported. Otherwise, `None` should be returned.

Used when:

* `HiveMetastoreCatalog` is requested to link:hive/HiveMetastoreCatalog.adoc#inferIfNeeded[inferIfNeeded] (when `RelationConversions` logical evaluation rule is requested to link:hive/RelationConversions.adoc#convert[convert a HiveTableRelation to a LogicalRelation] for `parquet`, `native` and `hive` ORC storage formats)

* `DataSource` is requested to <<spark-sql-DataSource.adoc#getOrInferFileFormatSchema, getOrInferFileFormatSchema>> and <<spark-sql-DataSource.adoc#resolveRelation, resolveRelation>>

| isSplitable
a| [[isSplitable]]

[source, scala]
----
isSplitable(
  sparkSession: SparkSession,
  options: Map[String, String],
  path: Path): Boolean
----

Controls whether the format (under the given path as Hadoop https://hadoop.apache.org/docs/current/api/org/apache/hadoop/fs/Path.html[Path]) can be split or not.

`isSplitable` is disabled (`false`) by default.

Used exclusively when `FileSourceScanExec` physical operator is requested to <<spark-sql-SparkPlan-FileSourceScanExec.adoc#createNonBucketedReadRDD, create an RDD for non-bucketed reads>> (when requested for the <<spark-sql-SparkPlan-FileSourceScanExec.adoc#inputRDD, inputRDD>> and neither the optional <<spark-sql-BaseRelation-HadoopFsRelation.adoc#bucketSpec, bucketing specification>> of the <<spark-sql-SparkPlan-FileSourceScanExec.adoc#relation, HadoopFsRelation>> is defined nor <<spark-sql-SQLConf.adoc#bucketingEnabled, bucketing is enabled>>)

| prepareWrite
a| [[prepareWrite]]

[source, scala]
----
prepareWrite(
  sparkSession: SparkSession,
  job: Job,
  options: Map[String, String],
  dataSchema: StructType): OutputWriterFactory
----

Prepares a write job and returns an `OutputWriterFactory`

Used exclusively when `FileFormatWriter` is requested to <<spark-sql-FileFormatWriter.adoc#write, write query result>>

| supportBatch
a| [[supportBatch]]

[source, scala]
----
supportBatch(
  sparkSession: SparkSession,
  dataSchema: StructType): Boolean
----

Flag that says whether the format supports <<spark-sql-vectorized-parquet-reader.adoc#, vectorized decoding>> (aka _columnar batch_) or not.

Default: `false`

Used exclusively when `FileSourceScanExec` physical operator is requested for the <<spark-sql-SparkPlan-FileSourceScanExec.adoc#supportsBatch, supportsBatch>>

| vectorTypes
a| [[vectorTypes]]

[source, scala]
----
vectorTypes(
  requiredSchema: StructType,
  partitionSchema: StructType,
  sqlConf: SQLConf): Option[Seq[String]]
----

Defines the fully-qualified class names (_types_) of the concrete <<spark-sql-ColumnVector.adoc#, ColumnVectors>> for every column in the input `requiredSchema` and `partitionSchema` schemas that are used in a columnar batch.

Default: undefined (`None`)

Used exclusively when `FileSourceScanExec` leaf physical operator is requested for the <<spark-sql-SparkPlan-FileSourceScanExec.adoc#vectorTypes, vectorTypes>>
|===

[[implementations]]
.FileFormats (Direct Implementations and Extensions)
[width="100%",cols="1,2",options="header"]
|===
| FileFormat
| Description

| <<spark-sql-AvroFileFormat.adoc#, AvroFileFormat>>
| [[AvroFileFormat]] Avro data source

| link:hive/HiveFileFormat.adoc[HiveFileFormat]
| [[HiveFileFormat]] Writes hive tables

| <<spark-sql-OrcFileFormat.adoc#, OrcFileFormat>>
| [[OrcFileFormat]] ORC data source

| <<spark-sql-ParquetFileFormat.adoc#, ParquetFileFormat>>
| [[ParquetFileFormat]] Parquet data source

| <<spark-sql-TextBasedFileFormat.adoc#, TextBasedFileFormat>>
| [[TextBasedFileFormat]] Base for text splitable `FileFormats`
|===

=== [[buildReaderWithPartitionValues-internals]] Building Data Reader With Partition Column Values Appended -- `buildReaderWithPartitionValues` Method

[source, scala]
----
buildReaderWithPartitionValues(
  sparkSession: SparkSession,
  dataSchema: StructType,
  partitionSchema: StructType,
  requiredSchema: StructType,
  filters: Seq[Filter],
  options: Map[String, String],
  hadoopConf: Configuration): PartitionedFile => Iterator[InternalRow]
----

`buildReaderWithPartitionValues` is simply an enhanced <<buildReader, buildReader>> that appends link:spark-sql-PartitionedFile.adoc#partitionValues[partition column values] to the internal rows produced by the reader function from <<buildReader, buildReader>>.

Internally, `buildReaderWithPartitionValues` <<buildReader, builds a data reader>> with the input parameters and gives a *data reader function* (of a link:spark-sql-PartitionedFile.adoc[PartitionedFile] to an `Iterator[InternalRow]`) that does the following:

. Creates a converter by requesting `GenerateUnsafeProjection` to link:spark-sql-GenerateUnsafeProjection.adoc#generate[generate an UnsafeProjection] for the attributes of the input `requiredSchema` and `partitionSchema`

. Applies the data reader to a `PartitionedFile` and converts the result using the converter on the joined row with the link:spark-sql-PartitionedFile.adoc#partitionValues[partition column values] appended.

NOTE: `buildReaderWithPartitionValues` is used exclusively when `FileSourceScanExec` physical operator is requested for the link:spark-sql-SparkPlan-FileSourceScanExec.adoc#inputRDDs[input RDDs].
