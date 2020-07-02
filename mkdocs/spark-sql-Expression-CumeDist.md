title: CumeDist

# CumeDist Declarative Window Aggregate Function Expression

`CumeDist` is a <<spark-sql-Expression-SizeBasedWindowFunction.adoc#, SizeBasedWindowFunction>> and a <<spark-sql-Expression-RowNumberLike.adoc#, RowNumberLike>> expression that is used for the following:

* <<spark-sql-functions.adoc#cume_dist, cume_dist>> standard function (Dataset API)

* <<spark-sql-FunctionRegistry.adoc#expressions, cume_dist>> SQL function

[[creating-instance]]
`CumeDist` takes no input parameters when created.

[source, scala]
----
import org.apache.spark.sql.catalyst.expressions.CumeDist
val cume_dist = CumeDist()
----

[[prettyName]]
`CumeDist` uses *cume_dist* for the <<spark-sql-Expression.adoc#prettyName, user-facing name>>.

[source, scala]
----
import org.apache.spark.sql.catalyst.expressions.CumeDist
val cume_dist = CumeDist()
scala> println(cume_dist)
cume_dist()
----

[[frame]]
As an <<spark-sql-Expression-WindowFunction.adoc#, WindowFunction>> expression (indirectly), `CumeDist` requires the `SpecifiedWindowFrame` (with the `RangeFrame` frame type, the `UnboundedPreceding` lower and the `CurrentRow` upper frame boundaries) as the <<spark-sql-Expression-WindowFunction.adoc#frame, frame>>.

NOTE: The <<frame, frame>> for `CumeDist` expression is range-based instead of row-based, because it has to return the same value for tie values in a window (equal values per `ORDER BY` specification).

[[evaluateExpression]]
As a <<spark-sql-Expression-DeclarativeAggregate.adoc#, DeclarativeAggregate>> expression (indirectly), `CumeDist` defines the <<spark-sql-Expression-DeclarativeAggregate.adoc#evaluateExpression, evaluateExpression>> expression which returns the final value when `CumeDist` is evaluated. The value uses the formula `rowNumber / n` where `rowNumber` is the row number in a window frame (the number of values before and including the current row) divided by the number of rows in the window frame.

[source, scala]
----
import org.apache.spark.sql.catalyst.expressions.CumeDist
val cume_dist = CumeDist()
scala> println(cume_dist.evaluateExpression.numberedTreeString)
00 (cast(rowNumber#0 as double) / cast(window__partition__size#1 as double))
01 :- cast(rowNumber#0 as double)
02 :  +- rowNumber#0: int
03 +- cast(window__partition__size#1 as double)
04    +- window__partition__size#1: int
----
