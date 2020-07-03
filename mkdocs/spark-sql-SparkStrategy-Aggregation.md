# Aggregation Execution Planning Strategy for Aggregate Physical Operators

`Aggregation` is an link:spark-sql-SparkStrategy.adoc[execution planning strategy] that link:spark-sql-SparkPlanner.adoc[SparkPlanner] uses to <<apply, select aggregate physical operator>> for <<spark-sql-LogicalPlan-Aggregate.adoc#, Aggregate>> logical operator in a <<spark-sql-LogicalPlan.adoc#, logical query plan>>.

[source, scala]
----
scala> :type spark
org.apache.spark.sql.SparkSession

// structured query with count aggregate function
val q = spark
  .range(5)
  .groupBy($"id" % 2 as "group")
  .agg(count("id") as "count")
val plan = q.queryExecution.optimizedPlan
scala> println(plan.numberedTreeString)
00 Aggregate [(id#0L % 2)], [(id#0L % 2) AS group#3L, count(1) AS count#8L]
01 +- Range (0, 5, step=1, splits=Some(8))

import spark.sessionState.planner.Aggregation
val physicalPlan = Aggregation.apply(plan)

// HashAggregateExec selected
scala> println(physicalPlan.head.numberedTreeString)
00 HashAggregate(keys=[(id#0L % 2)#12L], functions=[count(1)], output=[group#3L, count#8L])
01 +- HashAggregate(keys=[(id#0L % 2) AS (id#0L % 2)#12L], functions=[partial_count(1)], output=[(id#0L % 2)#12L, count#14L])
02    +- PlanLater Range (0, 5, step=1, splits=Some(8))
----

[[aggregate-physical-operator-preference]]
`Aggregation` <<spark-sql-AggUtils.adoc#aggregate-physical-operator-selection-criteria, can select>> the following aggregate physical operators (in the order of preference):

. <<spark-sql-SparkPlan-HashAggregateExec.adoc#, HashAggregateExec>>

. <<spark-sql-SparkPlan-ObjectHashAggregateExec.adoc#, ObjectHashAggregateExec>>

. <<spark-sql-SparkPlan-SortAggregateExec.adoc#, SortAggregateExec>>

=== [[apply]] Applying Aggregation Strategy to Logical Plan (Executing Aggregation) -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): Seq[SparkPlan]
----

NOTE: `apply` is part of link:spark-sql-catalyst-GenericStrategy.adoc#apply[GenericStrategy Contract] to generate a collection of link:spark-sql-SparkPlan.adoc[SparkPlans] for a given link:spark-sql-LogicalPlan.adoc[logical plan].

`apply` requests `PhysicalAggregation` extractor for link:spark-sql-PhysicalAggregation.adoc#unapply[Aggregate logical operators] and creates a single aggregate physical operator for every link:spark-sql-LogicalPlan-Aggregate.adoc[Aggregate] logical operator found.

Internally, `apply` requests `PhysicalAggregation` to link:spark-sql-PhysicalAggregation.adoc#unapply[destructure a Aggregate logical operator] (into a four-element tuple) and splits link:spark-sql-Expression-AggregateExpression.adoc[aggregate expressions] per whether they are distinct or not (using their link:spark-sql-Expression-AggregateExpression.adoc#isDistinct[isDistinct] flag).

`apply` then creates a physical operator using the following helper methods:

* <<spark-sql-AggUtils.adoc#planAggregateWithoutDistinct, AggUtils.planAggregateWithoutDistinct>> when no distinct aggregate expression is used

* <<spark-sql-AggUtils.adoc#planAggregateWithOneDistinct, AggUtils.planAggregateWithOneDistinct>> when at least one distinct aggregate expression is used.
