title: AggregateWindowFunction

# AggregateWindowFunction -- Declarative Window Aggregate Function Expressions

`AggregateWindowFunction` is the <<contract, extension>> of the <<spark-sql-Expression-DeclarativeAggregate.adoc#, DeclarativeAggregate Contract>> for <<extensions, declarative aggregate function expressions>> that are also <<spark-sql-Expression-WindowFunction.adoc#, WindowFunction>> expressions.

[[contract]]
[source, scala]
----
package org.apache.spark.sql.catalyst.expressions

abstract class AggregateWindowFunction extends DeclarativeAggregate with WindowFunction {
  self: Product =>
  // No required properties (vals and methods) that have no implementation
}
----

[[dataType]]
`AggregateWindowFunction` uses <<spark-sql-DataType.adoc#IntegerType, IntegerType>> as the <<spark-sql-Expression.adoc#dataType, data type>> of the result of evaluating itself.

[[nullable]]
`AggregateWindowFunction` is <<spark-sql-Expression.adoc#nullable, nullable>> by default.

[[frame]]
As a <<spark-sql-Expression-WindowFunction.adoc#, WindowFunction>> expression, `AggregateWindowFunction` uses a `SpecifiedWindowFrame` (with the `RowFrame` frame type, the `UnboundedPreceding` lower and the `CurrentRow` upper frame boundaries) as the <<spark-sql-Expression-WindowFunction.adoc#frame, frame>>.

[[mergeExpressions]]
`AggregateWindowFunction` is a <<spark-sql-Expression-DeclarativeAggregate.adoc#, DeclarativeAggregate>> expression that does not support <<spark-sql-Expression-DeclarativeAggregate.adoc#mergeExpressions, merging>> (two aggregation buffers together) and throws an `UnsupportedOperationException` whenever requested for it.

```
Window Functions do not support merging.
```

[[extensions]]
.AggregateWindowFunctions (Direct Implementations)
[cols="1,2",options="header",width="100%"]
|===
| AggregateWindowFunction
| Description

| <<spark-sql-Expression-RankLike.adoc#, RankLike>>
| [[RankLike]]

| <<spark-sql-Expression-RowNumberLike.adoc#, RowNumberLike>>
| [[RowNumberLike]]

| <<spark-sql-Expression-SizeBasedWindowFunction.adoc#, SizeBasedWindowFunction>>
| [[SizeBasedWindowFunction]] Window functions that require the size of the current window for calculation
|===
