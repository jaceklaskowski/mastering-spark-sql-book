# DataSourceStrategy Execution Planning Strategy

`DataSourceStrategy` is an link:spark-sql-SparkStrategy.adoc[execution planning strategy] (of link:spark-sql-SparkPlanner.adoc[SparkPlanner]) that <<apply, plans LogicalRelation logical operators as RowDataSourceScanExec physical operators>> (possibly under `FilterExec` and `ProjectExec` operators).

[[apply]]
[[selection-requirements]]
.DataSourceStrategy's Selection Requirements (in execution order)
[cols="1,2",options="header",width="100%"]
|===
| Logical Operator
| Description

| link:spark-sql-LogicalPlan-LogicalRelation.adoc[LogicalRelation] with a link:spark-sql-CatalystScan.adoc[CatalystScan] relation
| [[CatalystScan]] Uses <<pruneFilterProjectRaw, pruneFilterProjectRaw>> (with the <<toCatalystRDD, RDD conversion to RDD[InternalRow]>> as part of `scanBuilder`).

`CatalystScan` does not seem to be used in Spark SQL.

| link:spark-sql-LogicalPlan-LogicalRelation.adoc[LogicalRelation] with link:spark-sql-PrunedFilteredScan.adoc[PrunedFilteredScan] relation
| [[PrunedFilteredScan]] Uses <<pruneFilterProject, pruneFilterProject>> (with the <<toCatalystRDD, RDD conversion to RDD[InternalRow]>> as part of `scanBuilder`).

Matches link:spark-sql-JDBCRelation.adoc[JDBCRelation] exclusively

| link:spark-sql-LogicalPlan-LogicalRelation.adoc[LogicalRelation] with a link:spark-sql-PrunedScan.adoc[PrunedScan] relation
| [[PrunedScan]] Uses <<pruneFilterProject, pruneFilterProject>> (with the <<toCatalystRDD, RDD conversion to RDD[InternalRow]>> as part of `scanBuilder`).

`PrunedScan` does not seem to be used in Spark SQL.

| link:spark-sql-LogicalPlan-LogicalRelation.adoc[LogicalRelation] with a link:spark-sql-TableScan.adoc[TableScan] relation
a| [[TableScan]] Creates a link:spark-sql-SparkPlan-RowDataSourceScanExec.adoc#creating-instance[RowDataSourceScanExec] directly (requesting the `TableScan` to link:spark-sql-TableScan.adoc#buildScan[buildScan] followed by <<toCatalystRDD, RDD conversion to RDD[InternalRow]>>)

Matches <<spark-sql-KafkaRelation.adoc#, KafkaRelation>> exclusively
|===

[source, scala]
----
import org.apache.spark.sql.execution.datasources.DataSourceStrategy
val strategy = DataSourceStrategy(spark.sessionState.conf)

import org.apache.spark.sql.catalyst.plans.logical.LogicalPlan
val plan: LogicalPlan = ???

val sparkPlan = strategy(plan).head
----

NOTE: `DataSourceStrategy` uses link:spark-sql-PhysicalOperation.adoc[PhysicalOperation] Scala extractor object to destructure a logical query plan.

=== [[pruneFilterProject]] `pruneFilterProject` Internal Method

[source, scala]
----
pruneFilterProject(
  relation: LogicalRelation,
  projects: Seq[NamedExpression],
  filterPredicates: Seq[Expression],
  scanBuilder: (Seq[Attribute], Array[Filter]) => RDD[InternalRow])
----

`pruneFilterProject` simply calls <<pruneFilterProjectRaw, pruneFilterProjectRaw>> with `scanBuilder` ignoring the `Seq[Expression]` input parameter.

NOTE: `pruneFilterProject` is used when `DataSourceStrategy` execution planning strategy is <<apply, executed>> (for link:spark-sql-LogicalPlan-LogicalRelation.adoc[LogicalRelation] logical operators with a link:spark-sql-PrunedFilteredScan.adoc[PrunedFilteredScan] or a link:spark-sql-PrunedScan.adoc[PrunedScan]).

=== [[selectFilters]] Selecting Catalyst Expressions Convertible to Data Source Filter Predicates (and Handled by BaseRelation) -- `selectFilters` Method

[source, scala]
----
selectFilters(
  relation: BaseRelation,
  predicates: Seq[Expression]): (Seq[Expression], Seq[Filter], Set[Filter])
----

