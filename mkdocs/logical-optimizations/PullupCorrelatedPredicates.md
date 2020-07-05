# PullupCorrelatedPredicates Logical Optimization

`PullupCorrelatedPredicates` is a <<spark-sql-Optimizer.adoc#batches, base logical optimization>> that <<apply, transforms logical plans>> with the following operators:

. link:spark-sql-LogicalPlan-Filter.adoc[Filter] operators with an link:spark-sql-LogicalPlan-Aggregate.adoc[Aggregate] child operator

. link:spark-sql-LogicalPlan.adoc#UnaryNode[UnaryNode] operators

`PullupCorrelatedPredicates` is part of the <<spark-sql-Optimizer.adoc#Pullup-Correlated-Expressions, Pullup Correlated Expressions>> once-executed batch in the standard batches of the <<spark-sql-Optimizer.adoc#, Catalyst Optimizer>>.

`PullupCorrelatedPredicates` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

[source, scala]
----
import org.apache.spark.sql.catalyst.optimizer.PullupCorrelatedPredicates

// FIXME
// Demo: Filter + Aggregate
// Demo: Filter + UnaryNode

val plan = ???
val optimizedPlan = PullupCorrelatedPredicates(plan)
----

`PullupCorrelatedPredicates` uses link:spark-sql-PredicateHelper.adoc[PredicateHelper] for...FIXME

=== [[pullOutCorrelatedPredicates]] `pullOutCorrelatedPredicates` Internal Method

[source, scala]
----
pullOutCorrelatedPredicates(
  sub: LogicalPlan,
  outer: Seq[LogicalPlan]): (LogicalPlan, Seq[Expression])
----

`pullOutCorrelatedPredicates`...FIXME

NOTE: `pullOutCorrelatedPredicates` is used exclusively when `PullupCorrelatedPredicates` is requested to <<rewriteSubQueries, rewriteSubQueries>>.

=== [[rewriteSubQueries]] `rewriteSubQueries` Internal Method

[source, scala]
----
rewriteSubQueries(plan: LogicalPlan, outerPlans: Seq[LogicalPlan]): LogicalPlan
----

`rewriteSubQueries`...FIXME

NOTE: `rewriteSubQueries` is used exclusively when `PullupCorrelatedPredicates` is <<apply, executed>> (i.e. applied to a link:spark-sql-LogicalPlan.adoc[logical plan]).

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply` transforms the input link:spark-sql-LogicalPlan.adoc[logical plan] as follows:

. For link:spark-sql-LogicalPlan-Filter.adoc[Filter] operators with an link:spark-sql-LogicalPlan-Aggregate.adoc[Aggregate] child operator, `apply` <<rewriteSubQueries, rewriteSubQueries>> with the `Filter` and the `Aggregate` and its link:spark-sql-LogicalPlan-Aggregate.adoc#child[child] as the outer plans

. For link:spark-sql-LogicalPlan.adoc#UnaryNode[UnaryNode] operators, `apply` <<rewriteSubQueries, rewriteSubQueries>> with the operator and its children as the outer plans
