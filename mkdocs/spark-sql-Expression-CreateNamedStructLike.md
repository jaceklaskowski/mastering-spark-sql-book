# CreateNamedStructLike

`CreateNamedStructLike` is the <<contract, base>> of <<implementations, Catalyst expressions>> that <<FIXME, FIXME>>.

[[contract]]
[source, scala]
----
package org.apache.spark.sql.catalyst.expressions

trait CreateNamedStructLike extends Expression {
  // no required properties (vals and methods) that have no implementation
}
----

[[nullable]]
`CreateNamedStructLike` is not <<spark-sql-Expression.adoc#nullable, nullable>>.

[[foldable]]
`CreateNamedStructLike` is <<spark-sql-Expression.adoc#foldable, foldable>> only if all <<valExprs, value expressions>> are.

[[implementations]]
.CreateNamedStructLikes (Direct Implementations)
[cols="1,2",options="header",width="100%"]
|===
| CreateNamedStructLike
| Description

| [[CreateNamedStruct]] <<spark-sql-Expression-CreateNamedStruct.adoc#, CreateNamedStruct>>
|

| [[CreateNamedStructUnsafe]] <<spark-sql-Expression-CreateNamedStructUnsafe.adoc#, CreateNamedStructUnsafe>>
|
|===

[[internal-registries]]
.CreateNamedStructLike's Internal Properties (e.g. Registries, Counters and Flags)
[cols="1m,2",options="header",width="100%"]
|===
| Name
| Description

| dataType
| [[dataType]]

| nameExprs
| [[nameExprs]] <<spark-sql-Expression.adoc#, Catalyst expressions>> for names

| names
| [[names]]

| valExprs
| [[valExprs]] <<spark-sql-Expression.adoc#, Catalyst expressions>> for values
|===

=== [[checkInputDataTypes]] Checking Input Data Types -- `checkInputDataTypes` Method

[source, scala]
----
checkInputDataTypes(): TypeCheckResult
----

NOTE: `checkInputDataTypes` is part of the <<spark-sql-Expression.adoc#checkInputDataTypes, Expression Contract>> to verify (check the correctness of) the input data types.

`checkInputDataTypes`...FIXME

=== [[eval]] Evaluating Expression -- `eval` Method

[source, scala]
----
eval(input: InternalRow): Any
----

NOTE: `eval` is part of <<spark-sql-Expression.adoc#eval, Expression Contract>> for the *interpreted (non-code-generated) expression evaluation*, i.e. evaluating a Catalyst expression to a JVM object for a given <<spark-sql-InternalRow.adoc#, internal binary row>>.

`eval`...FIXME
