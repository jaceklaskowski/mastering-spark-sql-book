title: BroadcastExchangeExec

# BroadcastExchangeExec Unary Physical Operator for Broadcast Joins

`BroadcastExchangeExec` is a link:spark-sql-SparkPlan-Exchange.adoc[Exchange] unary physical operator to collect and broadcast rows of a child relation (to worker nodes).

`BroadcastExchangeExec` is <<creating-instance, created>> exclusively when `EnsureRequirements` physical query plan optimization link:spark-sql-EnsureRequirements.adoc#ensureDistributionAndOrdering[ensures BroadcastDistribution of the input data of a physical operator] (that can really be either link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc[BroadcastHashJoinExec] or link:spark-sql-SparkPlan-BroadcastNestedLoopJoinExec.adoc[BroadcastNestedLoopJoinExec] operators).

[source, scala]
----
val t1 = spark.range(5)
val t2 = spark.range(5)
val q = t1.join(t2).where(t1("id") === t2("id"))

scala> q.explain
== Physical Plan ==
*BroadcastHashJoin [id#19L], [id#22L], Inner, BuildRight
:- *Range (0, 5, step=1, splits=Some(8))
+- BroadcastExchange HashedRelationBroadcastMode(List(input[0, bigint, false]))
   +- *Range (0, 5, step=1, splits=Some(8))
----

[[metrics]]
.BroadcastExchangeExec's Performance Metrics
[cols="1,2,2",options="header",width="100%"]
|===
| Key
| Name (in web UI)
| Description

| [[broadcastTime]] `broadcastTime`
| time to broadcast (ms)
|

| [[buildTime]] `buildTime`
| time to build (ms)
|

| [[collectTime]] `collectTime`
| time to collect (ms)
|

| [[dataSize]] `dataSize`
| data size (bytes)
|
|===

.BroadcastExchangeExec in web UI (Details for Query)
image::images/spark-sql-BroadcastExchangeExec-webui-details-for-query.png[align="center"]

[[outputPartitioning]]
`BroadcastExchangeExec` uses link:spark-sql-SparkPlan-Partitioning.adoc#BroadcastPartitioning[BroadcastPartitioning] partitioning scheme (with the input <<mode, BroadcastMode>>).

=== [[doExecuteBroadcast]] Waiting Until Relation Has Been Broadcast -- `doExecuteBroadcast` Method

[source, scala]
----
def doExecuteBroadcast[T](): broadcast.Broadcast[T]
----

`doExecuteBroadcast` waits until the <<relationFuture, rows are broadcast>>.

NOTE: `doExecuteBroadcast` waits link:spark-sql-SQLConf.adoc#broadcastTimeout[spark.sql.broadcastTimeout] (defaults to 5 minutes).

NOTE: `doExecuteBroadcast` is part of link:spark-sql-SparkPlan.adoc#doExecuteBroadcast[SparkPlan Contract] to return the result of a structured query as a broadcast variable.

=== [[relationFuture]] Lazily-Once-Initialized Asynchronously-Broadcast `relationFuture` Internal Attribute

[source, scala]
----
relationFuture: Future[broadcast.Broadcast[Any]]
----

When "materialized" (aka _executed_), `relationFuture` finds the current link:spark-sql-SQLExecution.adoc#spark.sql.execution.id[execution id] and sets it to the `Future` thread.

`relationFuture` requests <<child, child physical operator>> to link:spark-sql-SparkPlan.adoc#executeCollectIterator[executeCollectIterator].

`relationFuture` records the time for `executeCollectIterator` in <<collectTime, collectTime>> metrics.

NOTE: `relationFuture` accepts a relation with up to 512 millions rows and 8GB in size, and reports a `SparkException` if the conditions are violated.

`relationFuture` requests the input <<mode, BroadcastMode>> to `transform` the internal rows to create a relation, e.g. link:spark-sql-HashedRelation.adoc[HashedRelation] or a `Array[InternalRow]`.

`relationFuture` calculates the data size:

* For a `HashedRelation`, `relationFuture` requests it to link:spark-sql-KnownSizeEstimation.adoc#estimatedSize[estimatedSize]

* For a `Array[InternalRow]`, `relationFuture` transforms the `InternalRows` to link:spark-sql-UnsafeRow.adoc[UnsafeRows] and requests each to link:spark-sql-UnsafeRow.adoc#getSizeInBytes[getSizeInBytes] that it sums all up.

`relationFuture` records the data size as the <<dataSize, dataSize>> metric.

`relationFuture` records the <<buildTime, buildTime>> metric.

`relationFuture` requests the link:spark-sql-SparkPlan.adoc#sparkContext[SparkContext] to `broadcast` the relation and records the time in <<broadcastTime, broadcastTime>> metrics.

In the end, `relationFuture` requests `SQLMetrics` to link:spark-sql-SQLMetric.adoc#postDriverMetricUpdates[post a SparkListenerDriverAccumUpdates] (with the execution id and the SQL metrics) and returns the broadcast internal rows.

NOTE: Since initialization of `relationFuture` happens on the driver, link:spark-sql-SQLMetric.adoc#postDriverMetricUpdates[posting a SparkListenerDriverAccumUpdates] is the only way how all the SQL metrics could be accessible to other subsystems using `SparkListener` listeners (incl. web UI).

In case of `OutOfMemoryError`, `relationFuture` reports another `OutOfMemoryError` with the following message:

[options="wrap"]
----
Not enough memory to build and broadcast the table to all worker nodes. As a workaround, you can either disable broadcast by setting spark.sql.autoBroadcastJoinThreshold to -1 or increase the spark driver memory by setting spark.driver.memory to a higher value
----

[[executionContext]]
NOTE: `relationFuture` is executed on a separate thread from a custom https://www.scala-lang.org/api/2.11.8/index.html#scala.concurrent.ExecutionContext[scala.concurrent.ExecutionContext] (built from a cached https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ThreadPoolExecutor.html[java.util.concurrent.ThreadPoolExecutor] with the prefix *broadcast-exchange* and up to 128 threads).

NOTE: `relationFuture` is used when `BroadcastExchangeExec` is requested to <<doPrepare, prepare for execution>> (that triggers asynchronous execution of the child operator and broadcasting the result) and <<doExecuteBroadcast, execute broadcast>> (that waits until the broadcasting has finished).

=== [[doPrepare]] Broadcasting Relation (Rows) Asynchronously -- `doPrepare` Method

[source, scala]
----
doPrepare(): Unit
----

NOTE: `doPrepare` is part of link:spark-sql-SparkPlan.adoc#doPrepare[SparkPlan Contract] to prepare a physical operator for execution.

`doPrepare` simply "materializes" the internal lazily-once-initialized <<relationFuture, asynchronous broadcast>>.

=== [[creating-instance]] Creating BroadcastExchangeExec Instance

`BroadcastExchangeExec` takes the following when created:

* [[mode]] link:spark-sql-BroadcastMode.adoc[BroadcastMode]
* [[child]] Child link:spark-sql-LogicalPlan.adoc[logical plan]
