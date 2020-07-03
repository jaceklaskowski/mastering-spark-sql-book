# LimitPushDown Logical Optimization

`LimitPushDown` is a <<spark-sql-Optimizer.adoc#batches, base logical optimization>> that <<apply, transforms>> the following logical plans:

* `LocalLimit` with `Union`
* `LocalLimit` with link:spark-sql-LogicalPlan-Join.adoc[Join]

`LimitPushDown` is part of the <<spark-sql-Optimizer.adoc#Operator_Optimization_before_Inferring_Filters, Operator Optimization before Inferring Filters>> fixed-point batch in the standard batches of the <<spark-sql-Optimizer.adoc#, Catalyst Optimizer>>.

`LimitPushDown` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

[source, scala]
----
// test datasets
scala> val ds1 = spark.range(4)
ds1: org.apache.spark.sql.Dataset[Long] = [value: bigint]

scala> val ds2 = spark.range(2)
ds2: org.apache.spark.sql.Dataset[Long] = [value: bigint]

// Case 1. Rather than `LocalLimit` of `Union` do `Union` of `LocalLimit`
scala> ds1.union(ds2).limit(2).explain(true)
== Parsed Logical Plan ==
GlobalLimit 2
+- LocalLimit 2
   +- Union
      :- Range (0, 4, step=1, splits=Some(8))
      +- Range (0, 2, step=1, splits=Some(8))

== Analyzed Logical Plan ==
id: bigint
GlobalLimit 2
+- LocalLimit 2
   +- Union
      :- Range (0, 4, step=1, splits=Some(8))
      +- Range (0, 2, step=1, splits=Some(8))

== Optimized Logical Plan ==
GlobalLimit 2
+- LocalLimit 2
   +- Union
      :- LocalLimit 2
      :  +- Range (0, 4, step=1, splits=Some(8))
      +- LocalLimit 2
         +- Range (0, 2, step=1, splits=Some(8))

== Physical Plan ==
CollectLimit 2
+- Union
   :- *LocalLimit 2
   :  +- *Range (0, 4, step=1, splits=Some(8))
   +- *LocalLimit 2
      +- *Range (0, 2, step=1, splits=Some(8))
----

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply`...FIXME

=== [[creating-instance]] Creating LimitPushDown Instance

`LimitPushDown` takes the following when created:

* [[conf]] link:spark-sql-CatalystConf.adoc[CatalystConf]

`LimitPushDown` initializes the <<internal-registries, internal registries and counters>>.

NOTE: `LimitPushDown` is created when
