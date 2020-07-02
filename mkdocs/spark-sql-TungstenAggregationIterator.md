title: TungstenAggregationIterator

# TungstenAggregationIterator -- Iterator of UnsafeRows for HashAggregateExec Physical Operator

`TungstenAggregationIterator` is a <<spark-sql-AggregationIterator.adoc#, AggregationIterator>> that the `HashAggregateExec` aggregate physical operator uses when <<spark-sql-SparkPlan-HashAggregateExec.adoc#doExecute, executed>> (to process <<spark-sql-UnsafeRow.adoc#, UnsafeRows>> per partition and calculate aggregations).

`TungstenAggregationIterator` prefers hash-based aggregation (before <<switchToSortBasedAggregation, switching to sort-based aggregation>>).

[source, scala]
----
val q = spark.range(10).
  groupBy('id % 2 as "group").
  agg(sum("id") as "sum")
val execPlan = q.queryExecution.sparkPlan
scala> println(execPlan.numberedTreeString)
00 HashAggregate(keys=[(id#0L % 2)#11L], functions=[sum(id#0L)], output=[group#3L, sum#7L])
01 +- HashAggregate(keys=[(id#0L % 2) AS (id#0L % 2)#11L], functions=[partial_sum(id#0L)], output=[(id#0L % 2)#11L, sum#13L])
02    +- Range (0, 10, step=1, splits=8)

import org.apache.spark.sql.execution.aggregate.HashAggregateExec
val hashAggExec = execPlan.asInstanceOf[HashAggregateExec]
val hashAggExecRDD = hashAggExec.execute

// MapPartitionsRDD is in private[spark] scope
// Use :paste -raw for the following helper object
package org.apache.spark
object AccessPrivateSpark {
  import org.apache.spark.rdd.RDD
  def mapPartitionsRDD[T](hashAggExecRDD: RDD[T]) = {
    import org.apache.spark.rdd.MapPartitionsRDD
    hashAggExecRDD.asInstanceOf[MapPartitionsRDD[_, _]]
  }
}
// END :paste -raw

import org.apache.spark.AccessPrivateSpark
val mpRDD = AccessPrivateSpark.mapPartitionsRDD(hashAggExecRDD)
val f = mpRDD.iterator(_, _)

import org.apache.spark.sql.execution.aggregate.TungstenAggregationIterator
// FIXME How to show that TungstenAggregationIterator is used?
----

When <<creating-instance, created>>, `TungstenAggregationIterator` gets SQL metrics from the <<spark-sql-SparkPlan-HashAggregateExec.adoc#metrics, HashAggregateExec>> aggregate physical operator being executed, i.e. <<numOutputRows, numOutputRows>>, <<peakMemory, peakMemory>>, <<spillSize, spillSize>> and <<avgHashProbe, avgHashProbe>> metrics.

* <<numOutputRows, numOutputRows>> is used when `TungstenAggregationIterator` is requested for the <<next, next UnsafeRow>> (and it <<hasNext, has one>>)

* <<peakMemory, peakMemory>>, <<spillSize, spillSize>> and <<avgHashProbe, avgHashProbe>> are used at the <<TaskCompletionListener, end of every task>> (one per partition)

The metrics are then displayed as part of <<spark-sql-SparkPlan-HashAggregateExec.adoc#, HashAggregateExec>> aggregate physical operator (e.g. in web UI in <<spark-sql-webui.adoc#ExecutionPage, Details for Query>>).

.HashAggregateExec in web UI (Details for Query)
image::images/spark-sql-HashAggregateExec-webui-details-for-query.png[align="center"]

[[internal-registries]]
.TungstenAggregationIterator's Internal Properties (e.g. Registries, Counters and Flags)
[cols="1m,2",options="header",width="100%"]
|===
| Name
| Description

| aggregationBufferMapIterator
| [[aggregationBufferMapIterator]] `KVIterator[UnsafeRow, UnsafeRow]`

Used when...FIXME

| hashMap
a| [[hashMap]] <<spark-sql-UnsafeFixedWidthAggregationMap.adoc#, UnsafeFixedWidthAggregationMap>> with the following:

* <<initialAggregationBuffer, initialAggregationBuffer>>

* <<spark-sql-StructType.adoc#fromAttributes, StructType>> built from (the <<spark-sql-Expression-AggregateFunction.adoc#aggBufferAttributes, aggBufferAttributes>> of) the <<spark-sql-AggregationIterator.adoc#aggregateFunctions, aggregate function expressions>>

* <<spark-sql-StructType.adoc#fromAttributes, StructType>> built from (the <<spark-sql-Expression-NamedExpression.adoc#toAttribute, attributes>> of) the <<groupingExpressions, groupingExpressions>>

* `1024 * 16` initial capacity

* The page size of the `TaskMemoryManager` (defaults to `spark.buffer.pageSize` configuration)

Used when `TungstenAggregationIterator` is requested for the <<next, next UnsafeRow>>, to <<outputForEmptyGroupingKeyWithoutInput, outputForEmptyGroupingKeyWithoutInput>>, <<processInputs, processInputs>>, to initialize the <<aggregationBufferMapIterator, aggregationBufferMapIterator>> and <<TaskCompletionListener, every time a partition has been processed>>.

| initialAggregationBuffer
| [[initialAggregationBuffer]] <<spark-sql-UnsafeRow.adoc#, UnsafeRow>> that is the aggregation buffer containing initial buffer values.

Used when...FIXME

| externalSorter
| [[externalSorter]] `UnsafeKVExternalSorter` used for sort-based aggregation

| sortBased
| [[sortBased]] Flag to indicate whether `TungstenAggregationIterator` uses sort-based aggregation (not hash-based aggregation).

`sortBased` flag is disabled (`false`) by default.

Enabled (`true`) when `TungstenAggregationIterator` is requested to <<switchToSortBasedAggregation, switch to sort-based aggregation>>.

Used when...FIXME
|===

=== [[processInputs]] `processInputs` Internal Method

[source, scala]
----
processInputs(fallbackStartsAt: (Int, Int)): Unit
----

`processInputs`...FIXME

NOTE: `processInputs` is used exclusively when `TungstenAggregationIterator` is <<creating-instance, created>> (and sets the internal flags to indicate whether to use a hash-based aggregation or, in the worst case, a sort-based aggregation when there is not enough memory for groups and their buffers).

=== [[switchToSortBasedAggregation]] Switching to Sort-Based Aggregation (From Preferred Hash-Based Aggregation) -- `switchToSortBasedAggregation` Internal Method

[source, scala]
----
switchToSortBasedAggregation(): Unit
----

`switchToSortBasedAggregation`...FIXME

NOTE: `switchToSortBasedAggregation` is used exclusively when `TungstenAggregationIterator` is requested to <<processInputs, processInputs>> (and the <<externalSorter, externalSorter>> is used).

==== [[next]] Getting Next UnsafeRow -- `next` Method

[source, scala]
----
next(): UnsafeRow
----

NOTE: `next` is part of Scala's http://www.scala-lang.org/api/2.11.11/#scala.collection.Iterator[scala.collection.Iterator] interface that returns the next element and discards it from the iterator.

`next`...FIXME

=== [[hasNext]] `hasNext` Method

[source, scala]
----
hasNext: Boolean
----

NOTE: `hasNext` is part of Scala's http://www.scala-lang.org/api/2.11.11/#scala.collection.Iterator[scala.collection.Iterator] interface that tests whether this iterator can provide another element.

`hasNext`...FIXME

=== [[creating-instance]] Creating TungstenAggregationIterator Instance

`TungstenAggregationIterator` takes the following when created:

* [[partIndex]] Partition index
* [[groupingExpressions]] Grouping <<spark-sql-Expression-NamedExpression.adoc#, named expressions>>
* [[aggregateExpressions]] <<spark-sql-Expression-AggregateExpression.adoc#, Aggregate expressions>>
* [[aggregateAttributes]] Aggregate <<spark-sql-Expression-Attribute.adoc#, attributes>>
* [[initialInputBufferOffset]] Initial input buffer offset
* [[resultExpressions]] Output <<spark-sql-Expression-NamedExpression.adoc#, named expressions>>
* [[newMutableProjection]] Function to create a new `MutableProjection` given Catalyst expressions and attributes (i.e. `(Seq[Expression], Seq[Attribute]) => MutableProjection`)
* [[originalInputAttributes]] Output attributes (of the <<spark-sql-SparkPlan-HashAggregateExec.adoc#child, child>> of the <<spark-sql-SparkPlan-HashAggregateExec.adoc#, HashAggregateExec>> physical operator)
* [[inputIter]] Iterator of <<spark-sql-InternalRow.adoc#, InternalRows>> (from a single partition of the <<spark-sql-SparkPlan-HashAggregateExec.adoc#child, child>> of the <<spark-sql-SparkPlan-HashAggregateExec.adoc#, HashAggregateExec>> physical operator)
* [[testFallbackStartsAt]] (used for testing) Optional ``HashAggregateExec``'s link:spark-sql-SparkPlan-HashAggregateExec.adoc#testFallbackStartsAt[testFallbackStartsAt]
* [[numOutputRows]] `numOutputRows` <<spark-sql-SQLMetric.adoc#, SQLMetric>>
* [[peakMemory]] `peakMemory` <<spark-sql-SQLMetric.adoc#, SQLMetric>>
* [[spillSize]] `spillSize` <<spark-sql-SQLMetric.adoc#, SQLMetric>>
* [[avgHashProbe]] `avgHashProbe` <<spark-sql-SQLMetric.adoc#, SQLMetric>>

NOTE: The SQL metrics (<<numOutputRows, numOutputRows>>, <<peakMemory, peakMemory>>, <<spillSize, spillSize>> and <<avgHashProbe, avgHashProbe>>) belong to the <<spark-sql-SparkPlan-HashAggregateExec.adoc#metrics, HashAggregateExec>> physical operator that created the `TungstenAggregationIterator`.

`TungstenAggregationIterator` initializes the <<internal-registries, internal registries and counters>>.

`TungstenAggregationIterator` starts <<processInputs, processing input rows>> and pre-loads the first key-value pair from the <<hashMap, UnsafeFixedWidthAggregationMap>> if did not <<sortBased, switch to sort-based aggregation>>.

=== [[generateResultProjection]] `generateResultProjection` Method

[source, scala]
----
generateResultProjection(): (UnsafeRow, InternalRow) => UnsafeRow
----

NOTE: `generateResultProjection` is part of the <<spark-sql-AggregationIterator.adoc#generateResultProjection, AggregationIterator Contract>> to...FIXME.

`generateResultProjection`...FIXME

=== [[outputForEmptyGroupingKeyWithoutInput]] Creating UnsafeRow -- `outputForEmptyGroupingKeyWithoutInput` Method

[source, scala]
----
outputForEmptyGroupingKeyWithoutInput(): UnsafeRow
----

`outputForEmptyGroupingKeyWithoutInput`...FIXME

NOTE: `outputForEmptyGroupingKeyWithoutInput` is used when...FIXME

=== [[TaskCompletionListener]] TaskCompletionListener

`TungstenAggregationIterator` registers a `TaskCompletionListener` that is executed on task completion (for every task that processes a partition).

When executed (once per partition), the `TaskCompletionListener` updates the following metrics:

* <<peakMemory, peakMemory>>

* <<spillSize, spillSize>>

* <<avgHashProbe, avgHashProbe>>
