title: InsertIntoHadoopFsRelationCommand

# InsertIntoHadoopFsRelationCommand Logical Command

`InsertIntoHadoopFsRelationCommand` is a <<spark-sql-LogicalPlan-DataWritingCommand.adoc#, logical command>> that writes the result of executing a <<query, query>> to an <<outputPath, output path>> in the given <<fileFormat, FileFormat>> (and <<creating-instance, other properties>>).

`InsertIntoHadoopFsRelationCommand` is <<creating-instance, created>> when:

* `DataSource` is requested to xref:spark-sql-DataSource.adoc#planForWritingFileFormat[plan for writing to a FileFormat-based data source] for the following:
** xref:spark-sql-LogicalPlan-CreateDataSourceTableAsSelectCommand.adoc[CreateDataSourceTableAsSelectCommand] logical command
** xref:spark-sql-LogicalPlan-InsertIntoDataSourceDirCommand.adoc[InsertIntoDataSourceDirCommand] logical command
** xref:spark-sql-DataFrameWriter.adoc#save[DataFrameWriter.save] operator with DataSource V1 data sources

* xref:spark-sql-Analyzer-DataSourceAnalysis.adoc[DataSourceAnalysis] post-hoc logical resolution rule is executed (and resolves a xref:InsertIntoTable.adoc[InsertIntoTable] logical operator with a xref:spark-sql-BaseRelation-HadoopFsRelation.adoc[HadoopFsRelation])

[[partitionOverwriteMode]][[PartitionOverwriteMode]]
`InsertIntoHadoopFsRelationCommand` uses *partitionOverwriteMode* option that overrides <<spark-sql-properties.adoc#spark.sql.sources.partitionOverwriteMode, spark.sql.sources.partitionOverwriteMode>> property for <<spark-sql-dynamic-partition-inserts.adoc#, dynamic partition inserts>>.

=== [[creating-instance]] Creating InsertIntoHadoopFsRelationCommand Instance

`InsertIntoHadoopFsRelationCommand` takes the following to be created:

* [[outputPath]] Output Hadoop's https://hadoop.apache.org/docs/r2.7.3/api/index.html?org/apache/hadoop/fs/Path.html[Path]
* [[staticPartitions]] Static table partitions (`Map[String, String]`)
* [[ifPartitionNotExists]] `ifPartitionNotExists` flag
* [[partitionColumns]] Partition columns (`Seq[Attribute]`)
* [[bucketSpec]] <<spark-sql-BucketSpec.adoc#, BucketSpec>>
* [[fileFormat]] <<spark-sql-FileFormat.adoc#, FileFormat>>
* [[options]] Options (`Map[String, String]`)
* [[query]] <<spark-sql-LogicalPlan.adoc#, Logical plan>>
* [[mode]] `SaveMode`
* [[catalogTable]] <<spark-sql-CatalogTable.adoc#, CatalogTable>>
* [[fileIndex]] `FileIndex`
* [[outputColumnNames]] Output column names

[NOTE]
====
<<staticPartitions, staticPartitions>> may hold zero or more partitions as follows:

* Always empty when <<creating-instance, created>> when `DataSource` is requested to <<spark-sql-DataSource.adoc#planForWritingFileFormat, planForWritingFileFormat>>

* Possibly with partitions when <<creating-instance, created>> when `DataSourceAnalysis` posthoc logical resolution rule is <<spark-sql-Analyzer-DataSourceAnalysis.adoc#apply, applied>> to an <<InsertIntoTable.adoc#, InsertIntoTable>> logical operator over a <<spark-sql-BaseRelation-HadoopFsRelation.adoc#, HadoopFsRelation>> relation

With that, <<staticPartitions, staticPartitions>> are simply the partitions of an <<InsertIntoTable.adoc#, InsertIntoTable>> logical operator.
====

=== [[run]] Executing Data-Writing Logical Command -- `run` Method

[source, scala]
----
run(
  sparkSession: SparkSession,
  child: SparkPlan): Seq[Row]
----

NOTE: `run` is part of link:spark-sql-LogicalPlan-DataWritingCommand.adoc#run[DataWritingCommand] contract.

`run` uses the <<spark-sql-SQLConf.adoc#manageFilesourcePartitions, spark.sql.hive.manageFilesourcePartitions>> configuration property to...FIXME

CAUTION: FIXME When is the <<catalogTable, catalogTable>> defined?

CAUTION: FIXME When is `tracksPartitionsInCatalog` of `CatalogTable` enabled?

`run` gets the <<partitionOverwriteMode, partitionOverwriteMode>> option...FIXME

`run` uses `FileCommitProtocol` utility to instantiate a committer based on the <<spark-sql-properties.adoc#spark.sql.sources.commitProtocolClass, spark.sql.sources.commitProtocolClass>> (default: <<spark-sql-SQLHadoopMapReduceCommitProtocol.adoc#, SQLHadoopMapReduceCommitProtocol>>) and the <<outputPath, outputPath>>, the <<dynamicPartitionOverwrite, dynamicPartitionOverwrite>>, and random `jobId`.

For insertion, `run` simply uses the `FileFormatWriter` utility to `write` and then...FIXME (does some table-specific "tasks").

Otherwise (for non-insertion case), `run` simply prints out the following INFO message to the logs and finishes.

```
Skipping insertion into a relation that already exists.
```

`run` uses `SchemaUtils` to <<spark-sql-SchemaUtils.adoc#checkColumnNameDuplication, make sure that there are no duplicates>> in the <<outputColumnNames, outputColumnNames>>.
