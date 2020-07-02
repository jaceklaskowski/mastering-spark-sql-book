title: Statistics

# Statistics -- Estimates of Plan Statistics and Query Hints

[[creating-instance]]
`Statistics` holds the statistics estimates and query hints of a logical operator:

* [[sizeInBytes]] Total (output) size (in bytes)
* [[rowCount]] Estimated number of rows (aka _row count_)
* [[attributeStats]] Column attribute statistics (aka _column (equi-height) histograms_)
* [[hints]] link:spark-sql-HintInfo.adoc[Query hints]

NOTE: *Cost statistics*, *plan statistics* or *query statistics* are all synonyms and used interchangeably.

You can access statistics and query hints of a logical plan using link:spark-sql-LogicalPlanStats.adoc#stats[stats] property.

[source, scala]
----
val q = spark.range(5).hint("broadcast").join(spark.range(1), "id")
val plan = q.queryExecution.optimizedPlan
val stats = plan.stats

scala> :type stats
org.apache.spark.sql.catalyst.plans.logical.Statistics

scala> println(stats.simpleString)
sizeInBytes=213.0 B, hints=none
----

NOTE: Use link:spark-sql-cost-based-optimization.adoc#ANALYZE-TABLE[ANALYZE TABLE COMPUTE STATISTICS] SQL command to compute <<sizeInBytes, total size>> and <<rowCount, row count>> statistics of a table.

NOTE: Use link:spark-sql-cost-based-optimization.adoc#ANALYZE-TABLE[ANALYZE TABLE COMPUTE STATISTICS FOR COLUMNS] SQL Command to generate <<attributeStats, column (equi-height) histograms>> of a table.

NOTE: Use `Dataset.hint` or `SELECT` SQL statement with hints to link:spark-sql-hint-framework.adoc#specifying-query-hints[specify query hints].

`Statistics` is <<creating-instance, created>> when:

* link:spark-sql-LogicalPlan-LeafNode.adoc#computeStats[Leaf logical operators] (specifically) and link:spark-sql-LogicalPlanStats.adoc#stats[logical operators] (in general) are requested for statistics estimates

* link:hive/HiveTableRelation.adoc#computeStats[HiveTableRelation] and link:spark-sql-LogicalPlan-LogicalRelation.adoc#computeStats[LogicalRelation] are requested for statistics estimates (through link:spark-sql-CatalogStatistics.adoc#toPlanStats[CatalogStatistics])

NOTE: <<rowCount, row count>> estimate is used in link:spark-sql-Optimizer-CostBasedJoinReorder.adoc[CostBasedJoinReorder] logical optimization when link:spark-sql-cost-based-optimization.adoc#spark.sql.cbo.enabled[cost-based optimization is enabled].

[NOTE]
====
link:spark-sql-CatalogStatistics.adoc[CatalogStatistics] is a "subset" of all possible `Statistics` (as there are no concepts of <<attributeStats, attributes>> and <<hints, query hints>> in link:spark-sql-ExternalCatalog.adoc[metastore]).

`CatalogStatistics` are statistics stored in an external catalog (usually a Hive metastore) and are often referred as *Hive statistics* while `Statistics` represents the *Spark statistics*.
====

[[simpleString]][[toString]]
`Statistics` comes with `simpleString` method that is used for the readable *text representation* (that is `toString` with *Statistics* prefix).

[source, scala]
----
import org.apache.spark.sql.catalyst.plans.logical.Statistics
import org.apache.spark.sql.catalyst.plans.logical.HintInfo
val stats = Statistics(sizeInBytes = 10, rowCount = Some(20), hints = HintInfo(broadcast = true))

scala> println(stats)
Statistics(sizeInBytes=10.0 B, rowCount=20, hints=(broadcast))

scala> println(stats.simpleString)
sizeInBytes=10.0 B, rowCount=20, hints=(broadcast)
----
