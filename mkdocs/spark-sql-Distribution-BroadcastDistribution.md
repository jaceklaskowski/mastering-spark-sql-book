# BroadcastDistribution

[[requiredNumPartitions]]
`BroadcastDistribution` is a link:spark-sql-Distribution.adoc[Distribution] that indicates to use one partition only and...FIXME.

`BroadcastDistribution` is <<creating-instance, created>> when:

. `BroadcastHashJoinExec` is requested for link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc#requiredChildDistribution[required child output distributions] (with link:spark-sql-HashedRelationBroadcastMode.adoc[HashedRelationBroadcastMode] of the link:spark-sql-HashJoin.adoc#buildKeys[build join keys])

. `BroadcastNestedLoopJoinExec` is requested for link:spark-sql-SparkPlan-BroadcastNestedLoopJoinExec.adoc#requiredChildDistribution[required child output distributions] (with link:spark-sql-IdentityBroadcastMode.adoc[IdentityBroadcastMode])

[[creating-instance]]
[[mode]]
`BroadcastDistribution` takes a link:spark-sql-BroadcastMode.adoc[BroadcastMode] when created.

NOTE: `BroadcastDistribution` is converted to a link:spark-sql-SparkPlan-BroadcastExchangeExec.adoc[BroadcastExchangeExec] physical operator when link:spark-sql-EnsureRequirements.adoc[EnsureRequirements] physical query plan optimization is executed (and link:spark-sql-EnsureRequirements.adoc#ensureDistributionAndOrdering[enforces partition requirements for data distribution and ordering]).

=== [[createPartitioning]] `createPartitioning` Method

[source, scala]
----
createPartitioning(numPartitions: Int): Partitioning
----

NOTE: `createPartitioning` is part of link:spark-sql-Distribution.adoc#createPartitioning[Distribution Contract] to create a link:spark-sql-SparkPlan-Partitioning.adoc[Partitioning] for a given number of partitions.

`createPartitioning`...FIXME
