title: HiveMetastoreCatalog

# HiveMetastoreCatalog -- Legacy SessionCatalog for Converting Hive Metastore Relations to Data Source Relations

`HiveMetastoreCatalog` is a link:../spark-sql-SessionCatalog.adoc[session-scoped catalog of relational entities] that knows how to <<convertToLogicalRelation, convert Hive metastore relations to data source relations>>.

`HiveMetastoreCatalog` is used by link:HiveSessionCatalog.adoc#metastoreCatalog[HiveSessionCatalog] for link:RelationConversions.adoc[RelationConversions] logical evaluation rule.

`HiveMetastoreCatalog` is <<creating-instance, created>> when `HiveSessionStateBuilder` is requested for a link:HiveSessionStateBuilder.adoc#catalog[SessionCatalog] (and creates a link:HiveSessionCatalog.adoc#metastoreCatalog[HiveSessionCatalog]).

.HiveMetastoreCatalog, HiveSessionCatalog and HiveSessionStateBuilder
image::../images/spark-sql-HiveMetastoreCatalog.png[align="center"]

=== [[creating-instance]] Creating HiveMetastoreCatalog Instance

`HiveMetastoreCatalog` takes the following to be created:

* [[sparkSession]] link:../spark-sql-SparkSession.adoc[SparkSession]

`HiveMetastoreCatalog` initializes the <<internal-properties, internal properties>>.

=== [[convertToLogicalRelation]] Converting HiveTableRelation to LogicalRelation -- `convertToLogicalRelation` Method

[source, scala]
----
convertToLogicalRelation(
  relation: HiveTableRelation,
  options: Map[String, String],
  fileFormatClass: Class[_ <: FileFormat],
  fileType: String): LogicalRelation
----

`convertToLogicalRelation` branches based on whether the input link:HiveTableRelation.adoc[HiveTableRelation] is <<convertToLogicalRelation-partitioned, partitioned>> or <<convertToLogicalRelation-not-partitioned, not>>.

[[convertToLogicalRelation-partitioned]]
When the `HiveTableRelation` is link:HiveTableRelation.adoc#isPartitioned[partitioned], `convertToLogicalRelation` uses link:configuration-properties.adoc#spark.sql.hive.manageFilesourcePartitions[spark.sql.hive.manageFilesourcePartitions] configuration property to compute the root paths. With the property enabled, the root path is simply the link:../spark-sql-CatalogTable.adoc#location[table location] (aka _locationUri_). Otherwise, the root paths are the `locationUri` of the link:../spark-sql-ExternalCatalog.adoc#listPartitions[partitions] (using the link:../spark-sql-SharedState.adoc#externalCatalog[shared ExternalCatalog]).

`convertToLogicalRelation` creates a new link:../spark-sql-LogicalPlan-LogicalRelation.adoc[LogicalRelation] with a link:../spark-sql-BaseRelation-HadoopFsRelation.adoc[HadoopFsRelation] (with no bucketing specification among things) unless a `LogicalRelation` for the table is already in a <<getCached, cache>>.

[[convertToLogicalRelation-not-partitioned]]
When the `HiveTableRelation` is not partitioned, `convertToLogicalRelation`...FIXME

In the end, `convertToLogicalRelation` replaces `exprIds` in the link:../spark-sql-LogicalPlan-LogicalRelation.adoc#output[table relation output (schema)].

NOTE: `convertToLogicalRelation` is used when link:RelationConversions.adoc[RelationConversions] logical evaluation rule is executed (with Hive tables in `parquet` as well as `native` and `hive` ORC storage formats).

=== [[inferIfNeeded]] `inferIfNeeded` Internal Method

[source, scala]
----
inferIfNeeded(
  relation: HiveTableRelation,
  options: Map[String, String],
  fileFormat: FileFormat,
  fileIndexOpt: Option[FileIndex] = None): CatalogTable
----

`inferIfNeeded`...FIXME

NOTE: `inferIfNeeded` is used when `HiveMetastoreCatalog` is requested to <<convertToLogicalRelation, convert a HiveTableRelation to a LogicalRelation over a HadoopFsRelation>>.

=== [[getCached]] `getCached` Internal Method

[source, scala]
----
getCached(
  tableIdentifier: QualifiedTableName,
  pathsInMetastore: Seq[Path],
  schemaInMetastore: StructType,
  expectedFileFormat: Class[_ <: FileFormat],
  partitionSchema: Option[StructType]): Option[LogicalRelation]
----

`getCached`...FIXME

NOTE: `getCached` is used when `HiveMetastoreCatalog` is requested to <<convertToLogicalRelation, convert a HiveTableRelation to a LogicalRelation over a HadoopFsRelation>>.

=== [[internal-properties]] Internal Properties

[cols="30m,70",options="header",width="100%"]
|===
| Name
| Description

| catalogProxy
a| [[catalogProxy]] link:../spark-sql-SessionCatalog.adoc[SessionCatalog] (of the <<sparkSession, SparkSession>>).

Used when `HiveMetastoreCatalog` is requested to <<getCached, getCached>>, <<convertToLogicalRelation, convertToLogicalRelation>>

|===
