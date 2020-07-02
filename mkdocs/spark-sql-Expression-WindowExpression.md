title: WindowExpression

# WindowExpression Unevaluable Expression

`WindowExpression` is an <<spark-sql-Expression.adoc#Unevaluable, unevaluable expression>> that represents a <<windowFunction, window function>> (over some <<windowSpec, WindowSpecDefinition>>).

NOTE: An <<spark-sql-Expression.adoc#Unevaluable, unevaluable expression>> cannot be evaluated to produce a value (neither in <<spark-sql-Expression.adoc#eval, interpreted>> nor <<spark-sql-Expression.adoc#doGenCode, code-generated>> expression evaluations) and has to be resolved (replaced) to some other expressions or logical operators at <<spark-sql-QueryExecution.adoc#analyzed, analysis>> or <<spark-sql-QueryExecution.adoc#optimizedPlan, optimization>> phases or they fail analysis.

`WindowExpression` is <<creating-instance, created>> when:

* `WindowSpec` is requested to <<spark-sql-WindowSpec.adoc#withAggregate, withAggregate>> (when <<spark-sql-Column.adoc#over, Column.over>> operator is used)

* `WindowsSubstitution` logical evaluation rule is <<spark-sql-Analyzer-WindowsSubstitution.adoc#apply, executed>> (with <<spark-sql-LogicalPlan-WithWindowDefinition.adoc#, WithWindowDefinition>> logical operators with <<spark-sql-Expression-UnresolvedWindowExpression.adoc#, UnresolvedWindowExpression>> expressions)

* `AstBuilder` is requested to <<spark-sql-AstBuilder.adoc#visitFunctionCall, parse a function call>> in a SQL statement

NOTE: `WindowExpression` can only be <<creating-instance, created>> with <<spark-sql-Expression-AggregateExpression.adoc#, AggregateExpression>>, <<spark-sql-Expression-AggregateWindowFunction.adoc#, AggregateWindowFunction>> or <<spark-sql-Expression-OffsetWindowFunction.adoc#, OffsetWindowFunction>> expressions which is enforced at <<spark-sql-Analyzer-CheckAnalysis.adoc#WindowExpression, analysis>>.

[source, scala]
----
// Using Catalyst DSL
val wf = 'count.function(star())
val windowSpec = ???
----

NOTE: `WindowExpression` is resolved in <<spark-sql-Analyzer-ExtractWindowExpressions.adoc#, ExtractWindowExpressions>>, <<spark-sql-Analyzer-ResolveWindowFrame.adoc#, ResolveWindowFrame>> and <<spark-sql-Analyzer-ResolveWindowOrder.adoc#, ResolveWindowOrder>> logical rules.

[source, scala]
----
import org.apache.spark.sql.catalyst.expressions.WindowExpression
// relation - Dataset as a table to query
val table = spark.emptyDataset[Int]

scala> val windowExpr = table
  .selectExpr("count() OVER (PARTITION BY value) AS count")
  .queryExecution
  .logical      // <1>
  .expressions
  .toList(0)
  .children(0)
  .asInstanceOf[WindowExpression]
windowExpr: org.apache.spark.sql.catalyst.expressions.WindowExpression = 'count() windowspecdefinition('value, UnspecifiedFrame)

scala> windowExpr.sql
res2: String = count() OVER (PARTITION BY `value` UnspecifiedFrame)
----
<1> Use `sqlParser` directly as in link:spark-sql-LogicalPlan-WithWindowDefinition.adoc#example[WithWindowDefinition Example]

[[properties]]
.WindowExpression's Properties
[width="100%",cols="1,2",options="header"]
|===
| Name
| Description

| `children`
| Collection of two link:spark-sql-Expression.adoc[expressions], i.e. <<windowFunction, windowFunction>> and <<windowSpec, WindowSpecDefinition>>, for which `WindowExpression` was created.

| `dataType`
| link:spark-sql-DataType.adoc[DataType] of <<windowFunction, windowFunction>>

| `foldable`
| Whether or not <<windowFunction, windowFunction>> is foldable.

| `nullable`
| Whether or not <<windowFunction, windowFunction>> is nullable.

| `sql`
| `"[windowFunction].sql OVER [windowSpec].sql"`

| `toString`
| `"[windowFunction] [windowSpec]"`
|===

NOTE: `WindowExpression` is subject to <<spark-sql-Optimizer-NullPropagation.adoc#, NullPropagation>> and <<spark-sql-Optimizer-DecimalAggregates.adoc#, DecimalAggregates>> logical optimizations.

NOTE: Distinct window functions are not supported which is enforced at <<spark-sql-Analyzer-CheckAnalysis.adoc#WindowExpression-AggregateExpression-isDistinct, analysis>>.

NOTE: An offset window function can only be evaluated in an ordered row-based window frame with a single offset which is enforced at <<spark-sql-Analyzer-CheckAnalysis.adoc#WindowExpression-OffsetWindowFunction, analysis>>.

=== [[catalyst-dsl]][[windowExpr]] Catalyst DSL -- `windowExpr` Operator

[source, scala]
----
windowExpr(
  windowFunc: Expression,
  windowSpec: WindowSpecDefinition): WindowExpression
----

<<spark-sql-catalyst-dsl.adoc#windowExpr, windowExpr>> operator in Catalyst DSL creates a <<creating-instance, WindowExpression>> expression, e.g. for testing or Spark SQL internals exploration.

[source, scala]
----
// FIXME: DEMO
----

=== [[creating-instance]] Creating WindowExpression Instance

`WindowExpression` takes the following when created:

* [[windowFunction]] Window function <<spark-sql-Expression.adoc#, expression>>
* [[windowSpec]] <<spark-sql-Expression-WindowSpecDefinition.adoc#, WindowSpecDefinition>> expression
