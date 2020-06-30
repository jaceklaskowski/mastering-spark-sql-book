title: WindowSpec

# WindowSpec -- Window Specification

`WindowSpec` is a *window specification* that defines which rows are included in a *window* (_frame_), i.e. the set of rows that are associated with the current row by some _relation_.

[[creating-instance]]
`WindowSpec` takes the following when created:

* [[partitionSpec]] *Partition specification* (`Seq[Expression]`) which defines which records are in the same partition. With no partition defined, all records belong to a single partition

* [[orderSpec]] *Ordering Specification* (`Seq[SortOrder]`) which defines how records in a partition are ordered that in turn defines the position of a record in a partition. The ordering could be ascending (`ASC` in SQL or `asc` in Scala) or descending (`DESC` or `desc`).

* [[frame]] *Frame Specification* (`WindowFrame`) which defines the rows to be included in the frame for the current row, based on their relative position to the current row. For example, _"the three rows preceding the current row to the current row"_ describes a frame including the current input row and three rows appearing before the current row.

You use <<spark-sql-WindowSpec-Window.adoc#, Window object>> to create a `WindowSpec`.

[source, scala]
----
import org.apache.spark.sql.expressions.Window
scala> val byHTokens = Window.partitionBy('token startsWith "h")
byHTokens: org.apache.spark.sql.expressions.WindowSpec = org.apache.spark.sql.expressions.WindowSpec@574985d8
----

Once the initial version of a `WindowSpec` is created, you use the <<methods, methods>> to further configure the window specification.

[[methods]]
.WindowSpec API
[cols="1m,3",options="header",width="100%"]
|===
| Method
| Description

| orderBy
a| [[orderBy]]

[source, scala]
----
orderBy(cols: Column*): WindowSpec
orderBy(colName: String, colNames: String*): WindowSpec
----

| partitionBy
a| [[partitionBy]]

[source, scala]
----
partitionBy(cols: Column*): WindowSpec
partitionBy(colName: String, colNames: String*): WindowSpec
----

| rangeBetween
a| [[rangeBetween]]

[source, scala]
----
rangeBetween(start: Column, end: Column): WindowSpec
rangeBetween(start: Long, end: Long): WindowSpec
----

| rowsBetween
a| [[rowsBetween]]

[source, scala]
----
rowsBetween(start: Long, end: Long): WindowSpec
----
|===

With a window specification fully defined, you use <<spark-sql-Column.adoc#over, Column.over>> operator that associates the `WindowSpec` with an <<spark-sql-functions.adoc#aggregate-functions, aggregate>> or <<spark-sql-functions.adoc#window-functions, window>> function.

[source, scala]
----
scala> :type windowSpec
org.apache.spark.sql.expressions.WindowSpec

import org.apache.spark.sql.functions.rank
val c = rank over windowSpec
----

=== [[withAggregate]] `withAggregate` Internal Method

[source, scala]
----
withAggregate(aggregate: Column): Column
----

`withAggregate`...FIXME

NOTE: `withAggregate` is used exclusively when <<spark-sql-Column.adoc#over, Column.over>> operator is used.
