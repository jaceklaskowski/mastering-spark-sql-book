# FileIndex

:hadoop-version: 2.10.0
:url-hadoop-javadoc: https://hadoop.apache.org/docs/r{hadoop-version}/api

`FileIndex` is the <<contract, abstraction>> of <<implementations, file indices>> that knows the <<rootPaths, root paths>> and <<partitionSchema, partition schema>> of a relation.

`FileIndex` is associated with a link:spark-sql-BaseRelation-HadoopFsRelation.adoc[HadoopFsRelation].

[[contract]]
.FileIndex Contract
[cols="30m,70",options="header",width="100%"]
|===
| Method
| Description

| inputFiles
a| [[inputFiles]]

[source, scala]
----
inputFiles: Array[String]
----

File names to read when scanning this relation

Used when:

* `CatalogFileIndex` is requested for the link:CatalogFileIndex.adoc#inputFiles[input files]

* `HadoopFsRelation` is requested for the link:spark-sql-BaseRelation-HadoopFsRelation.adoc#inputFiles[input files]

| listFiles
a| [[listFiles]]

[source, scala]
----
listFiles(
  partitionFilters: Seq[Expression],
  dataFilters: Seq[Expression]): Seq[PartitionDirectory]
----

File names (grouped into partitions when the data is partitioned)

Used when:

* `FileSourceScanExec` physical operator is requested for link:spark-sql-SparkPlan-FileSourceScanExec.adoc#selectedPartitions[selectedPartitions]

* `HiveMetastoreCatalog` is requested to link:hive/HiveMetastoreCatalog.adoc#inferIfNeeded[inferIfNeeded]

* link:spark-sql-SparkOptimizer-OptimizeMetadataOnlyQuery.adoc[OptimizeMetadataOnlyQuery] logical optimization is executed

* `CatalogFileIndex` is requested for the link:CatalogFileIndex.adoc#listFiles[files]

| metadataOpsTimeNs
a| [[metadataOpsTimeNs]]

[source, scala]
----
metadataOpsTimeNs: Option[Long] = None
----

Metadata operation time for listing files (in nanoseconds)

Used when `FileSourceScanExec` leaf physical operator is requested for <<spark-sql-SparkPlan-FileSourceScanExec.adoc#selectedPartitions, selectedPartitions>>

| partitionSchema
a| [[partitionSchema]]

[source, scala]
----
partitionSchema: StructType
----

Used when:

* `CatalogFileIndex` is requested to <<CatalogFileIndex.adoc#filterPartitions, filterPartitions>>

* `DataSource` is requested to <<spark-sql-DataSource.adoc#getOrInferFileFormatSchema, getOrInferFileFormatSchema>> and <<spark-sql-DataSource.adoc#resolveRelation, resolve a FileFormat-based relation>>

| refresh
a| [[refresh]]

[source, scala]
----
refresh(): Unit
----

Refreshes cached file listings

Used when:

* `CacheManager` is requested to <<spark-sql-CacheManager.adoc#lookupAndRefresh, lookupAndRefresh>>

* <<spark-sql-LogicalPlan-InsertIntoHadoopFsRelationCommand.adoc#, InsertIntoHadoopFsRelationCommand>> is executed

* `LogicalRelation` leaf logical operator is requested to <<spark-sql-LogicalPlan-LogicalRelation.adoc#refresh, refresh>> (for a <<spark-sql-BaseRelation-HadoopFsRelation.adoc#, HadoopFsRelation>>)

| rootPaths
a| [[rootPaths]]

[source, scala]
----
rootPaths: Seq[Path]
----

Root paths from which the catalog gets the files (as Hadoop {url-hadoop-javadoc}/org/apache/hadoop/fs/Path.html[Paths]). There could be a single root path of the entire table (with partition directories) or individual partitions.

Used when:

* `HiveMetastoreCatalog` is requested for a link:hive/HiveMetastoreCatalog.adoc#getCached[LogicalRelation over a HadoopFsRelation cached] (when requested to link:hive/HiveMetastoreCatalog.adoc#convertToLogicalRelation[convert a HiveTableRelation])

* `CacheManager` is requested to link:spark-sql-CacheManager.adoc#lookupAndRefresh[lookupAndRefresh]

* `FileSourceScanExec` physical operator is requested for the link:spark-sql-SparkPlan-FileSourceScanExec.adoc#metadata[metadata]

* `DDLUtils` utility is used to link:spark-sql-DDLUtils.adoc#verifyNotReadPath[verifyNotReadPath]

* link:spark-sql-Analyzer-DataSourceAnalysis.adoc[DataSourceAnalysis] logical resolution rule is executed (for a link:InsertIntoTable.adoc[InsertIntoTable] with a link:spark-sql-BaseRelation-HadoopFsRelation.adoc[HadoopFsRelation])

| sizeInBytes
a| [[sizeInBytes]]

[source, scala]
----
sizeInBytes: Long
----

Estimated size of the data of the relation (in bytes)

Used when:

* `HadoopFsRelation` is requested for the <<spark-sql-BaseRelation-HadoopFsRelation.adoc#sizeInBytes, estimated size>>

* link:spark-sql-SparkOptimizer-PruneFileSourcePartitions.adoc[PruneFileSourcePartitions] logical optimization is executed

|===

[[implementations]]
.FileIndexes (Direct Implementations and Extensions Only)
[cols="30,70",options="header",width="100%"]
|===
| FileIndex
| Description

| link:CatalogFileIndex.adoc[CatalogFileIndex]
| [[CatalogFileIndex]]

| link:PartitioningAwareFileIndex.adoc[PartitioningAwareFileIndex]
| [[PartitioningAwareFileIndex]]

|===
