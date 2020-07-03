# KnownSizeEstimation

`KnownSizeEstimation` is the <<contract, contract>> that allows a class to give link:spark-sql-SizeEstimator.adoc[SizeEstimator] a more accurate <<estimatedSize, size estimation>>.

[[contract]]
`KnownSizeEstimation` defines the single `estimatedSize` method.

[source, scala]
----
package org.apache.spark.util

trait KnownSizeEstimation {
  def estimatedSize: Long
}
----

`estimatedSize` is used when:

* `SizeEstimator` is requested to link:spark-sql-SizeEstimator.adoc#visitSingleObject[visitSingleObject]

* `BroadcastExchangeExec` is requested for link:spark-sql-SparkPlan-BroadcastExchangeExec.adoc#relationFuture[relationFuture]

* `BroadcastHashJoinExec` is requested to link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc#doExecute[execute]

* `ShuffledHashJoinExec` is requested to link:spark-sql-SparkPlan-ShuffledHashJoinExec.adoc#buildHashedRelation[buildHashedRelation]

NOTE: `KnownSizeEstimation` is a `private[spark]` contract.

NOTE: link:spark-sql-HashedRelation.adoc[HashedRelation] is the only `KnownSizeEstimation` available.
