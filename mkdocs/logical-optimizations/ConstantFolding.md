# ConstantFolding Logical Optimization

`ConstantFolding` is a <<spark-sql-Optimizer.adoc#batches, base logical optimization>> that <<apply, replaces expressions that can be statically evaluated with their equivalent literal values>>.

`ConstantFolding` is part of the <<spark-sql-Optimizer.adoc#Operator_Optimization_before_Inferring_Filters, Operator Optimization before Inferring Filters>> fixed-point batch in the standard batches of the <<spark-sql-Optimizer.adoc#, Catalyst Optimizer>>.

`ConstantFolding` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

[source, scala]
----
scala> spark.range(1).select(lit(3) > 2).explain(true)
...
TRACE SparkOptimizer:
=== Applying Rule org.apache.spark.sql.catalyst.optimizer.ConstantFolding ===
!Project [(3 > 2) AS (3 > 2)#3]            Project [true AS (3 > 2)#3]
 +- Range (0, 1, step=1, splits=Some(8))   +- Range (0, 1, step=1, splits=Some(8))

scala> spark.range(1).select('id + 'id > 0).explain(true)
...
TRACE SparkOptimizer:
=== Applying Rule org.apache.spark.sql.catalyst.optimizer.ConstantFolding ===
!Project [((id#7L + id#7L) > cast(0 as bigint)) AS ((id + id) > 0)#10]   Project [((id#7L + id#7L) > 0) AS ((id + id) > 0)#10]
 +- Range (0, 1, step=1, splits=Some(8))                                 +- Range (0, 1, step=1, splits=Some(8))
----

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply`...FIXME
