# RewritePredicateSubquery Logical Optimization

`RewritePredicateSubquery` is a <<spark-sql-Optimizer.adoc#batches, base logical optimization>> that <<apply, transforms Filter operators with Exists and In (with ListQuery) expressions to Join operators>> as follows:

* `Filter` operators with `Exists` and `In` with `ListQuery` expressions give *left-semi joins*

* `Filter` operators with `Not` with `Exists` and `In` with `ListQuery` expressions give *left-anti joins*

NOTE: Prefer `EXISTS` (over `Not` with `In` with `ListQuery` subquery expression) if performance matters since https://github.com/apache/spark/blob/master/sql/catalyst/src/main/scala/org/apache/spark/sql/catalyst/optimizer/subquery.scala?utf8=%E2%9C%93#L110[they say] "that will almost certainly be planned as a Broadcast Nested Loop join".

`RewritePredicateSubquery` is part of the <<spark-sql-Optimizer.adoc#RewriteSubquery, RewriteSubquery>> once-executed batch in the standard batches of the <<spark-sql-Optimizer.adoc#, Catalyst Optimizer>>.

`RewritePredicateSubquery` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

[source, scala]
----
// FIXME Examples of RewritePredicateSubquery
// 1. Filters with Exists and In (with ListQuery) expressions
// 2. NOTs

// Based on RewriteSubquerySuite
// FIXME Contribute back to RewriteSubquerySuite
import org.apache.spark.sql.catalyst.plans.logical.LogicalPlan
import org.apache.spark.sql.catalyst.rules.RuleExecutor
object Optimize extends RuleExecutor[LogicalPlan] {
  import org.apache.spark.sql.catalyst.optimizer._
  val batches = Seq(
    Batch("Column Pruning", FixedPoint(100), ColumnPruning),
    Batch("Rewrite Subquery", Once,
      RewritePredicateSubquery,
      ColumnPruning,
      CollapseProject,
      RemoveRedundantProject))
}

val q = ...
val optimized = Optimize.execute(q.analyze)
----

`RewritePredicateSubquery` is part of the link:spark-sql-Optimizer.adoc#RewriteSubquery[RewriteSubquery] once-executed batch in the standard batches of the link:spark-sql-Optimizer.adoc[Catalyst Optimizer].

=== [[rewriteExistentialExpr]] `rewriteExistentialExpr` Internal Method

[source, scala]
----
rewriteExistentialExpr(
  exprs: Seq[Expression],
  plan: LogicalPlan): (Option[Expression], LogicalPlan)
----

`rewriteExistentialExpr`...FIXME

NOTE: `rewriteExistentialExpr` is used when...FIXME

=== [[dedupJoin]] `dedupJoin` Internal Method

[source, scala]
----
dedupJoin(joinPlan: LogicalPlan): LogicalPlan
----

`dedupJoin`...FIXME

NOTE: `dedupJoin` is used when...FIXME

=== [[getValueExpression]] `getValueExpression` Internal Method

[source, scala]
----
getValueExpression(e: Expression): Seq[Expression]
----

`getValueExpression`...FIXME

NOTE: `getValueExpression` is used when...FIXME

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply` transforms link:spark-sql-LogicalPlan-Filter.adoc[Filter] unary operators in the input link:spark-sql-LogicalPlan.adoc[logical plan].

`apply` link:spark-sql-PredicateHelper.adoc#splitConjunctivePredicates[splits conjunctive predicates] in the link:spark-sql-LogicalPlan-Filter.adoc#condition[condition expression] (i.e. expressions separated by `And` expression) and then partitions them into two collections of expressions link:spark-sql-Expression-SubqueryExpression.adoc#hasInOrExistsSubquery[with and without In or Exists subquery expressions].

`apply` creates a link:spark-sql-LogicalPlan-Filter.adoc#creating-instance[Filter] operator for condition (sub)expressions without subqueries (combined with `And` expression) if available or takes the link:spark-sql-LogicalPlan-Filter.adoc#child[child] operator (of the input `Filter` unary operator).

In the end, `apply` creates a new logical plan with link:spark-sql-LogicalPlan-Join.adoc[Join] operators for link:spark-sql-Expression-Exists.adoc[Exists] and link:spark-sql-Expression-In.adoc[In] expressions (and their negations) as follows:

* For link:spark-sql-Expression-Exists.adoc[Exists] predicate expressions, `apply` <<rewriteExistentialExpr, rewriteExistentialExpr>> and creates a link:spark-sql-LogicalPlan-Join.adoc#creating-instance[Join] operator with link:spark-sql-joins.adoc#LeftSemi[LeftSemi] join type. In the end, `apply` <<dedupJoin, dedupJoin>>

* For `Not` expressions with a link:spark-sql-Expression-Exists.adoc[Exists] predicate expression, `apply` <<rewriteExistentialExpr, rewriteExistentialExpr>> and creates a link:spark-sql-LogicalPlan-Join.adoc#creating-instance[Join] operator with link:spark-sql-joins.adoc#LeftAnti[LeftAnti] join type. In the end, `apply` <<dedupJoin, dedupJoin>>

* For link:spark-sql-Expression-In.adoc[In] predicate expressions with a link:spark-sql-Expression-ListQuery.adoc[ListQuery] subquery expression, `apply` <<getValueExpression, getValueExpression>> followed by <<rewriteExistentialExpr, rewriteExistentialExpr>> and creates a link:spark-sql-LogicalPlan-Join.adoc#creating-instance[Join] operator with link:spark-sql-joins.adoc#LeftSemi[LeftSemi] join type. In the end, `apply` <<dedupJoin, dedupJoin>>

* For `Not` expressions with a link:spark-sql-Expression-In.adoc[In] predicate expression with a link:spark-sql-Expression-ListQuery.adoc[ListQuery] subquery expression, `apply` <<getValueExpression, getValueExpression>>, <<rewriteExistentialExpr, rewriteExistentialExpr>> followed by link:spark-sql-PredicateHelper.adoc#splitConjunctivePredicates[splitting conjunctive predicates] and creates a link:spark-sql-LogicalPlan-Join.adoc#creating-instance[Join] operator with link:spark-sql-joins.adoc#LeftAnti[LeftAnti] join type. In the end, `apply` <<dedupJoin, dedupJoin>>

* For other predicate expressions, `apply` <<rewriteExistentialExpr, rewriteExistentialExpr>> and creates a link:spark-sql-LogicalPlan-Project.adoc#creating-instance[Project] unary operator with a link:spark-sql-LogicalPlan-Filter.adoc#creating-instance[Filter] operator
