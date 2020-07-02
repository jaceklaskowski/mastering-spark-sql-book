title: WindowFunction

# WindowFunction -- Window Function Expressions With WindowFrame

`WindowFunction` is the <<contract, contract>> of <<implementations, function expressions>> that define a <<frame, WindowFrame>> in which the window operator must be executed.

[[contract]]
[source, scala]
----
package org.apache.spark.sql.catalyst.expressions

trait WindowFunction extends Expression {
  // No required properties (vals and methods) that have no implementation
}
----

[[implementations]]
.WindowFunctions (Direct Implementations)
[cols="1,2",options="header",width="100%"]
|===
| WindowFunction
| Description

| <<spark-sql-Expression-AggregateWindowFunction.adoc#, AggregateWindowFunction>>
| [[AggregateWindowFunction]]

| <<spark-sql-Expression-OffsetWindowFunction.adoc#, OffsetWindowFunction>>
| [[OffsetWindowFunction]]
|===

=== [[frame]] Defining WindowFrame for Execution -- `frame` Method

[source, scala]
----
frame: WindowFrame
----

`frame` defines the `WindowFrame` for function execution, i.e. the `WindowFrame` in which the window operator must be executed.

`frame` is `UnspecifiedFrame` by default.

NOTE: `frame` is used when...FIXME
