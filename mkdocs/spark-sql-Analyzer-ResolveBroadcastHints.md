# ResolveBroadcastHints Logical Resolution Rule -- Resolving UnresolvedHint Operators with BROADCAST, BROADCASTJOIN and MAPJOIN Hint Names

`ResolveBroadcastHints` is a logical resolution rule that the link:spark-sql-Analyzer.adoc#ResolveBroadcastHints[Spark Analyzer] uses to <<apply, resolve UnresolvedHint logical operators>> with `BROADCAST`, `BROADCASTJOIN` or `MAPJOIN` hints (case-insensitive) to <<spark-sql-LogicalPlan-ResolvedHint.adoc#, ResolvedHint>> operators.

Technically, `ResolveBroadcastHints` is a link:spark-sql-catalyst-Rule.adoc[Catalyst rule] for transforming link:spark-sql-LogicalPlan.adoc[logical plans], i.e. `Rule[LogicalPlan]`.

`ResolveBroadcastHints` is part of link:spark-sql-Analyzer.adoc#Hints[Hints] fixed-point batch of rules (that is executed before any other rule).

[[conf]]
[[creating-instance]]
`ResolveBroadcastHints` takes a link:spark-sql-SQLConf.adoc[SQLConf] when created.

[source, scala]
----
// Use Catalyst DSL to create a logical plan
import org.apache.spark.sql.catalyst.dsl.plans._
val plan = table("t1").join(table("t2")).hint(name = "broadcast", "t1", "table2")
scala> println(plan.numberedTreeString)
00 'UnresolvedHint broadcast, [t1, table2]
01 +- 'Join Inner
02    :- 'UnresolvedRelation `t1`
03    +- 'UnresolvedRelation `t2`

import org.apache.spark.sql.catalyst.analysis.ResolveHints.ResolveBroadcastHints
val resolver = new ResolveBroadcastHints(spark.sessionState.conf)
val analyzedPlan = resolver(plan)
scala> println(analyzedPlan.numberedTreeString)
00 'Join Inner
01 :- 'ResolvedHint (broadcast)
02 :  +- 'UnresolvedRelation `t1`
03 +- 'UnresolvedRelation `t2`
----

=== [[apply]] Resolving UnresolvedHint with BROADCAST, BROADCASTJOIN or MAPJOIN Hint Names (Applying ResolveBroadcastHints to Logical Plan) -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of link:spark-sql-catalyst-Rule.adoc#apply[Rule Contract] to apply a rule to a link:spark-sql-LogicalPlan.adoc[logical plan].

`apply` transforms link:spark-sql-LogicalPlan-UnresolvedHint.adoc[UnresolvedHint] operators into link:spark-sql-LogicalPlan-ResolvedHint.adoc[ResolvedHint] for the hint names as `BROADCAST`, `BROADCASTJOIN` or `MAPJOIN` (case-insensitive).

For `UnresolvedHints` with no link:spark-sql-LogicalPlan-UnresolvedHint.adoc#parameters[parameters], `apply` marks the entire child logical plan as eligible for broadcast, i.e.  link:spark-sql-LogicalPlan-ResolvedHint.adoc#creating-instance[creates] a `ResolvedHint` with the child operator and `HintInfo` with link:spark-sql-HintInfo.adoc#broadcast[broadcast] flag on.

For `UnresolvedHints` with parameters defined, `apply` considers the parameters the names of the tables to <<applyBroadcastHint, apply broadcast hint>> to.

NOTE: The table names can be of `String` or `UnresolvedAttribute` types.

`apply` reports an `AnalysisException` for the parameters that are not of `String` or `UnresolvedAttribute` types.

```
org.apache.spark.sql.AnalysisException: Broadcast hint parameter should be an identifier or string but was [unsupported] ([className]
```

[source, scala]
----
// Use Catalyst DSL to create a logical plan
import org.apache.spark.sql.catalyst.dsl.plans._
// !!! IT WON'T WORK !!!
// 1 is not a table name or of type `UnresolvedAttribute`
val plan = table("t1").hint(name = "broadcast", 1)

scala> println(plan.numberedTreeString)
00 'UnresolvedHint broadcast, [1]
01 +- 'UnresolvedRelation `t1`

// Resolve hints
import org.apache.spark.sql.catalyst.analysis.ResolveHints
val broadcastHintResolver = new ResolveHints.ResolveBroadcastHints(spark.sessionState.conf)
scala> broadcastHintResolver(plan)
org.apache.spark.sql.AnalysisException: Broadcast hint parameter should be an identifier or string but was 1 (class java.lang.Integer;
  at org.apache.spark.sql.catalyst.analysis.ResolveHints$ResolveBroadcastHints$$anonfun$apply$1$$anonfun$applyOrElse$1.apply(ResolveHints.scala:98)
  at org.apache.spark.sql.catalyst.analysis.ResolveHints$ResolveBroadcastHints$$anonfun$apply$1$$anonfun$applyOrElse$1.apply(ResolveHints.scala:95)
  at scala.collection.TraversableLike$$anonfun$map$1.apply(TraversableLike.scala:234)
  at scala.collection.TraversableLike$$anonfun$map$1.apply(TraversableLike.scala:234)
  at scala.collection.IndexedSeqOptimized$class.foreach(IndexedSeqOptimized.scala:33)
  at scala.collection.mutable.WrappedArray.foreach(WrappedArray.scala:35)
  at scala.collection.TraversableLike$class.map(TraversableLike.scala:234)
  at scala.collection.AbstractTraversable.map(Traversable.scala:104)
  at org.apache.spark.sql.catalyst.analysis.ResolveHints$ResolveBroadcastHints$$anonfun$apply$1.applyOrElse(ResolveHints.scala:95)
  at org.apache.spark.sql.catalyst.analysis.ResolveHints$ResolveBroadcastHints$$anonfun$apply$1.applyOrElse(ResolveHints.scala:88)
  at org.apache.spark.sql.catalyst.trees.TreeNode$$anonfun$transformUp$1.apply(TreeNode.scala:289)
  at org.apache.spark.sql.catalyst.trees.TreeNode$$anonfun$transformUp$1.apply(TreeNode.scala:289)
  at org.apache.spark.sql.catalyst.trees.CurrentOrigin$.withOrigin(TreeNode.scala:70)
  at org.apache.spark.sql.catalyst.trees.TreeNode.transformUp(TreeNode.scala:288)
  at org.apache.spark.sql.catalyst.analysis.ResolveHints$ResolveBroadcastHints.apply(ResolveHints.scala:88)
  ... 51 elided
----

=== [[applyBroadcastHint]] `applyBroadcastHint` Internal Method

[source, scala]
----
applyBroadcastHint(plan: LogicalPlan, toBroadcast: Set[String]): LogicalPlan
----

`applyBroadcastHint`...FIXME

NOTE: `applyBroadcastHint` is used exclusively when `ResolveBroadcastHints` is requested to <<apply, execute>>.
