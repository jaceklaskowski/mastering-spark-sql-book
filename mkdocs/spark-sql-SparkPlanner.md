title: SparkPlanner

# SparkPlanner -- Spark Query Planner

`SparkPlanner` is a concrete link:spark-sql-catalyst-QueryPlanner.adoc[Catalyst Query Planner] that converts a link:spark-sql-LogicalPlan.adoc[logical plan] to one or more link:spark-sql-SparkPlan.adoc[physical plans] using <<strategies, execution planning strategies>> with support for <<extraStrategies, extra strategies>> (by means of <<experimentalMethods, ExperimentalMethods>>) and <<extraPlanningStrategies, extraPlanningStrategies>>.

NOTE: `SparkPlanner` is expected to plan (aka _generate_) at least one link:spark-sql-SparkPlan.adoc[physical plan] per link:spark-sql-LogicalPlan.adoc[logical plan].

`SparkPlanner` is available as link:spark-sql-SessionState.adoc#planner[planner] of a `SessionState`.

[source, scala]
----
val spark: SparkSession = ...
scala> :type spark.sessionState.planner
org.apache.spark.sql.execution.SparkPlanner
----

[[strategies]]
.SparkPlanner's Execution Planning Strategies (in execution order)
[cols="1,2",options="header",width="100%"]
|===
| SparkStrategy
| Description

| [[extraStrategies]] ``ExperimentalMethods``'s link:spark-sql-ExperimentalMethods.adoc#extraStrategies[extraStrategies]
|

| <<extraPlanningStrategies, extraPlanningStrategies>>
| Extension point for extra link:spark-sql-SparkStrategy.adoc[planning strategies]

| link:spark-sql-SparkStrategy-DataSourceV2Strategy.adoc[DataSourceV2Strategy]
|

| link:spark-sql-SparkStrategy-FileSourceStrategy.adoc[FileSourceStrategy]
|

| link:spark-sql-SparkStrategy-DataSourceStrategy.adoc[DataSourceStrategy]
|

| link:spark-sql-SparkStrategy-SpecialLimits.adoc[SpecialLimits]
|

| link:spark-sql-SparkStrategy-Aggregation.adoc[Aggregation]
|

| link:spark-sql-SparkStrategy-JoinSelection.adoc[JoinSelection]
|

| link:spark-sql-SparkStrategy-InMemoryScans.adoc[InMemoryScans]
|

| link:spark-sql-SparkStrategy-BasicOperators.adoc[BasicOperators]
|
|===

NOTE: `SparkPlanner` extends link:spark-sql-SparkStrategies.adoc[SparkStrategies] abstract class.

=== [[creating-instance]] Creating SparkPlanner Instance

`SparkPlanner` takes the following when created:

* [[sparkContext]] link:spark-SparkContext.adoc[SparkContext]
* [[conf]] link:spark-sql-SQLConf.adoc[SQLConf]
* [[experimentalMethods]] link:spark-sql-ExperimentalMethods.adoc[ExperimentalMethods]

[NOTE]
====
`SparkPlanner` is <<creating-instance, created>> in:

* link:spark-sql-BaseSessionStateBuilder.adoc[BaseSessionStateBuilder]
* link:hive/HiveSessionStateBuilder.adoc[HiveSessionStateBuilder]
* Structured Streaming's `IncrementalExecution`
====

=== [[extraPlanningStrategies]] Extension Point for Extra Planning Strategies -- `extraPlanningStrategies` Method

[source, scala]
----
extraPlanningStrategies: Seq[Strategy] = Nil
----

`extraPlanningStrategies` is an extension point to register extra link:spark-sql-SparkStrategy.adoc[planning strategies] with the query planner.

NOTE: `extraPlanningStrategies` are executed after <<extraStrategies, extraStrategies>>.

[NOTE]
====
`extraPlanningStrategies` is used when `SparkPlanner` <<strategies, is requested for planning strategies>>.

`extraPlanningStrategies` is overriden in the `SessionState` builders -- link:spark-sql-BaseSessionStateBuilder.adoc#planner[BaseSessionStateBuilder] and link:hive/HiveSessionStateBuilder.adoc#planner[HiveSessionStateBuilder].
====

=== [[collectPlaceholders]] Collecting PlanLater Physical Operators -- `collectPlaceholders` Method

[source, scala]
----
collectPlaceholders(plan: SparkPlan): Seq[(SparkPlan, LogicalPlan)]
----

`collectPlaceholders` collects all link:spark-sql-SparkStrategy.adoc#PlanLater[PlanLater] physical operators in the `plan` link:spark-sql-SparkPlan.adoc[physical plan].

NOTE: `collectPlaceholders` is part of link:spark-sql-catalyst-QueryPlanner.adoc#collectPlaceholders[QueryPlanner Contract].

=== [[prunePlans]] Pruning "Bad" Physical Plans -- `prunePlans` Method

[source, scala]
----
prunePlans(plans: Iterator[SparkPlan]): Iterator[SparkPlan]
----

`prunePlans` gives the input `plans` link:spark-sql-SparkPlan.adoc[physical plans] back (i.e. with no changes).

NOTE: `prunePlans` is part of link:spark-sql-catalyst-QueryPlanner.adoc#prunePlans[QueryPlanner Contract] to remove somehow "bad" plans.

=== [[pruneFilterProject]] Creating Physical Operator (Possibly Under FilterExec and ProjectExec Operators) -- `pruneFilterProject` Method

[source, scala]
----
pruneFilterProject(
  projectList: Seq[NamedExpression],
  filterPredicates: Seq[Expression],
  prunePushedDownFilters: Seq[Expression] => Seq[Expression],
  scanBuilder: Seq[Attribute] => SparkPlan): SparkPlan
----

NOTE: `pruneFilterProject` is almost like <<spark-sql-SparkStrategy-DataSourceStrategy.adoc#pruneFilterProjectRaw, DataSourceStrategy.pruneFilterProjectRaw>>.

`pruneFilterProject` branches off per whether it is possible to use a column pruning only (to get the right projection) and the input `projectList` columns of this projection are enough to evaluate all input `filterPredicates` filter conditions.

If so, `pruneFilterProject` does the following:

. Applies the input `scanBuilder` function to the input `projectList` columns that creates a new <<spark-sql-SparkPlan.adoc#, physical operator>>

. If there are Catalyst predicate expressions in the input `prunePushedDownFilters` that cannot be pushed down, `pruneFilterProject` creates a <<spark-sql-SparkPlan-FilterExec.adoc#creating-instance, FilterExec>> unary physical operator (with the unhandled predicate expressions)

. Otherwise, `pruneFilterProject` simply returns the physical operator

NOTE: In this case no extra <<spark-sql-SparkPlan-ProjectExec.adoc#, ProjectExec>> unary physical operator is created.

If not (i.e. it is neither possible to use a column pruning only nor evaluate filter conditions), `pruneFilterProject` does the following:

. Applies the input `scanBuilder` function to the projection and filtering columns that creates a new <<spark-sql-SparkPlan.adoc#, physical operator>>

. Creates a <<spark-sql-SparkPlan-FilterExec.adoc#creating-instance, FilterExec>> unary physical operator (with the unhandled predicate expressions if available)

. Creates a <<spark-sql-SparkPlan-ProjectExec.adoc#creating-instance, ProjectExec>> unary physical operator with the optional `FilterExec` operator (with the scan physical operator) or simply the scan physical operator alone

NOTE: `pruneFilterProject` is used when link:hive/HiveTableScans.adoc[HiveTableScans] and link:spark-sql-SparkStrategy-InMemoryScans.adoc[InMemoryScans] execution planning strategies are executed.
