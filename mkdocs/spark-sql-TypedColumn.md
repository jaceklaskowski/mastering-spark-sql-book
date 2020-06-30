# TypedColumn

`TypedColumn` is a link:spark-sql-Column.adoc[Column] with the <<encoder, ExpressionEncoder>> for the types of the input and the output.

`TypedColumn` is <<creating-instance, created>> using link:spark-sql-Column.adoc#as[as] operator on a `Column`.

[source, scala]
----
scala> val id = $"id".as[Int]
id: org.apache.spark.sql.TypedColumn[Any,Int] = id

scala> id.expr
res1: org.apache.spark.sql.catalyst.expressions.Expression = 'id
----

=== [[name]] `name` Operator

[source, scala]
----
name(alias: String): TypedColumn[T, U]
----

NOTE: `name` is part of link:spark-sql-Column.adoc#name[Column Contract] to...FIXME.

`name`...FIXME

NOTE: `name` is used when...FIXME

=== [[withInputType]] Creating TypedColumn -- `withInputType` Internal Method

[source, scala]
----
withInputType(
  inputEncoder: ExpressionEncoder[_],
  inputAttributes: Seq[Attribute]): TypedColumn[T, U]
----

`withInputType`...FIXME

[NOTE]
====
`withInputType` is used when the following typed operators are used:

* link:spark-sql-dataset-operators.adoc#select[Dataset.select]

* link:spark-sql-KeyValueGroupedDataset.adoc#agg[KeyValueGroupedDataset.agg]

* link:spark-sql-RelationalGroupedDataset.adoc#agg[RelationalGroupedDataset.agg]
====

=== [[creating-instance]] Creating TypedColumn Instance

`TypedColumn` takes the following when created:

* [[expr]] Catalyst link:spark-sql-Expression.adoc[expression]
* [[encoder]] link:spark-sql-ExpressionEncoder.adoc[ExpressionEncoder] of the column results

`TypedColumn` initializes the <<internal-registries, internal registries and counters>>.
