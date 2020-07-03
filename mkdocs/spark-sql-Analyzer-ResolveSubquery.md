# ResolveSubquery Logical Resolution Rule

`ResolveSubquery` is a *logical resolution* that <<resolveSubQueries, resolves subquery expressions>> (<<spark-sql-Expression-SubqueryExpression-ScalarSubquery.adoc#, ScalarSubquery>>, <<spark-sql-Expression-Exists.adoc#, Exists>> and <<spark-sql-Expression-In.adoc#, In>>) when <<apply, transforming a logical plan>> with the following logical operators:

. `Filter` operators with an `Aggregate` child operator

. Unary operators with the children resolved

`ResolveSubquery` is part of link:spark-sql-Analyzer.adoc#Resolution[Resolution] fixed-point batch of rules of the link:spark-sql-Analyzer.adoc[Spark Analyzer].

Technically, `ResolveSubquery` is a link:spark-sql-catalyst-Rule.adoc[Catalyst rule] for transforming link:spark-sql-LogicalPlan.adoc[logical plans], i.e. `Rule[LogicalPlan]`.

[source, scala]
----
// Use Catalyst DSL
import org.apache.spark.sql.catalyst.expressions._
val a = 'a.int

import org.apache.spark.sql.catalyst.plans.logical.LocalRelation
val rel = LocalRelation(a)

import org.apache.spark.sql.catalyst.expressions.Literal
val list = Seq[Literal](1)

// FIXME Use a correct query to demo ResolveSubquery
import org.apache.spark.sql.catalyst.plans.logical.Filter
import org.apache.spark.sql.catalyst.expressions.In
val plan = Filter(condition = In(value = a, list), child = rel)

scala> println(plan.numberedTreeString)
00 Filter a#9 IN (1)
01 +- LocalRelation <empty>, [a#9]

import spark.sessionState.analyzer.ResolveSubquery
val analyzedPlan = ResolveSubquery(plan)
scala> println(analyzedPlan.numberedTreeString)
00 Filter a#9 IN (1)
01 +- LocalRelation <empty>, [a#9]
----

=== [[resolveSubQueries]] Resolving Subquery Expressions (ScalarSubquery, Exists and In) -- `resolveSubQueries` Internal Method

[source, scala]
----
resolveSubQueries(plan: LogicalPlan, plans: Seq[LogicalPlan]): LogicalPlan
----

`resolveSubQueries` requests the input link:spark-sql-LogicalPlan.adoc[logical plan] to link:spark-sql-catalyst-QueryPlan.adoc#transformExpressions[transform expressions] (down the operator tree) as follows:

. For link:spark-sql-Expression-ExecSubqueryExpression-ScalarSubquery.adoc[ScalarSubquery] expressions with link:spark-sql-Expression-ExecSubqueryExpression-ScalarSubquery.adoc#plan[subquery plan] not link:spark-sql-LogicalPlan.adoc#resolved[resolved] and <<resolveSubQuery, resolveSubQuery>> to create resolved `ScalarSubquery` expressions

. For link:spark-sql-Expression-Exists.adoc[Exists] expressions with link:spark-sql-Expression-Exists.adoc#plan[subquery plan] not link:spark-sql-LogicalPlan.adoc#resolved[resolved] and <<resolveSubQuery, resolveSubQuery>> to create resolved `Exists` expressions

. For link:spark-sql-Expression-In.adoc[In] expressions with link:spark-sql-Expression-ListQuery.adoc[ListQuery] not link:spark-sql-Expression-ListQuery.adoc#resolved[resolved] and <<resolveSubQuery, resolveSubQuery>> to create resolved `In` expressions

NOTE: `resolveSubQueries` is used exclusively when `ResolveSubquery` is <<apply, executed>>.

=== [[resolveSubQuery]] `resolveSubQuery` Internal Method

[source, scala]
----
resolveSubQuery(
  e: SubqueryExpression,
  plans: Seq[LogicalPlan])(
  f: (LogicalPlan, Seq[Expression]) => SubqueryExpression): SubqueryExpression
----

`resolveSubQuery`...FIXME

NOTE: `resolveSubQuery` is used exclusively when `ResolveSubquery` is requested to <<resolveSubQueries, resolve subquery expressions (ScalarSubquery, Exists and In)>>.

=== [[apply]] Applying ResolveSubquery to Logical Plan (Executing ResolveSubquery) -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of link:spark-sql-catalyst-Rule.adoc#apply[Rule Contract] to apply a rule to a link:spark-sql-catalyst-TreeNode.adoc[TreeNode], e.g. link:spark-sql-LogicalPlan.adoc[logical plan].

`apply` transforms the input link:spark-sql-LogicalPlan.adoc[logical plan] as follows:

. For link:spark-sql-LogicalPlan-Filter.adoc[Filter] operators with an link:spark-sql-LogicalPlan-Aggregate.adoc[Aggregate] operator (as the link:spark-sql-LogicalPlan-Filter.adoc#child[child] operator) and the link:spark-sql-LogicalPlan.adoc#childrenResolved[children resolved], `apply` <<resolveSubQueries, resolves subquery expressions (ScalarSubquery, Exists and In)>> with the `Filter` operator and the plans with the `Aggregate` operator and its single link:spark-sql-LogicalPlan-Aggregate.adoc#child[child]

. For link:spark-sql-LogicalPlan.adoc#UnaryNode[unary operators] with the link:spark-sql-LogicalPlan.adoc#childrenResolved[children resolved], `apply` <<resolveSubQueries, resolves subquery expressions (ScalarSubquery, Exists and In)>> with the unary operator and its single child
