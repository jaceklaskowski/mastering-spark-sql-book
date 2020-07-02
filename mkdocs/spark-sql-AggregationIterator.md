title: AggregationIterator

# AggregationIterator -- Generic Iterator of UnsafeRows for Aggregate Physical Operators

`AggregationIterator` is the base for <<implementations, iterators>> of <<spark-sql-UnsafeRow.adoc, UnsafeRows>> that...FIXME

> Iterators are data structures that allow to iterate over a sequence of elements. They have a `hasNext` method for checking if there is a next element available, and a `next` method which returns the next element and discards it from the iterator.

[[implementations]]
.AggregationIterator's Implementations
[width="100%",cols="1,2",options="header"]
|===
| Name
| Description

| <<spark-sql-ObjectAggregationIterator.adoc#, ObjectAggregationIterator>>
| Used exclusively when `ObjectHashAggregateExec` physical operator is link:spark-sql-SparkPlan-ObjectHashAggregateExec.adoc#doExecute[executed].

| <<spark-sql-SortBasedAggregationIterator.adoc#, SortBasedAggregationIterator>>
| Used exclusively when `SortAggregateExec` physical operator is link:spark-sql-SparkPlan-SortAggregateExec.adoc#doExecute[executed].

| link:spark-sql-TungstenAggregationIterator.adoc[TungstenAggregationIterator]
a| Used exclusively when `HashAggregateExec` physical operator is link:spark-sql-SparkPlan-HashAggregateExec.adoc#doExecute[executed].

NOTE: link:spark-sql-SparkPlan-HashAggregateExec.adoc[HashAggregateExec] operator is the preferred aggregate physical operator for link:spark-sql-SparkStrategy-Aggregation.adoc[Aggregation] execution planning strategy (over `ObjectHashAggregateExec` and `SortAggregateExec`).
|===

[[internal-registries]]
.AggregationIterator's Internal Registries and Counters
[cols="1,2",options="header",width="100%"]
|===
| Name
| Description

| [[aggregateFunctions]] `aggregateFunctions`
| link:spark-sql-Expression-AggregateFunction.adoc[Aggregate functions]

Used when...FIXME

| [[allImperativeAggregateFunctions]] `allImperativeAggregateFunctions`
| link:spark-sql-Expression-ImperativeAggregate.adoc[ImperativeAggregate] functions

Used when...FIXME

| [[allImperativeAggregateFunctionPositions]] `allImperativeAggregateFunctionPositions`
| Positions

Used when...FIXME

| [[expressionAggInitialProjection]] `expressionAggInitialProjection`
| `MutableProjection`

Used when...FIXME

| `generateOutput`
a| [[generateOutput]] Function used to generate an <<spark-sql-UnsafeRow.adoc#, unsafe row>> (i.e. `(UnsafeRow, InternalRow) => UnsafeRow`)

Used when:

* `ObjectAggregationIterator` is requested for the <<spark-sql-ObjectAggregationIterator.adoc#next, next unsafe row>> and <<spark-sql-ObjectAggregationIterator.adoc#outputForEmptyGroupingKeyWithoutInput, outputForEmptyGroupingKeyWithoutInput>>

* `SortBasedAggregationIterator` is requested for the <<spark-sql-SortBasedAggregationIterator.adoc#next, next unsafe row>> and <<spark-sql-SortBasedAggregationIterator.adoc#outputForEmptyGroupingKeyWithoutInput, outputForEmptyGroupingKeyWithoutInput>>

* `TungstenAggregationIterator` is requested for the <<spark-sql-TungstenAggregationIterator.adoc#next, next unsafe row>> and <<spark-sql-TungstenAggregationIterator.adoc#outputForEmptyGroupingKeyWithoutInput, outputForEmptyGroupingKeyWithoutInput>>

| [[groupingAttributes]] `groupingAttributes`
| Grouping link:spark-sql-Expression-Attribute.adoc[attributes]

Used when...FIXME

| [[groupingProjection]] `groupingProjection`
| link:spark-sql-UnsafeProjection.adoc[UnsafeProjection]

Used when...FIXME

| [[processRow]] `processRow`
| `(InternalRow, InternalRow) => Unit`

Used when...FIXME
|===

=== [[creating-instance]] Creating AggregationIterator Instance

`AggregationIterator` takes the following when created:

* [[groupingExpressions]] Grouping link:spark-sql-Expression-NamedExpression.adoc[named expressions]
* [[inputAttributes]] Input link:spark-sql-Expression-Attribute.adoc[attributes]
* [[aggregateExpressions]] link:spark-sql-Expression-AggregateExpression.adoc[Aggregate expressions]
* [[aggregateAttributes]] Aggregate link:spark-sql-Expression-Attribute.adoc[attributes]
* [[initialInputBufferOffset]] Initial input buffer offset
* [[resultExpressions]] Result link:spark-sql-Expression-NamedExpression.adoc[named expressions]
* [[newMutableProjection]] Function to create a new `MutableProjection` given expressions and attributes

`AggregationIterator` initializes the <<internal-registries, internal registries and counters>>.

NOTE: `AggregationIterator` is a Scala abstract class and cannot be <<creating-instance, created>> directly. It is created indirectly for the <<implementations, concrete AggregationIterators>>.

=== [[initializeAggregateFunctions]] `initializeAggregateFunctions` Internal Method

[source, scala]
----
initializeAggregateFunctions(
  expressions: Seq[AggregateExpression],
  startingInputBufferOffset: Int): Array[AggregateFunction]
----

`initializeAggregateFunctions`...FIXME

NOTE: `initializeAggregateFunctions` is used when...FIXME

=== [[generateProcessRow]] `generateProcessRow` Internal Method

[source, scala]
----
generateProcessRow(
  expressions: Seq[AggregateExpression],
  functions: Seq[AggregateFunction],
  inputAttributes: Seq[Attribute]): (InternalRow, InternalRow) => Unit
----

`generateProcessRow`...FIXME

NOTE: `generateProcessRow` is used when...FIXME

=== [[generateResultProjection]] `generateResultProjection` Method

[source, scala]
----
generateResultProjection(): (UnsafeRow, InternalRow) => UnsafeRow
----

`generateResultProjection`...FIXME

[NOTE]
====
`generateResultProjection` is used when:

* `AggregationIterator` is <<generateOutput, created>>

* `TungstenAggregationIterator` is requested for the <<spark-sql-TungstenAggregationIterator.adoc#generateResultProjection, generateResultProjection>>
====
