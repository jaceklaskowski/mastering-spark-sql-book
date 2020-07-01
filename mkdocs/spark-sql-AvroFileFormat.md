title: AvroFileFormat

# AvroFileFormat -- FileFormat For Avro-Encoded Files

`AvroFileFormat` is a <<spark-sql-FileFormat.adoc#, FileFormat>> for Apache Avro, i.e. a data source format that can read and write Avro-encoded data in files.

[[shortName]]
`AvroFileFormat` is a <<spark-sql-DataSourceRegister.adoc#, DataSourceRegister>> and <<spark-sql-DataSourceRegister.adoc#shortName, registers itself>> as *avro* data source.

[source, scala]
----
// ./bin/spark-shell --packages org.apache.spark:spark-avro_2.12:2.4.0

// Writing data to Avro file(s)
spark
  .range(1)
  .write
  .format("avro") // <-- Triggers AvroFileFormat
  .save("data.avro")

// Reading Avro data from file(s)
val q = spark
  .read
  .format("avro") // <-- Triggers AvroFileFormat
  .load("data.avro")
scala> q.show
+---+
| id|
+---+
|  0|
+---+
----

[[isSplitable]]
`AvroFileFormat` is <<spark-sql-FileFormat.adoc#isSplitable, splitable>>, i.e. FIXME

=== [[buildReader]] Building Partitioned Data Reader -- `buildReader` Method

[source, scala]
----
buildReader(
  spark: SparkSession,
  dataSchema: StructType,
  partitionSchema: StructType,
  requiredSchema: StructType,
  filters: Seq[Filter],
  options: Map[String, String],
  hadoopConf: Configuration): (PartitionedFile) => Iterator[InternalRow]
----

NOTE: `buildReader` is part of the <<spark-sql-FileFormat.adoc#buildReader, FileFormat Contract>> to build a <<spark-sql-PartitionedFile.adoc#, PartitionedFile>> reader.

`buildReader`...FIXME

=== [[inferSchema]] Inferring Schema -- `inferSchema` Method

[source, scala]
----
inferSchema(
  spark: SparkSession,
  options: Map[String, String],
  files: Seq[FileStatus]): Option[StructType]
----

NOTE: `inferSchema` is part of the <<spark-sql-FileFormat.adoc#inferSchema, FileFormat Contract>> to infer (return) the <<spark-sql-StructType.adoc#, schema>> of the given files.

`inferSchema`...FIXME

=== [[prepareWrite]] Preparing Write Job -- `prepareWrite` Method

[source, scala]
----
prepareWrite(
  spark: SparkSession,
  job: Job,
  options: Map[String, String],
  dataSchema: StructType): OutputWriterFactory
----

NOTE: `prepareWrite` is part of the <<spark-sql-FileFormat.adoc#prepareWrite, FileFormat Contract>> to prepare a write job.

`prepareWrite`...FIXME
