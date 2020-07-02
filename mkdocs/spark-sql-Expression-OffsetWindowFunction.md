title: OffsetWindowFunction

# OffsetWindowFunction -- Unevaluable Window Function Expressions

`OffsetWindowFunction` is the <<contract, base>> of <<extensions, window function expressions>> that are <<spark-sql-Expression.adoc#Unevaluable, unevaluable>> and `ImplicitCastInputTypes`.

NOTE: An <<spark-sql-Expression.adoc#Unevaluable, unevaluable expression>> cannot be evaluated to produce a value (neither in <<spark-sql-Expression.adoc#eval, interpreted>> nor <<spark-sql-Expression.adoc#doGenCode, code-generated>> expression evaluations) and has to be resolved (replaced) to some other expressions or logical operators at <<spark-sql-QueryExecution.adoc#analyzed, analysis>> or <<spark-sql-QueryExecution.adoc#optimizedPlan, optimization>> phases or they fail analysis.

[[contract]]
[source, scala]
----
package org.apache.spark.sql.catalyst.expressions

abstract class OffsetWindowFunction ... {
  // only required properties (vals and methods) that have no implementation
  // the others follow
  val default: Expression
  val direction: SortDirection
  val input: Expression
  val offset: Expression
}
----

.(Subset of) OffsetWindowFunction Contract
[cols="1m,2",options="header",width="100%"]
|===
| Property
| Description

| default
| [[default]]

| direction
| [[direction]]

| input
| [[input]]

| offset
| [[offset]]
|===

[[children]]
`OffsetWindowFunction` uses the <<input, input>>, <<offset, offset>> and <<default, default>> expressions as the <<spark-sql-catalyst-TreeNode.adoc#children, children>>.

[[foldable]]
`OffsetWindowFunction` is not <<spark-sql-Expression.adoc#foldable, foldable>>.

[[nullable]]
`OffsetWindowFunction` is <<spark-sql-Expression.adoc#nullable, nullable>> when the <<default, default>> is not defined or the <<default, default>> or the <<input, input>> expressions are.

[[dataType]]
When requested for the <<spark-sql-Expression.adoc#dataType, dataType>>, `OffsetWindowFunction` simply requests the <<input, input>> expression for the data type.

[[dataType]]
When requested for the <<spark-sql-Expression-ExpectsInputTypes.adoc#inputTypes, inputTypes>>, `OffsetWindowFunction` returns the `AnyDataType`, <<spark-sql-DataType.adoc#IntegerType, IntegerType>> with the <<spark-sql-Expression.adoc#dataType, data type>> of the <<input, input>> expression and the <<spark-sql-DataType.adoc#NullType, NullType>>.

[[toString]]
`OffsetWindowFunction` uses the following *text representation* (i.e. `toString`):

```
[prettyName]([input], [offset], [default])
```

[[extensions]]
.OffsetWindowFunctions (Direct Implementations)
[cols="1,2",options="header",width="100%"]
|===
| OffsetWindowFunction
| Description

| Lag
| [[Lag]]

| Lead
| [[Lead]]
|===

=== [[frame]] `frame` Lazy Property

[source, scala]
----
frame: WindowFrame
----

NOTE: `frame` is part of the <<spark-sql-Expression-WindowFunction.adoc#frame, WindowFunction Contract>> to define the `WindowFrame` for function expression execution.

`frame`...FIXME

=== [[checkInputDataTypes]] Verifying Input Data Types -- `checkInputDataTypes` Method

[source, scala]
----
checkInputDataTypes(): TypeCheckResult
----

NOTE: `checkInputDataTypes` is part of the <<spark-sql-Expression.adoc#checkInputDataTypes, Expression Contract>> to verify (check the correctness of) the input data types.

`checkInputDataTypes`...FIXME
