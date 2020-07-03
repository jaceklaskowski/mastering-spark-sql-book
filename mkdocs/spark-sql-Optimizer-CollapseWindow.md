# CollapseWindow Logical Optimization

`CollapseWindow` is a <<spark-sql-Optimizer.adoc#batches, base logical optimization>> that <<apply, FIXME>>.

`CollapseWindow` is part of the <<spark-sql-Optimizer.adoc#Operator_Optimization, Operator Optimization>> fixed-point batch in the standard batches of the <<spark-sql-Optimizer.adoc#, Catalyst Optimizer>>.

`CollapseWindow` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

[source, scala]
----
// FIXME: DEMO
import org.apache.spark.sql.catalyst.optimizer.CollapseWindow

val logicalPlan = ???
val afterCollapseWindow = CollapseWindow(logicalPlan)
----

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply`...FIXME
