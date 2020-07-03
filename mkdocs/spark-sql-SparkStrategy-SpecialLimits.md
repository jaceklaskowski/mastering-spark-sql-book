# SpecialLimits Execution Planning Strategy

`SpecialLimits` is an link:spark-sql-SparkStrategy.adoc[execution planning strategy] that link:spark-sql-SparkPlanner.adoc[Spark Planner] uses to <<apply, FIXME>>.

=== [[apply]] Applying SpecialLimits Strategy to Logical Plan (Executing SpecialLimits) -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): Seq[SparkPlan]
----

NOTE: `apply` is part of link:spark-sql-catalyst-GenericStrategy.adoc#apply[GenericStrategy Contract] to generate a collection of link:spark-sql-SparkPlan.adoc[SparkPlans] for a given link:spark-sql-LogicalPlan.adoc[logical plan].

`apply`...FIXME
