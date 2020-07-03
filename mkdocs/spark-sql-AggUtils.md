# AggUtils Helper Object

`AggUtils` is a Scala object that defines the methods used exclusively when <<spark-sql-SparkStrategy-Aggregation.adoc#, Aggregation>> execution planning strategy is executed.

* <<planAggregateWithoutDistinct, planAggregateWithoutDistinct>>

* <<planAggregateWithOneDistinct, planAggregateWithOneDistinct>>

=== [[planAggregateWithOneDistinct]] `planAggregateWithOneDistinct` Method

[source, scala]
----
planAggregateWithOneDistinct(
  groupingExpressions: Seq[NamedExpression],
  functionsWithDistinct: Seq[AggregateExpression],
  functionsWithoutDistinct: Seq[AggregateExpression],
  resultExpressions: Seq[NamedExpression],
  child: SparkPlan): Seq[SparkPlan]
----

`planAggregateWithOneDistinct`...FIXME

NOTE: `planAggregateWithOneDistinct` is used exclusively when `Aggregation` execution planning strategy is <<spark-sql-SparkStrategy-Aggregation.adoc#apply, executed>>.

=== [[planAggregateWithoutDistinct]] Creating Physical Plan with Two Aggregate Physical Operators for Partial and Final Aggregations -- `planAggregateWithoutDistinct` Method

[source, scala]
----
planAggregateWithoutDistinct(
  groupingExpressions: Seq[NamedExpression],
  aggregateExpressions: Seq[AggregateExpression],
  resultExpressions: Seq[NamedExpression],
  child: SparkPlan): Seq[SparkPlan]
----

`planAggregateWithoutDistinct` is a two-step physical operator generator.

`planAggregateWithoutDistinct` first <<AggUtils-createAggregate, creates an aggregate physical operator>> with `aggregateExpressions` in `Partial` mode (for partial aggregations).

NOTE: `requiredChildDistributionExpressions` for the aggregate physical operator for partial aggregation "stage" is empty.

In the end, `planAggregateWithoutDistinct` <<AggUtils-createAggregate, creates another aggregate physical operator>> (of the same type as before), but `aggregateExpressions` are now in `Final` mode (for final aggregations). The aggregate physical operator becomes the parent of the first aggregate operator.

NOTE: `requiredChildDistributionExpressions` for the parent aggregate physical operator for final aggregation "stage" are the link:spark-sql-Expression-Attribute.adoc[attributes] of `groupingExpressions`.

NOTE: `planAggregateWithoutDistinct` is used exclusively when `Aggregation` execution planning strategy is <<spark-sql-SparkStrategy-Aggregation.adoc#apply, executed>> (with no `AggregateExpressions` being <<spark-sql-Expression-AggregateExpression.adoc#isDistinct, distinct>>).

=== [[createAggregate]] Creating Aggregate Physical Operator -- `createAggregate` Internal Method

[source, scala]
----
createAggregate(
  requiredChildDistributionExpressions: Option[Seq[Expression]] = None,
  groupingExpressions: Seq[NamedExpression] = Nil,
  aggregateExpressions: Seq[AggregateExpression] = Nil,
  aggregateAttributes: Seq[Attribute] = Nil,
  initialInputBufferOffset: Int = 0,
  resultExpressions: Seq[NamedExpression] = Nil,
  child: SparkPlan): SparkPlan
----

`createAggregate` creates a <<spark-sql-SparkPlan.adoc#, physical operator>> given the input `aggregateExpressions` <<spark-sql-Expression-AggregateExpression.adoc#, aggregate expressions>>.

[[aggregate-physical-operator-selection-criteria]]
.createAggregate's Aggregate Physical Operator Selection Criteria (in execution order)
[cols="1,2",options="header",width="100%"]
|===
| Aggregate Physical Operator
| Selection Criteria

| link:spark-sql-SparkPlan-HashAggregateExec.adoc[HashAggregateExec]
a| `HashAggregateExec` link:spark-sql-SparkPlan-HashAggregateExec.adoc#supportsAggregate[supports] all `aggBufferAttributes` of the input `aggregateExpressions` link:spark-sql-Expression-AggregateExpression.adoc[aggregate expressions].

| link:spark-sql-SparkPlan-ObjectHashAggregateExec.adoc[ObjectHashAggregateExec]
a|

. link:spark-sql-properties.adoc#spark.sql.execution.useObjectHashAggregateExec[spark.sql.execution.useObjectHashAggregateExec] internal flag enabled (it is by default)

. `ObjectHashAggregateExec` link:spark-sql-SparkPlan-ObjectHashAggregateExec.adoc#supportsAggregate[supports] the input `aggregateExpressions` link:spark-sql-Expression-AggregateExpression.adoc[aggregate expressions].

| link:spark-sql-SparkPlan-SortAggregateExec.adoc[SortAggregateExec]
| When all the above requirements could not be met.
|===

NOTE: `createAggregate` is used when `AggUtils` is requested to <<planAggregateWithoutDistinct, planAggregateWithoutDistinct>>, <<planAggregateWithOneDistinct, planAggregateWithOneDistinct>> (and `planStreamingAggregation` for Spark Structured Streaming)
