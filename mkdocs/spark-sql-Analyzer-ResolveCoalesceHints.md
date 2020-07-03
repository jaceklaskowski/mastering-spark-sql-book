# ResolveCoalesceHints Logical Resolution Rule -- Resolving UnresolvedHint Operators with COALESCE and REPARTITION Hints

`ResolveCoalesceHints` is a logical resolution rule that the <<spark-sql-Analyzer.adoc#ResolveCoalesceHints, Spark Analyzer>> uses to <<apply, resolve UnresolvedHint logical operators>> with `COALESCE` or `REPARTITION` hints (case-insensitive) to <<spark-sql-LogicalPlan-ResolvedHint.adoc#, ResolvedHint>> operators.

`COALESCE` or `REPARTITION` hints expect a partition number as the only parameter.

Technically, `ResolveCoalesceHints` is a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

`ResolveCoalesceHints` is part of <<spark-sql-Analyzer.adoc#Hints, Hints>> fixed-point batch of rules (that is executed before any other rule).

[[creating-instance]]
`ResolveCoalesceHints` takes no input parameters when created.

.Example: Using COALESCE Hint
[source, scala]
----
// Use Catalyst DSL to create a logical plan
import org.apache.spark.sql.catalyst.dsl.plans._
val plan = table("t1").hint(name = "COALESCE", 3)
scala> println(plan.numberedTreeString)
00 'UnresolvedHint COALESCE, [3]
01 +- 'UnresolvedRelation `t1`

import org.apache.spark.sql.catalyst.analysis.ResolveHints.ResolveCoalesceHints
val analyzedPlan = ResolveCoalesceHints(plan)
scala> println(analyzedPlan.numberedTreeString)
00 'Repartition 3, false
01 +- 'UnresolvedRelation `t1`
----

.Example: Using REPARTITION Hint
[source, scala]
----
// Use Catalyst DSL to create a logical plan
import org.apache.spark.sql.catalyst.dsl.plans._
val plan = table("t1").hint(name = "REPARTITION", 3)
scala> println(plan.numberedTreeString)
00 'UnresolvedHint REPARTITION, [3]
01 +- 'UnresolvedRelation `t1`

import org.apache.spark.sql.catalyst.analysis.ResolveHints.ResolveCoalesceHints
val analyzedPlan = ResolveCoalesceHints(plan)
scala> println(analyzedPlan.numberedTreeString)
00 'Repartition 3, true
01 +- 'UnresolvedRelation `t1`
----

.Example: Using COALESCE Hint in SQL
[source, scala]
----

val q = sql("SELECT /*+ COALESCE(10) */ * FROM VALUES 1 t(id)")
val plan = q.queryExecution.logical
scala> println(plan.numberedTreeString)
00 'UnresolvedHint COALESCE, [10]
01 +- 'Project [*]
02    +- 'SubqueryAlias `t`
03       +- 'UnresolvedInlineTable [id], [List(1)]

import org.apache.spark.sql.catalyst.analysis.ResolveHints.ResolveCoalesceHints
val analyzedPlan = ResolveCoalesceHints(plan)
scala> println(analyzedPlan.numberedTreeString)
00 'Repartition 10, false
01 +- 'Project [*]
02    +- 'SubqueryAlias `t`
03       +- 'UnresolvedInlineTable [id], [List(1)]
----
