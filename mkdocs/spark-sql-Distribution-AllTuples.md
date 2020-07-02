# AllTuples

[[requiredNumPartitions]]
`AllTuples` is a link:spark-sql-Distribution.adoc[Distribution] that indicates to use one partition only.

=== [[createPartitioning]] `createPartitioning` Method

[source, scala]
----
createPartitioning(numPartitions: Int): Partitioning
----

NOTE: `createPartitioning` is part of link:spark-sql-Distribution.adoc#createPartitioning[Distribution Contract] to create a link:spark-sql-SparkPlan-Partitioning.adoc[Partitioning] for a given number of partitions.

`createPartitioning`...FIXME
