# ResolveOrdinalInOrderByAndGroupBy Logical Resolution Rule

`ResolveOrdinalInOrderByAndGroupBy` is a <<spark-sql-Analyzer.adoc#batches, logical resolution rule>> that <<apply, converts ordinal positions in Sort and Aggregate logical operators with corresponding expressions>>  in a logical query plan.

`ResolveOrdinalInOrderByAndGroupBy` is part of the <<spark-sql-Analyzer.adoc#Resolution, Resolution>> fixed-point batch in the standard batches of the <<spark-sql-Analyzer.adoc#, Analyzer>>.

`ResolveOrdinalInOrderByAndGroupBy` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

[[creating-instance]]
`ResolveOrdinalInOrderByAndGroupBy` takes no arguments when created.

[source, scala]
----
// FIXME: DEMO
val rule = spark.sessionState.analyzer.ResolveOrdinalInOrderByAndGroupBy

val plan = ???
val planResolved = rule(plan)
scala> println(planResolved.numberedTreeString)
00 'UnresolvedRelation `t1`
----

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply` <<spark-sql-catalyst-TreeNode.adoc#transformUp, walks the logical plan from children up the tree>> and looks for <<spark-sql-LogicalPlan-Sort.adoc#, Sort>> and <<spark-sql-LogicalPlan-Aggregate.adoc#, Aggregate>> logical operators with <<spark-sql-Expression-UnresolvedOrdinal.adoc#, UnresolvedOrdinal>> leaf expressions (in <<spark-sql-LogicalPlan-Sort.adoc#order, ordering>> and <<spark-sql-LogicalPlan-Aggregate.adoc#groupingExpressions, grouping>> expressions, respectively).

For a <<spark-sql-LogicalPlan-Sort.adoc#, Sort>> logical operator with <<spark-sql-Expression-UnresolvedOrdinal.adoc#, UnresolvedOrdinal>> expressions, `apply` replaces all the <<spark-sql-Expression-SortOrder.adoc#, SortOrder>> expressions (with <<spark-sql-Expression-UnresolvedOrdinal.adoc#, UnresolvedOrdinal>> child expressions) with `SortOrder` expressions and the expression at the `index - 1` position in the <<spark-sql-catalyst-QueryPlan.adoc#output, output schema>> of the <<spark-sql-LogicalPlan-Sort.adoc#child, child>> logical operator.

For a <<spark-sql-LogicalPlan-Aggregate.adoc#, Aggregate>> logical operator with <<spark-sql-Expression-UnresolvedOrdinal.adoc#, UnresolvedOrdinal>> expressions, `apply` replaces all the expressions (with <<spark-sql-Expression-UnresolvedOrdinal.adoc#, UnresolvedOrdinal>> child expressions) with the expression at the `index - 1` position in the <<spark-sql-LogicalPlan-Aggregate.adoc#aggregateExpressions, aggregate named expressions>> of the current `Aggregate` logical operator.

`apply` throws a `AnalysisException` (and hence fails an analysis) if the ordinal is outside the range:

```
ORDER BY position [index] is not in select list (valid range is [1, [output.size]])
GROUP BY position [index] is not in select list (valid range is [1, [aggs.size]])
```
