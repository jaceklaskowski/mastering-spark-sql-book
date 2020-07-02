title: Partitioning

# Partitioning -- Specification of Physical Operator's Output Partitions

`Partitioning` is the <<contract, contract>> to hint the Spark Physical Optimizer for the number of partitions the output of a <<spark-sql-SparkPlan.adoc#, physical operator>> should be split across.

[[contract]]
[[numPartitions]]
[source, scala]
----
numPartitions: Int
----

`numPartitions` is used in:

* `EnsureRequirements` physical preparation rule to link:spark-sql-EnsureRequirements.adoc#ensureDistributionAndOrdering[enforce partition requirements of a physical operator]

* link:spark-sql-SparkPlan-SortMergeJoinExec.adoc[SortMergeJoinExec] for `outputPartitioning` for `FullOuter` join type
* `Partitioning.allCompatible`

[[implementations]]
.Partitioning Schemes (Partitionings) and Their Properties
[width="100%",cols="1,1,1,1,1",options="header"]
|===
| Partitioning
| compatibleWith
| guarantees
| numPartitions
| satisfies

m| BroadcastPartitioning
| `BroadcastPartitioning` with the same `BroadcastMode`
| Exactly the same `BroadcastPartitioning`
^| 1
| [[BroadcastPartitioning]] link:spark-sql-Distribution-BroadcastDistribution.adoc[BroadcastDistribution] with the same `BroadcastMode`

a| `HashPartitioning`

* `clustering` expressions
* `numPartitions`

| `HashPartitioning` (when their underlying expressions are semantically equal, i.e. deterministic and canonically equal)
| `HashPartitioning` (when their underlying expressions are semantically equal, i.e. deterministic and canonically equal)
| Input `numPartitions`
a| [[HashPartitioning]]

* link:spark-sql-Distribution-UnspecifiedDistribution.adoc[UnspecifiedDistribution]

* link:spark-sql-Distribution-ClusteredDistribution.adoc[ClusteredDistribution] with all the hashing link:spark-sql-Expression.adoc[expressions] included in `clustering` expressions

a| `PartitioningCollection`

* `partitionings`

| Any `Partitioning` that is compatible with one of the input `partitionings`
| Any `Partitioning` that is guaranteed by any of the input `partitionings`
| Number of partitions of the first `Partitioning` in the input `partitionings`
| [[PartitioningCollection]] Any `Distribution` that is satisfied by any of the input `partitionings`

a| `RangePartitioning`

* `ordering` collection of `SortOrder`
* `numPartitions`

| `RangePartitioning` (when semantically equal, i.e. underlying expressions are deterministic and canonically equal)
| `RangePartitioning` (when semantically equal, i.e. underlying expressions are deterministic and canonically equal)
| Input `numPartitions`
a| [[RangePartitioning]]

* link:spark-sql-Distribution-UnspecifiedDistribution.adoc[UnspecifiedDistribution]
* link:spark-sql-Distribution-OrderedDistribution.adoc[OrderedDistribution] with `requiredOrdering` that matches the input `ordering`
* link:spark-sql-Distribution-ClusteredDistribution.adoc[ClusteredDistribution] with all the children of the input `ordering` semantically equal to one of the `clustering` expressions

a| `RoundRobinPartitioning`

* `numPartitions`

| Always negative
| Always negative
| Input `numPartitions`
| [[RoundRobinPartitioning]] link:spark-sql-Distribution-UnspecifiedDistribution.adoc[UnspecifiedDistribution]

| `SinglePartition`
| Any `Partitioning` with exactly one partition
| Any `Partitioning` with exactly one partition
^| 1
| [[SinglePartition]] Any `Distribution` except link:spark-sql-Distribution-BroadcastDistribution.adoc[BroadcastDistribution]

a| `UnknownPartitioning`

* `numPartitions`
| Always negative
| Always negative
| Input `numPartitions`
| [[UnknownPartitioning]] link:spark-sql-Distribution-UnspecifiedDistribution.adoc[UnspecifiedDistribution]
|===
