# JoinSelection Execution Planning Strategy

`JoinSelection` is an link:spark-sql-SparkStrategy.adoc[execution planning strategy] that link:spark-sql-SparkPlanner.adoc[SparkPlanner] uses to <<apply, plan a Join logical operator to one of the supported join physical operators>> (as described by <<join-selection-requirements, join physical operator selection requirements>>).

`JoinSelection` firstly <<apply, considers>> join physical operators per whether join keys are used or not. When join keys are used, `JoinSelection` considers <<BroadcastHashJoinExec, BroadcastHashJoinExec>>, <<ShuffledHashJoinExec, ShuffledHashJoinExec>> or <<SortMergeJoinExec, SortMergeJoinExec>> operators. Without join keys, `JoinSelection` considers <<BroadcastNestedLoopJoinExec, BroadcastNestedLoopJoinExec>> or <<CartesianProductExec, CartesianProductExec>>.

[[join-selection-requirements]]
.Join Physical Operator Selection Requirements (in the order of preference)
[cols="1,3",options="header",width="100%"]
|===
| Physical Join Operator
| Selection Requirements

| link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc[BroadcastHashJoinExec]
a| [[BroadcastHashJoinExec]] There are join keys and one of the following holds:

* Join type is link:spark-sql-joins.adoc#CROSS[CROSS], link:spark-sql-joins.adoc#INNER[INNER], link:spark-sql-joins.adoc#LEFT_ANTI[LEFT ANTI], link:spark-sql-joins.adoc#LEFT_OUTER[LEFT OUTER], link:spark-sql-joins.adoc#LEFT_SEMI[LEFT SEMI] or link:spark-sql-joins.adoc#ExistenceJoin[ExistenceJoin] (i.e. <<canBuildRight, canBuildRight>> for the input `joinType` is positive) and right join side <<canBroadcast, can be broadcast>>

* Join type is link:spark-sql-joins.adoc#CROSS[CROSS], link:spark-sql-joins.adoc#INNER[INNER] or link:spark-sql-joins.adoc#RIGHT_OUTER[RIGHT OUTER] (i.e. <<canBuildLeft, canBuildLeft>> for the input `joinType` is positive) and left join side <<canBroadcast, can be broadcast>>

| link:spark-sql-SparkPlan-ShuffledHashJoinExec.adoc[ShuffledHashJoinExec]
a| [[ShuffledHashJoinExec]] There are join keys and one of the following holds:

* link:spark-sql-properties.adoc#spark.sql.join.preferSortMergeJoin[spark.sql.join.preferSortMergeJoin] is disabled, the join type is link:spark-sql-joins.adoc#CROSS[CROSS], link:spark-sql-joins.adoc#INNER[INNER], link:spark-sql-joins.adoc#LEFT_ANTI[LEFT ANTI], link:spark-sql-joins.adoc#LEFT_OUTER[LEFT OUTER], link:spark-sql-joins.adoc#LEFT_SEMI[LEFT SEMI] or link:spark-sql-joins.adoc#ExistenceJoin[ExistenceJoin] (i.e. <<canBuildRight, canBuildRight>> for the input `joinType` is positive), <<canBuildLocalHashMap, canBuildLocalHashMap>> for right join side and finally right join side is <<muchSmaller, much smaller>> than left side

* link:spark-sql-properties.adoc#spark.sql.join.preferSortMergeJoin[spark.sql.join.preferSortMergeJoin] is disabled, the join type is link:spark-sql-joins.adoc#CROSS[CROSS], link:spark-sql-joins.adoc#INNER[INNER] or link:spark-sql-joins.adoc#RIGHT_OUTER[RIGHT OUTER] (i.e. <<canBuildLeft, canBuildLeft>> for the input `joinType` is positive), <<canBuildLocalHashMap, canBuildLocalHashMap>> for left join side and finally left join side is <<muchSmaller, much smaller>> than right

* Left join keys are *not* link:spark-sql-SparkPlan-SortMergeJoinExec.adoc#orderable[orderable]

| link:spark-sql-SparkPlan-SortMergeJoinExec.adoc[SortMergeJoinExec]
| [[SortMergeJoinExec]] Left join keys are link:spark-sql-SparkPlan-SortMergeJoinExec.adoc#orderable[orderable]

