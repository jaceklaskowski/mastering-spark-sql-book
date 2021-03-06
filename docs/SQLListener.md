# SQLListener Spark Listener

`SQLListener` is a `SparkListener` (Spark Core) that collects information about SQL query executions for web UI (to display in [SQL tab](SQLTab.md)). It relies on [spark.sql.execution.id](SQLExecution.md#spark.sql.execution.id) key to distinguish between queries.

Internally, it uses <<SQLExecutionUIData, SQLExecutionUIData>> data structure exclusively to record all the necessary data for a single SQL query execution. `SQLExecutionUIData` is tracked in the internal registries, i.e. `activeExecutions`, `failedExecutions`, and `completedExecutions` as well as lookup tables, i.e. `_executionIdToData`, `_jobIdToExecutionId`, and `_stageIdToStageMetrics`.

`SQLListener` starts recording a query execution by intercepting a <<SparkListenerSQLExecutionStart, SparkListenerSQLExecutionStart>> event (using <<onOtherEvent, onOtherEvent>> callback).

`SQLListener` stops recording information about a SQL query execution when <<SparkListenerSQLExecutionEnd, SparkListenerSQLExecutionEnd>> event arrives.

=== [[onJobStart]] Registering Job and Stages under Active Execution -- `onJobStart` Callback

[source, scala]
----
onJobStart(jobStart: SparkListenerJobStart): Unit
----

`onJobStart` reads the [`spark.sql.execution.id` key](SQLExecution.md#spark.sql.execution.id), the identifiers of the job and the stages and then updates the <<SQLExecutionUIData, SQLExecutionUIData>> for the execution id in `activeExecutions` internal registry.

NOTE: When `onJobStart` is executed, it is assumed that <<SQLExecutionUIData, SQLExecutionUIData>> has already been created and available in the internal `activeExecutions` registry.

The job in <<SQLExecutionUIData, SQLExecutionUIData>> is marked as running with the stages added (to `stages`). For each stage, a `SQLStageMetrics` is created in the internal `_stageIdToStageMetrics` registry. At the end, the execution id is recorded for the job id in the internal `_jobIdToExecutionId`.

=== [[onOtherEvent]] `onOtherEvent` Callback

In `onOtherEvent`, `SQLListener` listens to the following `SparkListenerEvent` events:

* <<SparkListenerSQLExecutionStart, SparkListenerSQLExecutionStart>>
* <<SparkListenerSQLExecutionEnd, SparkListenerSQLExecutionEnd>>
* <<SparkListenerDriverAccumUpdates, SparkListenerDriverAccumUpdates>>

==== [[SparkListenerSQLExecutionStart]] Registering Active Execution -- `SparkListenerSQLExecutionStart` Event

[source, scala]
----
case class SparkListenerSQLExecutionStart(
  executionId: Long,
  description: String,
  details: String,
  physicalPlanDescription: String,
  sparkPlanInfo: SparkPlanInfo,
  time: Long)
extends SparkListenerEvent
----

`SparkListenerSQLExecutionStart` events starts recording information about the `executionId` SQL query execution.

When a `SparkListenerSQLExecutionStart` event arrives, a new <<SQLExecutionUIData, SQLExecutionUIData>> for the `executionId` query execution is created and stored in `activeExecutions` internal registry. It is also stored in `_executionIdToData` lookup table.

==== [[SparkListenerSQLExecutionEnd]] `SparkListenerSQLExecutionEnd` Event

[source, scala]
----
case class SparkListenerSQLExecutionEnd(
  executionId: Long,
  time: Long)
extends SparkListenerEvent
----

`SparkListenerSQLExecutionEnd` event stops recording information about the `executionId` SQL query execution (tracked as <<SQLExecutionUIData, SQLExecutionUIData>>). `SQLListener` saves the input `time` as `completionTime`.

If there are no other running jobs (registered in <<SQLExecutionUIData, SQLExecutionUIData>>), the query execution is removed from the `activeExecutions` internal registry and moved to either `completedExecutions` or `failedExecutions` registry.

This is when `SQLListener` checks the number of `SQLExecutionUIData` entires in either registry -- `failedExecutions` or `completedExecutions` -- and removes the excess of the old entries beyond [spark.sql.ui.retainedExecutions](configuration-properties.md#spark.sql.ui.retainedExecutions) configuration property.

==== [[SparkListenerDriverAccumUpdates]] `SparkListenerDriverAccumUpdates` Event

[source, scala]
----
case class SparkListenerDriverAccumUpdates(
  executionId: Long,
  accumUpdates: Seq[(Long, Long)])
extends SparkListenerEvent
----

When `SparkListenerDriverAccumUpdates` comes, <<SQLExecutionUIData, SQLExecutionUIData>> for the input `executionId` is looked up (in `_executionIdToData`) and `SQLExecutionUIData.driverAccumUpdates` is updated with the input `accumUpdates`.

=== [[onJobEnd]] `onJobEnd` Callback

[source, scala]
----
onJobEnd(jobEnd: SparkListenerJobEnd): Unit
----

When called, `onJobEnd` retrieves the <<SQLExecutionUIData, SQLExecutionUIData>> for the job and records it either successful or failed depending on the job result.

If it is the last job of the query execution (tracked as <<SQLExecutionUIData, SQLExecutionUIData>>), the execution is removed from `activeExecutions` internal registry and moved to either

If the query execution has already been marked as completed (using `completionTime`) and there are no other running jobs (registered in <<SQLExecutionUIData, SQLExecutionUIData>>), the query execution is removed from the `activeExecutions` internal registry and moved to either `completedExecutions` or `failedExecutions` registry.

This is when `SQLListener` checks the number of `SQLExecutionUIData` entires in either registry -- `failedExecutions` or `completedExecutions` -- and removes the excess of the old entries beyond [spark.sql.ui.retainedExecutions](configuration-properties.md#spark.sql.ui.retainedExecutions) Spark property.

=== [[getExecution]] Getting SQL Execution Data -- `getExecution` Method

[source, scala]
----
getExecution(executionId: Long): Option[SQLExecutionUIData]
----

=== [[getExecutionMetrics]] Getting Execution Metrics -- `getExecutionMetrics` Method

[source, scala]
----
getExecutionMetrics(executionId: Long): Map[Long, String]
----

`getExecutionMetrics` gets the metrics (aka _accumulator updates_) for `executionId` (by which it collects all the tasks that were used for an execution).

It is exclusively used to render the [ExecutionPage](SQLTab.md#ExecutionPage) page in web UI.

=== [[mergeAccumulatorUpdates]] `mergeAccumulatorUpdates` Method

`mergeAccumulatorUpdates` is a `private` helper method for...TK

It is used exclusively in <<getExecutionMetrics, getExecutionMetrics>> method.

=== [[SQLExecutionUIData]] SQLExecutionUIData

`SQLExecutionUIData` is the data abstraction of `SQLListener` to describe SQL query executions. It is a container for jobs, stages, and accumulator updates for a single query execution.
