title: Subqueries

# Subqueries (Subquery Expressions)

As of Spark 2.0, Spark SQL supports subqueries.

A *subquery* (aka *subquery expression*) is a query that is nested inside of another query.

There are the following kinds of subqueries:

. A subquery as a source (inside a SQL `FROM` clause)
. A scalar subquery or a predicate subquery (as a column)

Every subquery can also be *correlated* or *uncorrelated*.

[[scalar-subquery]]
A *scalar subquery* is a structured query that returns a single row and a single column only. Spark SQL uses link:spark-sql-Expression-SubqueryExpression-ScalarSubquery.adoc[ScalarSubquery (SubqueryExpression)] expression to represent scalar subqueries (while link:spark-sql-AstBuilder.adoc#visitSubqueryExpression[parsing a SQL statement]).

[source, scala]
----
// FIXME: ScalarSubquery in a logical plan
----

A `ScalarSubquery` expression appears as *scalar-subquery#[exprId] [conditionString]* in a logical plan.

[source, scala]
----
// FIXME: Name of a ScalarSubquery in a logical plan
----

It is said that scalar subqueries should be used very rarely if at all and you should join instead.

Spark Analyzer uses link:spark-sql-Analyzer-ResolveSubquery.adoc[ResolveSubquery] resolution rule to link:spark-sql-Analyzer-ResolveSubquery.adoc#resolveSubQueries[resolve subqueries] and at the end link:spark-sql-Analyzer-CheckAnalysis.adoc#checkSubqueryExpression[makes sure that they are valid].

Catalyst Optimizer uses the following optimizations for subqueries:

* link:spark-sql-Optimizer-PullupCorrelatedPredicates.adoc[PullupCorrelatedPredicates] optimization to link:spark-sql-Optimizer-PullupCorrelatedPredicates.adoc#rewriteSubQueries[rewrite subqueries] and pull up correlated predicates

* link:spark-sql-Optimizer-RewriteCorrelatedScalarSubquery.adoc[RewriteCorrelatedScalarSubquery] optimization to link:spark-sql-Optimizer-RewriteCorrelatedScalarSubquery.adoc#constructLeftJoins[constructLeftJoins]

Spark Physical Optimizer uses link:spark-sql-PlanSubqueries.adoc[PlanSubqueries] physical optimization to link:spark-sql-PlanSubqueries.adoc#apply[plan queries with scalar subqueries].

CAUTION: FIXME Describe how a physical link:spark-sql-Expression-ExecSubqueryExpression-ScalarSubquery.adoc[ScalarSubquery] is executed (cf. `updateResult`, `eval` and `doGenCode`).