| link:spark-sql-SparkPlan-BroadcastNestedLoopJoinExec.adoc[BroadcastNestedLoopJoinExec]
a| [[BroadcastNestedLoopJoinExec]] There are no join keys and one of the following holds:

* Join type is link:spark-sql-joins.adoc#CROSS[CROSS], link:spark-sql-joins.adoc#INNER[INNER], link:spark-sql-joins.adoc#LEFT_ANTI[LEFT ANTI], link:spark-sql-joins.adoc#LEFT_OUTER[LEFT OUTER], link:spark-sql-joins.adoc#LEFT_SEMI[LEFT SEMI] or link:spark-sql-joins.adoc#ExistenceJoin[ExistenceJoin] (i.e. <<canBuildRight, canBuildRight>> for the input `joinType` is positive) and right join side <<canBroadcast, can be broadcast>>

* Join type is link:spark-sql-joins.adoc#CROSS[CROSS], link:spark-sql-joins.adoc#INNER[INNER] or link:spark-sql-joins.adoc#RIGHT_OUTER[RIGHT OUTER] (i.e. <<canBuildLeft, canBuildLeft>> for the input `joinType` is positive) and left join side <<canBroadcast, can be broadcast>>

| link:spark-sql-SparkPlan-CartesianProductExec.adoc[CartesianProductExec]
| [[CartesianProductExec]] There are no join keys and link:spark-sql-joins.adoc#join-types[join type] is link:spark-sql-joins.adoc#CROSS[CROSS] or link:spark-sql-joins.adoc#INNER[INNER]

| link:spark-sql-SparkPlan-BroadcastNestedLoopJoinExec.adoc[BroadcastNestedLoopJoinExec]
| No other join operator has matched already
|===

NOTE: `JoinSelection` uses link:spark-sql-ExtractEquiJoinKeys.adoc[ExtractEquiJoinKeys] Scala extractor to destructure a `Join` logical operator.

=== [[muchSmaller]] Is Left-Side Plan At Least 3 Times Smaller Than Right-Side Plan? -- `muchSmaller` Internal Condition

[source, scala]
----
muchSmaller(a: LogicalPlan, b: LogicalPlan): Boolean
----

`muchSmaller` condition holds when plan `a` is at least 3 times smaller than plan `b`.

Internally, `muchSmaller` link:spark-sql-LogicalPlan.adoc#stats[calculates the estimated statistics for the input logical plans] and compares their physical size in bytes (`sizeInBytes`).

NOTE: `muchSmaller` is used when `JoinSelection` checks <<join-selection-requirements, join selection requirements>> for `ShuffledHashJoinExec` physical operator.

=== [[canBuildLocalHashMap]] `canBuildLocalHashMap` Internal Condition

[source, scala]
----
canBuildLocalHashMap(plan: LogicalPlan): Boolean
----

