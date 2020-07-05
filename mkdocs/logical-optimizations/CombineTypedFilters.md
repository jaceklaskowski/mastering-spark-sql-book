# CombineTypedFilters Logical Optimization

`CombineTypedFilters` is a <<spark-sql-Optimizer.adoc#batches, base logical optimization>> that <<apply, combines two back to back (typed) filters into one>> that ultimately ends up as a single method call.

`CombineTypedFilters` is part of the <<spark-sql-Optimizer.adoc#Object_Expressions_Optimization, Object Expressions Optimization>> fixed-point batch in the standard batches of the <<spark-sql-Optimizer.adoc#, Catalyst Optimizer>>.

`CombineTypedFilters` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

[source, scala]
----
scala> :type spark
org.apache.spark.sql.SparkSession

// A query with two consecutive typed filters
val q = spark.range(10).filter(_ % 2 == 0).filter(_ == 0)
scala> q.queryExecution.optimizedPlan
...
TRACE SparkOptimizer:
=== Applying Rule org.apache.spark.sql.catalyst.optimizer.CombineTypedFilters ===
 TypedFilter <function1>, class java.lang.Long, [StructField(value,LongType,true)], newInstance(class java.lang.Long)      TypedFilter <function1>, class java.lang.Long, [StructField(value,LongType,true)], newInstance(class java.lang.Long)
!+- TypedFilter <function1>, class java.lang.Long, [StructField(value,LongType,true)], newInstance(class java.lang.Long)   +- Range (0, 10, step=1, splits=Some(8))
!   +- Range (0, 10, step=1, splits=Some(8))

TRACE SparkOptimizer: Fixed point reached for batch Typed Filter Optimization after 2 iterations.
DEBUG SparkOptimizer:
=== Result of Batch Typed Filter Optimization ===
 TypedFilter <function1>, class java.lang.Long, [StructField(value,LongType,true)], newInstance(class java.lang.Long)      TypedFilter <function1>, class java.lang.Long, [StructField(value,LongType,true)], newInstance(class java.lang.Long)
!+- TypedFilter <function1>, class java.lang.Long, [StructField(value,LongType,true)], newInstance(class java.lang.Long)   +- Range (0, 10, step=1, splits=Some(8))
!   +- Range (0, 10, step=1, splits=Some(8))
...
----

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply`...FIXME
