title: HashAggregateExec

# HashAggregateExec Aggregate Physical Operator for Hash-Based Aggregation

`HashAggregateExec` is a link:spark-sql-SparkPlan.adoc#UnaryExecNode[unary physical operator] (i.e. with one <<child, child>> physical operator) for **hash-based aggregation** that is <<creating-instance, created>> (indirectly through <<spark-sql-AggUtils.adoc#createAggregate, AggUtils.createAggregate>>) when:

* link:spark-sql-SparkStrategy-Aggregation.adoc[Aggregation] execution planning strategy selects the aggregate physical operator for an link:spark-sql-LogicalPlan-Aggregate.adoc[Aggregate] logical operator

* Structured Streaming's `StatefulAggregationStrategy` strategy creates plan for streaming `EventTimeWatermark` or link:spark-sql-LogicalPlan-Aggregate.adoc[Aggregate] logical operators

NOTE: `HashAggregateExec` is the link:spark-sql-SparkStrategy-Aggregation.adoc#aggregate-physical-operator-preference[preferred aggregate physical operator] for link:spark-sql-SparkStrategy-Aggregation.adoc[Aggregation] execution planning strategy (over `ObjectHashAggregateExec` and `SortAggregateExec`).

`HashAggregateExec` supports link:spark-sql-CodegenSupport.adoc[Java code generation] (aka _codegen_).

`HashAggregateExec` uses <<spark-sql-TungstenAggregationIterator.adoc#, TungstenAggregationIterator>> (to iterate over `UnsafeRows` in partitions) when <<doExecute, executed>>.

