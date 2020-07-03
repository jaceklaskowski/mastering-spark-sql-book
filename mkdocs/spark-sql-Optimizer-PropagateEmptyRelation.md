# PropagateEmptyRelation Logical Optimization

`PropagateEmptyRelation` is a <<spark-sql-Optimizer.adoc#batches, base logical optimization>> that <<apply, collapses plans with empty LocalRelation logical operators>>, e.g. <<explode, explode>> or <<join, join>>.

`PropagateEmptyRelation` is part of the <<spark-sql-Optimizer.adoc#LocalRelation, LocalRelation>> fixed-point batch in the standard batches of the <<spark-sql-Optimizer.adoc#, Catalyst Optimizer>>.

`PropagateEmptyRelation` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

=== [[explode]] Explode

[source, scala]
----
scala> val emp = spark.emptyDataset[Seq[String]]
emp: org.apache.spark.sql.Dataset[Seq[String]] = [value: array<string>]

scala> emp.select(explode($"value")).show
+---+
|col|
+---+
+---+

scala> emp.select(explode($"value")).explain(true)
== Parsed Logical Plan ==
'Project [explode('value) AS List()]
+- LocalRelation <empty>, [value#77]

== Analyzed Logical Plan ==
col: string
Project [col#89]
+- Generate explode(value#77), false, false, [col#89]
   +- LocalRelation <empty>, [value#77]

== Optimized Logical Plan ==
LocalRelation <empty>, [col#89]

== Physical Plan ==
LocalTableScan <empty>, [col#89]
----

=== [[join]] Join

[source, scala]
----
scala> spark.emptyDataset[Int].join(spark.range(1)).explain(extended = true)
...
TRACE SparkOptimizer:
=== Applying Rule org.apache.spark.sql.catalyst.optimizer.PropagateEmptyRelation ===
!Join Inner                                LocalRelation <empty>, [value#40, id#42L]
!:- LocalRelation <empty>, [value#40]
!+- Range (0, 1, step=1, splits=Some(8))

TRACE SparkOptimizer: Fixed point reached for batch LocalRelation after 2 iterations.
DEBUG SparkOptimizer:
=== Result of Batch LocalRelation ===
!Join Inner                                LocalRelation <empty>, [value#40, id#42L]
!:- LocalRelation <empty>, [value#40]
!+- Range (0, 1, step=1, splits=Some(8))
...
== Parsed Logical Plan ==
Join Inner
:- LocalRelation <empty>, [value#40]
+- Range (0, 1, step=1, splits=Some(8))

== Analyzed Logical Plan ==
value: int, id: bigint
Join Inner
:- LocalRelation <empty>, [value#40]
+- Range (0, 1, step=1, splits=Some(8))

== Optimized Logical Plan ==
LocalRelation <empty>, [value#40, id#42L]

== Physical Plan ==
LocalTableScan <empty>, [value#40, id#42L]
----

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply`...FIXME
