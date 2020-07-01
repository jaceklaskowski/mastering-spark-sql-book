title: ParquetReadSupport

# ParquetReadSupport -- Non-Vectorized ReadSupport in Parquet Data Source

`ParquetReadSupport` is a concrete `ReadSupport` (from Apache Parquet) of <<spark-sql-UnsafeRow.adoc#, UnsafeRows>>.

`ParquetReadSupport` is <<creating-instance, created>> exclusively when `ParquetFileFormat` is requested for a <<spark-sql-ParquetFileFormat.adoc#buildReaderWithPartitionValues, data reader>> (with no support for <<spark-sql-vectorized-parquet-reader.adoc#, Vectorized Parquet Decoding>> and so falling back to parquet-mr).

[[parquet.read.support.class]]
`ParquetReadSupport` is registered as the fully-qualified class name for <<spark-sql-ParquetFileFormat.adoc#parquet.read.support.class, parquet.read.support.class>> Hadoop configuration when `ParquetFileFormat` is requested for a <<spark-sql-ParquetFileFormat.adoc#buildReaderWithPartitionValues, data reader>>.

[[creating-instance]]
[[convertTz]]
`ParquetReadSupport` takes an optional Java `TimeZone` to be created.

[[logging]]
[TIP]
====
Enable `ALL` logging level for `org.apache.spark.sql.execution.datasources.parquet.ParquetReadSupport` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.execution.datasources.parquet.ParquetReadSupport=ALL
```

Refer to <<spark-logging.adoc#, Logging>>.
====

=== [[init]] Initializing ReadSupport -- `init` Method

[source, scala]
----
init(context: InitContext): ReadContext
----

NOTE: `init` is part of the `ReadSupport` Contract to...FIXME.

`init`...FIXME

=== [[prepareForRead]] `prepareForRead` Method

[source, scala]
----
prepareForRead(
  conf: Configuration,
  keyValueMetaData: JMap[String, String],
  fileSchema: MessageType,
  readContext: ReadContext): RecordMaterializer[UnsafeRow]
----

NOTE: `prepareForRead` is part of the `ReadSupport` Contract to...FIXME.

`prepareForRead`...FIXME