[source, scala]
----
val q = spark.range(10).
  groupBy('id % 2 as "group").
  agg(sum("id") as "sum")

// HashAggregateExec selected due to:
// 1. sum uses mutable types for aggregate expression
// 2. just a single id column reference of LongType data type
scala> q.explain
== Physical Plan ==
*HashAggregate(keys=[(id#0L % 2)#12L], functions=[sum(id#0L)])
+- Exchange hashpartitioning((id#0L % 2)#12L, 200)
   +- *HashAggregate(keys=[(id#0L % 2) AS (id#0L % 2)#12L], functions=[partial_sum(id#0L)])
      +- *Range (0, 10, step=1, splits=8)

val execPlan = q.queryExecution.sparkPlan
scala> println(execPlan.numberedTreeString)
00 HashAggregate(keys=[(id#0L % 2)#15L], functions=[sum(id#0L)], output=[group#3L, sum#7L])
01 +- HashAggregate(keys=[(id#0L % 2) AS (id#0L % 2)#15L], functions=[partial_sum(id#0L)], output=[(id#0L % 2)#15L, sum#17L])
02    +- Range (0, 10, step=1, splits=8)

// Going low level...watch your steps :)

import q.queryExecution.optimizedPlan
import org.apache.spark.sql.catalyst.plans.logical.Aggregate
val aggLog = optimizedPlan.asInstanceOf[Aggregate]
import org.apache.spark.sql.catalyst.planning.PhysicalAggregation
import org.apache.spark.sql.catalyst.expressions.aggregate.AggregateExpression
val aggregateExpressions: Seq[AggregateExpression] = PhysicalAggregation.unapply(aggLog).get._2
val aggregateBufferAttributes = aggregateExpressions.
 flatMap(_.aggregateFunction.aggBufferAttributes)
import org.apache.spark.sql.execution.aggregate.HashAggregateExec
// that's the exact reason why HashAggregateExec was selected
// Aggregation execution planning strategy prefers HashAggregateExec
scala> val useHash = HashAggregateExec.supportsAggregate(aggregateBufferAttributes)
useHash: Boolean = true

val hashAggExec = execPlan.asInstanceOf[HashAggregateExec]
scala> println(execPlan.numberedTreeString)
00 HashAggregate(keys=[(id#0L % 2)#15L], functions=[sum(id#0L)], output=[group#3L, sum#7L])
01 +- HashAggregate(keys=[(id#0L % 2) AS (id#0L % 2)#15L], functions=[partial_sum(id#0L)], output=[(id#0L % 2)#15L, sum#17L])
02    +- Range (0, 10, step=1, splits=8)

val hashAggExecRDD = hashAggExec.execute // <-- calls doExecute
scala> println(hashAggExecRDD.toDebugString)
(8) MapPartitionsRDD[3] at execute at <console>:30 []
 |  MapPartitionsRDD[2] at execute at <console>:30 []
 |  MapPartitionsRDD[1] at execute at <console>:30 []
 |  ParallelCollectionRDD[0] at execute at <console>:30 []
----

[[metrics]]
.HashAggregateExec's Performance Metrics
[cols="1,2,2",options="header",width="100%"]
|===
| Key
| Name (in web UI)
| Description

| `aggTime`
| aggregate time
| [[aggTime]]

| `avgHashProbe`
| avg hash probe
a| [[avgHashProbe]] Average hash map probe per lookup (i.e. `numProbes` / `numKeyLookups`)

NOTE: `numProbes` and `numKeyLookups` are used in link:spark-sql-BytesToBytesMap.adoc[BytesToBytesMap] append-only hash map for the number of iteration to look up a single key and the number of all the lookups in total, respectively.

| `numOutputRows`
| number of output rows
a| [[numOutputRows]] Number of groups (per partition) that (depending on the number of partitions and the side of link:spark-sql-SparkPlan-ShuffleExchangeExec.adoc[ShuffleExchangeExec] operator) is the number of groups

* `0` for no input with a grouping expression, e.g. `spark.range(0).groupBy($"id").count.show`

* `1` for no grouping expression and no input, e.g. `spark.range(0).groupBy().count.show`

TIP: Use different number of elements and partitions in `range` operator to observe the difference in `numOutputRows` metric, e.g.

[source, scala]
----
spark.
  range(0, 10, 1, numPartitions = 1).
  groupBy($"id" % 5 as "gid").
  count.
  show

spark.
  range(0, 10, 1, numPartitions = 5).
  groupBy($"id" % 5 as "gid").
  count.
  show
----

| `peakMemory`
| peak memory
| [[peakMemory]]

| `spillSize`
| spill size
| [[spillSize]]
|===

.HashAggregateExec in web UI (Details for Query)
image::images/spark-sql-HashAggregateExec-webui-details-for-query.png[align="center"]

[[properties]]
.HashAggregateExec's Properties
[width="100%",cols="1,2",options="header"]
|===
| Name
| Description

| [[output]] `output`
| link:spark-sql-catalyst-QueryPlan.adoc#output[Output schema] for the input <<resultExpressions, NamedExpressions>>
|===

[[requiredChildDistribution]]
`requiredChildDistribution` varies per the input <<requiredChildDistributionExpressions, required child distribution expressions>>.

.HashAggregateExec's Required Child Output Distributions
[cols="1,2",options="header",width="100%"]
|===
| requiredChildDistributionExpressions
| Distribution

| Defined, but empty
| link:spark-sql-Distribution-AllTuples.adoc[AllTuples]

| Non-empty
| link:spark-sql-Distribution-ClusteredDistribution.adoc[ClusteredDistribution] for `exprs`

| Undefined (`None`)
| link:spark-sql-Distribution-UnspecifiedDistribution.adoc[UnspecifiedDistribution]
|===

[NOTE]
====
`requiredChildDistributionExpressions` is exactly `requiredChildDistributionExpressions` from <<spark-sql-AggUtils.adoc#createAggregate, AggUtils.createAggregate>> and is undefined by default.

---

(No distinct in aggregation) `requiredChildDistributionExpressions` is undefined when `HashAggregateExec` is created for partial aggregations (i.e. `mode` is `Partial` for aggregate expressions).

`requiredChildDistributionExpressions` is defined, but could possibly be empty, when `HashAggregateExec` is created for final aggregations (i.e. `mode` is `Final` for aggregate expressions).

---

(one distinct in aggregation) `requiredChildDistributionExpressions` is undefined when `HashAggregateExec` is created for partial aggregations (i.e. `mode` is `Partial` for aggregate expressions) with one distinct in aggregation.

`requiredChildDistributionExpressions` is defined, but could possibly be empty, when `HashAggregateExec` is created for partial merge aggregations (i.e. `mode` is `PartialMerge` for aggregate expressions).

*FIXME* for the following two cases in aggregation with one distinct.
====

NOTE: The prefix for variable names for `HashAggregateExec` operators in link:spark-sql-CodegenSupport.adoc[CodegenSupport]-generated code is *agg*.

[[internal-registries]]
.HashAggregateExec's Internal Properties (e.g. Registries, Counters and Flags)
[cols="1m,2",options="header",width="100%"]
|===
| Name
| Description

| aggregateBufferAttributes
| [[aggregateBufferAttributes]] All the <<spark-sql-Expression-AggregateFunction.adoc#aggBufferAttributes, AttributeReferences>> of the <<spark-sql-Expression-AggregateExpression.adoc#aggregateFunction, AggregateFunctions>> of the <<aggregateExpressions, AggregateExpressions>>

| testFallbackStartsAt
| [[testFallbackStartsAt]] Optional pair of numbers for controlled fall-back to a sort-based aggregation when the hash-based approach is unable to acquire enough memory.

| declFunctions
| [[declFunctions]] <<spark-sql-Expression-DeclarativeAggregate.adoc#, DeclarativeAggregate>> expressions (from the <<spark-sql-Expression-AggregateExpression.adoc#aggregateFunction, AggregateFunctions>> of the <<aggregateExpressions, AggregateExpressions>>)

| bufferSchema
| [[bufferSchema]] <<spark-sql-StructType.adoc#fromAttributes, StructType>> built from the <<aggregateBufferAttributes, aggregateBufferAttributes>>

| groupingKeySchema
| [[groupingKeySchema]] <<spark-sql-StructType.adoc#fromAttributes, StructType>> built from the <<groupingAttributes, groupingAttributes>>

| groupingAttributes
| [[groupingAttributes]] <<spark-sql-Expression-NamedExpression.adoc#toAttribute, Attributes>> of the <<groupingExpressions, groupingExpressions>>
|===

[NOTE]
====
`HashAggregateExec` uses `TungstenAggregationIterator` that can (theoretically) link:spark-sql-TungstenAggregationIterator.adoc#switchToSortBasedAggregation[switch to a sort-based aggregation when the hash-based approach is unable to acquire enough memory].

See <<testFallbackStartsAt, testFallbackStartsAt>> internal property and link:spark-sql-properties.adoc#spark.sql.TungstenAggregate.testFallbackStartsAt[spark.sql.TungstenAggregate.testFallbackStartsAt] Spark property.

Search logs for the following INFO message to know whether the switch has happened.

```
INFO TungstenAggregationIterator: falling back to sort based aggregation.
```
====

=== [[finishAggregate]] `finishAggregate` Method

[source, scala]
----
finishAggregate(
  hashMap: UnsafeFixedWidthAggregationMap,
  sorter: UnsafeKVExternalSorter,
  peakMemory: SQLMetric,
  spillSize: SQLMetric,
  avgHashProbe: SQLMetric): KVIterator[UnsafeRow, UnsafeRow]
----

`finishAggregate`...FIXME

NOTE: `finishAggregate` is used exclusively when `HashAggregateExec` is requested to <<doProduceWithKeys, generate the Java code for doProduceWithKeys>>.

=== [[doConsumeWithKeys]] Generating Java Source Code for Whole-Stage Consume Path with Grouping Keys -- `doConsumeWithKeys` Internal Method

[source, scala]
----
doConsumeWithKeys(ctx: CodegenContext, input: Seq[ExprCode]): String
----

`doConsumeWithKeys`...FIXME

NOTE: `doConsumeWithKeys` is used exclusively when `HashAggregateExec` is requested to <<doConsume, generate the Java code for whole-stage consume path>> (with <<groupingExpressions, named expressions for the grouping keys>>).

=== [[doConsumeWithoutKeys]] Generating Java Source Code for Whole-Stage Consume Path without Grouping Keys -- `doConsumeWithoutKeys` Internal Method

[source, scala]
----
doConsumeWithoutKeys(ctx: CodegenContext, input: Seq[ExprCode]): String
----

`doConsumeWithoutKeys`...FIXME

NOTE: `doConsumeWithoutKeys` is used exclusively when `HashAggregateExec` is requested to <<doConsume, generate the Java code for whole-stage consume path>> (with no <<groupingExpressions, named expressions for the grouping keys>>).

=== [[doConsume]] Generating Java Source Code for Consume Path in Whole-Stage Code Generation -- `doConsume` Method

[source, scala]
----
doConsume(ctx: CodegenContext, input: Seq[ExprCode], row: ExprCode): String
----

NOTE: `doConsume` is part of <<spark-sql-CodegenSupport.adoc#doConsume, CodegenSupport Contract>> to generate the Java source code for <<spark-sql-whole-stage-codegen.adoc#consume-path, consume path>> in Whole-Stage Code Generation.

`doConsume` executes <<doConsumeWithoutKeys, doConsumeWithoutKeys>> when no <<groupingExpressions, named expressions for the grouping keys>> were specified for the `HashAggregateExec` or <<doConsumeWithKeys, doConsumeWithKeys>> otherwise.

=== [[doProduceWithKeys]] Generating Java Source Code For "produce" Path (In Whole-Stage Code Generation) -- `doProduceWithKeys` Internal Method

[source, scala]
----
doProduceWithKeys(ctx: CodegenContext): String
----

`doProduceWithKeys`...FIXME

NOTE: `doProduceWithKeys` is used exclusively when `HashAggregateExec` physical operator is requested to <<doProduce, generate the Java source code for "produce" path in whole-stage code generation>> (when there are no <<groupingExpressions, groupingExpressions>>).

=== [[doProduceWithoutKeys]] `doProduceWithoutKeys` Internal Method

[source, scala]
----
doProduceWithoutKeys(ctx: CodegenContext): String
----

`doProduceWithoutKeys`...FIXME

NOTE: `doProduceWithoutKeys` is used exclusively when `HashAggregateExec` physical operator is requested to <<doProduce, generate the Java source code for "produce" path in whole-stage code generation>>.

=== [[generateResultFunction]] `generateResultFunction` Internal Method

[source, scala]
----
generateResultFunction(ctx: CodegenContext): String
----

`generateResultFunction`...FIXME

NOTE: `generateResultFunction` is used exclusively when `HashAggregateExec` physical operator is requested to <<doProduceWithKeys, doProduceWithKeys>> (when `HashAggregateExec` physical operator is requested to <<doProduce, generate the Java source code for "produce" path in whole-stage code generation>>)

=== [[supportsAggregate]] `supportsAggregate` Object Method

[source, scala]
----
supportsAggregate(aggregateBufferAttributes: Seq[Attribute]): Boolean
----

`supportsAggregate` firstly <<spark-sql-StructType.adoc#fromAttributes, creates the schema>> (from the input aggregation buffer attributes) and requests `UnsafeFixedWidthAggregationMap` to <<spark-sql-UnsafeFixedWidthAggregationMap.adoc#supportsAggregationBufferSchema, supportsAggregationBufferSchema>> (i.e. the schema uses link:spark-sql-UnsafeRow.adoc#mutableFieldTypes[mutable field data types] only that have fixed length and can be mutated in place in an link:spark-sql-UnsafeRow.adoc[UnsafeRow]).

[NOTE]
====
`supportsAggregate` is used when:

* `AggUtils` is requested to <<spark-sql-AggUtils.adoc#createAggregate, creates an aggregate physical operator given aggregate expressions>>

* `HashAggregateExec` physical operator is <<creating-instance, created>> (to assert that the <<aggregateBufferAttributes, aggregateBufferAttributes>> are supported)
====

=== [[doExecute]] Executing Physical Operator (Generating RDD[InternalRow]) -- `doExecute` Method

[source, scala]
----
doExecute(): RDD[InternalRow]
----

NOTE: `doExecute` is part of <<spark-sql-SparkPlan.adoc#doExecute, SparkPlan Contract>> to generate the runtime representation of a structured query as a distributed computation over <<spark-sql-InternalRow.adoc#, internal binary rows>> on Apache Spark (i.e. `RDD[InternalRow]`).

`doExecute` requests the <<child, child>> physical operator to <<spark-sql-SparkPlan.adoc#execute, execute>> (that triggers physical query planning and generates an `RDD[InternalRow]`) and transforms it by executing the following function on internal rows per partition with index (using `RDD.mapPartitionsWithIndex` that creates another RDD):

. Records the start execution time (`beforeAgg`)

. Requests the `Iterator[InternalRow]` (from executing the <<child, child>> physical operator) for the next element

.. If there is no input (an empty partition), but there are <<groupingExpressions, grouping keys>> used, `doExecute` simply returns an empty iterator

.. Otherwise, `doExecute` creates a <<spark-sql-TungstenAggregationIterator.adoc#creating-instance, TungstenAggregationIterator>> and branches off per whether there are rows to process and the <<groupingExpressions, grouping keys>>.

For empty partitions and no <<groupingExpressions, grouping keys>>, `doExecute` increments the <<numOutputRows, numOutputRows>> metric and requests the `TungstenAggregationIterator` to <<spark-sql-TungstenAggregationIterator.adoc#outputForEmptyGroupingKeyWithoutInput, create a single UnsafeRow>> as the only element of the result iterator.

For non-empty partitions or there are <<groupingExpressions, grouping keys>> used, `doExecute` returns the `TungstenAggregationIterator`.

In the end, `doExecute` calculates the <<aggTime, aggTime>> metric and returns an `Iterator[UnsafeRow]` that can be as follows:

* Empty

* A single-element `Iterator[UnsafeRow]` with the <<spark-sql-TungstenAggregationIterator.adoc#outputForEmptyGroupingKeyWithoutInput, single UnsafeRow>>

* The <<spark-sql-TungstenAggregationIterator.adoc#, TungstenAggregationIterator>>

NOTE: The <<numOutputRows, numOutputRows>>, <<peakMemory, peakMemory>>, <<spillSize, spillSize>> and <<avgHashProbe, avgHashProbe>> metrics are used exclusively to create the <<spark-sql-TungstenAggregationIterator.adoc#, TungstenAggregationIterator>>.

[NOTE]
====
`doExecute` (by `RDD.mapPartitionsWithIndex` transformation) adds a new `MapPartitionsRDD` to the RDD lineage. Use `RDD.toDebugString` to see the additional `MapPartitionsRDD`.

[source, scala]
----
val ids = spark.range(1)
scala> println(ids.queryExecution.toRdd.toDebugString)
(8) MapPartitionsRDD[12] at toRdd at <console>:29 []
 |  MapPartitionsRDD[11] at toRdd at <console>:29 []
 |  ParallelCollectionRDD[10] at toRdd at <console>:29 []

// Use groupBy that gives HashAggregateExec operator
val q = ids.groupBy('id).count
scala> q.explain
== Physical Plan ==
*(2) HashAggregate(keys=[id#30L], functions=[count(1)])
+- Exchange hashpartitioning(id#30L, 200)
   +- *(1) HashAggregate(keys=[id#30L], functions=[partial_count(1)])
      +- *(1) Range (0, 1, step=1, splits=8)

val rdd = q.queryExecution.toRdd
scala> println(rdd.toDebugString)
(200) MapPartitionsRDD[18] at toRdd at <console>:28 []
  |   ShuffledRowRDD[17] at toRdd at <console>:28 []
  +-(8) MapPartitionsRDD[16] at toRdd at <console>:28 []
     |  MapPartitionsRDD[15] at toRdd at <console>:28 []
     |  MapPartitionsRDD[14] at toRdd at <console>:28 []
     |  ParallelCollectionRDD[13] at toRdd at <console>:28 []
----
====

=== [[doProduce]] Generating Java Source Code for Produce Path in Whole-Stage Code Generation -- `doProduce` Method

[source, scala]
----
doProduce(ctx: CodegenContext): String
----

NOTE: `doProduce` is part of <<spark-sql-CodegenSupport.adoc#doProduce, CodegenSupport Contract>> to generate the Java source code for <<spark-sql-whole-stage-codegen.adoc#produce-path, produce path>> in Whole-Stage Code Generation.

`doProduce` executes <<doProduceWithoutKeys, doProduceWithoutKeys>> when no <<groupingExpressions, named expressions for the grouping keys>> were specified for the `HashAggregateExec` or <<doProduceWithKeys, doProduceWithKeys>> otherwise.

=== [[creating-instance]] Creating HashAggregateExec Instance

`HashAggregateExec` takes the following when created:

* [[requiredChildDistributionExpressions]] Required child distribution link:spark-sql-Expression.adoc[expressions]
* [[groupingExpressions]] link:spark-sql-Expression-NamedExpression.adoc[Named expressions] for grouping keys
* [[aggregateExpressions]] link:spark-sql-Expression-AggregateExpression.adoc[AggregateExpressions]
* [[aggregateAttributes]] Aggregate link:spark-sql-Expression-Attribute.adoc[attributes]
* [[initialInputBufferOffset]] Initial input buffer offset
* [[resultExpressions]] Output link:spark-sql-Expression-NamedExpression.adoc[named expressions]
* [[child]] Child link:spark-sql-SparkPlan.adoc[physical plan]

`HashAggregateExec` initializes the <<internal-registries, internal registries and counters>>.

=== [[createHashMap]] Creating UnsafeFixedWidthAggregationMap Instance -- `createHashMap` Method

[source, scala]
----
createHashMap(): UnsafeFixedWidthAggregationMap
----

`createHashMap` creates a <<spark-sql-UnsafeFixedWidthAggregationMap.adoc#creating-instance, UnsafeFixedWidthAggregationMap>> (with the <<getEmptyAggregationBuffer, empty aggregation buffer>>, the <<bufferSchema, bufferSchema>>, the <<groupingKeySchema, groupingKeySchema>>, the current `TaskMemoryManager`, `1024 * 16` initial capacity and the page size of the `TaskMemoryManager`)

NOTE: `createHashMap` is used exclusively when `HashAggregateExec` physical operator is requested to <<doProduceWithKeys, generate the Java source code for "produce" path (in Whole-Stage Code Generation)>>.
