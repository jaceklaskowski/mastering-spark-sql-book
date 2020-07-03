# BroadcastMode

`BroadcastMode` is the <<contract, contract>> for...FIXME

[[contract]]
[source, scala]
----
package org.apache.spark.sql.catalyst.plans.physical

trait BroadcastMode {
  def canonicalized: BroadcastMode
  def transform(rows: Array[InternalRow]): Any
  def transform(rows: Iterator[InternalRow], sizeHint: Option[Long]): Any
}
----

.BroadcastMode Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| [[canonicalized]] `canonicalized`
| Used when...FIXME

| [[transform]] `transform`
a|

Used when:

* `BroadcastExchangeExec` is requested for link:spark-sql-SparkPlan-BroadcastExchangeExec.adoc#relationFuture[relationFuture] for the first time (when `BroadcastExchangeExec` is requested to link:spark-sql-SparkPlan-BroadcastExchangeExec.adoc#doPrepare[prepare for execution] as part of link:spark-sql-SparkPlan.adoc#executeQuery[executing a physical operator])

* `HashedRelationBroadcastMode` is requested to link:spark-sql-HashedRelationBroadcastMode.adoc#transform[transform] internal rows (and build a link:spark-sql-HashedRelation.adoc#apply[HashedRelation])

NOTE: The `rows`-only variant does not seem to be used at all.
|===

[[implementations]]
.BroadcastModes
[cols="1,2",options="header",width="100%"]
|===
| BroadcastMode
| Description

| [[HashedRelationBroadcastMode]] link:spark-sql-HashedRelationBroadcastMode.adoc[HashedRelationBroadcastMode]
|

| [[IdentityBroadcastMode]] link:spark-sql-IdentityBroadcastMode.adoc[IdentityBroadcastMode]
|
|===
