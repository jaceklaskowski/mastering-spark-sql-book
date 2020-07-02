title: TypedImperativeAggregate

# TypedImperativeAggregate -- Imperative Aggregate Functions with Custom Aggregation Buffer

`TypedImperativeAggregate` is the <<contract, contract>> for link:spark-sql-Expression-ImperativeAggregate.adoc[imperative aggregation functions] that allows for an arbitrary user-defined java object to be used as <<createAggregationBuffer, internal aggregation buffer>>.

[[ImperativeAggregate]]
.TypedImperativeAggregate as ImperativeAggregate
[cols="1,2",options="header",width="100%"]
|===
| ImperativeAggregate Method
| Description

| [[aggBufferAttributes]] `aggBufferAttributes`
|

| [[aggBufferSchema]] `aggBufferSchema`
|

| [[initialize]] link:spark-sql-Expression-ImperativeAggregate.adoc#initialize[initialize]
| <<createAggregationBuffer, Creates an aggregation buffer>> and puts it at link:spark-sql-Expression-ImperativeAggregate.adoc#mutableAggBufferOffset[mutableAggBufferOffset] position in the input `buffer` link:spark-sql-InternalRow.adoc[InternalRow].

| [[inputAggBufferAttributes]] `inputAggBufferAttributes`
|
|===

[[implementations]]
.TypedImperativeAggregate's Direct Implementations
[width="100%",cols="1,2",options="header"]
|===
| Name
| Description

| `ApproximatePercentile`
|

| `Collect`
|

| link:spark-sql-Expression-ComplexTypedAggregateExpression.adoc[ComplexTypedAggregateExpression]
|

| `CountMinSketchAgg`
|

| `HiveUDAFFunction`
|

| `Percentile`
|
|===

=== [[contract]] TypedImperativeAggregate Contract

[source, scala]
----
package org.apache.spark.sql.catalyst.expressions.aggregate

abstract class TypedImperativeAggregate[T] extends ImperativeAggregate {
  def createAggregationBuffer(): T
  def deserialize(storageFormat: Array[Byte]): T
  def eval(buffer: T): Any
  def merge(buffer: T, input: T): T
  def serialize(buffer: T): Array[Byte]
  def update(buffer: T, input: InternalRow): T
}
----

.TypedImperativeAggregate Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| [[createAggregationBuffer]] `createAggregationBuffer`
| Used exclusively when a `TypedImperativeAggregate` is <<initialize, initialized>>

| [[deserialize]] `deserialize`
|

| [[eval]] `eval`
|

| [[merge]] `merge`
|

| [[serialize]] `serialize`
|

| [[update]] `update`
|
|===
