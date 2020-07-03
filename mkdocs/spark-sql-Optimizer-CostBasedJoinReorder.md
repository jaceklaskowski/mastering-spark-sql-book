# CostBasedJoinReorder Logical Optimization -- Join Reordering in Cost-Based Optimization

`CostBasedJoinReorder` is a <<spark-sql-Optimizer.adoc#batches, base logical optimization>> that <<apply, reorders joins>> in link:spark-sql-cost-based-optimization.adoc[cost-based optimization].

`ReorderJoin` is part of the <<spark-sql-Optimizer.adoc#Join_Reorder, Join Reorder>> once-executed batch in the standard batches of the <<spark-sql-Optimizer.adoc#, Catalyst Optimizer>>.

`ReorderJoin` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

`CostBasedJoinReorder` <<apply, applies>> the join optimizations on a logical plan with 2 or more <<extractInnerJoins, consecutive inner or cross joins>> (possibly separated by `Project` operators) when link:spark-sql-properties.adoc#spark.sql.cbo.enabled[spark.sql.cbo.enabled] and link:spark-sql-properties.adoc#spark.sql.cbo.joinReorder.enabled[spark.sql.cbo.joinReorder.enabled] configuration properties are both enabled.

[source, scala]
----
// Use shortcuts to read the values of the properties
scala> spark.sessionState.conf.cboEnabled
res0: Boolean = true

scala> spark.sessionState.conf.joinReorderEnabled
res1: Boolean = true
----

`CostBasedJoinReorder` uses link:spark-sql-cost-based-optimization.adoc#row-count-stat[row count] statistic that is computed using link:spark-sql-cost-based-optimization.adoc#ANALYZE-TABLE[ANALYZE TABLE COMPUTE STATISTICS] SQL command with no `NOSCAN` option.

[source, scala]
----
// Create tables and compute their row count statistics
// There have to be at least 2 joins
// Make the example reproducible
val tableNames = Seq("t1", "t2", "tiny")
import org.apache.spark.sql.catalyst.TableIdentifier
val tableIds = tableNames.map(TableIdentifier.apply)
val sessionCatalog = spark.sessionState.catalog
tableIds.foreach { tableId =>
  sessionCatalog.dropTable(tableId, ignoreIfNotExists = true, purge = true)
}

val belowBroadcastJoinThreshold = spark.sessionState.conf.autoBroadcastJoinThreshold - 1
spark.range(belowBroadcastJoinThreshold).write.saveAsTable("t1")
// t2 is twice as big as t1
spark.range(2 * belowBroadcastJoinThreshold).write.saveAsTable("t2")
spark.range(5).write.saveAsTable("tiny")

// Compute row count statistics
tableNames.foreach { t =>
  sql(s"ANALYZE TABLE $t COMPUTE STATISTICS")
}

// Load the tables
val t1 = spark.table("t1")
val t2 = spark.table("t2")
val tiny = spark.table("tiny")

