title: Window Utility

# Window Utility Object -- Defining Window Specification

`Window` utility object is a <<methods, set of static methods>> to define a <<spark-sql-WindowSpec.adoc#, window specification>>.

[[methods]]
.Window API
[cols="1m,3",options="header",width="100%"]
|===
| Method
| Description

| currentRow
a| [[currentRow]]

[source, scala]
----
currentRow: Long
----

Value representing the current row that is used to define <<spark-sql-WindowSpec.adoc#frame, frame boundaries>>.

| orderBy
a| [[orderBy]]

[source, scala]
----
orderBy(cols: Column*): WindowSpec
orderBy(colName: String, colNames: String*): WindowSpec
----

Creates a <<spark-sql-WindowSpec.adoc#, WindowSpec>> with the <<spark-sql-WindowSpec.adoc#orderSpec, ordering>> defined.

| partitionBy
a| [[partitionBy]]

[source, scala]
----
partitionBy(cols: Column*): WindowSpec
partitionBy(colName: String, colNames: String*): WindowSpec
----

Creates a <<spark-sql-WindowSpec.adoc#, WindowSpec>> with the <<spark-sql-WindowSpec.adoc#partitionSpec, partitioning>> defined.

| rangeBetween
a| [[rangeBetween]]

[source, scala]
----
rangeBetween(start: Column, end: Column): WindowSpec
rangeBetween(start: Long, end: Long): WindowSpec
----

Creates a <<spark-sql-WindowSpec.adoc#, WindowSpec>> with the <<spark-sql-WindowSpec.adoc#frame, frame boundaries>> defined, from `start` (inclusive) to `end` (inclusive). Both `start` and `end` are relative to the current row based on the actual value of the `ORDER BY` expression(s).

| rowsBetween
a| [[rowsBetween]]

[source, scala]
----
rowsBetween(start: Long, end: Long): WindowSpec
----

Creates a <<spark-sql-WindowSpec.adoc#, WindowSpec>> with the <<spark-sql-WindowSpec.adoc#frame, frame boundaries>> defined, from `start` (inclusive) to `end` (inclusive). Both `start` and `end` are positions relative to the current row based on the position of the row within the partition.

| unboundedFollowing
a| [[unboundedFollowing]]

[source, scala]
----
unboundedFollowing: Long
----

Value representing the last row in a partition (equivalent to "UNBOUNDED FOLLOWING" in SQL) that is used to define <<spark-sql-WindowSpec.adoc#frame, frame boundaries>>.

| unboundedPreceding
a| [[unboundedPreceding]]

[source, scala]
----
unboundedPreceding: Long
----

Value representing the first row in a partition (equivalent to "UNBOUNDED PRECEDING" in SQL) that is used to define <<spark-sql-WindowSpec.adoc#frame, frame boundaries>>.
|===

[source, scala]
----
import org.apache.spark.sql.expressions.Window
import org.apache.spark.sql.functions.{currentRow, lit}
val windowSpec = Window
  .partitionBy($"orderId")
  .orderBy($"time")
  .rangeBetween(currentRow, lit(1))
scala> :type windowSpec
org.apache.spark.sql.expressions.WindowSpec
----

=== [[spec]] Creating "Empty" WindowSpec -- `spec` Internal Method

[source, scala]
----
spec: WindowSpec
----

`spec` creates an "empty" <<spark-sql-WindowSpec.adoc#creating-instance, WindowSpec>>, i.e. with empty <<spark-sql-WindowSpec.adoc#partitionSpec, partition>> and <<spark-sql-WindowSpec.adoc#orderSpec, ordering>> specifications, and a `UnspecifiedFrame`.

[NOTE]
====
`spec` is used when:

* <<spark-sql-Column.adoc#over, Column.over>> operator is used (with no `WindowSpec`)

* `Window` utility object is requested to <<partitionBy, partitionBy>>, <<orderBy, orderBy>>, <<rowsBetween, rowsBetween>> and <<rangeBetween, rangeBetween>>
====
