title: Aggregator

# Aggregator -- User-Defined Typed Aggregate Functions (UDAFs)

`Aggregator` is the <<contract, contract>> for *user-defined typed aggregate functions* (aka _user-defined typed aggregations_ or _UDAFs_ in short).

[[contract]]
[source, scala]
----
package org.apache.spark.sql.expressions

abstract class Aggregator[-IN, BUF, OUT] extends Serializable {
  // only required methods that have no implementation
  def bufferEncoder: Encoder[BUF]
  def finish(reduction: BUF): OUT
  def merge(b1: BUF, b2: BUF): BUF
  def outputEncoder: Encoder[OUT]
  def reduce(b: BUF, a: IN): BUF
  def zero: BUF
}
----

After you create a custom `Aggregator`, you should use <<toColumn, toColumn>> method to convert it to a `TypedColumn` that can be used with link:spark-sql-dataset-operators.adoc#select[Dataset.select] and link:spark-sql-KeyValueGroupedDataset.adoc#agg[KeyValueGroupedDataset.agg] typed operators.

[source, scala]
----
// From Spark MLlib's org.apache.spark.ml.recommendation.ALSModel
// Step 1. Create Aggregator
val topKAggregator: Aggregator[Int, Int, Float] = ???
val recs = ratings
  .as[(Int, Int, Float)]
  .groupByKey(_._1)
  .agg(topKAggregator.toColumn) // <-- use the custom Aggregator
  .toDF("id", "recommendations")
----

[NOTE]
====
Use `org.apache.spark.sql.expressions.scalalang.typed` object to access the type-safe aggregate functions, i.e. `avg`, `count`, `sum` and `sumLong`.

[source, scala]
----
import org.apache.spark.sql.expressions.scalalang.typed

// Example 1
ds.groupByKey(_._1).agg(typed.sum(_._2))

// Example 2
ds.select(typed.sum((i: Int) => i))
----
====

[NOTE]
====
`Aggregator` is an `Experimental` and `Evolving` contract that is evolving towards becoming a stable API, but is not a stable API yet and can change from one feature release to another release.

In other words, using the contract is as treading on thin ice.
====

`Aggregator` is used when:

* link:spark-sql-Expression-SimpleTypedAggregateExpression.adoc#creating-instance[SimpleTypedAggregateExpression] and link:spark-sql-Expression-ComplexTypedAggregateExpression.adoc#creating-instance[ComplexTypedAggregateExpression] are created

* `TypedAggregateExpression` is requested for the link:spark-sql-Expression-TypedAggregateExpression.adoc#aggregator[aggregator]

.Aggregator Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| [[bufferEncoder]] `bufferEncoder`
| Used when...FIXME

| [[finish]] `finish`
| Used when...FIXME

| [[merge]] `merge`
| Used when...FIXME

| [[outputEncoder]] `outputEncoder`
| Used when...FIXME

| [[reduce]] `reduce`
| Used when...FIXME

| [[zero]] `zero`
| Used when...FIXME
|===

[[implementations]]
.Aggregators
[cols="1,2",options="header",width="100%"]
|===
| Aggregator
| Description

| [[ParameterizedTypeSum]] `ParameterizedTypeSum`
|

| [[ReduceAggregator]] `ReduceAggregator`
|

| [[TopByKeyAggregator]] `TopByKeyAggregator`
| Used exclusively in Spark MLlib

| [[TypedAverage]] `TypedAverage`
|

| [[TypedCount]] `TypedCount`
|

| [[TypedSumDouble]] `TypedSumDouble`
|

| [[TypedSumLong]] `TypedSumLong`
|
|===

=== [[toColumn]] Converting Aggregator to TypedColumn -- `toColumn` Method

[source, scala]
----
toColumn: TypedColumn[IN, OUT]
----

`toColumn`...FIXME

NOTE: `toColumn` is used when...FIXME
