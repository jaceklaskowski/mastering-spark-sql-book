# Hint Framework

Structured queries can be optimized using *Hint Framework* that allows for <<specifying-query-hints, specifying query hints>>.

*Query hints* allow for annotating a query and give a hint to the query optimizer how to optimize logical plans. This can be very useful when the query optimizer cannot make optimal decision, e.g. with respect to join methods due to conservativeness or the lack of proper statistics.

Spark SQL supports <<coalesce-repartition-hints, COALESCE and REPARTITION>> and <<broadcast-hints, BROADCAST>> hints. All remaining unresolved hints are silently removed from a query plan at <<spark-analyzer, analysis>>.

NOTE: Hint Framework was added in https://issues.apache.org/jira/browse/SPARK-20857[Spark SQL 2.2].

=== [[specifying-query-hints]] Specifying Query Hints

You can specify query hints using link:spark-sql-dataset-operators.adoc#hint[Dataset.hint] operator or <<sql-hints, SELECT SQL statements with hints>>.

[source, scala]
----
// Dataset API
val q = spark.range(1).hint(name = "myHint", 100, true)
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

=== [[sql-hints]] SELECT SQL Statements With Hints

`SELECT` SQL statement supports query hints as comments in SQL query that Spark SQL link:spark-sql-AstBuilder.adoc#withHints[translates] into a link:spark-sql-LogicalPlan-UnresolvedHint.adoc[UnresolvedHint] unary logical operator in a logical plan.

=== [[coalesce-repartition-hints]] COALESCE and REPARTITION Hints

Spark SQL 2.4 added support for *COALESCE* and *REPARTITION* hints (using <<sql-hints, SQL comments>>):

* `SELECT /*+ COALESCE(5) */ ...`

* `SELECT /*+ REPARTITION(3) */ ...`

=== [[broadcast-hints]] Broadcast Hints

Spark SQL 2.2 supports *BROADCAST* hints using <<broadcast-function, broadcast standard function>> or <<sql-hints, SQL comments>>:

* `SELECT /*+ MAPJOIN(b) */ ...`

* `SELECT /*+ BROADCASTJOIN(b) */ ...`

* `SELECT /*+ BROADCAST(b) */ ...`

=== [[broadcast-function]] broadcast Standard Function

While `hint` operator allows for attaching any hint to a logical plan link:spark-sql-functions.adoc#broadcast[broadcast] standard function attaches the broadcast hint only (that actually makes it a special case of `hint` operator).

`broadcast` standard function is used for link:spark-sql-joins-broadcast.adoc[broadcast joins (aka map-side joins)], i.e. to hint the Spark planner to broadcast a dataset regardless of the size.

[source, scala]
----
val small = spark.range(1)
val large = spark.range(100)

// Let's use broadcast standard function first
val q = large.join(broadcast(small), "id")
val plan = q.queryExecution.logical
scala> println(plan.numberedTreeString)
00 'Join UsingJoin(Inner,List(id))
01 :- Range (0, 100, step=1, splits=Some(8))
02 +- ResolvedHint (broadcast)
03    +- Range (0, 1, step=1, splits=Some(8))

// Please note that broadcast standard function uses ResolvedHint not UnresolvedHint

// Let's "replicate" standard function using hint operator
// Any of the names would work (case-insensitive)
// "BROADCAST", "BROADCASTJOIN", "MAPJOIN"
val smallHinted = small.hint("broadcast")
val plan = smallHinted.queryExecution.logical
scala> println(plan.numberedTreeString)
00 'UnresolvedHint broadcast
01 +- Range (0, 1, step=1, splits=Some(8))

// join is "clever"
// i.e. resolves UnresolvedHint into ResolvedHint immediately
val q = large.join(smallHinted, "id")
val plan = q.queryExecution.logical
scala> println(plan.numberedTreeString)
00 'Join UsingJoin(Inner,List(id))
01 :- Range (0, 100, step=1, splits=Some(8))
02 +- ResolvedHint (broadcast)
03    +- Range (0, 1, step=1, splits=Some(8))
----

=== [[spark-analyzer]] Spark Analyzer

There are the following logical rules that link:spark-sql-Analyzer.adoc[Spark Analyzer] uses to analyze logical plans with the link:spark-sql-LogicalPlan-UnresolvedHint.adoc[UnresolvedHint] logical operator:

. link:spark-sql-Analyzer-ResolveBroadcastHints.adoc[ResolveBroadcastHints] resolves `UnresolvedHint` operators with `BROADCAST`, `BROADCASTJOIN`, `MAPJOIN` hints to a link:spark-sql-LogicalPlan-ResolvedHint.adoc[ResolvedHint]

. <<spark-sql-Analyzer-ResolveCoalesceHints.adoc#, ResolveCoalesceHints>> resolves <<spark-sql-LogicalPlan-UnresolvedHint.adoc#, UnresolvedHint>> logical operators with `COALESCE` or `REPARTITION` hints

. `RemoveAllHints` simply removes all `UnresolvedHint` operators

The order of executing the above rules matters.

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

=== [[hint-catalyst-dsl]] Hint Operator in Catalyst DSL

You can use `hint` operator from link:spark-sql-catalyst-dsl.adoc#hint[Catalyst DSL] to create a `UnresolvedHint` logical operator, e.g. for testing or Spark SQL internals exploration.

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
