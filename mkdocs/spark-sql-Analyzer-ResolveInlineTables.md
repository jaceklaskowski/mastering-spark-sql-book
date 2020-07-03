# ResolveInlineTables Logical Resolution Rule

`ResolveInlineTables` is a <<spark-sql-Analyzer.adoc#batches, logical resolution rule>> that <<apply, resolves (replaces) UnresolvedInlineTable operators to LocalRelations>> in a logical query plan.

`ResolveInlineTables` is part of the <<spark-sql-Analyzer.adoc#Resolution, Resolution>> fixed-point batch in the standard batches of the <<spark-sql-Analyzer.adoc#, Analyzer>>.

`ResolveInlineTables` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

[[conf]]
[[creating-instance]]
`ResolveInlineTables` takes a <<spark-sql-SQLConf.adoc#, SQLConf>> when created.

[source, scala]
----
val q = sql("VALUES 1, 2, 3")
val plan = q.queryExecution.logical
scala> println(plan.numberedTreeString)
00 'UnresolvedInlineTable [col1], [List(1), List(2), List(3)]

scala> :type spark
org.apache.spark.sql.SparkSession

scala> :type spark.sessionState.conf
org.apache.spark.sql.internal.SQLConf

import org.apache.spark.sql.catalyst.analysis.ResolveInlineTables
val rule = ResolveInlineTables(spark.sessionState.conf)

val planAfterResolveInlineTables = rule(plan)
scala> println(planAfterResolveInlineTables.numberedTreeString)
00 LocalRelation [col1#2]
----

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply` simply <<spark-sql-catalyst-TreeNode.adoc#transformUp, searches the input plan up>> to find <<spark-sql-LogicalPlan-UnresolvedInlineTable.adoc#, UnresolvedInlineTable>> logical operators with <<spark-sql-LogicalPlan-UnresolvedInlineTable.adoc#expressionsResolved, rows expressions resolved>>.

For such a <<spark-sql-LogicalPlan-UnresolvedInlineTable.adoc#, UnresolvedInlineTable>> logical operator, `apply` <<validateInputDimension, validateInputDimension>> and <<validateInputEvaluable, validateInputEvaluable>>.

In the end, `apply` <<convert, converts the UnresolvedInlineTable to a LocalRelation>>.

=== [[validateInputDimension]] `validateInputDimension` Internal Method

[source, scala]
----
validateInputDimension(table: UnresolvedInlineTable): Unit
----

`validateInputDimension`...FIXME

NOTE: `validateInputDimension` is used exclusively when `ResolveInlineTables` logical resolution rule is <<apply, executed>>.

=== [[validateInputEvaluable]] `validateInputEvaluable` Internal Method

[source, scala]
----
validateInputEvaluable(table: UnresolvedInlineTable): Unit
----

`validateInputEvaluable`...FIXME

NOTE: `validateInputEvaluable` is used exclusively when `ResolveInlineTables` logical resolution rule is <<apply, executed>>.

=== [[convert]] Converting UnresolvedInlineTable to LocalRelation -- `convert` Internal Method

[source, scala]
----
convert(table: UnresolvedInlineTable): LocalRelation
----

`convert`...FIXME

NOTE: `convert` is used exclusively when `ResolveInlineTables` logical resolution rule is <<apply, executed>>.
