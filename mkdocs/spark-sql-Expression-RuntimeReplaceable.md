title: RuntimeReplaceable

# RuntimeReplaceable -- Replaceable SQL Expressions

`RuntimeReplaceable` is the <<contract, marker contract>> for <<spark-sql-Expression-UnaryExpression.adoc#, unary expressions>> that are replaced by link:spark-sql-Optimizer.adoc#ReplaceExpressions[Catalyst Optimizer] with their child expression (that can then be evaluated).

NOTE: Catalyst Optimizer uses link:spark-sql-Optimizer-ReplaceExpressions.adoc[ReplaceExpressions] logical optimization to replace `RuntimeReplaceable` expressions.

`RuntimeReplaceable` contract allows for *expression aliases*, i.e. expressions that are fairly complex in the inside than on the outside, and is used to provide compatibility with other SQL databases by supporting SQL functions with their more complex Catalyst expressions (that are already supported by Spark SQL).

NOTE: <<implementations, RuntimeReplaceables>> are tied up to their SQL functions in link:spark-sql-FunctionRegistry.adoc#expressions[FunctionRegistry].

[[Unevaluable]]
`RuntimeReplaceable` expressions link:spark-sql-Expression.adoc#Unevaluable[cannot be evaluated] (i.e. produce a value given an internal row) and therefore have to be replaced in the link:spark-sql-QueryExecution.adoc[query execution pipeline].

[[contract]]
[source, scala]
----
package org.apache.spark.sql.catalyst.expressions

trait RuntimeReplaceable extends UnaryExpression with Unevaluable {
  // as a marker contract it only marks a class
  // no methods are required
}
----

NOTE: To make sure the `explain` plan and expression SQL works correctly, a `RuntimeReplaceable` implementation should override link:spark-sql-Expression.adoc#flatArguments[flatArguments] and link:spark-sql-Expression.adoc#sql[sql] methods.

[[implementations]]
.RuntimeReplaceables
[cols="1,1,1",options="header",width="100%"]
|===
| RuntimeReplaceable
| Standard Function
| SQL Function

| [[IfNull]] `IfNull`
|
| link:spark-sql-FunctionRegistry.adoc#ifnull[ifnull]

| [[Left]] `Left`
|
| link:spark-sql-FunctionRegistry.adoc#left[left]

| [[NullIf]] `NullIf`
|
| link:spark-sql-FunctionRegistry.adoc#nullif[nullif]

| [[Nvl]] `Nvl`
|
| link:spark-sql-FunctionRegistry.adoc#nvl[nvl]

| [[Nvl2]] `Nvl2`
|
| link:spark-sql-FunctionRegistry.adoc#nvl2[nvl2]

| [[ParseToDate]] link:spark-sql-Expression-ParseToDate.adoc[ParseToDate]
| link:spark-sql-functions-datetime.adoc#to_date[to_date]
| link:spark-sql-FunctionRegistry.adoc#to_date[to_date]

| [[ParseToTimestamp]] link:spark-sql-Expression-ParseToTimestamp.adoc[ParseToTimestamp]
| link:spark-sql-functions-datetime.adoc#to_timestamp[to_timestamp]
| link:spark-sql-FunctionRegistry.adoc#to_timestamp[to_timestamp]

| [[Right]] `Right`
|
| link:spark-sql-FunctionRegistry.adoc#right[right]
|===
