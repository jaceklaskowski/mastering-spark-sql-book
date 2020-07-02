title: BasicStatsPlanVisitor

# BasicStatsPlanVisitor -- Computing Statistics for Cost-Based Optimization

`BasicStatsPlanVisitor` is a link:spark-sql-LogicalPlanVisitor.adoc[LogicalPlanVisitor] that computes the link:spark-sql-Statistics.adoc[statistics] of a logical query plan for link:spark-sql-cost-based-optimization.adoc[cost-based optimization] (i.e. when link:spark-sql-cost-based-optimization.adoc#spark.sql.cbo.enabled[cost-based optimization is enabled]).

NOTE: Cost-based optimization is enabled when link:spark-sql-properties.adoc#spark.sql.cbo.enabled[spark.sql.cbo.enabled] configuration property is on, i.e. `true`, and is disabled by default.

`BasicStatsPlanVisitor` is used exclusively when a link:spark-sql-LogicalPlanStats.adoc#stats[logical operator is requested for the statistics] with link:spark-sql-LogicalPlanStats.adoc#stats-cbo-enabled[cost-based optimization enabled].

`BasicStatsPlanVisitor` comes with custom <<handlers, handlers>> for a few logical operators and falls back to link:spark-sql-SizeInBytesOnlyStatsPlanVisitor.adoc[SizeInBytesOnlyStatsPlanVisitor] for the others.

[[handlers]]
.BasicStatsPlanVisitor's Visitor Handlers
[cols="1,1,2",options="header",width="100%"]
|===
| Logical Operator
| Handler
| Behaviour

| [[Aggregate]] link:spark-sql-LogicalPlan-Aggregate.adoc[Aggregate]
| [[visitAggregate]] link:spark-sql-LogicalPlanVisitor.adoc#visitAggregate[visitAggregate]
| Requests `AggregateEstimation` for link:spark-sql-AggregateEstimation.adoc#estimate[statistics estimates and query hints] or falls back to link:spark-sql-SizeInBytesOnlyStatsPlanVisitor.adoc[SizeInBytesOnlyStatsPlanVisitor]

| [[Filter]] `Filter`
| [[visitFilter]] link:spark-sql-LogicalPlanVisitor.adoc#visitFilter[visitFilter]
| Requests `FilterEstimation` for link:spark-sql-FilterEstimation.adoc#estimate[statistics estimates and query hints] or falls back to link:spark-sql-SizeInBytesOnlyStatsPlanVisitor.adoc[SizeInBytesOnlyStatsPlanVisitor]

| [[Join]] link:spark-sql-LogicalPlan-Join.adoc[Join]
| [[visitJoin]] link:spark-sql-LogicalPlanVisitor.adoc#visitJoin[visitJoin]
| Requests `JoinEstimation` for link:spark-sql-JoinEstimation.adoc#estimate[statistics estimates and query hints] or falls back to link:spark-sql-SizeInBytesOnlyStatsPlanVisitor.adoc[SizeInBytesOnlyStatsPlanVisitor]

| [[Project]] link:spark-sql-LogicalPlan-Project.adoc[Project]
| [[visitProject]] link:spark-sql-LogicalPlanVisitor.adoc#visitProject[visitProject]
| Requests `ProjectEstimation` for link:spark-sql-ProjectEstimation.adoc#estimate[statistics estimates and query hints] or falls back to link:spark-sql-SizeInBytesOnlyStatsPlanVisitor.adoc[SizeInBytesOnlyStatsPlanVisitor]
|===
