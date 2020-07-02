# HashClusteredDistribution

`HashClusteredDistribution` is a link:spark-sql-Distribution.adoc[Distribution] that <<createPartitioning, creates a HashPartitioning>> for the <<expressions, hash expressions>> and a requested number of partitions.

[[requiredNumPartitions]]
`HashClusteredDistribution` specifies `None` for the link:spark-sql-Distribution.adoc#requiredNumPartitions[required number of partitions].

NOTE: `None` for the required number of partitions indicates to use any number of partitions (possibly link:spark-sql-properties.adoc#spark.sql.shuffle.partitions[spark.sql.shuffle.partitions] configuration property with the default of `200` partitions).

`HashClusteredDistribution` is <<creating-instance, created>> when the following physical operators are requested for the link:spark-sql-SparkPlan.adoc#requiredChildDistribution[required partition requirements of the child operator(s)] (e.g. link:spark-sql-SparkPlan-CoGroupExec.adoc[CoGroupExec], link:spark-sql-SparkPlan-ShuffledHashJoinExec.adoc[ShuffledHashJoinExec], link:spark-sql-SparkPlan-SortMergeJoinExec.adoc[SortMergeJoinExec] and Spark Structured Streaming's `StreamingSymmetricHashJoinExec`).

[[creating-instance]][[expressions]]
`HashClusteredDistribution` takes hash link:spark-sql-Expression.adoc[expressions] when created.

`HashClusteredDistribution` requires that the <<expressions, hash expressions>> should not be empty (i.e. `Nil`).

`HashClusteredDistribution` is used when:

* `EnsureRequirements` is requested to link:spark-sql-EnsureRequirements.adoc#withExchangeCoordinator[add an ExchangeCoordinator] for Adaptive Query Execution

* `HashPartitioning` is requested to `satisfies`

=== [[createPartitioning]] `createPartitioning` Method

[source, scala]
----
createPartitioning(
  numPartitions: Int): Partitioning
----

NOTE: `createPartitioning` is part of link:spark-sql-Distribution.adoc#createPartitioning[Distribution Contract] to create a link:spark-sql-SparkPlan-Partitioning.adoc[Partitioning] for a given number of partitions.

`createPartitioning` creates a `HashPartitioning` for the <<expressions, hash expressions>> and the input `numPartitions`.
