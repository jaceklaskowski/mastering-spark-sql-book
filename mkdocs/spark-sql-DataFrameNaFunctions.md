title: DataFrameNaFunctions

# DataFrameNaFunctions -- Working With Missing Data

`DataFrameNaFunctions` is used to work with <<methods, missing data>> in a structured query (a <<spark-sql-DataFrame.adoc#, DataFrame>>).

[[methods]]
.DataFrameNaFunctions API
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| <<drop, drop>>
a|

[source, scala]
----
drop(): DataFrame
drop(cols: Array[String]): DataFrame
drop(minNonNulls: Int): DataFrame
drop(minNonNulls: Int, cols: Array[String]): DataFrame
drop(minNonNulls: Int, cols: Seq[String]): DataFrame
drop(cols: Seq[String]): DataFrame
drop(how: String): DataFrame
drop(how: String, cols: Array[String]): DataFrame
drop(how: String, cols: Seq[String]): DataFrame
----

| <<fill, fill>>
a|

[source, scala]
----
fill(value: Boolean): DataFrame
fill(value: Boolean, cols: Array[String]): DataFrame
fill(value: Boolean, cols: Seq[String]): DataFrame
fill(value: Double): DataFrame
fill(value: Double, cols: Array[String]): DataFrame
fill(value: Double, cols: Seq[String]): DataFrame
fill(value: Long): DataFrame
fill(value: Long, cols: Array[String]): DataFrame
fill(value: Long, cols: Seq[String]): DataFrame
fill(valueMap: Map[String, Any]): DataFrame
fill(value: String): DataFrame
fill(value: String, cols: Array[String]): DataFrame
fill(value: String, cols: Seq[String]): DataFrame
----

| <<replace, replace>>
a|

[source, scala]
----
replace[T](cols: Seq[String], replacement: Map[T, T]): DataFrame
replace[T](col: String, replacement: Map[T, T]): DataFrame
----
|===

[[creating-instance]]
`DataFrameNaFunctions` is available using <<spark-sql-Dataset-untyped-transformations.adoc#na, na>> untyped transformation.

[source, scala]
----
val q: DataFrame = ...
q.na
----

=== [[convertToDouble]] `convertToDouble` Internal Method

[source, scala]
----
convertToDouble(v: Any): Double
----

`convertToDouble`...FIXME

NOTE: `convertToDouble` is used when...FIXME

=== [[drop]] `drop` Method

[source, scala]
----
drop(): DataFrame
drop(cols: Array[String]): DataFrame
drop(minNonNulls: Int): DataFrame
drop(minNonNulls: Int, cols: Array[String]): DataFrame
drop(minNonNulls: Int, cols: Seq[String]): DataFrame
drop(cols: Seq[String]): DataFrame
drop(how: String): DataFrame
drop(how: String, cols: Array[String]): DataFrame
drop(how: String, cols: Seq[String]): DataFrame
----

`drop`...FIXME

=== [[fill]] `fill` Method

[source, scala]
----
fill(value: Boolean): DataFrame
fill(value: Boolean, cols: Array[String]): DataFrame
fill(value: Boolean, cols: Seq[String]): DataFrame
fill(value: Double): DataFrame
fill(value: Double, cols: Array[String]): DataFrame
fill(value: Double, cols: Seq[String]): DataFrame
fill(value: Long): DataFrame
fill(value: Long, cols: Array[String]): DataFrame
fill(value: Long, cols: Seq[String]): DataFrame
fill(valueMap: Map[String, Any]): DataFrame
fill(value: String): DataFrame
fill(value: String, cols: Array[String]): DataFrame
fill(value: String, cols: Seq[String]): DataFrame
----

`fill`...FIXME

=== [[fillCol]] `fillCol` Internal Method

[source, scala]
----
fillCol[T](col: StructField, replacement: T): Column
----

`fillCol`...FIXME

NOTE: `fillCol` is used when...FIXME

=== [[fillMap]] `fillMap` Internal Method

[source, scala]
----
fillMap(values: Seq[(String, Any)]): DataFrame
----

`fillMap`...FIXME

NOTE: `fillMap` is used when...FIXME

=== [[fillValue]] `fillValue` Internal Method

[source, scala]
----
fillValue[T](value: T, cols: Seq[String]): DataFrame
----

`fillValue`...FIXME

NOTE: `fillValue` is used when...FIXME

=== [[replace0]] `replace0` Internal Method

[source, scala]
----
replace0[T](cols: Seq[String], replacement: Map[T, T]): DataFrame
----

`replace0`...FIXME

NOTE: `replace0` is used when...FIXME

=== [[replace]] `replace` Method

[source, scala]
----
replace[T](cols: Seq[String], replacement: Map[T, T]): DataFrame
replace[T](col: String, replacement: Map[T, T]): DataFrame
----

`replace`...FIXME

=== [[replaceCol]] `replaceCol` Internal Method

[source, scala]
----
replaceCol(col: StructField, replacementMap: Map[_, _]): Column
----

`replaceCol`...FIXME

NOTE: `replaceCol` is used when...FIXME
