# InMemoryScans Execution Planning Strategy

`InMemoryScans` is an link:spark-sql-SparkStrategy.adoc[execution planning strategy] that <<apply, plans InMemoryRelation logical operators to InMemoryTableScanExec physical operators>>.

[source, scala]
----
val spark: SparkSession = ...
// query uses InMemoryRelation logical operator
val q = spark.range(5).cache
val plan = q.queryExecution.optimizedPlan
scala> println(plan.numberedTreeString)
00 InMemoryRelation [id#208L], true, 10000, StorageLevel(disk, memory, deserialized, 1 replicas)
01    +- *Range (0, 5, step=1, splits=8)

// InMemoryScans is an internal class of SparkStrategies
import spark.sessionState.planner.InMemoryScans
val physicalPlan = InMemoryScans.apply(plan).head
scala> println(physicalPlan.numberedTreeString)
00 InMemoryTableScan [id#208L]
01    +- InMemoryRelation [id#208L], true, 10000, StorageLevel(disk, memory, deserialized, 1 replicas)
02          +- *Range (0, 5, step=1, splits=8)
----

`InMemoryScans` is part of the link:spark-sql-SparkPlanner.adoc#strategies[standard execution planning strategies] of link:spark-sql-SparkPlanner.adoc[SparkPlanner].

=== [[apply]] Applying InMemoryScans Strategy to Logical Plan (Executing InMemoryScans) -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): Seq[SparkPlan]
----

NOTE: `apply` is part of link:spark-sql-catalyst-GenericStrategy.adoc#apply[GenericStrategy Contract] to generate a collection of link:spark-sql-SparkPlan.adoc[SparkPlans] for a given link:spark-sql-LogicalPlan.adoc[logical plan].

`apply` link:spark-sql-PhysicalOperation.adoc#unapply[destructures the input logical plan] to a link:spark-sql-LogicalPlan-InMemoryRelation.adoc[InMemoryRelation] logical operator.

In the end, `apply` link:spark-sql-SparkPlanner.adoc#pruneFilterProject[pruneFilterProject] with a new link:spark-sql-SparkPlan-InMemoryTableScanExec.adoc#creating-instance[InMemoryTableScanExec] physical operator.