`canBuildLocalHashMap` condition holds for the logical `plan` whose single partition is small enough to build a hash table (i.e. link:spark-sql-properties.adoc#spark.sql.autoBroadcastJoinThreshold[spark.sql.autoBroadcastJoinThreshold] multiplied by link:spark-sql-properties.adoc#spark.sql.shuffle.partitions[spark.sql.shuffle.partitions]).

Internally, `canBuildLocalHashMap` link:spark-sql-LogicalPlan.adoc#stats[calculates the estimated statistics for the input logical plans] and takes the size in bytes (`sizeInBytes`).

NOTE: `canBuildLocalHashMap` is used when `JoinSelection` checks <<join-selection-requirements, join selection requirements>> for `ShuffledHashJoinExec` physical operator.

=== [[canBroadcast]] Can Logical Plan Be Broadcast? -- `canBroadcast` Internal Condition

[source, scala]
----
canBroadcast(plan: LogicalPlan): Boolean
----

`canBroadcast` is enabled, i.e. `true`, when the size of the output of the input logical plan (aka _sizeInBytes_) is less than link:spark-sql-properties.adoc#spark.sql.autoBroadcastJoinThreshold[spark.sql.autoBroadcastJoinThreshold] configuration property.

NOTE: link:spark-sql-properties.adoc#spark.sql.autoBroadcastJoinThreshold[spark.sql.autoBroadcastJoinThreshold] is 10M by default.

NOTE: `canBroadcast` uses the total size statistic from link:spark-sql-LogicalPlanStats.adoc#stats[Statistics] of a logical operator.

NOTE: `canBroadcast` is used when `JoinSelection` is requested to <<canBroadcastBySizes, canBroadcastBySizes>> and <<broadcastSideBySizes, selects the build side per join type and total size statistic of join sides>>.

=== [[canBroadcastByHints]] `canBroadcastByHints` Internal Method

[source, scala]
----
canBroadcastByHints(joinType: JoinType, left: LogicalPlan, right: LogicalPlan): Boolean
----

`canBroadcastByHints` is positive (i.e. `true`) when either condition holds:

. Join type is link:spark-sql-joins.adoc#CROSS[CROSS], link:spark-sql-joins.adoc#INNER[INNER] or link:spark-sql-joins.adoc#RIGHT_OUTER[RIGHT OUTER] (i.e. <<canBuildLeft, canBuildLeft>> for the input `joinType` is positive) and `left` operator's link:spark-sql-HintInfo.adoc#broadcast[broadcast] hint flag is on

. Join type is link:spark-sql-joins.adoc#CROSS[CROSS], link:spark-sql-joins.adoc#INNER[INNER], link:spark-sql-joins.adoc#LEFT_ANTI[LEFT ANTI], link:spark-sql-joins.adoc#LEFT_OUTER[LEFT OUTER], link:spark-sql-joins.adoc#LEFT_SEMI[LEFT SEMI] or link:spark-sql-joins.adoc#ExistenceJoin[ExistenceJoin] (i.e. <<canBuildRight, canBuildRight>> for the input `joinType` is positive) and `right` operator's link:spark-sql-HintInfo.adoc#broadcast[broadcast] hint flag is on

Otherwise, `canBroadcastByHints` is negative (i.e. `false`).

NOTE: `canBroadcastByHints` is used when `JoinSelection` is requested to <<apply, plan a Join logical operator>> (and considers a link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc[BroadcastHashJoinExec] or a link:spark-sql-SparkPlan-BroadcastNestedLoopJoinExec.adoc[BroadcastNestedLoopJoinExec] physical operator).

=== [[broadcastSideByHints]] Selecting Build Side Per Join Type and Broadcast Hints -- `broadcastSideByHints` Internal Method

[source, scala]
----
broadcastSideByHints(joinType: JoinType, left: LogicalPlan, right: LogicalPlan): BuildSide
----

`broadcastSideByHints` computes `buildLeft` and `buildRight` flags:

* `buildLeft` flag is positive (i.e. `true`) when the join type is link:spark-sql-joins.adoc#CROSS[CROSS], link:spark-sql-joins.adoc#INNER[INNER] or link:spark-sql-joins.adoc#RIGHT_OUTER[RIGHT OUTER] (i.e. <<canBuildLeft, canBuildLeft>> for the input `joinType` is positive) and the `left` operator's link:spark-sql-HintInfo.adoc#broadcast[broadcast] hint flag is positive

* `buildRight` flag is positive (i.e. `true`) when the join type is link:spark-sql-joins.adoc#CROSS[CROSS], link:spark-sql-joins.adoc#INNER[INNER], link:spark-sql-joins.adoc#LEFT_ANTI[LEFT ANTI], link:spark-sql-joins.adoc#LEFT_OUTER[LEFT OUTER], link:spark-sql-joins.adoc#LEFT_SEMI[LEFT SEMI] or link:spark-sql-joins.adoc#ExistenceJoin[ExistenceJoin] (i.e. <<canBuildRight, canBuildRight>> for the input `joinType` is positive) and the `right` operator's link:spark-sql-HintInfo.adoc#broadcast[broadcast] hint flag is positive

In the end, `broadcastSideByHints` <<broadcastSide, gives the join side to broadcast>>.

NOTE: `broadcastSideByHints` is used when `JoinSelection` is requested to <<apply, plan a Join logical operator>> (and considers a link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc[BroadcastHashJoinExec] or a link:spark-sql-SparkPlan-BroadcastNestedLoopJoinExec.adoc[BroadcastNestedLoopJoinExec] physical operator).

=== [[broadcastSide]] Choosing Join Side to Broadcast -- `broadcastSide` Internal Method

[source, scala]
----
broadcastSide(
  canBuildLeft: Boolean,
  canBuildRight: Boolean,
  left: LogicalPlan,
  right: LogicalPlan): BuildSide
----

`broadcastSide` gives the smaller side (`BuildRight` or `BuildLeft`) per link:spark-sql-Statistics.adoc#sizeInBytes[total size] when `canBuildLeft` and `canBuildRight` are both positive (i.e. `true`).

`broadcastSide` gives `BuildRight` when `canBuildRight` is positive.

`broadcastSide` gives `BuildLeft` when `canBuildLeft` is positive.

When all the above conditions are not met, `broadcastSide` gives the smaller side (`BuildRight` or `BuildLeft`) per link:spark-sql-Statistics.adoc#sizeInBytes[total size] (similarly to the first case when `canBuildLeft` and `canBuildRight` are both positive).

NOTE: `broadcastSide` is used when `JoinSelection` is requested to <<broadcastSideByHints, broadcastSideByHints>>, <<broadcastSideBySizes, select the build side per join type and total size statistic of join sides>>, and <<apply, execute>> (and considers a link:spark-sql-SparkPlan-BroadcastNestedLoopJoinExec.adoc[BroadcastNestedLoopJoinExec] physical operator).

=== [[canBuildLeft]] Checking If Join Type Allows For Left Join Side As Build Side -- `canBuildLeft` Internal Condition

[source, scala]
----
canBuildLeft(joinType: JoinType): Boolean
----

`canBuildLeft` is positive (i.e. `true`) for link:spark-sql-joins.adoc#CROSS[CROSS], link:spark-sql-joins.adoc#INNER[INNER] and link:spark-sql-joins.adoc#RIGHT_OUTER[RIGHT OUTER] join types. Otherwise, `canBuildLeft` is negative (i.e. `false`).

NOTE: `canBuildLeft` is used when `JoinSelection` is requested to <<canBroadcastByHints, canBroadcastByHints>>, <<broadcastSideByHints, broadcastSideByHints>>, <<canBroadcastBySizes, canBroadcastBySizes>>, <<broadcastSideBySizes, broadcastSideBySizes>> and <<apply, execute>> (when selecting a <<ShuffledHashJoinExec>> physical operator).

=== [[canBuildRight]] Checking If Join Type Allows For Right Join Side As Build Side -- `canBuildRight` Internal Condition

[source, scala]
----
canBuildRight(joinType: JoinType): Boolean
----

`canBuildRight` is positive (i.e. `true`) if the input join type is one of the following:

* link:spark-sql-joins.adoc#CROSS[CROSS], link:spark-sql-joins.adoc#INNER[INNER], link:spark-sql-joins.adoc#LEFT_ANTI[LEFT ANTI], link:spark-sql-joins.adoc#LEFT_OUTER[LEFT OUTER], link:spark-sql-joins.adoc#LEFT_SEMI[LEFT SEMI] or link:spark-sql-joins.adoc#ExistenceJoin[ExistenceJoin]

Otherwise, `canBuildRight` is negative (i.e. `false`).

NOTE: `canBuildRight` is used when `JoinSelection` is requested to <<canBroadcastByHints, canBroadcastByHints>>, <<broadcastSideByHints, broadcastSideByHints>>, <<canBroadcastBySizes, canBroadcastBySizes>>, <<broadcastSideBySizes, broadcastSideBySizes>> and <<apply, execute>> (when selecting a <<ShuffledHashJoinExec>> physical operator).

=== [[canBroadcastBySizes]] Checking If Join Type and Total Size Statistic of Join Sides Allow for Broadcast Join -- `canBroadcastBySizes` Internal Method

[source, scala]
----
canBroadcastBySizes(joinType: JoinType, left: LogicalPlan, right: LogicalPlan): Boolean
----

`canBroadcastBySizes` is positive (i.e. `true`) when either condition holds:

. Join type is link:spark-sql-joins.adoc#CROSS[CROSS], link:spark-sql-joins.adoc#INNER[INNER] or link:spark-sql-joins.adoc#RIGHT_OUTER[RIGHT OUTER] (i.e. <<canBuildLeft, canBuildLeft>> for the input `joinType` is positive) and `left` operator <<canBroadcast, can be broadcast per total size statistic>>

. Join type is link:spark-sql-joins.adoc#CROSS[CROSS], link:spark-sql-joins.adoc#INNER[INNER], link:spark-sql-joins.adoc#LEFT_ANTI[LEFT ANTI], link:spark-sql-joins.adoc#LEFT_OUTER[LEFT OUTER], link:spark-sql-joins.adoc#LEFT_SEMI[LEFT SEMI] or link:spark-sql-joins.adoc#ExistenceJoin[ExistenceJoin] (i.e. <<canBuildRight, canBuildRight>> for the input `joinType` is positive) and `right` operator <<canBroadcast, can be broadcast per total size statistic>>

Otherwise, `canBroadcastByHints` is negative (i.e. `false`).

NOTE: `canBroadcastByHints` is used when `JoinSelection` is requested to <<apply, plan a Join logical operator>> (and considers a link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc[BroadcastHashJoinExec] or a link:spark-sql-SparkPlan-BroadcastNestedLoopJoinExec.adoc[BroadcastNestedLoopJoinExec] physical operator).

=== [[broadcastSideBySizes]] Selecting Build Side Per Join Type and Total Size Statistic of Join Sides -- `broadcastSideBySizes` Internal Method

[source, scala]
----
broadcastSideBySizes(joinType: JoinType, left: LogicalPlan, right: LogicalPlan): BuildSide
----

`broadcastSideBySizes` computes `buildLeft` and `buildRight` flags:

* `buildLeft` flag is positive (i.e. `true`) when the join type is link:spark-sql-joins.adoc#CROSS[CROSS], link:spark-sql-joins.adoc#INNER[INNER] or link:spark-sql-joins.adoc#RIGHT_OUTER[RIGHT OUTER] (i.e. <<canBuildLeft, canBuildLeft>> for the input `joinType` is positive) and `left` operator <<canBroadcast, can be broadcast per total size statistic>>

* `buildRight` flag is positive (i.e. `true`) when the join type is link:spark-sql-joins.adoc#CROSS[CROSS], link:spark-sql-joins.adoc#INNER[INNER], link:spark-sql-joins.adoc#LEFT_ANTI[LEFT ANTI], link:spark-sql-joins.adoc#LEFT_OUTER[LEFT OUTER], link:spark-sql-joins.adoc#LEFT_SEMI[LEFT SEMI] or link:spark-sql-joins.adoc#ExistenceJoin[ExistenceJoin] (i.e. <<canBuildRight, canBuildRight>> for the input `joinType` is positive) and `right` operator <<canBroadcast, can be broadcast per total size statistic>>

In the end, `broadcastSideByHints` <<broadcastSide, gives the join side to broadcast>>.

NOTE: `broadcastSideByHints` is used when `JoinSelection` is requested to <<apply, plan a Join logical operator>> (and considers a link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc[BroadcastHashJoinExec] or a link:spark-sql-SparkPlan-BroadcastNestedLoopJoinExec.adoc[BroadcastNestedLoopJoinExec] physical operator).

=== [[apply]] Applying JoinSelection Strategy to Logical Plan (Executing JoinSelection) -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): Seq[SparkPlan]
----

