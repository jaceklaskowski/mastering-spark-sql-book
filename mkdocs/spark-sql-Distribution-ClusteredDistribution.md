# ClusteredDistribution

`ClusteredDistribution` is a link:spark-sql-Distribution.adoc[Distribution] that <<createPartitioning, creates a HashPartitioning>> for the <<clustering, clustering expressions>> and a requested number of partitions.

`ClusteredDistribution` requires that the <<clustering, clustering expressions>> should not be empty (i.e. `Nil`).

`ClusteredDistribution` is <<creating-instance, created>> when the following physical operators are requested for a required child distribution:

* `MapGroupsExec`, link:spark-sql-SparkPlan-HashAggregateExec.adoc#requiredChildDistribution[HashAggregateExec], link:spark-sql-SparkPlan-ObjectHashAggregateExec.adoc#requiredChildDistribution[ObjectHashAggregateExec], link:spark-sql-SparkPlan-SortAggregateExec.adoc#requiredChildDistribution[SortAggregateExec], link:spark-sql-SparkPlan-WindowExec.adoc#requiredChildDistribution[WindowExec]

* Spark Structured Streaming's `FlatMapGroupsWithStateExec`, `StateStoreRestoreExec`, `StateStoreSaveExec`, `StreamingDeduplicateExec`, `StreamingSymmetricHashJoinExec`, `StreamingSymmetricHashJoinExec`

* SparkR's `FlatMapGroupsInRExec`

* PySpark's `FlatMapGroupsInPandasExec`

`ClusteredDistribution` is used when:

* `DataSourcePartitioning`, `SinglePartition`, `HashPartitioning`, and `RangePartitioning` are requested to `satisfies`

* `EnsureRequirements` is requested to link:spark-sql-EnsureRequirements.adoc#withExchangeCoordinator[add an ExchangeCoordinator] for Adaptive Query Execution

=== [[createPartitioning]] `createPartitioning` Method

[source, scala]
----
createPartitioning(numPartitions: Int): Partitioning
----

NOTE: `createPartitioning` is part of link:spark-sql-Distribution.adoc#createPartitioning[Distribution Contract] to create a link:spark-sql-SparkPlan-Partitioning.adoc[Partitioning] for a given number of partitions.

`createPartitioning` creates a `HashPartitioning` for the <<clustering, clustering expressions>> and the input `numPartitions`.

`createPartitioning` reports an `AssertionError` when the <<requiredNumPartitions, number of partitions>> is not the input `numPartitions`.

[options="wrap"]
```
This ClusteredDistribution requires [requiredNumPartitions] partitions, but the actual number of partitions is [numPartitions].
```

=== [[creating-instance]] Creating ClusteredDistribution Instance

`ClusteredDistribution` takes the following when created:

* [[clustering]] Clustering link:spark-sql-Expression.adoc[expressions]
* [[requiredNumPartitions]] Required number of partitions (default: `None`)

NOTE: `None` for the required number of partitions indicates to use any number of partitions (possibly link:spark-sql-properties.adoc#spark.sql.shuffle.partitions[spark.sql.shuffle.partitions] configuration property with the default of `200` partitions).
