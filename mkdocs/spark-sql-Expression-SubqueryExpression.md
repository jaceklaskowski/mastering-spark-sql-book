title: SubqueryExpression

# SubqueryExpression -- Expressions With Logical Query Plans

`SubqueryExpression` is the <<contract, contract>> for link:spark-sql-Expression-PlanExpression.adoc[expressions with logical query plans] (i.e. `PlanExpression[LogicalPlan]`).

[[contract]]
[source, scala]
----
package org.apache.spark.sql.catalyst.expressions

abstract class SubqueryExpression(
    plan: LogicalPlan,
    children: Seq[Expression],
    exprId: ExprId) extends PlanExpression[LogicalPlan] {
  // only required methods that have no implementation
  // the others follow
  override def withNewPlan(plan: LogicalPlan): SubqueryExpression
}
----

.(Subset of) SubqueryExpression Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| [[withNewPlan]] `withNewPlan`
a| Used when:

* `CTESubstitution` substitution analyzer rule is requested to `substituteCTE`

* `ResolveReferences` logical resolution rule is requested to link:spark-sql-Analyzer-ResolveReferences.adoc#dedupRight[dedupRight] and link:spark-sql-Analyzer-ResolveReferences.adoc#dedupOuterReferencesInSubquery[dedupOuterReferencesInSubquery]

* `ResolveSubquery` logical resolution rule is requested to link:spark-sql-Analyzer-ResolveSubquery.adoc#resolveSubQuery[resolveSubQuery]

* `UpdateOuterReferences` logical rule is link:spark-sql-Analyzer-UpdateOuterReferences.adoc#apply[executed]

* `ResolveTimeZone` logical resolution rule is link:spark-sql-ResolveTimeZone.adoc#apply[executed]

* `SubqueryExpression` is requested for a <<canonicalize, canonicalized version>>

* `OptimizeSubqueries` logical query optimization is link:spark-sql-Optimizer-OptimizeSubqueries.adoc#apply[executed]

* `CacheManager` is requested to link:spark-sql-CacheManager.adoc#useCachedData[replace logical query segments with cached query plans]
|===

[[implementations]]
.SubqueryExpressions
[cols="1,2",options="header",width="100%"]
|===
| SubqueryExpression
| Description

| [[Exists]] link:spark-sql-Expression-Exists.adoc[Exists]
|

| [[ListQuery]] link:spark-sql-Expression-ListQuery.adoc[ListQuery]
|

| [[ScalarSubquery]] link:spark-sql-Expression-ExecSubqueryExpression-ScalarSubquery.adoc[ScalarSubquery]
|
|===

[[resolved]]
`SubqueryExpression` is link:spark-sql-Expression.adoc#resolved[resolved] when the link:spark-sql-Expression.adoc#childrenResolved[children are resolved] and the <<plan, subquery logical plan>> is link:spark-sql-LogicalPlan.adoc#resolved[resolved].

[[references]]
`references`...FIXME

[[semanticEquals]]
`semanticEquals`...FIXME

[[canonicalize]]
`canonicalize`...FIXME

=== [[hasInOrExistsSubquery]] `hasInOrExistsSubquery` Object Method

[source, scala]
----
hasInOrExistsSubquery(e: Expression): Boolean
----

`hasInOrExistsSubquery`...FIXME

NOTE: `hasInOrExistsSubquery` is used when...FIXME

=== [[hasCorrelatedSubquery]] `hasCorrelatedSubquery` Object Method

[source, scala]
----
hasCorrelatedSubquery(e: Expression): Boolean
----

`hasCorrelatedSubquery`...FIXME

NOTE: `hasCorrelatedSubquery` is used when...FIXME

=== [[hasSubquery]] `hasSubquery` Utility

[source, scala]
----
hasSubquery(
  e: Expression): Boolean
----

`hasSubquery`...FIXME

NOTE: `hasSubquery` is used when...FIXME

=== [[creating-instance]] Creating SubqueryExpression Instance

`SubqueryExpression` takes the following when created:

* [[plan]] Subquery link:spark-sql-LogicalPlan.adoc[logical plan]
* [[children]] Child link:spark-sql-Expression.adoc[expressions]
* [[exprId]] Expression ID (as `ExprId`)
