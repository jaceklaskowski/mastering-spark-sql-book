title: Column Operators

# Column API -- Column Operators

Column API is a <<methods, set of operators>> to work with values in a column (of a <<spark-sql-Dataset.adoc#, Dataset>>).

[[methods]]
[[operators]]
.Column Operators
[cols="1m,3",options="header",width="100%"]
|===
| Operator
| Description

| asc
a| [[asc]]

[source, scala]
----
asc: Column
----

| asc_nulls_first
a| [[asc_nulls_first]]

[source, scala]
----
asc_nulls_first: Column
----

| asc_nulls_last
a| [[asc_nulls_last]]

[source, scala]
----
asc_nulls_last: Column
----

| desc
a| [[desc]]

[source, scala]
----
desc: Column
----

| desc_nulls_first
a| [[desc_nulls_first]]

[source, scala]
----
desc_nulls_first: Column
----

| desc_nulls_last
a| [[desc_nulls_last]]

[source, scala]
----
desc_nulls_last: Column
----

| <<isin-internals, isin>>
a| [[isin]]

[source, scala]
----
isin(list: Any*): Column
----

| isInCollection
a| [[isInCollection]]

[source, scala]
----
isInCollection(values: scala.collection.Iterable[_]): Column
----

(*New in 2.4.0*) An expression operator that is `true` if the value of the column is in the given `values` collection

`isInCollection` is simply a synonym of <<isin, isin>> operator.
|===

=== [[isin-internals]] `isin` Operator

[source, scala]
----
isin(list: Any*): Column
----

Internally, `isin` creates a `Column` with <<spark-sql-Expression-In.adoc#, In>> predicate expression.

[source, scala]
----
val ids = Seq((1, 2, 2), (2, 3, 1)).toDF("x", "y", "id")
scala> ids.show
+---+---+---+
|  x|  y| id|
+---+---+---+
|  1|  2|  2|
|  2|  3|  1|
+---+---+---+

val c = $"id" isin ($"x", $"y")
val q = ids.filter(c)
scala> q.show
+---+---+---+
|  x|  y| id|
+---+---+---+
|  1|  2|  2|
+---+---+---+

// Note that isin accepts non-Column values
val c = $"id" isin ("x", "y")
val q = ids.filter(c)
scala> q.show
+---+---+---+
|  x|  y| id|
+---+---+---+
+---+---+---+
----
