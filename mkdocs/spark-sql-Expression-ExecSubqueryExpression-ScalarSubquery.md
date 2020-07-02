title: ScalarSubquery

# ScalarSubquery (ExecSubqueryExpression) Expression

`ScalarSubquery` is an link:spark-sql-Expression-ExecSubqueryExpression.adoc[ExecSubqueryExpression] that <<updateResult, can give exactly one value>> (i.e. the value of executing <<plan, SubqueryExec>> subquery that can result in a single row and a single column or `null` if no row were computed).

IMPORTANT: Spark SQL uses the name of `ScalarSubquery` twice to represent an `ExecSubqueryExpression` (this page) and a link:spark-sql-Expression-SubqueryExpression-ScalarSubquery.adoc[SubqueryExpression]. It _is_ confusing and you should _not_ be anymore.

`ScalarSubquery` is <<creating-instance, created>> exclusively when `PlanSubqueries` physical optimization is link:spark-sql-PlanSubqueries.adoc#apply[executed] (and plans a link:spark-sql-Expression-SubqueryExpression-ScalarSubquery.adoc[ScalarSubquery] expression).

[source, scala]
----
// FIXME DEMO
import org.apache.spark.sql.execution.PlanSubqueries
val spark = ...
val planSubqueries = PlanSubqueries(spark)
val plan = ...
val executedPlan = planSubqueries(plan)
----

[[Unevaluable]]
`ScalarSubquery` expression link:spark-sql-Expression.adoc#Unevaluable[cannot be evaluated], i.e. produce a value given an internal row.

[[dataType]]
`ScalarSubquery` uses...FIXME...for the <<spark-sql-Expression.adoc#dataType, data type>>.

[[internal-registries]]
.ScalarSubquery's Internal Properties (e.g. Registries, Counters and Flags)
[cols="1,2",options="header",width="100%"]
|===
| Name
| Description

| `result`
| [[result]] The value of the single column in a single row after collecting the rows from executing the <<plan, subquery plan>> or `null` if no rows were collected.

| `updated`
| [[updated]] Flag that says whether `ScalarSubquery` was <<updateResult, updated>> with collected result of executing the <<plan, subquery plan>>.
|===

=== [[creating-instance]] Creating ScalarSubquery Instance

`ScalarSubquery` takes the following when created:

* [[plan]] link:spark-sql-SparkPlan-SubqueryExec.adoc[SubqueryExec] plan
* [[exprId]] Expression ID (as `ExprId`)

=== [[updateResult]] Updating ScalarSubquery With Collected Result -- `updateResult` Method

[source, scala]
----
updateResult(): Unit
----

NOTE: `updateResult` is part of link:spark-sql-Expression-ExecSubqueryExpression.adoc#updateResult[ExecSubqueryExpression Contract] to fill an Catalyst expression with a collected result from executing a subquery plan.

`updateResult` requests <<plan, SubqueryExec>> physical plan to link:spark-sql-SparkPlan-SubqueryExec.adoc#executeCollect[execute and collect internal rows].

`updateResult` sets <<result, result>> to the value of the only column of the single row or `null` if no row were collected.

In the end, `updateResult` marks the `ScalarSubquery` instance as <<updated, updated>>.

`updateResult` reports a `RuntimeException` when there are more than 1 rows in the result.

```
more than one row returned by a subquery used as an expression:
[plan]
```

`updateResult` reports an `AssertionError` when the number of fields is not exactly 1.

```
Expects 1 field, but got [numFields] something went wrong in analysis
```

=== [[eval]] Evaluating Expression -- `eval` Method

[source, scala]
----
eval(input: InternalRow): Any
----

NOTE: `eval` is part of <<spark-sql-Expression.adoc#eval, Expression Contract>> for the *interpreted (non-code-generated) expression evaluation*, i.e. evaluating a Catalyst expression to a JVM object for a given <<spark-sql-InternalRow.adoc#, internal binary row>>.

`eval` simply returns <<result, result>> value.

`eval` reports an `IllegalArgumentException` if the `ScalarSubquery` expression has not been <<updated, updated>> yet.

=== [[doGenCode]] Generating Java Source Code (ExprCode) For Code-Generated Expression Evaluation -- `doGenCode` Method

[source, scala]
----
doGenCode(ctx: CodegenContext, ev: ExprCode): ExprCode
----

NOTE: `doGenCode` is part of <<spark-sql-Expression.adoc#doGenCode, Expression Contract>> to generate a Java source code (ExprCode) for code-generated expression evaluation.

`doGenCode` first makes sure that the <<updated, updated>> flag is on (`true`). If not, `doGenCode` throws an `IllegalArgumentException` exception with the following message:

```
requirement failed: [this] has not finished
```

`doGenCode` then creates a <<spark-sql-Expression-Literal.adoc#create, Literal>> (for the <<result, result>> and the <<dataType, dataType>>) and simply requests it to <<spark-sql-Expression-Literal.adoc#doGenCode, generate a Java source code>>.
