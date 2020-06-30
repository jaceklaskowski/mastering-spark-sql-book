title: CatalogStatistics

# CatalogStatistics -- Table Statistics From External Catalog (Metastore)

[[creating-instance]][[table-statistics]]
`CatalogStatistics` are *table statistics* that are stored in an link:spark-sql-ExternalCatalog.adoc[external catalog] (aka _metastore_):

* [[sizeInBytes]] Physical *total size* (in bytes)
* [[rowCount]] Estimated *number of rows* (aka _row count_)
* [[colStats]] *Column statistics* (i.e. column names and their link:spark-sql-ColumnStat.adoc[statistics])

[NOTE]
====
`CatalogStatistics` is a "subset" of the statistics in link:spark-sql-Statistics.adoc[Statistics] (as there are no concepts of link:spark-sql-Statistics.adoc#attributeStats[attributes] and link:spark-sql-Statistics.adoc#hints[broadcast hint] in metastore).

`CatalogStatistics` are often stored in a Hive metastore and are referred as *Hive statistics* while `Statistics` are the *Spark statistics*.
====

`CatalogStatistics` can be converted to link:spark-sql-Statistics.adoc[Spark statistics] using <<toPlanStats, toPlanStats>> method.

`CatalogStatistics` is <<creating-instance, created>> when:

* link:spark-sql-LogicalPlan-AnalyzeColumnCommand.adoc#run[AnalyzeColumnCommand], `AlterTableAddPartitionCommand` and `TruncateTableCommand` commands are executed (and store statistics in link:spark-sql-ExternalCatalog.adoc[ExternalCatalog])

* `CommandUtils` is requested for link:spark-sql-CommandUtils.adoc#updateTableStats[updating existing table statistics], the link:spark-sql-CommandUtils.adoc#compareAndGetNewStats[current statistics (if changed)]

* `HiveExternalCatalog` is requested for link:hive/HiveExternalCatalog.adoc#statsFromProperties[restoring Spark statistics from properties] (from a Hive Metastore)

* link:hive/DetermineTableStats.adoc#apply[DetermineTableStats] and link:spark-sql-SparkOptimizer-PruneFileSourcePartitions.adoc[PruneFileSourcePartitions] logical optimizations are executed (i.e. applied to a logical plan)

* `HiveClientImpl` is requested for a table or partition link:hive/HiveClientImpl.adoc#readHiveStats[statistics from Hive's parameters]

[source, scala]
----
scala> :type spark.sessionState.catalog
org.apache.spark.sql.catalyst.catalog.SessionCatalog

// Using higher-level interface to access CatalogStatistics
// Make sure that you ran ANALYZE TABLE (as described above)
val db = spark.catalog.currentDatabase
val tableName = "t1"
val metadata = spark.sharedState.externalCatalog.getTable(db, tableName)
val stats = metadata.stats

scala> :type stats
Option[org.apache.spark.sql.catalyst.catalog.CatalogStatistics]

// Using low-level internal SessionCatalog interface to access CatalogTables
val tid = spark.sessionState.sqlParser.parseTableIdentifier(tableName)
val metadata = spark.sessionState.catalog.getTempViewOrPermanentTableMetadata(tid)
val stats = metadata.stats

scala> :type stats
Option[org.apache.spark.sql.catalyst.catalog.CatalogStatistics]
----

[[simpleString]]
`CatalogStatistics` has a text representation.

[source, scala]
----
scala> :type stats
Option[org.apache.spark.sql.catalyst.catalog.CatalogStatistics]

scala> stats.map(_.simpleString).foreach(println)
714 bytes, 2 rows
----

=== [[toPlanStats]] Converting Metastore Statistics to Spark Statistics -- `toPlanStats` Method

[source, scala]
----
toPlanStats(planOutput: Seq[Attribute], cboEnabled: Boolean): Statistics
----

`toPlanStats` converts the table statistics (from an external metastore) to link:spark-sql-Statistics.adoc[Spark statistics].

With link:spark-sql-cost-based-optimization.adoc[cost-based optimization] enabled and <<rowCount, row count>> statistics available, `toPlanStats` creates a link:spark-sql-Statistics.adoc[Statistics] with the link:spark-sql-EstimationUtils.adoc#getOutputSize[estimated total (output) size], <<rowCount, row count>> and column statistics.

NOTE: Cost-based optimization is enabled when link:spark-sql-properties.adoc#spark.sql.cbo.enabled[spark.sql.cbo.enabled] configuration property is turned on, i.e. `true`, and is disabled by default.

Otherwise, when link:spark-sql-cost-based-optimization.adoc[cost-based optimization] is disabled, `toPlanStats` creates a link:spark-sql-Statistics.adoc[Statistics] with just the mandatory <<sizeInBytes, sizeInBytes>>.

CAUTION: FIXME Why does `toPlanStats` compute `sizeInBytes` differently per CBO?

[NOTE]
====
`toPlanStats` does the reverse of link:hive/HiveExternalCatalog.adoc#statsToProperties[HiveExternalCatalog.statsToProperties].

[source, scala]
----
FIXME Example
----
====

NOTE: `toPlanStats` is used when link:hive/HiveTableRelation.adoc#computeStats[HiveTableRelation] and link:spark-sql-LogicalPlan-LogicalRelation.adoc#computeStats[LogicalRelation] are requested for statistics.
