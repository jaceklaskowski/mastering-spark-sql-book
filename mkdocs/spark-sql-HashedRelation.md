# HashedRelation

`HashedRelation` is the <<contract, contract>> for "relations" with values hashed by some key.

`HashedRelation` is a link:spark-sql-KnownSizeEstimation.adoc[KnownSizeEstimation].

[[contract]]
[source, scala]
----
package org.apache.spark.sql.execution.joins

trait HashedRelation extends KnownSizeEstimation {
  // only required methods that have no implementation
  // the others follow
  def asReadOnlyCopy(): HashedRelation
  def close(): Unit
  def get(key: InternalRow): Iterator[InternalRow]
  def getAverageProbesPerLookup: Double
  def getValue(key: InternalRow): InternalRow
  def keyIsUnique: Boolean
}
----

NOTE: `HashedRelation` is a `private[execution]` contract.

.HashedRelation Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| [[asReadOnlyCopy]] `asReadOnlyCopy`
| Gives a read-only copy of this `HashedRelation` to be safely used in a separate thread.

Used exclusively when `BroadcastHashJoinExec` is requested to link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc#doExecute[execute] (and transform every partitions of `streamedPlan` physical operator using the broadcast variable of `buildPlan` physical operator).

| [[get]] `get`
| Gives link:spark-sql-InternalRow.adoc[internal rows] for the given key or `null`

Used when `HashJoin` is requested to <<innerJoin, innerJoin>>, <<outerJoin, outerJoin>>, <<semiJoin, semiJoin>>, <<existenceJoin, existenceJoin>> and <<antiJoin, antiJoin>>.

| [[getValue]] `getValue`
a| Gives the value link:spark-sql-InternalRow.adoc[internal row] for a given key

NOTE: `HashedRelation` has two variants of `getValue`, i.e. one that accepts an `InternalRow` and <<getValue-long, another>> a `Long`. `getValue` with an `InternalRow` does not seem to be used at all.

| [[getAverageProbesPerLookup]] `getAverageProbesPerLookup`
| Used when...FIXME
|===

=== [[getValue-long]] `getValue` Method

[source, scala]
----
getValue(key: Long): InternalRow
----

NOTE: This is `getValue` that takes a long key. There is the more generic <<getValue, getValue>> that takes an internal row instead.

`getValue` simply reports an `UnsupportedOperationException` (and expects concrete `HashedRelations` to provide a more meaningful implementation).

NOTE: `getValue` is used exclusively when `LongHashedRelation` is requested to link:spark-sql-LongHashedRelation.adoc#getValue[get the value for a given key].

=== [[apply]] Creating Concrete HashedRelation Instance (for Build Side of Hash-based Join) -- `apply` Factory Method

[source, scala]
----
apply(
  input: Iterator[InternalRow],
  key: Seq[Expression],
  sizeEstimate: Int = 64,
  taskMemoryManager: TaskMemoryManager = null): HashedRelation
----

`apply` creates a link:spark-sql-LongHashedRelation.adoc#apply[LongHashedRelation] when the input `key` collection has a single link:spark-sql-Expression.adoc[expression] of type long or link:spark-sql-UnsafeHashedRelation.adoc#apply[UnsafeHashedRelation] otherwise.

[NOTE]
====
The input `key` expressions are:

* link:spark-sql-HashJoin.adoc#buildKeys[Build join keys] of `ShuffledHashJoinExec` physical operator

* link:spark-sql-HashedRelationBroadcastMode.adoc#canonicalized[Canonicalized build-side join keys] of `HashedRelationBroadcastMode` (of link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc#requiredChildDistribution[BroadcastHashJoinExec] physical operator)
====

[NOTE]
====
`apply` is used when:

* `ShuffledHashJoinExec` is requested to link:spark-sql-SparkPlan-ShuffledHashJoinExec.adoc#buildHashedRelation[build a HashedRelation for given internal rows]

* `HashedRelationBroadcastMode` is requested to link:spark-sql-HashedRelationBroadcastMode.adoc#transform[transform]
====