// Example: Inner join with join condition
val q = t1.join(t2, Seq("id")).join(tiny, Seq("id"))
val plan = q.queryExecution.analyzed
scala> println(plan.numberedTreeString)
00 Project [id#51L]
01 +- Join Inner, (id#51L = id#57L)
02    :- Project [id#51L]
03    :  +- Join Inner, (id#51L = id#54L)
04    :     :- SubqueryAlias t1
05    :     :  +- Relation[id#51L] parquet
06    :     +- SubqueryAlias t2
07    :        +- Relation[id#54L] parquet
08    +- SubqueryAlias tiny
09       +- Relation[id#57L] parquet

// Eliminate SubqueryAlias logical operators as they no longer needed
// And "confuse" CostBasedJoinReorder
// CostBasedJoinReorder cares about how deep Joins are and reorders consecutive joins only
import org.apache.spark.sql.catalyst.analysis.EliminateSubqueryAliases
val noAliasesPlan = EliminateSubqueryAliases(plan)
scala> println(noAliasesPlan.numberedTreeString)
00 Project [id#51L]
01 +- Join Inner, (id#51L = id#57L)
02    :- Project [id#51L]
03    :  +- Join Inner, (id#51L = id#54L)
04    :     :- Relation[id#51L] parquet
05    :     +- Relation[id#54L] parquet
06    +- Relation[id#57L] parquet

// Let's go pro and create a custom RuleExecutor (i.e. an Optimizer)
import org.apache.spark.sql.catalyst.rules.RuleExecutor
import org.apache.spark.sql.catalyst.plans.logical.LogicalPlan
import org.apache.spark.sql.catalyst.analysis.EliminateSubqueryAliases
import org.apache.spark.sql.catalyst.optimizer.CostBasedJoinReorder
object Optimize extends RuleExecutor[LogicalPlan] {
  val batches =
    Batch("EliminateSubqueryAliases", Once, EliminateSubqueryAliases) ::
    Batch("Join Reorder", Once, CostBasedJoinReorder) :: Nil
}

val joinsReordered = Optimize.execute(plan)
scala> println(joinsReordered.numberedTreeString)
00 Project [id#51L]
01 +- Join Inner, (id#51L = id#54L)
02    :- Project [id#51L]
03    :  +- Join Inner, (id#51L = id#57L)
04    :     :- Relation[id#51L] parquet
05    :     +- Relation[id#57L] parquet
06    +- Relation[id#54L] parquet

// Execute the plans
// Compare the plans as diagrams in web UI @ http://localhost:4040/SQL
// We'd have to use too many internals so let's turn CBO on and off
// Moreover, please remember that the query "phases" are cached
// That's why we copy and paste the entire query for execution
import org.apache.spark.sql.internal.SQLConf
val cc = SQLConf.get
cc.setConf(SQLConf.CBO_ENABLED, false)
val q = t1.join(t2, Seq("id")).join(tiny, Seq("id"))
q.collect.foreach(_ => ())

cc.setConf(SQLConf.CBO_ENABLED, true)
val q = t1.join(t2, Seq("id")).join(tiny, Seq("id"))
q.collect.foreach(_ => ())
----

[CAUTION]
====
FIXME Examples of other join queries

* Cross join with join condition
* Project with attributes only and Inner join with join condition
* Project with attributes only and Cross join with join condition
====

[[logging]]
[TIP]
====
Enable `DEBUG` logging level for `org.apache.spark.sql.catalyst.optimizer.JoinReorderDP` logger to see the join reordering duration.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.catalyst.optimizer.JoinReorderDP=DEBUG
```

Refer to link:spark-logging.adoc[Logging].
====

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply` traverses the input link:spark-sql-LogicalPlan.adoc[logical plan] down and tries to <<reorder, reorder>> the following logical operators:

* link:spark-sql-LogicalPlan-Join.adoc[Join] for `CROSS` or `INNER` joins with a join condition

* link:spark-sql-LogicalPlan-Project.adoc[Project] with the above link:spark-sql-LogicalPlan-Join.adoc[Join] child operator and the project list of link:spark-sql-Expression-Attribute.adoc[Attribute] leaf expressions only

=== [[reorder]] Reordering Logical Plan with Join Operators -- `reorder` Internal Method

[source, scala]
----
reorder(plan: LogicalPlan, output: Seq[Attribute]): LogicalPlan
----

`reorder`...FIXME

NOTE: `reorder` is used exclusively when `CostBasedJoinReorder` is <<apply, applied>> to a logical plan.

=== [[replaceWithOrderedJoin]] `replaceWithOrderedJoin` Internal Method

[source, scala]
----
replaceWithOrderedJoin(plan: LogicalPlan): LogicalPlan
----

`replaceWithOrderedJoin`...FIXME

NOTE: `replaceWithOrderedJoin` is used recursively and when `CostBasedJoinReorder` is <<reorder, reordering>>...FIXME

=== [[extractInnerJoins]] Extracting Consecutive Join Operators -- `extractInnerJoins` Internal Method

[source, scala]
----
extractInnerJoins(plan: LogicalPlan): (Seq[LogicalPlan], Set[Expression])
----

`extractInnerJoins` finds consecutive link:spark-sql-LogicalPlan-Join.adoc[Join] logical operators (inner or cross) with join conditions or link:spark-sql-LogicalPlan-Project.adoc[Project] logical operators with `Join` logical operator and the project list of link:spark-sql-Expression-Attribute.adoc[Attribute] leaf expressions only.

For `Project` operators `extractInnerJoins` calls itself recursively with the `Join` operator inside.

In the end, `extractInnerJoins` gives the collection of logical plans under the consecutive `Join` logical operators (possibly separated by `Project` operators only) and their join conditions (for which `And` expressions have been split).

NOTE: `extractInnerJoins` is used recursively when `CostBasedJoinReorder` is <<reorder, reordering>> a logical plan.
