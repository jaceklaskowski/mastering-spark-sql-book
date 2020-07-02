# JoinEstimation

`JoinEstimation` is a utility that <<estimate, computes statistics estimates and query hints of a Join logical operator>>.

`JoinEstimation` is <<creating-instance, created>> exclusively for `BasicStatsPlanVisitor` to link:spark-sql-BasicStatsPlanVisitor.adoc#visitJoin[estimate statistics of a Join logical operator].

NOTE: `BasicStatsPlanVisitor` is used only when link:spark-sql-cost-based-optimization.adoc#spark.sql.cbo.enabled[cost-based optimization is enabled].

[[creating-instance]]
[[join]]
`JoinEstimation` takes a link:spark-sql-LogicalPlan-Join.adoc[Join] logical operator when created.

[[leftStats]]
[[rightStats]]
When <<creating-instance, created>>, `JoinEstimation` immediately takes the link:spark-sql-LogicalPlanStats.adoc#stats[estimated statistics and query hints] of the link:spark-sql-LogicalPlan-Join.adoc#left[left] and link:spark-sql-LogicalPlan-Join.adoc#right[right] sides of the <<join, Join>> logical operator.

[source, scala]
----
// JoinEstimation requires row count stats for join statistics estimates
// With cost-based optimization off, size in bytes is available only
// That would give no join estimates whatsoever (except size in bytes)
// Make sure that you `--conf spark.sql.cbo.enabled=true`
scala> println(spark.sessionState.conf.cboEnabled)
true

// Build a query with join operator
// From the available data sources tables seem the best...so far
val r1 = spark.range(5)
scala> println(r1.queryExecution.analyzed.stats.simpleString)
sizeInBytes=40.0 B, hints=none

// Make the demo reproducible
val db = spark.catalog.currentDatabase
spark.sharedState.externalCatalog.dropTable(db, table = "t1", ignoreIfNotExists = true, purge = true)
spark.sharedState.externalCatalog.dropTable(db, table = "t2", ignoreIfNotExists = true, purge = true)

// FIXME What relations give row count stats?

// Register tables
spark.range(5).write.saveAsTable("t1")
spark.range(10).write.saveAsTable("t2")

// Refresh internal registries
sql("REFRESH TABLE t1")
sql("REFRESH TABLE t2")

// Calculate row count stats
val tables = Seq("t1", "t2")
tables.map(t => s"ANALYZE TABLE $t COMPUTE STATISTICS").foreach(sql)

val t1 = spark.table("t1")
val t2 = spark.table("t2")

// analyzed plan is just before withCachedData and optimizedPlan plans
// where CostBasedJoinReorder kicks in and optimizes a query using statistics

