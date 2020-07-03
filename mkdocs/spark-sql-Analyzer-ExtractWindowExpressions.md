# ExtractWindowExpressions Logical Resolution Rule

`ExtractWindowExpressions` is a <<spark-sql-Analyzer.adoc#batches, logical resolution rule>> that <<apply, transforms a logical query plan>> and replaces (extracts) <<spark-sql-Expression-WindowExpression.adoc#, WindowExpression>> expressions with <<spark-sql-LogicalPlan-Window.adoc#, Window>> logical operators.

`ExtractWindowExpressions` is part of the <<spark-sql-Analyzer.adoc#Resolution, Resolution>> fixed-point batch in the standard batches of the <<spark-sql-Analyzer.adoc#, Analyzer>>.

`ExtractWindowExpressions` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

NOTE: `ExtractWindowExpressions` is a Scala object inside <<spark-sql-Analyzer.adoc#, Analyzer>> class (so you have to create an instance of the `Analyzer` class to access it or simply use <<spark-sql-SessionState.adoc#analyzer, SessionState>>).

[source, scala]
----
import spark.sessionState.analyzer.ExtractWindowExpressions

// Example 1: Filter + Aggregate with WindowExpressions in aggregateExprs
val q = ???
val plan = q.queryExecution.logical
val afterExtractWindowExpressions = ExtractWindowExpressions(plan)

// Example 2: Aggregate with WindowExpressions in aggregateExprs
val q = ???
val plan = q.queryExecution.logical
val afterExtractWindowExpressions = ExtractWindowExpressions(plan)

// Example 3: Project with WindowExpressions in projectList
val q = ???
val plan = q.queryExecution.logical
val afterExtractWindowExpressions = ExtractWindowExpressions(plan)
----

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply` <<spark-sql-catalyst-TreeNode.adoc#transformDown, transforms the logical operators downwards>> in the input <<spark-sql-LogicalPlan.adoc#, logical plan>> as follows:

* For <<spark-sql-LogicalPlan-Filter.adoc#, Filter>> unary operators with <<spark-sql-LogicalPlan-Aggregate.adoc#, Aggregate>> operator (as the <<spark-sql-LogicalPlan-Filter.adoc#child, child>>) that <<hasWindowFunction, has a window function>> in the <<spark-sql-LogicalPlan-Aggregate.adoc#aggregateExpressions, aggregateExpressions>>, `apply`...FIXME

* For <<spark-sql-LogicalPlan-Aggregate.adoc#, Aggregate>> logical operators that <<hasWindowFunction, have a window function>> in the <<spark-sql-LogicalPlan-Aggregate.adoc#aggregateExpressions, aggregateExpressions>>, `apply`...FIXME

* For <<spark-sql-LogicalPlan-Project.adoc#, Project>> logical operators that <<hasWindowFunction, have a window function>> in the <<spark-sql-LogicalPlan-Project.adoc#projectList, projectList>>, `apply`...FIXME

=== [[hasWindowFunction]] `hasWindowFunction` Internal Method

[source, scala]
----
hasWindowFunction(projectList: Seq[NamedExpression]): Boolean // <1>
hasWindowFunction(expr: NamedExpression): Boolean
----
<1> Executes the other `hasWindowFunction` on every `NamedExpression` in the `projectList`

`hasWindowFunction` is positive (`true`) when the input `expr` <<spark-sql-Expression-NamedExpression.adoc#, named expression>> is a <<spark-sql-Expression-WindowExpression.adoc#, WindowExpression>> expression. Otherwise, `hasWindowFunction` is negative (`false`).

NOTE: `hasWindowFunction` is used when `ExtractWindowExpressions` logical resolution rule is requested to <<extract, extract>> and <<apply, execute>>.

=== [[extract]] `extract` Internal Method

[source, scala]
----
extract(expressions: Seq[NamedExpression]): (Seq[NamedExpression], Seq[NamedExpression])
----

`extract`...FIXME

NOTE: `extract` is used exclusively when `ExtractWindowExpressions` logical resolution rule is <<apply, executed>>.

=== [[addWindow]] Adding Project and Window Logical Operators to Logical Plan -- `addWindow` Internal Method

[source, scala]
----
addWindow(
  expressionsWithWindowFunctions: Seq[NamedExpression],
  child: LogicalPlan): LogicalPlan
----

`addWindow` adds a <<spark-sql-LogicalPlan-Project.adoc#, Project>> logical operator with one or more <<spark-sql-LogicalPlan-Window.adoc#, Window>> logical operators (for every <<spark-sql-Expression-WindowExpression.adoc#, WindowExpression>> in the input <<spark-sql-Expression-NamedExpression.adoc#, named expressions>>) to the input <<spark-sql-LogicalPlan.adoc#, logical plan>>.

Internally, `addWindow`...FIXME

NOTE: `addWindow` is used exclusively when `ExtractWindowExpressions` logical resolution rule is <<apply, executed>>.
