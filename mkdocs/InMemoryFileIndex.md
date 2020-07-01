# InMemoryFileIndex

:hadoop-version: 2.10.0
:url-hadoop-javadoc: https://hadoop.apache.org/docs/r{hadoop-version}/api

`InMemoryFileIndex` is a link:PartitioningAwareFileIndex.adoc[PartitioningAwareFileIndex] for a partition schema and file list.

`InMemoryFileIndex` is <<creating-instance, created>> when:

* `HiveMetastoreCatalog` is requested to link:hive/HiveMetastoreCatalog.adoc#inferIfNeeded[inferIfNeeded] (when requested to link:hive/HiveMetastoreCatalog.adoc#convertToLogicalRelation[convert a HiveTableRelation])

* `CatalogFileIndex` is requested for the link:CatalogFileIndex.adoc#filterPartitions[partitions by the given predicate expressions] for a non-partitioned Hive table

* `DataSource` is requested to link:spark-sql-DataSource.adoc#createInMemoryFileIndex[createInMemoryFileIndex]

* Spark Structured Streaming's `FileStreamSource` is used

=== [[creating-instance]] Creating InMemoryFileIndex Instance

`InMemoryFileIndex` takes the following to be created:

* [[sparkSession]] link:spark-sql-SparkSession.adoc[SparkSession]
* [[rootPathsSpecified]] Root paths (as Hadoop {url-hadoop-javadoc}/org/apache/hadoop/fs/Path.html[Paths])
* [[parameters]] Options for partition discovery
* [[userSpecifiedSchema]] Optional user-defined link:spark-sql-StructType.adoc[schema]
* [[fileStatusCache]] `FileStatusCache` (default: `NoopCache`)

`InMemoryFileIndex` initializes the <<internal-properties, internal properties>>.

=== [[internal-properties]] Internal Properties

[cols="30m,70",options="header",width="100%"]
|===
| Name
| Description

| rootPaths
a| [[rootPaths]] The <<rootPathsSpecified, root paths>> with no `_spark_metadata` streaming metadata directories (of Spark Structured Streaming's `FileStreamSink` when reading the output of a streaming query)

NOTE: `rootPaths` is part of the link:FileIndex.adoc#rootPaths[FileIndex] contract.

|===
