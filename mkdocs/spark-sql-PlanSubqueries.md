# PlanSubqueries Physical Query Optimization

`PlanSubqueries` is a *physical query optimization* (aka _physical query preparation rule_ or simply _preparation rule_) that <<apply, plans ScalarSubquery (SubqueryExpression) expressions>> (as `ScalarSubquery ExecSubqueryExpression` expressions).

[source, scala]
----
import org.apache.spark.sql.SparkSession
val spark: SparkSession = ...

import org.apache.spark.sql.execution.PlanSubqueries
val planSubqueries = PlanSubqueries(spark)

Seq(
  (0, 0),
  (1, 0),
  (2, 1)
).toDF("id", "gid").createOrReplaceTempView("t")

Seq(
  (0, 3),
  (1, 20)
).toDF("gid", "lvl").createOrReplaceTempView("v")

val sql = """
  select * from t where gid > (select max(gid) from v)
"""
val q = spark.sql(sql)

val sparkPlan = q.queryExecution.sparkPlan
scala> println(sparkPlan.numberedTreeString)
00 Project [_1#49 AS id#52, _2#50 AS gid#53]
01 +- Filter (_2#50 > scalar-subquery#128 [])
02    :  +- Aggregate [max(gid#61) AS max(gid)#130]
03    :     +- LocalRelation [gid#61]
04    +- LocalTableScan [_1#49, _2#50]

val optimizedPlan = planSubqueries(sparkPlan)
scala> println(optimizedPlan.numberedTreeString)
00 Project [_1#49 AS id#52, _2#50 AS gid#53]
01 +- Filter (_2#50 > Subquery subquery128)
02    :  +- Subquery subquery128
03    :     +- *(2) HashAggregate(keys=[], functions=[max(gid#61)], output=[max(gid)#130])
04    :        +- Exchange SinglePartition
05    :           +- *(1) HashAggregate(keys=[], functions=[partial_max(gid#61)], output=[max#134])
06    :              +- LocalTableScan [gid#61]
07    +- LocalTableScan [_1#49, _2#50]
----

`PlanSubqueries` is part of link:spark-sql-QueryExecution.adoc#preparations[preparations] batch of physical query plan rules and is executed when `QueryExecution` is requested for the link:spark-sql-QueryExecution.adoc#executedPlan[optimized physical query plan] (i.e. in *executedPlan* phase of a query execution).

Technically, `PlanSubqueries` is just a link:spark-sql-catalyst-Rule.adoc[Catalyst rule] for transforming link:spark-sql-SparkPlan.adoc[physical query plans], i.e. `Rule[SparkPlan]`.

=== [[apply]] Applying PlanSubqueries Rule to Physical Plan (Executing PlanSubqueries) -- `apply` Method

[source, scala]
----
apply(plan: SparkPlan): SparkPlan
----

NOTE: `apply` is part of link:spark-sql-catalyst-Rule.adoc#apply[Rule Contract] to apply a rule to a link:spark-sql-catalyst-TreeNode.adoc[TreeNode], e.g. link:spark-sql-SparkPlan.adoc[physical plan].

For every link:spark-sql-Expression-SubqueryExpression-ScalarSubquery.adoc[ScalarSubquery (SubqueryExpression)] expression in the input link:spark-sql-SparkPlan.adoc[physical plan], `apply` does the following:

. Builds the link:spark-sql-QueryExecution.adoc#executedPlan[optimized physical plan] (aka `executedPlan`) of the link:spark-sql-Expression-SubqueryExpression-ScalarSubquery.adoc#plan[subquery logical plan], i.e. creates a link:spark-sql-QueryExecution.adoc#creating-instance[QueryExecution] for the subquery logical plan and requests the optimized physical plan.

. Plans the scalar subquery, i.e. creates a link:spark-sql-Expression-ExecSubqueryExpression-ScalarSubquery.adoc[ScalarSubquery (ExecSubqueryExpression)] expression with a new link:spark-sql-SparkPlan-SubqueryExec.adoc#creating-instance[SubqueryExec] physical operator (with the name *subquery[id]* and the optimized physical plan) and the link:spark-sql-Expression-SubqueryExpression-ScalarSubquery.adoc#exprId[ExprId].
