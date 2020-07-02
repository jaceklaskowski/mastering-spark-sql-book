title: Literal

# Literal Leaf Expression

`Literal` is a link:spark-sql-Expression.adoc#LeafExpression[leaf expression] that is <<creating-instance, created>> to represent a Scala <<value, value>> of a <<dataType, specific type>>.

`Literal` is <<creating-instance, created>> when...MEFIXME

[[properties]]
.Literal's Properties
[width="100%",cols="1,2",options="header"]
|===
| Property
| Description

| `foldable`
| [[foldable]] Enabled (i.e. `true`)

| `nullable`
| [[nullable]] Enabled when <<value, value>> is `null`
|===

=== [[create]] Creating Literal Instance -- `create` Object Method

[source, scala]
----
create(v: Any, dataType: DataType): Literal
----

`create` uses `CatalystTypeConverters` helper object to <<spark-sql-CatalystTypeConverters.adoc#convertToCatalyst, convert>> the input `v` Scala value to a Catalyst rows or types and creates a <<creating-instance, Literal>> (with the Catalyst value and the input <<spark-sql-DataType.adoc#, DataType>>).

NOTE: `create` is used when...FIXME

=== [[creating-instance]] Creating Literal Instance

`Literal` takes the following when created:

* [[value]] Scala value (of type `Any`)
* [[dataType]] <<spark-sql-DataType.adoc#, DataType>>

=== [[doGenCode]] Generating Java Source Code (ExprCode) For Code-Generated Expression Evaluation -- `doGenCode` Method

[source, scala]
----
doGenCode(ctx: CodegenContext, ev: ExprCode): ExprCode
----

NOTE: `doGenCode` is part of <<spark-sql-Expression.adoc#doGenCode, Expression Contract>> to generate a Java source code (ExprCode) for code-generated expression evaluation.

`doGenCode`...FIXME
