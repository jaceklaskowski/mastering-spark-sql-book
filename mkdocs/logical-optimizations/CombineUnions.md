# CombineUnions Logical Optimization

`CombineUnions` is a <<spark-sql-Optimizer.adoc#batches, base logical optimization>> that <<apply, FIXME>>.

`CombineUnions` is part of the <<spark-sql-Optimizer.adoc#Union, Union>> once-executed batch in the standard batches of the <<spark-sql-Optimizer.adoc#, Catalyst Optimizer>>.

`CombineUnions` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

[source, scala]
----
// FIXME Demo
----

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply`...FIXME
