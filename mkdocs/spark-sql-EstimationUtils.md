# EstimationUtils

`EstimationUtils` is...FIXME

=== [[getOutputSize]] `getOutputSize` Method

[source, scala]
----
getOutputSize(
  attributes: Seq[Attribute],
  outputRowCount: BigInt,
  attrStats: AttributeMap[ColumnStat] = AttributeMap(Nil)): BigInt
----

`getOutputSize`...FIXME

NOTE: `getOutputSize` is used when...FIXME

=== [[nullColumnStat]] `nullColumnStat` Method

[source, scala]
----
nullColumnStat(dataType: DataType, rowCount: BigInt): ColumnStat
----

`nullColumnStat`...FIXME

NOTE: `nullColumnStat` is used exclusively when `JoinEstimation` is requested to link:spark-sql-JoinEstimation.adoc#estimateInnerOuterJoin[estimateInnerOuterJoin] for `LeftOuter` and `RightOuter` joins.

=== [[rowCountsExist]] Checking Availability of Row Count Statistic -- `rowCountsExist` Method

[source, scala]
----
rowCountsExist(plans: LogicalPlan*): Boolean
----

`rowCountsExist` is positive (i.e. `true`) when every logical plan (in the input `plans`) has link:spark-sql-Statistics.adoc#rowCount[estimated number of rows] (aka _row count_) statistic computed.

Otherwise, `rowCountsExist` is negative (i.e. `false`).

NOTE: `rowCountsExist` uses `LogicalPlanStats` to access the link:spark-sql-LogicalPlanStats.adoc#stats[estimated statistics and query hints] of a logical plan.

[NOTE]
====
`rowCountsExist` is used when:

* `AggregateEstimation` is requested to link:spark-sql-AggregateEstimation.adoc#estimate[estimate statistics and query hints of a Aggregate logical operator]

* `JoinEstimation` is requested to link:spark-sql-JoinEstimation.adoc#estimate[estimate statistics and query hints of a Join logical operator] (regardless of the join type)

* `ProjectEstimation` is requested to link:spark-sql-ProjectEstimation.adoc#estimate[estimate statistics and query hints of a Project logical operator]
====
