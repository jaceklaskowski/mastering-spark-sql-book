# HashJoin -- Hash-based Join Physical Operators

`HashJoin` is the <<contract, contract>> for hash-based join physical operators (e.g. link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc[BroadcastHashJoinExec] and link:spark-sql-SparkPlan-ShuffledHashJoinExec.adoc[ShuffledHashJoinExec]).

[[contract]]
[source, scala]
----
package org.apache.spark.sql.execution.joins

trait HashJoin {
  // only required methods that have no implementation
  // the others follow
  val leftKeys: Seq[Expression]
  val rightKeys: Seq[Expression]
  val joinType: JoinType
  val buildSide: BuildSide
  val condition: Option[Expression]
  val left: SparkPlan
  val right: SparkPlan
}
----

.HashJoin Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| [[buildSide]] `buildSide`
a| Left or right build side

Used when:

* `HashJoin` is requested for <<buildPlan, buildPlan>>, <<streamedPlan, streamedPlan>>, <<buildKeys, buildKeys>> and <<streamedKeys, streamedKeys>>

* `BroadcastHashJoinExec` physical operator is requested for link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc#requiredChildDistribution[requiredChildDistribution], to link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc#codegenInner[codegenInner] and link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc#codegenOuter[codegenOuter]

| [[joinType]] `joinType`
| link:spark-sql-joins.adoc[JoinType]
|===

[[internal-registries]]
.HashJoin's Internal Properties (e.g. Registries, Counters and Flags)
[cols="1,2",options="header",width="100%"]
|===
| Name
| Description

| [[boundCondition]] `boundCondition`
|

| [[buildKeys]] `buildKeys`
| Build join keys (as link:spark-sql-Expression.adoc[Catalyst expressions])

| [[buildPlan]] `buildPlan`
|

| [[streamedKeys]] `streamedKeys`
| Streamed join keys (as link:spark-sql-Expression.adoc[Catalyst expressions])

| [[streamedPlan]] `streamedPlan`
|
|===

=== [[join]] `join` Method

[source, scala]
----
join(
  streamedIter: Iterator[InternalRow],
  hashed: HashedRelation,
  numOutputRows: SQLMetric,
  avgHashProbe: SQLMetric): Iterator[InternalRow]
----

`join` branches off per <<joinType, joinType>> to create a join iterator of internal rows (i.e. `Iterator[InternalRow]`) for the input `streamedIter` and `hashed`:

* <<innerJoin, innerJoin>> for a link:spark-sql-joins.adoc#InnerLike[InnerLike] join

* <<outerJoin, outerJoin>> for a link:spark-sql-joins.adoc#LeftOuter[LeftOuter] or a link:spark-sql-joins.adoc#RightOuter[RightOuter] join

* <<semiJoin, semiJoin>> for a link:spark-sql-joins.adoc#LeftSemi[LeftSemi] join

* <<antiJoin, antiJoin>> for a link:spark-sql-joins.adoc#LeftAnti[LeftAnti] join

* <<existenceJoin, existenceJoin>> for a link:spark-sql-joins.adoc#ExistenceJoin[ExistenceJoin] join

`join` requests `TaskContext` to add a `TaskCompletionListener` to update the input avg hash probe SQL metric. The `TaskCompletionListener` is executed on a task completion (regardless of the task status: success, failure, or cancellation) and uses link:spark-sql-HashedRelation.adoc#getAverageProbesPerLookup[getAverageProbesPerLookup] from the input `hashed` to set the input avg hash probe.

`join` <<createResultProjection, createResultProjection>>.

In the end, for every row in the join iterator of internal rows `join` increments the input `numOutputRows` SQL metric and applies the result projection.

`join` reports a `IllegalArgumentException` when the <<joinType, joinType>> is incorrect.

```
[x] JoinType is not supported
```

NOTE: `join` is used when link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc#doExecute[BroadcastHashJoinExec] and link:spark-sql-SparkPlan-ShuffledHashJoinExec.adoc#doExecute[ShuffledHashJoinExec] are executed.

=== [[innerJoin]] `innerJoin` Internal Method

[source, scala]
----
innerJoin(
  streamIter: Iterator[InternalRow],
  hashedRelation: HashedRelation): Iterator[InternalRow]
----

`innerJoin`...FIXME

NOTE: `innerJoin` is used when...FIXME

=== [[outerJoin]] `outerJoin` Internal Method

[source, scala]
----
outerJoin(
  streamedIter: Iterator[InternalRow],
  hashedRelation: HashedRelation): Iterator[InternalRow]
----

`outerJoin`...FIXME

NOTE: `outerJoin` is used when...FIXME

=== [[semiJoin]] `semiJoin` Internal Method

[source, scala]
----
semiJoin(
  streamIter: Iterator[InternalRow],
  hashedRelation: HashedRelation): Iterator[InternalRow]
----

`semiJoin`...FIXME

NOTE: `semiJoin` is used when...FIXME

=== [[antiJoin]] `antiJoin` Internal Method

[source, scala]
----
antiJoin(
  streamIter: Iterator[InternalRow],
  hashedRelation: HashedRelation): Iterator[InternalRow]
----

`antiJoin`...FIXME

NOTE: `antiJoin` is used when...FIXME

=== [[existenceJoin]] `existenceJoin` Internal Method

[source, scala]
----
existenceJoin(
  streamIter: Iterator[InternalRow],
  hashedRelation: HashedRelation): Iterator[InternalRow]
----

`existenceJoin`...FIXME

NOTE: `existenceJoin` is used when...FIXME

=== [[createResultProjection]] `createResultProjection` Method

[source, scala]
----
createResultProjection(): (InternalRow) => InternalRow
----

`createResultProjection`...FIXME

NOTE: `createResultProjection` is used when...FIXME
