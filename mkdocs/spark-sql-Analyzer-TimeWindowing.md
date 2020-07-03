# TimeWindowing Logical Resolution Rule

`TimeWindowing` is a <<spark-sql-Analyzer.adoc#batches, logical resolution rule>> that <<apply, FIXME>> in a logical query plan.

`TimeWindowing` is part of the <<spark-sql-Analyzer.adoc#Resolution, Resolution>> fixed-point batch in the standard batches of the <<spark-sql-Analyzer.adoc#, Analyzer>>.

`TimeWindowing` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

[source, scala]
----
// FIXME: DEMO
----

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply`...FIXME
