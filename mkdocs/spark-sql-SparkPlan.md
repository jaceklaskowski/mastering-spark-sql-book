title: SparkPlan

# SparkPlan -- Physical Operators in Physical Query Plan of Structured Query

`SparkPlan` is the <<contract, contract>> of *physical operators* to build a *physical query plan* (aka _query execution plan_).

[[contract]]
`SparkPlan` contract requires that a concrete physical operator implements <<doExecute, doExecute>> method.

[[doExecute]]
[source, scala]
----
doExecute(): RDD[InternalRow]
----

`doExecute` allows a physical operator to describe a distributed computation (that is a runtime representation of the operator in particular and a structured query in general) as an RDD of link:spark-sql-InternalRow.adoc[internal binary rows], i.e. `RDD[InternalRow]`, and thus _execute_.

[[hooks]]
.SparkPlan's Extension Hooks
[cols="1m,2",options="header",width="100%"]
|===
| Name
| Description

| doExecuteBroadcast
a| [[doExecuteBroadcast]]

By default reports a `UnsupportedOperationException`.

```
[nodeName] does not implement doExecuteBroadcast
```

Executed exclusively as part of <<executeBroadcast, executeBroadcast>> to return the result of a structured query as a broadcast variable.

| doPrepare
a| [[doPrepare]] Prepares a physical operator for execution

