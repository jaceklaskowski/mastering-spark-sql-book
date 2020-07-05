# HiveFileFormat

`HiveFileFormat` is a link:../spark-sql-FileFormat.adoc[FileFormat] for <<prepareWrite, writing Hive tables>>.

[[shortName]]
`HiveFileFormat` is a link:../spark-sql-DataSourceRegister.adoc[DataSourceRegister] and link:../spark-sql-DataSourceRegister.adoc#shortName[registers] itself as *hive* data source.

NOTE: Hive data source can only be used with tables and you cannot read or write files of Hive data source directly. Use [DataFrameReader.table](../DataFrameReader.md#table) to load from or link:../spark-sql-DataFrameWriter.adoc#saveAsTable[DataFrameWriter.saveAsTable] to write data to a Hive table.

`HiveFileFormat` is <<creating-instance, created>> exclusively when `SaveAsHiveFile` is requested to link:../hive/SaveAsHiveFile.adoc#saveAsHiveFile[saveAsHiveFile] (when link:InsertIntoHiveDirCommand.adoc[InsertIntoHiveDirCommand] and link:InsertIntoHiveTable.adoc[InsertIntoHiveTable] logical commands are executed).

[[fileSinkConf]][[creating-instance]]
`HiveFileFormat` takes a `FileSinkDesc` when created.

[[inferSchema]]
`HiveFileFormat` throws a `UnsupportedOperationException` when requested to link:../spark-sql-FileFormat.adoc#inferSchema[inferSchema].

```
inferSchema is not supported for hive data source.
```

=== [[prepareWrite]] Preparing Write Job -- `prepareWrite` Method

[source, scala]
----
prepareWrite(
  sparkSession: SparkSession,
  job: Job,
  options: Map[String, String],
  dataSchema: StructType): OutputWriterFactory
----

NOTE: `prepareWrite` is part of the link:../spark-sql-FileFormat.adoc#prepareWrite[FileFormat] contract.

`prepareWrite` sets the *mapred.output.format.class* property to be the `getOutputFileFormatClassName` of the Hive `TableDesc` of the <<fileSinkConf, FileSinkDesc>>.

`prepareWrite` requests the `HiveTableUtil` helper object to `configureJobPropertiesForStorageHandler`.

`prepareWrite` requests the Hive `Utilities` helper object to `copyTableJobPropertiesToConf`.

In the end, `prepareWrite` creates a new `OutputWriterFactory` that creates a new `HiveOutputWriter` when requested for a new `OutputWriter` instance.