NOTE: `apply` is part of link:spark-sql-catalyst-GenericStrategy.adoc#apply[GenericStrategy Contract] to generate a collection of link:spark-sql-SparkPlan.adoc[SparkPlans] for a given link:spark-sql-LogicalPlan.adoc[logical plan].

`apply` uses link:spark-sql-ExtractEquiJoinKeys.adoc[ExtractEquiJoinKeys] Scala extractor to destructure the input logical `plan`.

==== [[apply-BroadcastHashJoinExec]] Considering BroadcastHashJoinExec Physical Operator

`apply` gives a link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc#creating-instance[BroadcastHashJoinExec] physical operator if the plan <<canBroadcastByHints, should be broadcast per join type and broadcast hints used>> (for the join type and left or right side of the join). `apply` <<broadcastSideByHints, selects the build side per join type and broadcast hints>>.

`apply` gives a link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc#creating-instance[BroadcastHashJoinExec] physical operator if the plan <<canBroadcastBySizes, should be broadcast per join type and size of join sides>> (for the join type and left or right side of the join). `apply` <<broadcastSideBySizes, selects the build side per join type and total size statistic of join sides>>.

==== [[apply-ShuffledHashJoinExec]] Considering ShuffledHashJoinExec Physical Operator

`apply` gives...FIXME

==== [[apply-SortMergeJoinExec]] Considering SortMergeJoinExec Physical Operator

`apply` gives...FIXME

==== [[apply-BroadcastNestedLoopJoinExec]] Considering BroadcastNestedLoopJoinExec Physical Operator

`apply` gives...FIXME

==== [[apply-CartesianProductExec]] Considering CartesianProductExec Physical Operator

`apply` gives...FIXME