`selectFilters` builds a map of link:spark-sql-Expression.adoc[Catalyst predicate expressions] (from the input `predicates`) that can be <<translateFilter, translated>> to a link:spark-sql-Filter.adoc[data source filter predicate].

`selectFilters` then requests the input `BaseRelation` for link:spark-sql-BaseRelation.adoc#unhandledFilters[unhandled filters] (out of the convertible ones that `selectFilters` built the map with).

In the end, `selectFilters` returns a 3-element tuple with the following:

. Inconvertible and unhandled Catalyst predicate expressions

. All converted data source filters

. Pushed-down data source filters (that the input `BaseRelation` can handle)

NOTE: `selectFilters` is used exclusively when `DataSourceStrategy` execution planning strategy is requested to <<pruneFilterProjectRaw, create a RowDataSourceScanExec physical operator (possibly under FilterExec and ProjectExec operators)>> (which is when `DataSourceStrategy` is <<apply, executed>> and <<pruneFilterProject, pruneFilterProject>>).

=== [[translateFilter]] Translating Catalyst Expression Into Data Source Filter Predicate -- `translateFilter` Method

[source, scala]
----
translateFilter(predicate: Expression): Option[Filter]
----

`translateFilter` translates a link:spark-sql-Expression.adoc[Catalyst expression] into a corresponding link:spark-sql-Filter.adoc[Filter predicate] if possible. If not, `translateFilter` returns `None`.

[[translateFilter-conversions]]
.translateFilter's Conversions
[cols="1,1",options="header",width="100%"]
|===
| Catalyst Expression
| Filter Predicate

| `EqualTo`
| `EqualTo`

| `EqualNullSafe`
| `EqualNullSafe`

| `GreaterThan`
| `GreaterThan`

| `LessThan`
| `LessThan`

| `GreaterThanOrEqual`
| `GreaterThanOrEqual`

| `LessThanOrEqual`
| `LessThanOrEqual`

| link:spark-sql-Expression-InSet.adoc[InSet]
| `In`

| link:spark-sql-Expression-In.adoc[In]
| `In`

| `IsNull`
| `IsNull`

| `IsNotNull`
| `IsNotNull`

| `And`
| `And`

| `Or`
| `Or`

| `Not`
| `Not`

| `StartsWith`
| `StringStartsWith`

| `EndsWith`
| `StringEndsWith`

| `Contains`
| `StringContains`
|===

NOTE: The Catalyst expressions and their corresponding data source filter predicates have the same names _in most cases_ but belong to different Scala packages, i.e. `org.apache.spark.sql.catalyst.expressions` and `org.apache.spark.sql.sources`, respectively.

[NOTE]
====
`translateFilter` is used when:

* `FileSourceScanExec` is link:spark-sql-SparkPlan-FileSourceScanExec.adoc#creating-instance[created] (and initializes link:spark-sql-SparkPlan-FileSourceScanExec.adoc#pushedDownFilters[pushedDownFilters])

* `DataSourceStrategy` is requested to <<selectFilters, selectFilters>>

* `PushDownOperatorsToDataSource` logical optimization is link:spark-sql-SparkOptimizer-PushDownOperatorsToDataSource.adoc#apply[executed] (for link:spark-sql-LogicalPlan-DataSourceV2Relation.adoc[DataSourceV2Relation] leaf operators with a link:spark-sql-SupportsPushDownFilters.adoc[SupportsPushDownFilters] data source reader)
====

=== [[toCatalystRDD]] RDD Conversion (Converting RDD of Rows to Catalyst RDD of InternalRows) -- `toCatalystRDD` Internal Method

[source, scala]
----
toCatalystRDD(
  relation: LogicalRelation,
  output: Seq[Attribute],
  rdd: RDD[Row]): RDD[InternalRow]
toCatalystRDD(relation: LogicalRelation, rdd: RDD[Row]) // <1>
----
<1> Calls the former `toCatalystRDD` with the link:spark-sql-LogicalPlan-LogicalRelation.adoc#output[output] of the `LogicalRelation`

`toCatalystRDD` branches off per the link:spark-sql-BaseRelation.adoc#needConversion[needConversion] flag of the link:spark-sql-LogicalPlan-LogicalRelation.adoc#relation[BaseRelation] of the input link:spark-sql-LogicalPlan-LogicalRelation.adoc[LogicalRelation].

When enabled (`true`), `toCatalystRDD` link:spark-sql-RDDConversions.adoc#rowToRowRdd[converts the objects inside Rows to Catalyst types].

