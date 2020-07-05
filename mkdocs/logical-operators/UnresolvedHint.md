title: UnresolvedHint

# UnresolvedHint Unary Logical Operator -- Attaching Hint to Logical Plan

`UnresolvedHint` is a link:spark-sql-LogicalPlan.adoc#UnaryNode[unary logical operator] that represents a hint (by <<name, name>> and <<parameters, parameters>>) for the <<child, child>> logical plan.

`UnresolvedHint` is <<creating-instance, created>> and added to a link:spark-sql-LogicalPlan.adoc[logical plan] when:

* link:spark-sql-dataset-operators.adoc#hint[Dataset.hint] operator is used

* `AstBuilder` link:spark-sql-AstBuilder.adoc#withHints[converts] `/*+ hint */` in `SELECT` SQL queries

[source, scala]
----
// Dataset API
val q = spark.range(1).hint("myHint", 100, true)
val plan = q.queryExecution.logical
scala> println(plan.numberedTreeString)
00 'UnresolvedHint myHint, [100, true]
01 +- Range (0, 1, step=1, splits=Some(8))

// SQL
val q = sql("SELECT /*+ myHint (100, true) */ 1")
val plan = q.queryExecution.logical
scala> println(plan.numberedTreeString)
00 'UnresolvedHint myHint, [100, true]
01 +- 'Project [unresolvedalias(1, None)]
02    +- OneRowRelation
----

[[creating-instance]]
When created `UnresolvedHint` takes:

* [[name]] Name of a hint
* [[parameters]] Parameters of a hint
* [[child]] Child link:spark-sql-LogicalPlan.adoc[logical plan]

[[resolved]]
`UnresolvedHint` can never be link:spark-sql-LogicalPlan.adoc#resolved[resolved] and is supposed to be converted to a link:spark-sql-LogicalPlan-ResolvedHint.adoc[ResolvedHint] unary logical operator during <<spark-sql-Analyzer.adoc#Hints, query analysis>> (or simply removed from a logical plan).

[NOTE]
====
There are the following logical rules that link:spark-sql-Analyzer.adoc[Spark Analyzer] uses to analyze logical plans with the link:spark-sql-LogicalPlan-UnresolvedHint.adoc[UnresolvedHint] logical operator:

. link:spark-sql-Analyzer-ResolveBroadcastHints.adoc[ResolveBroadcastHints] resolves `UnresolvedHint` operators with `BROADCAST`, `BROADCASTJOIN`, `MAPJOIN` hints to a link:spark-sql-LogicalPlan-ResolvedHint.adoc[ResolvedHint]

. <<spark-sql-Analyzer-ResolveCoalesceHints.adoc#, ResolveCoalesceHints>> resolves <<spark-sql-LogicalPlan-UnresolvedHint.adoc#, UnresolvedHint>> logical operators with `COALESCE` or `REPARTITION` hints

. `RemoveAllHints` simply removes all `UnresolvedHint` operators

The order of executing the above rules matters.
====

[source, scala]
----
// Let's hint the query twice
// The order of hints matters as every hint operator executes Spark analyzer
// That will resolve all but the last hint
val q = spark.range(100).
  hint("broadcast").
  hint("myHint", 100, true)
val plan = q.queryExecution.logical
scala> println(plan.numberedTreeString)
00 'UnresolvedHint myHint, [100, true]
01 +- ResolvedHint (broadcast)
02    +- Range (0, 100, step=1, splits=Some(8))

// Let's resolve unresolved hints
import org.apache.spark.sql.catalyst.rules.RuleExecutor
import org.apache.spark.sql.catalyst.plans.logical.LogicalPlan
import org.apache.spark.sql.catalyst.analysis.ResolveHints
import org.apache.spark.sql.internal.SQLConf
object HintResolver extends RuleExecutor[LogicalPlan] {
  lazy val batches =
    Batch("Hints", FixedPoint(maxIterations = 100),
      new ResolveHints.ResolveBroadcastHints(SQLConf.get),
      ResolveHints.RemoveAllHints) :: Nil
}
val resolvedPlan = HintResolver.execute(plan)
scala> println(resolvedPlan.numberedTreeString)
00 ResolvedHint (broadcast)
01 +- Range (0, 100, step=1, splits=Some(8))
----

[[output]]
`UnresolvedHint` uses the <<child, child>> operator's output schema for yours.

[TIP]
====
Use `hint` operator from link:spark-sql-catalyst-dsl.adoc#hint[Catalyst DSL] to create a `UnresolvedHint` logical operator, e.g. for testing or Spark SQL internals exploration.

[source, scala]
----
// Create a logical plan to add hint to
import org.apache.spark.sql.catalyst.plans.logical.LocalRelation
val r1 = LocalRelation('a.int, 'b.timestamp, 'c.boolean)
scala> println(r1.numberedTreeString)
00 LocalRelation <empty>, [a#0, b#1, c#2]

// Attach hint to the plan
import org.apache.spark.sql.catalyst.dsl.plans._
val plan = r1.hint(name = "myHint", 100, true)
scala> println(plan.numberedTreeString)
00 'UnresolvedHint myHint, [100, true]
01 +- LocalRelation <empty>, [a#0, b#1, c#2]
----
====