val t1plan = t1.queryExecution.analyzed
scala> println(t1plan.numberedTreeString)
00 SubqueryAlias t1
01 +- Relation[id#45L] parquet

// Show the stats of every node in the analyzed query plan

val p0 = t1plan.p(0)
scala> println(s"Statistics of ${p0.simpleString}: ${p0.stats.simpleString}")
Statistics of SubqueryAlias t1: sizeInBytes=80.0 B, hints=none

val p1 = t1plan.p(1)
scala> println(s"Statistics of ${p1.simpleString}: ${p1.stats.simpleString}")
Statistics of Relation[id#45L] parquet: sizeInBytes=80.0 B, rowCount=5, hints=none

val t2plan = t2.queryExecution.analyzed

// let's get rid of the SubqueryAlias operator

import org.apache.spark.sql.catalyst.analysis.EliminateSubqueryAliases
val t1NoAliasesPlan = EliminateSubqueryAliases(t1plan)
val t2NoAliasesPlan = EliminateSubqueryAliases(t2plan)

// Using Catalyst DSL
import org.apache.spark.sql.catalyst.dsl.plans._
import org.apache.spark.sql.catalyst.plans._
val plan = t1NoAliasesPlan.join(
  otherPlan = t2NoAliasesPlan,
  joinType = Inner,
  condition = Some($"id".expr))
scala> println(plan.numberedTreeString)
00 'Join Inner, 'id
01 :- Relation[id#45L] parquet
02 +- Relation[id#57L] parquet

// Take Join operator off the logical plan
// JoinEstimation works with Joins only
import org.apache.spark.sql.catalyst.plans.logical.Join
val join = plan.collect { case j: Join => j }.head

// Make sure that row count stats are defined per join side
scala> join.left.stats.rowCount.isDefined
res1: Boolean = true

scala> join.right.stats.rowCount.isDefined
res2: Boolean = true

// Make the example reproducible
// Computing stats is once-only process and the estimates are cached
join.invalidateStatsCache

import org.apache.spark.sql.catalyst.plans.logical.statsEstimation.JoinEstimation
val stats = JoinEstimation(join).estimate
scala> :type stats
Option[org.apache.spark.sql.catalyst.plans.logical.Statistics]

// Stats have to be available so Option.get should just work
scala> println(stats.get.simpleString)
Some(sizeInBytes=1200.0 B, rowCount=50, hints=none)
----

`JoinEstimation` can <<estimate, estimate statistics and query hints of a Join logical operator>> with the following link:spark-sql-LogicalPlan-Join.adoc#joinType[join types]:

* `Inner`, `Cross`, `LeftOuter`, `RightOuter`, `FullOuter`, `LeftSemi` and `LeftAnti`

For the other join types (e.g. `ExistenceJoin`), `JoinEstimation` prints out a DEBUG message to the logs and returns `None` (to "announce" that no statistics could be computed).

[source, scala]
----
// Demo: Unsupported join type, i.e. ExistenceJoin

// Some parts were copied from the earlier demo
// FIXME Make it self-contained

// Using Catalyst DSL
// Don't even know if such existance join could ever be possible in Spark SQL
// For demo purposes it's OK, isn't it?
import org.apache.spark.sql.catalyst.plans.ExistenceJoin
val left = t1NoAliasesPlan
val right = t2NoAliasesPlan
val plan = left.join(right,
  joinType = ExistenceJoin(exists = 'id.long))

// Take Join operator off the logical plan
// JoinEstimation works with Joins only
import org.apache.spark.sql.catalyst.plans.logical.Join
val join = plan.collect { case j: Join => j }.head

// Enable DEBUG logging level
import org.apache.log4j.{Level, Logger}
Logger.getLogger("org.apache.spark.sql.catalyst.plans.logical.statsEstimation.JoinEstimation").setLevel(Level.DEBUG)

scala> val stats = JoinEstimation(join).estimate
18/06/13 10:29:37 DEBUG JoinEstimation: [CBO] Unsupported join type: ExistenceJoin(id#35L)
stats: Option[org.apache.spark.sql.catalyst.plans.logical.Statistics] = None
----

[source, scala]
----
// FIXME Describe the purpose of the demo

// Using Catalyst DSL
import org.apache.spark.sql.catalyst.dsl.plans._

val t1 = table(ref = "t1")

// HACK: Disable symbolToColumn implicit conversion
// It is imported automatically in spark-shell (and makes demos impossible)
// implicit def symbolToColumn(s: Symbol): org.apache.spark.sql.ColumnName
trait ThatWasABadIdea
implicit def symbolToColumn(ack: ThatWasABadIdea) = ack

import org.apache.spark.sql.catalyst.dsl.expressions._
val id = 'id.long

val t2 = table("t2")
import org.apache.spark.sql.catalyst.plans.LeftSemi
val plan = t1.join(t2, joinType = LeftSemi, condition = Some(id))
scala> println(plan.numberedTreeString)
00 'Join LeftSemi, id#2: bigint
01 :- 'UnresolvedRelation `t1`
02 +- 'UnresolvedRelation `t2`

import org.apache.spark.sql.catalyst.plans.logical.Join
val join = plan match { case j: Join => j }

import org.apache.spark.sql.catalyst.plans.logical.statsEstimation.JoinEstimation

// FIXME java.lang.UnsupportedOperationException
val stats = JoinEstimation(join).estimate
----

[[logging]]
[TIP]
====
Enable `DEBUG` logging level for `org.apache.spark.sql.catalyst.plans.logical.statsEstimation.JoinEstimation` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.catalyst.plans.logical.statsEstimation.JoinEstimation=DEBUG
```

Refer to link:spark-logging.adoc[Logging].
====

=== [[estimateInnerOuterJoin]] `estimateInnerOuterJoin` Internal Method

[source, scala]
----
estimateInnerOuterJoin(): Option[Statistics]
----

`estimateInnerOuterJoin` destructures <<join, Join logical operator>> into a join type with the left and right keys.

`estimateInnerOuterJoin` simply returns `None` (i.e. _nothing_) when either side of the <<join, Join logical operator>> have no link:spark-sql-EstimationUtils.adoc#rowCountsExist[row count statistic].

NOTE: `estimateInnerOuterJoin` is used exclusively when `JoinEstimation` is requested to <<estimate, estimate statistics and query hints of a Join logical operator>> for `Inner`, `Cross`, `LeftOuter`, `RightOuter` and `FullOuter` joins.

=== [[computeByNdv]] `computeByNdv` Internal Method

[source, scala]
----
computeByNdv(
  leftKey: AttributeReference,
  rightKey: AttributeReference,
  newMin: Option[Any],
  newMax: Option[Any]): (BigInt, ColumnStat)
----

`computeByNdv`...FIXME

NOTE: `computeByNdv` is used exclusively when `JoinEstimation` is requested for <<computeCardinalityAndStats, computeCardinalityAndStats>>

=== [[computeCardinalityAndStats]] `computeCardinalityAndStats` Internal Method

[source, scala]
----
computeCardinalityAndStats(
  keyPairs: Seq[(AttributeReference, AttributeReference)]): (BigInt, AttributeMap[ColumnStat])
----

`computeCardinalityAndStats`...FIXME

NOTE: `computeCardinalityAndStats` is used exclusively when `JoinEstimation` is requested for <<estimateInnerOuterJoin, estimateInnerOuterJoin>>

=== [[computeByHistogram]] Computing Join Cardinality Using Equi-Height Histograms -- `computeByHistogram` Internal Method

[source, scala]
----
computeByHistogram(
  leftKey: AttributeReference,
  rightKey: AttributeReference,
  leftHistogram: Histogram,
  rightHistogram: Histogram,
  newMin: Option[Any],
  newMax: Option[Any]): (BigInt, ColumnStat)
----

`computeByHistogram`...FIXME

NOTE: `computeByHistogram` is used exclusively when `JoinEstimation` is requested for <<computeCardinalityAndStats, computeCardinalityAndStats>> (and the histograms of both column attributes used in a join are available).

=== [[estimateLeftSemiAntiJoin]] Estimating Statistics for Left Semi and Left Anti Joins -- `estimateLeftSemiAntiJoin` Internal Method

[source, scala]
----
estimateLeftSemiAntiJoin(): Option[Statistics]
----

`estimateLeftSemiAntiJoin` estimates statistics of the <<join, Join>> logical operator only when link:spark-sql-EstimationUtils.adoc#rowCountsExist[estimated row count statistic is available]. Otherwise, `estimateLeftSemiAntiJoin` simply returns `None` (i.e. no statistics estimated).

NOTE: link:spark-sql-cost-based-optimization.adoc#rowCount[row count] statistic of a table is available only after link:spark-sql-cost-based-optimization.adoc#ANALYZE-TABLE[ANALYZE TABLE COMPUTE STATISTICS] SQL command.

If available, `estimateLeftSemiAntiJoin` takes the link:spark-sql-Statistics.adoc#rowCount[estimated row count statistic] of the link:spark-sql-LogicalPlan-Join.adoc#left[left side] of the <<join, Join>> operator.

NOTE: Use link:spark-sql-cost-based-optimization.adoc#ANALYZE-TABLE[ANALYZE TABLE COMPUTE STATISTICS] SQL command on the left logical plan to compute link:spark-sql-cost-based-optimization.adoc#rowCount[row count] statistics.

NOTE: Use link:spark-sql-cost-based-optimization.adoc#ANALYZE-TABLE[ANALYZE TABLE COMPUTE STATISTICS FOR COLUMNS] SQL command on the left logical plan to generate link:spark-sql-Statistics.adoc#attributeStats[column (equi-height) histograms] for more accurate estimations.

In the end, `estimateLeftSemiAntiJoin` creates a new link:spark-sql-Statistics.adoc#creating-instance[Statistics] with the following estimates:

. link:spark-sql-Statistics.adoc#sizeInBytes[Total size (in bytes)] is the link:spark-sql-EstimationUtils.adoc#getOutputSize[output size] for the link:spark-sql-LogicalPlan-Join.adoc#output[output schema] of the join, the row count statistic (aka _output rows_) and link:spark-sql-Statistics.adoc#attributeStats[column histograms].

. link:spark-sql-Statistics.adoc#rowCount[Row count] is exactly the row count of the left side

. link:spark-sql-Statistics.adoc#attributeStats[Column histograms] is exactly the column histograms of the left side

NOTE: `estimateLeftSemiAntiJoin` is used exclusively when `JoinEstimation` is requested to <<estimate, estimate statistics and query hints>> for `LeftSemi` and `LeftAnti` joins.

=== [[estimate]] Estimating Statistics and Query Hints of Join Logical Operator -- `estimate` Method

[source, scala]
----
estimate: Option[Statistics]
----

`estimate` estimates statistics and query hints of the <<join, Join>> logical operator per link:spark-sql-LogicalPlan-Join.adoc#joinType[join type]:

* For `Inner`, `Cross`, `LeftOuter`, `RightOuter` and `FullOuter` join types, `estimate` <<estimateInnerOuterJoin, estimateInnerOuterJoin>>

* For `LeftSemi` and `LeftAnti` join types, `estimate` <<estimateLeftSemiAntiJoin, estimateLeftSemiAntiJoin>>

For other join types, `estimate` prints out the following DEBUG message to the logs and returns `None` (to "announce" that no statistics could be computed).

```
[CBO] Unsupported join type: [joinType]
```

NOTE: `estimate` is used exclusively when `BasicStatsPlanVisitor` is requested to link:spark-sql-BasicStatsPlanVisitor.adoc#visitJoin[estimate statistics and query hints of a Join logical operator].
