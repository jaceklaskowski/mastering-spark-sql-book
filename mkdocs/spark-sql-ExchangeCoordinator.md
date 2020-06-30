# ExchangeCoordinator

`ExchangeCoordinator` is <<creating-instance, created>> when `EnsureRequirements` physical query optimization is requested to <<spark-sql-EnsureRequirements.adoc#withExchangeCoordinator, add an ExchangeCoordinator>> for <<spark-sql-adaptive-query-execution.adoc#, Adaptive Query Execution>>.

[[creating-instance]]
`ExchangeCoordinator` takes the following to be created:

* [[numExchanges]] Number of <<spark-sql-SparkPlan-ShuffleExchangeExec.adoc#, ShuffleExchangeExec>> unary physical operators
* [[advisoryTargetPostShuffleInputSize]] Recommended size of the input data of a post-shuffle partition (configured by <<spark-sql-properties.adoc#spark.sql.adaptive.shuffle.targetPostShuffleInputSize, spark.sql.adaptive.shuffle.targetPostShuffleInputSize>> property)
* [[minNumPostShufflePartitions]] Optional advisory minimum number of post-shuffle partitions (default: `None`) (configured by <<spark-sql-properties.adoc#spark.sql.adaptive.minNumPostShufflePartitions, spark.sql.adaptive.minNumPostShufflePartitions>> property)

[[exchanges]]
`ExchangeCoordinator` keeps track of <<spark-sql-SparkPlan-ShuffleExchangeExec.adoc#, ShuffleExchangeExec>> unary physical operators that were <<registerExchange, registered>> (when `ShuffleExchangeExec` unary physical operator was requested to <<spark-sql-SparkPlan-ShuffleExchangeExec.adoc#doPrepare, prepare itself for execution>>).

[[toString]]
`ExchangeCoordinator` uses the following *text representation* (i.e. `toString`):

```
coordinator[target post-shuffle partition size: [advisoryTargetPostShuffleInputSize]]
```

=== [[postShuffleRDD]] `postShuffleRDD` Method

[source, scala]
----
postShuffleRDD(exchange: ShuffleExchangeExec): ShuffledRowRDD
----

`postShuffleRDD`...FIXME

NOTE: `postShuffleRDD` is used exclusively when `ShuffleExchangeExec` unary physical operator is requested to <<spark-sql-SparkPlan-ShuffleExchangeExec.adoc#doExecute, execute>>.

=== [[doEstimationIfNecessary]] `doEstimationIfNecessary` Internal Method

[source, scala]
----
doEstimationIfNecessary(): Unit
----

`doEstimationIfNecessary`...FIXME

NOTE: `doEstimationIfNecessary` is used exclusively when `ExchangeCoordinator` is requested for a <<postShuffleRDD, post-shuffle RDD (ShuffledRowRDD)>>.

=== [[estimatePartitionStartIndices]] `estimatePartitionStartIndices` Method

[source, scala]
----
estimatePartitionStartIndices(
  mapOutputStatistics: Array[MapOutputStatistics]): Array[Int]
----

`estimatePartitionStartIndices`...FIXME

NOTE: `estimatePartitionStartIndices` is used exclusively when `ExchangeCoordinator` is requested for a <<doEstimationIfNecessary, doEstimationIfNecessary>>.

=== [[registerExchange]] `registerExchange` Method

[source, scala]
----
registerExchange(exchange: ShuffleExchangeExec): Unit
----

`registerExchange` simply adds the <<spark-sql-SparkPlan-ShuffleExchangeExec.adoc#, ShuffleExchangeExec>> unary physical operator to the <<exchanges, exchanges>> internal registry.

NOTE: `registerExchange` is used exclusively when `ShuffleExchangeExec` unary physical operator is requested to <<spark-sql-SparkPlan-ShuffleExchangeExec.adoc#doPrepare, prepare itself for execution>>.