Executed exclusively as part of <<prepare, prepare>> and is supposed to set some state up before executing a query (e.g. link:spark-sql-SparkPlan-BroadcastExchangeExec.adoc#doPrepare[BroadcastExchangeExec] to broadcast a relation asynchronously or link:spark-sql-SparkPlan-SubqueryExec.adoc#doPrepare[SubqueryExec] to execute a child operator)

| requiredChildDistribution
a| [[requiredChildDistribution]]

[source, scala]
----
requiredChildDistribution: Seq[Distribution]
----

The *required partition requirements* (_aka_ *child output distributions*) of the input data, i.e. how link:spark-sql-catalyst-TreeNode.adoc#children[children] physical operators' output is split across partitions.

Defaults to a link:spark-sql-Distribution-UnspecifiedDistribution.adoc[UnspecifiedDistribution] for all of the link:spark-sql-catalyst-TreeNode.adoc#children[child] operators.

Used exclusively when `EnsureRequirements` physical query plan optimization is link:spark-sql-EnsureRequirements.adoc#apply[executed] (and link:spark-sql-EnsureRequirements.adoc#ensureDistributionAndOrdering[enforces partition requirements]).

| requiredChildOrdering
a| [[requiredChildOrdering]]

[source, scala]
----
requiredChildOrdering: Seq[Seq[SortOrder]]
----

Specifies required sort ordering for each partition requirement (from link:spark-sql-catalyst-TreeNode.adoc#children[children] operators)

Defaults to no sort ordering for all of the physical operator's link:spark-sql-catalyst-TreeNode.adoc#children[children].

Used exclusively when `EnsureRequirements` physical query plan optimization is link:spark-sql-EnsureRequirements.adoc#apply[executed] (and link:spark-sql-EnsureRequirements.adoc#ensureDistributionAndOrdering[enforces partition requirements]).
|===

`SparkPlan` is a recursive data structure in Spark SQL's link:spark-sql-catalyst.adoc[Catalyst tree manipulation framework] and as such represents a single *physical operator* in a physical execution query plan as well as a *physical execution query plan* itself (i.e. a tree of physical operators in a query plan of a structured query).

.Physical Plan of Structured Query (i.e. Tree of SparkPlans)
image::images/spark-sql-SparkPlan-webui-physical-plan.png[align="center"]

NOTE: A structured query can be expressed using Spark SQL's high-level link:spark-sql-Dataset.adoc[Dataset] API for Scala, Java, Python, R or good ol' SQL.

A `SparkPlan` physical operator is a link:spark-sql-catalyst-TreeNode.adoc[Catalyst tree node] that may have zero or more link:spark-sql-catalyst-TreeNode.adoc#children[child physical operators].

NOTE: A structured query is basically a single `SparkPlan` physical operator with link:spark-sql-catalyst-TreeNode.adoc#children[child physical operators].

NOTE: Spark SQL uses link:spark-sql-catalyst.adoc[Catalyst] tree manipulation framework to compose nodes to build a tree of (logical or physical) operators that, in this particular case, is composing `SparkPlan` physical operator nodes to build the physical execution plan tree of a structured query.

[[Physical-Operator-Execution-Pipeline]]
The entry point to *Physical Operator Execution Pipeline* is <<execute, execute>>.

When <<execute, executed>>, `SparkPlan` <<executeQuery, executes the internal query implementation>> in a named scope (for visualization purposes, e.g. web UI) that triggers <<prepare, prepare>> of the children physical operators first followed by <<prepareSubqueries, prepareSubqueries>> and finally <<doPrepare, doPrepare>> methods. After <<waitForSubqueries, subqueries have finished>>, <<doExecute, doExecute>> method is eventually triggered.

.SparkPlan's Execution (execute Method)
image::images/spark-sql-SparkPlan-execute.png[align="center"]

The result of <<execute, executing>> a `SparkPlan` is an `RDD` of link:spark-sql-InternalRow.adoc[internal binary rows], i.e. `RDD[InternalRow]`.

NOTE: Executing a structured query is simply a translation of the higher-level Dataset-based description to an RDD-based runtime representation that Spark will in the end execute (once an Dataset action is used).

CAUTION: FIXME Picture between Spark SQL's Dataset => Spark Core's RDD

[[sparkContext]]
`SparkPlan` has access to the owning `SparkContext` (from the Spark Core).

[NOTE]
====
<<execute, execute>> is called when `QueryExecution` is requested for the link:spark-sql-QueryExecution.adoc#toRdd[RDD] that is Spark Core's physical execution plan (as a RDD lineage) that triggers query execution (i.e. physical planning, but not execution of the plan) and _could_ be considered execution of a structured query.

The _could_ part above refers to the fact that the final execution of a structured query happens only when a RDD action is executed on the RDD of a structured query. And hence the need for Spark SQL's high-level Dataset API in which the Dataset operators simply execute a RDD action on the corresponding RDD. _Easy, isn't it?_
====

[TIP]
====
Use link:spark-sql-dataset-operators.adoc#explain[explain] operator to see the execution plan of a structured query.

[source, scala]
----
val q = // your query here
q.explain
----

You may also access the execution plan of a `Dataset` using its link:spark-sql-Dataset.adoc#queryExecution[queryExecution] property.

[source, scala]
----
val q = // your query here
q.queryExecution.sparkPlan
----
====

The <<contract, SparkPlan contract>> assumes that concrete physical operators define <<doExecute, doExecute>> method (with optional <<hooks, hooks>> like <<doPrepare, doPrepare>>) which is executed when the physical operator is <<execute, executed>>.

.SparkPlan.execute -- Physical Operator Execution Pipeline
image::images/spark-sql-SparkPlan-execute-pipeline.png[align="center"]

`SparkPlan` has the following `final` methods that prepare execution environment and pass calls to corresponding methods (that constitute <<contract, SparkPlan Contract>>).

[[final-methods]]
.SparkPlan's Final Methods
[cols="1m,3",options="header",width="100%"]
|===
| Name
| Description

| execute
a| [[execute]]

[source, scala]
----
execute(): RDD[InternalRow]
----

"Executes" a physical operator (and its link:spark-sql-catalyst-TreeNode.adoc#children[children]) that triggers physical query planning and in the end generates an `RDD` of link:spark-sql-InternalRow.adoc[internal binary rows] (i.e. `RDD[InternalRow]`).

Used _mostly_ when `QueryExecution` is requested for the <<toRdd, RDD-based runtime representation of a structured query>> (that describes a distributed computation using Spark Core's RDD).

Internally, `execute` first <<executeQuery, prepares the physical operator for execution>> and eventually requests it to <<doExecute, doExecute>>.

NOTE: Executing `doExecute` in a named scope happens only after the operator is <<prepare, prepared for execution>> followed by <<waitForSubqueries, waiting for any subqueries to finish>>.

| <<executeQuery, executeQuery>>
a|

[source, scala]
----
executeQuery[T](query: => T): T
----

Executes a physical operator in a single RDD scope, i.e. all RDDs created during execution of the physical operator have the same scope.

`executeQuery` executes the input `query` after the following methods (in order):

1. <<prepare, prepare>>
2. <<waitForSubqueries, waitForSubqueries>>

[NOTE]
====
`executeQuery` is used when:

* `SparkPlan` is <<execute, executed>> (in which the input `query` is just <<doExecute, doExecute>>)
* `SparkPlan` is requested to <<executeBroadcast, executeBroadcast>> (in which the input `query` is just <<doExecuteBroadcast, doExecuteBroadcast>>)
* `CodegenSupport` is requested for the link:spark-sql-CodegenSupport.adoc#produce[Java source code] of a physical operator (in which the input `query` is <<doProduce, doProduce>>)
====

| prepare
a| [[prepare]]

[source, scala]
----
prepare(): Unit
----

Prepares a physical operator for execution

`prepare` is used mainly when a physical operator is requested to <<executeQuery, execute a structured query>>

`prepare` is also used recursively for every link:spark-sql-catalyst-TreeNode.adoc#children[child] physical operator (down the physical plan) and when a physical operator is requested to <<prepareSubqueries, prepare subqueries>>.

NOTE: `prepare` is idempotent, i.e. can be called multiple times with no change to the final result. It uses <<prepared, prepared>> internal flag to execute the physical operator once only.

Internally, `prepare` calls <<doPrepare, doPrepare>> of its link:spark-sql-catalyst-TreeNode.adoc#children[children] before <<prepareSubqueries, prepareSubqueries>> and <<doPrepare, doPrepare>>.

| <<executeBroadcast, executeBroadcast>>
a|

[source, scala]
----
executeBroadcast[T](): broadcast.Broadcast[T]
----

Calls <<doExecuteBroadcast, doExecuteBroadcast>>
|===

[[specialized-spark-plans]]
.Physical Query Operators / Specialized SparkPlans
[cols="1m,2",options="header",width="100%"]
|===
| Name
| Description

| BinaryExecNode
| [[BinaryExecNode]] Binary physical operator with two child `left` and `right` physical operators

| LeafExecNode
| [[LeafExecNode]] Leaf physical operator with no children

By default, the link:spark-sql-catalyst-QueryPlan.adoc#producedAttributes[set of all attributes that are produced] is exactly the link:spark-sql-catalyst-QueryPlan.adoc#outputSet[set of attributes that are output].

| UnaryExecNode
| [[UnaryExecNode]] Unary physical operator with one `child` physical operator
|===

NOTE: The naming convention for physical operators in Spark's source code is to have their names end with the *Exec* prefix, e.g. `DebugExec` or link:spark-sql-SparkPlan-LocalTableScanExec.adoc[LocalTableScanExec] that is however removed when the operator is displayed, e.g. in link:spark-sql-webui.adoc[web UI].

[[internal-registries]]
.SparkPlan's Internal Properties (e.g. Registries, Counters and Flags)
[cols="1m,2",options="header",width="100%"]
|===
| Name
| Description

| prepared
| [[prepared]] Flag that controls that <<prepare, prepare>> is executed only once.

| subexpressionEliminationEnabled
a| [[subexpressionEliminationEnabled]] Flag that controls whether the link:spark-sql-subexpression-elimination.adoc[subexpression elimination optimization] is enabled or not.

Used when the following physical operators are requested to execute (i.e. describe a distributed computation as an RDD of internal rows):

* link:spark-sql-SparkPlan-ProjectExec.adoc#doExecute[ProjectExec]

* link:spark-sql-SparkPlan-HashAggregateExec.adoc#doExecute[HashAggregateExec] (and for link:spark-sql-SparkPlan-HashAggregateExec.adoc#finishAggregate[finishAggregate])

* link:spark-sql-SparkPlan-ObjectHashAggregateExec.adoc#doExecute[ObjectHashAggregateExec]

* link:spark-sql-SparkPlan-SortAggregateExec.adoc#doExecute[SortAggregateExec]

* link:spark-sql-SparkPlan-WindowExec.adoc#doExecute[WindowExec] (and creates a link:spark-sql-SparkPlan-WindowExec.adoc#windowFrameExpressionFactoryPairs[lookup table for WindowExpressions and factory functions for WindowFunctionFrame])
|===

CAUTION: FIXME `SparkPlan` is `Serializable`. Why? Is this because `Dataset.cache` persists executed query plans?

=== [[decodeUnsafeRows]] Decoding Byte Arrays Back to UnsafeRows -- `decodeUnsafeRows` Method

CAUTION: FIXME

=== [[getByteArrayRdd]] Compressing Partitions of UnsafeRows (to Byte Arrays) After Executing Physical Operator -- `getByteArrayRdd` Internal Method

[source, scala]
----
getByteArrayRdd(n: Int = -1): RDD[Array[Byte]]
----

CAUTION: FIXME

=== [[resetMetrics]] `resetMetrics` Method

[source, scala]
----
resetMetrics(): Unit
----

`resetMetrics` takes <<metrics, metrics>> and request them to link:spark-sql-SQLMetric.adoc[reset].

NOTE: `resetMetrics` is used when...FIXME

=== [[prepareSubqueries]] `prepareSubqueries` Method

CAUTION: FIXME

=== [[executeToIterator]] `executeToIterator` Method

CAUTION: FIXME

=== [[executeCollectIterator]] `executeCollectIterator` Method

[source, scala]
----
executeCollectIterator(): (Long, Iterator[InternalRow])
----

`executeCollectIterator`...FIXME

NOTE: `executeCollectIterator` is used when...FIXME

=== [[executeQuery]] Preparing SparkPlan for Query Execution -- `executeQuery` Final Method

[source, scala]
----
executeQuery[T](query: => T): T
----

`executeQuery` executes the input `query` in a named scope (i.e. so that all RDDs created will have the same scope for visualization like web UI).

Internally, `executeQuery` calls <<prepare, prepare>> and <<waitForSubqueries, waitForSubqueries>> followed by executing `query`.

NOTE: `executeQuery` is executed as part of <<execute, execute>>, <<executeBroadcast, executeBroadcast>> and when ``CodegenSupport``-enabled physical operator link:spark-sql-CodegenSupport.adoc#produce[produces a Java source code].

=== [[executeBroadcast]] Broadcasting Result of Structured Query -- `executeBroadcast` Final Method

[source, scala]
----
executeBroadcast[T](): broadcast.Broadcast[T]
----

`executeBroadcast` returns the result of a structured query as a broadcast variable.

Internally, `executeBroadcast` calls <<doExecuteBroadcast, doExecuteBroadcast>> inside <<executeQuery, executeQuery>>.

NOTE: `executeBroadcast` is called in link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc[BroadcastHashJoinExec], link:spark-sql-SparkPlan-BroadcastNestedLoopJoinExec.adoc[BroadcastNestedLoopJoinExec] and link:spark-sql-SparkPlan-ReusedExchangeExec.adoc[ReusedExchangeExec] physical operators.

=== [[metrics]] Performance Metrics -- `metrics` Method

[source, scala]
----
metrics: Map[String, SQLMetric] = Map.empty
----

`metrics` returns the <<spark-sql-SQLMetric.adoc#, SQLMetrics>> by their names.

By default, `metrics` contains no `SQLMetrics` (i.e. `Map.empty`).

NOTE: `metrics` is used when...FIXME

=== [[executeTake]] Taking First N UnsafeRows -- `executeTake` Method

[source, scala]
----
executeTake(n: Int): Array[InternalRow]
----

`executeTake` gives an array of up to `n` first link:spark-sql-InternalRow.adoc[internal rows].

.SparkPlan's executeTake takes 5 elements
image::images/spark-sql-SparkPlan-executeTake.png[align="center"]

Internally, `executeTake` <<getByteArrayRdd, gets an RDD of byte array of `n` unsafe rows>> and scans the RDD partitions one by one until `n` is reached or all partitions were processed.

`executeTake` runs Spark jobs that take all the elements from requested number of partitions, starting from the 0th partition and increasing their number by link:spark-sql-properties.adoc#spark.sql.limit.scaleUpFactor[spark.sql.limit.scaleUpFactor] property (but minimum twice as many).

NOTE: `executeTake` uses `SparkContext.runJob` to run a Spark job.

In the end, `executeTake` <<decodeUnsafeRows, decodes the unsafe rows>>.

NOTE: `executeTake` gives an empty collection when `n` is 0 (and no Spark job is executed).

NOTE: `executeTake` may take and decode more unsafe rows than really needed since all unsafe rows from a partition are read (if the partition is included in the scan).

[source, scala]
----
import org.apache.spark.sql.internal.SQLConf.SHUFFLE_PARTITIONS
spark.sessionState.conf.setConf(SHUFFLE_PARTITIONS, 10)

// 8 groups over 10 partitions
// only 7 partitions are with numbers
val nums = spark.
  range(start = 0, end = 20, step = 1, numPartitions = 4).
  repartition($"id" % 8)

import scala.collection.Iterator
val showElements = (it: Iterator[java.lang.Long]) => {
  val ns = it.toSeq
  import org.apache.spark.TaskContext
  val pid = TaskContext.get.partitionId
  println(s"[partition: $pid][size: ${ns.size}] ${ns.mkString(" ")}")
}
// ordered by partition id manually for demo purposes
scala> nums.foreachPartition(showElements)
[partition: 0][size: 2] 4 12
[partition: 1][size: 2] 7 15
[partition: 2][size: 0]
[partition: 3][size: 0]
[partition: 4][size: 0]
[partition: 5][size: 5] 0 6 8 14 16
[partition: 6][size: 0]
[partition: 7][size: 3] 3 11 19
[partition: 8][size: 5] 2 5 10 13 18
[partition: 9][size: 3] 1 9 17

scala> println(spark.sessionState.conf.limitScaleUpFactor)
4

// Think how many Spark jobs will the following queries run?
// Answers follow
scala> nums.take(13)
res0: Array[Long] = Array(4, 12, 7, 15, 0, 6, 8, 14, 16, 3, 11, 19, 2)

// The number of Spark jobs = 3

scala> nums.take(5)
res34: Array[Long] = Array(4, 12, 7, 15, 0)

// The number of Spark jobs = 4

scala> nums.take(3)
res38: Array[Long] = Array(4, 12, 7)

// The number of Spark jobs = 2
----

[NOTE]
====
`executeTake` is used when:

* `CollectLimitExec` is requested to <<executeCollect, executeCollect>>
* `AnalyzeColumnCommand` is link:spark-sql-LogicalPlan-AnalyzeColumnCommand.adoc#run[executed]
====

=== [[executeCollect]] Executing Physical Operator and Collecting Results -- `executeCollect` Method

[source, scala]
----
executeCollect(): Array[InternalRow]
----

`executeCollect` <<getByteArrayRdd, executes the physical operator and compresses partitions of UnsafeRows as byte arrays>> (that yields a `RDD[(Long, Array[Byte])]` and so no real Spark jobs may have been submitted).

`executeCollect` runs a Spark job to `collect` the elements of the RDD and for every pair in the result (of a count and bytes per partition) <<decodeUnsafeRows, decodes the byte arrays back to UnsafeRows>> and stores the decoded arrays together as the final `Array[InternalRow]`.

NOTE: `executeCollect` runs a Spark job using Spark Core's `RDD.collect` operator.

NOTE: `executeCollect` returns `Array[InternalRow]`, i.e. keeps the internal representation of rows unchanged and does not convert rows to JVM types.

[NOTE]
====
`executeCollect` is used when:

* `Dataset` is requested for the link:spark-sql-Dataset.adoc#logicalPlan[logical plan] (being a single link:spark-sql-LogicalPlan-Command.adoc[Command] or their `Union`)

* link:spark-sql-dataset-operators.adoc#explain[explain] and link:spark-sql-dataset-operators.adoc#count[count] operators are executed

* `Dataset` is requested to `collectFromPlan`

* `SubqueryExec` is requested to link:spark-sql-SparkPlan-SubqueryExec.adoc#doPrepare[prepare for execution] (and initializes link:spark-sql-SparkPlan-SubqueryExec.adoc#relationFuture[relationFuture] for the first time)

* `SparkPlan` is requested to <<executeCollectPublic, executeCollectPublic>>

* `ScalarSubquery` and `InSubquery` plan expressions are requested to `updateResult`
====

=== [[executeCollectPublic]] `executeCollectPublic` Method

[source, scala]
----
executeCollectPublic(): Array[Row]
----

`executeCollectPublic`...FIXME

NOTE: `executeCollectPublic` is used when...FIXME

=== [[newPredicate]] `newPredicate` Method

[source, scala]
----
newPredicate(expression: Expression, inputSchema: Seq[Attribute]): GenPredicate
----

`newPredicate`...FIXME

NOTE: `newPredicate` is used when...FIXME

=== [[waitForSubqueries]] Waiting for Subqueries to Finish -- `waitForSubqueries` Method

[source, scala]
----
waitForSubqueries(): Unit
----

`waitForSubqueries` requests every link:spark-sql-Expression-ExecSubqueryExpression.adoc[ExecSubqueryExpression] in <<runningSubqueries, runningSubqueries>> to link:spark-sql-Expression-ExecSubqueryExpression.adoc#updateResult[updateResult].

NOTE: `waitForSubqueries` is used exclusively when a physical operator is requested to <<executeQuery, prepare itself for query execution>> (when it is <<execute, executed>> or requested to <<executeBroadcast, executeBroadcast>>).

=== [[outputPartitioning]] Output Data Partitioning Requirements -- `outputPartitioning` Method

[source, scala]
----
outputPartitioning: Partitioning
----

`outputPartitioning` specifies the *output data partitioning requirements*, i.e. a hint for the Spark Physical Optimizer for the number of partitions the output of the physical operator should be split across.

`outputPartitioning` defaults to a `UnknownPartitioning` (with `0` partitions).

[NOTE]
====
`outputPartitioning` is used when:

* <<spark-sql-EnsureRequirements.adoc#, EnsureRequirements>> physical query optimization is executed (and in particular <<spark-sql-EnsureRequirements.adoc#withExchangeCoordinator, adds an ExchangeCoordinator for adaptive query execution>>, <<spark-sql-EnsureRequirements.adoc#ensureDistributionAndOrdering, enforces partition requirements>> and <<spark-sql-EnsureRequirements.adoc#reorderJoinPredicates, reorderJoinPredicates>>)

* `Dataset` is requested to <<spark-sql-dataset-operators.adoc#checkpoint, checkpoint>>
====

=== [[outputOrdering]] Output Data Ordering Requirements -- `outputOrdering` Method

[source, scala]
----
outputOrdering: Seq[SortOrder]
----

`outputOrdering` specifies the *output data ordering requirements* of the physical operator, i.e. a hint for the Spark Physical Optimizer for the sorting (ordering) of the data (within and across partitions).

`outputOrdering` defaults to no ordering (`Nil`).

[NOTE]
====
`outputOrdering` is used when:

* <<spark-sql-EnsureRequirements.adoc#, EnsureRequirements>> physical query optimization is executed (and <<spark-sql-EnsureRequirements.adoc#ensureDistributionAndOrdering, enforces partition requirements>>)

* `Dataset` is requested to <<spark-sql-dataset-operators.adoc#checkpoint, checkpoint>>

* `FileFormatWriter` is requested to <<spark-sql-FileFormatWriter.adoc#write, write a query result>>
====
