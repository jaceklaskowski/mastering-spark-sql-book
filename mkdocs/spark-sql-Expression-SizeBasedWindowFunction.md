title: SizeBasedWindowFunction

# SizeBasedWindowFunction -- Declarative Window Aggregate Functions with Window Size

`SizeBasedWindowFunction` is the <<contract, extension>> of the <<spark-sql-Expression-AggregateWindowFunction.adoc#, AggregateWindowFunction Contract>> for <<implementations, window functions>> that require the <<n, size of the current window>> for calculation.

[[contract]]
[source, scala]
----
package org.apache.spark.sql.catalyst.expressions

trait SizeBasedWindowFunction extends AggregateWindowFunction {
  // No required properties (vals and methods) that have no implementation
}
----

.SizeBasedWindowFunction Contract
[cols="1m,2",options="header",width="100%"]
|===
| Property
| Description

| n
| [[n]] Size of the current window as a <<spark-sql-Expression-AttributeReference.adoc#, AttributeReference>> expression with `++window__partition__size++` name, <<spark-sql-DataType.adoc#IntegerType, IntegerType>> data type and not nullable
|===

[[implementations]]
.SizeBasedWindowFunctions (Direct Implementations)
[cols="1,2",options="header",width="100%"]
|===
| SizeBasedWindowFunction
| Description

| <<spark-sql-Expression-CumeDist.adoc#, CumeDist>>
| [[CumeDist]] Window function expression for <<spark-sql-functions.adoc#cume_dist, cume_dist>> standard function (Dataset API) and <<spark-sql-FunctionRegistry.adoc#expressions, cume_dist>> SQL function

| NTile
| [[NTile]]

| PercentRank
| [[PercentRank]]
|===
