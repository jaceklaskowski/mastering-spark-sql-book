# CatalogFileIndex

:hadoop-version: 2.10.0
:java-version: 8
:url-hadoop-javadoc: https://hadoop.apache.org/docs/r{hadoop-version}/api
:java-api: https://docs.oracle.com/javase/{java-version}/docs/api

`CatalogFileIndex` is a link:FileIndex.adoc[FileIndex] that is <<creating-instance, created>> when:

* `HiveMetastoreCatalog` is requested to link:hive/HiveMetastoreCatalog.adoc#convertToLogicalRelation[convert a HiveTableRelation to a LogicalRelation]

* `DataSource` is requested to link:spark-sql-DataSource.adoc#resolveRelation[create a BaseRelation for a FileFormat]

=== [[creating-instance]] Creating CatalogFileIndex Instance

`CatalogFileIndex` takes the following to be created:

* [[sparkSession]] link:spark-sql-SparkSession.adoc[SparkSession]
* [[table]] link:spark-sql-CatalogTable.adoc[CatalogTable]
* [[sizeInBytes]] Size (in bytes)

`CatalogFileIndex` initializes the <<internal-properties, internal properties>>.

=== [[listFiles]] Partition Files -- `listFiles` Method

[source, scala]
----
listFiles(
  partitionFilters: Seq[Expression],
  dataFilters: Seq[Expression]): Seq[PartitionDirectory]
----

NOTE: `listFiles` is part of the link:FileIndex.adoc#listFiles[FileIndex] contract.

`listFiles` <<filterPartitions, lists the partitions>> for the input partition filters and then requests them for the link:PartitioningAwareFileIndex.adoc#listFiles[underlying partition files].

=== [[inputFiles]] `inputFiles` Method

[source, scala]
----
inputFiles: Array[String]
----

NOTE: `inputFiles` is part of the link:FileIndex.adoc#inputFiles[FileIndex] contract.

`inputFiles` <<filterPartitions, lists all the partitions>> and then requests them for the link:PartitioningAwareFileIndex.adoc#inputFiles[input files].

=== [[rootPaths]] `rootPaths` Method

[source, scala]
----
rootPaths: Seq[Path]
----

NOTE: `rootPaths` is part of the link:FileIndex.adoc#rootPaths[FileIndex] contract.

`rootPaths` simply returns the <<baseLocation, baseLocation>> converted to a Hadoop {url-hadoop-javadoc}/org/apache/hadoop/fs/Path.html[Path].

=== [[filterPartitions]] Listing Partitions By Given Predicate Expressions -- `filterPartitions` Method

[source, scala]
----
filterPartitions(
  filters: Seq[Expression]): InMemoryFileIndex
----

`filterPartitions` requests the <<table, CatalogTable>> for the link:spark-sql-CatalogTable.adoc#partitionColumnNames[partition columns].

For a partitioned table, `filterPartitions` starts tracking time. `filterPartitions` requests the link:spark-sql-SessionState.adoc#catalog[SessionCatalog] for the link:spark-sql-SessionCatalog.adoc#listPartitionsByFilter[partitions by filter] and creates a link:PrunedInMemoryFileIndex.adoc[PrunedInMemoryFileIndex] (with the partition listing time).

For an unpartitioned table (no partition columns defined), `filterPartitions` simply returns a link:InMemoryFileIndex.adoc[InMemoryFileIndex] (with the <<rootPaths, rootPaths>> and no user-specified schema).

[NOTE]
====
`filterPartitions` is used when:

* `HiveMetastoreCatalog` is requested to link:hive/HiveMetastoreCatalog.adoc#convertToLogicalRelation[convert a HiveTableRelation to a LogicalRelation]

* `CatalogFileIndex` is requested to <<listFiles, listFiles>> and <<inputFiles, inputFiles>>

* link:spark-sql-SparkOptimizer-PruneFileSourcePartitions.adoc[PruneFileSourcePartitions] logical optimization is executed
====

=== [[internal-properties]] Internal Properties

[cols="30m,70",options="header",width="100%"]
|===
| Name
| Description

| baseLocation
a| [[baseLocation]] Base location (as a Java {java-api}/java/net/URI.html[URI]) as defined in the <<table, CatalogTable>> metadata (under the link:spark-sql-CatalogStorageFormat.adoc#locationUri[locationUri] of the link:spark-sql-CatalogTable.adoc#storage[storage])

Used when `CatalogFileIndex` is requested to <<filterPartitions, filter the partitions>> and for the <<rootPaths, root paths>>

| hadoopConf
a| [[hadoopConf]] Hadoop {url-hadoop-javadoc}/org/apache/hadoop/conf/Configuration.html[Configuration]

Used when `CatalogFileIndex` is requested to <<filterPartitions, filter the partitions>>

|===