NOTE: link:spark-sql-BaseRelation.adoc#needConversion[needConversion] flag is enabled (`true`) by default.

Otherwise, `toCatalystRDD` simply casts the input `RDD[Row]` to a `RDD[InternalRow]` (as a simple untyped Scala type conversion using Java's `asInstanceOf` operator).

NOTE: `toCatalystRDD` is used when `DataSourceStrategy` execution planning strategy is <<apply, executed>> (for all kinds of <<selection-requirements, BaseRelations>>).

=== [[pruneFilterProjectRaw]] Creating RowDataSourceScanExec Physical Operator for LogicalRelation (Possibly Under FilterExec and ProjectExec Operators) -- `pruneFilterProjectRaw` Internal Method

[source, scala]
----
pruneFilterProjectRaw(
  relation: LogicalRelation,
  projects: Seq[NamedExpression],
  filterPredicates: Seq[Expression],
  scanBuilder: (Seq[Attribute], Seq[Expression], Seq[Filter]) => RDD[InternalRow]): SparkPlan
----

`pruneFilterProjectRaw` creates a <<spark-sql-SparkPlan-RowDataSourceScanExec.adoc#creating-instance, RowDataSourceScanExec>> leaf physical operator given a <<spark-sql-LogicalPlan-LogicalRelation.adoc#, LogicalRelation>> leaf logical operator (possibly as a child of a <<spark-sql-SparkPlan-FilterExec.adoc#, FilterExec>> and a <<spark-sql-SparkPlan-ProjectExec.adoc#, ProjectExec>> unary physical operators).

In other words, `pruneFilterProjectRaw` simply converts a <<spark-sql-LogicalPlan-LogicalRelation.adoc#, LogicalRelation>> leaf logical operator into a <<spark-sql-SparkPlan-RowDataSourceScanExec.adoc#, RowDataSourceScanExec>> leaf physical operator (possibly under a <<spark-sql-SparkPlan-FilterExec.adoc#, FilterExec>> and a <<spark-sql-SparkPlan-ProjectExec.adoc#, ProjectExec>> unary physical operators).

NOTE: `pruneFilterProjectRaw` is almost like <<spark-sql-SparkPlanner.adoc#pruneFilterProject, SparkPlanner.pruneFilterProject>>.

Internally, `pruneFilterProjectRaw` splits the input `filterPredicates` expressions to <<selectFilters, select the Catalyst expressions that can be converted to data source filter predicates>> (and handled by the <<spark-sql-LogicalPlan-LogicalRelation.adoc#relation, BaseRelation>> of the `LogicalRelation`).

`pruneFilterProjectRaw` combines all expressions that are neither convertible to data source filters nor can be handled by the relation using `And` binary expression (that creates a so-called `filterCondition` that will eventually be used to create a <<spark-sql-SparkPlan-FilterExec.adoc#, FilterExec>> physical operator if non-empty).

`pruneFilterProjectRaw` creates a <<spark-sql-SparkPlan-RowDataSourceScanExec.adoc#creating-instance, RowDataSourceScanExec>> leaf physical operator.

If it is possible to use a column pruning only to get the right projection and if the columns of this projection are enough to evaluate all filter conditions, `pruneFilterProjectRaw` creates a <<spark-sql-SparkPlan-FilterExec.adoc#creating-instance, FilterExec>> unary physical operator (with the unhandled predicate expressions and the `RowDataSourceScanExec` leaf physical operator as the child).

NOTE: In this case no extra <<spark-sql-SparkPlan-ProjectExec.adoc#, ProjectExec>> unary physical operator is created.

Otherwise, `pruneFilterProjectRaw` creates a <<spark-sql-SparkPlan-FilterExec.adoc#creating-instance, FilterExec>> unary physical operator (with the unhandled predicate expressions and the `RowDataSourceScanExec` leaf physical operator as the child) that in turn becomes the <<spark-sql-SparkPlan-ProjectExec.adoc#child, child>> of a new <<spark-sql-SparkPlan-ProjectExec.adoc#creating-instance, ProjectExec>> unary physical operator.

NOTE: `pruneFilterProjectRaw` is used exclusively when `DataSourceStrategy` execution planning strategy is <<apply, executed>> (for a `LogicalRelation` with a `CatalystScan` relation) and <<pruneFilterProject, pruneFilterProject>> (when <<apply, executed>> for a `LogicalRelation` with a `PrunedFilteredScan` or a `PrunedScan` relation).
