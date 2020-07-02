title: TypedAggregateExpression

# TypedAggregateExpression Expression

`TypedAggregateExpression` is the <<contract, contract>> for link:spark-sql-Expression-AggregateFunction.adoc[AggregateFunction] expressions that...FIXME

[[contract]]
[source, scala]
----
package org.apache.spark.sql.execution.aggregate

trait TypedAggregateExpression extends AggregateFunction {
  // only required methods that have no implementation
  def aggregator: Aggregator[Any, Any, Any]
  def inputClass: Option[Class[_]]
  def inputDeserializer: Option[Expression]
  def inputSchema: Option[StructType]
  def withInputInfo(deser: Expression, cls: Class[_], schema: StructType): TypedAggregateExpression
}
----

`TypedAggregateExpression` is used when:

* `TypedColumn` is requested to link:spark-sql-TypedColumn.adoc#withInputType[withInputType] (for link:spark-sql-dataset-operators.adoc#select[Dataset.select], link:spark-sql-KeyValueGroupedDataset.adoc#agg[KeyValueGroupedDataset.agg] and link:spark-sql-RelationalGroupedDataset.adoc#agg[RelationalGroupedDataset.agg] operators)

* `Column` is requested to link:spark-sql-Column.adoc#generateAlias[generateAlias] and link:spark-sql-Column.adoc#named[named] (for link:spark-sql-dataset-operators.adoc#select[Dataset.select] and link:spark-sql-KeyValueGroupedDataset.adoc#agg[KeyValueGroupedDataset.agg] operators)

* `RelationalGroupedDataset` is requested to link:spark-sql-RelationalGroupedDataset.adoc#alias[alias] (when `RelationalGroupedDataset` is requested to link:spark-sql-RelationalGroupedDataset.adoc#toDF[create a DataFrame from aggregate expressions])

.TypedAggregateExpression Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| [[aggregator]] `aggregator`
| link:spark-sql-Aggregator.adoc[Aggregator]

| [[inputClass]] `inputClass`
| Used when...FIXME

| [[inputDeserializer]] `inputDeserializer`
| Used when...FIXME

| [[inputSchema]] `inputSchema`
| Used when...FIXME

| [[withInputInfo]] `withInputInfo`
| Used when...FIXME
|===

[[implementations]]
.TypedAggregateExpressions
[cols="1,2",options="header",width="100%"]
|===
| Aggregator
| Description

| [[ComplexTypedAggregateExpression]] link:spark-sql-Expression-ComplexTypedAggregateExpression.adoc[ComplexTypedAggregateExpression]
|

| [[SimpleTypedAggregateExpression]] link:spark-sql-Expression-SimpleTypedAggregateExpression.adoc[SimpleTypedAggregateExpression]
|
|===

=== [[apply]] Creating TypedAggregateExpression -- `apply` Factory Method

[source, scala]
----
apply[BUF : Encoder, OUT : Encoder](
  aggregator: Aggregator[_, BUF, OUT]): TypedAggregateExpression
----

`apply`...FIXME

NOTE: `apply` is used exclusively when `Aggregator` is requested to link:spark-sql-Aggregator.adoc#toColumn[convert itself to a TypedColumn].
