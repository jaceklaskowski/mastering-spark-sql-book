# CoalesceShufflePartitions Physical Optimization

`CoalesceShufflePartitions` is a physical query plan optimization for [Adaptive Query Execution](../adaptive-query-execution/index.md).

`CoalesceShufflePartitions` uses [spark.sql.adaptive.coalescePartitions.enabled](../configuration-properties.md#spark.sql.adaptive.coalescePartitions.enabled) configuration property.

`CoalesceShufflePartitions` is a [Catalyst rule](../catalyst/Rule.md) for transforming [physical plans](../physical-operators/SparkPlan.md) (`Rule[SparkPlan]`).

## Creating Instance

`CoalesceShufflePartitions` takes the following to be created:

* <span id="session"> [SparkSession](../SparkSession.md)

`CoalesceShufflePartitions` is created when `AdaptiveSparkPlanExec` physical operator is requested for the [QueryStage Optimizer Rules](../adaptive-query-execution/AdaptiveSparkPlanExec.md#queryStageOptimizerRules).

## <span id="apply"> Executing Rule

```scala
apply(
  plan: SparkPlan): SparkPlan
```

!!! note "spark.sql.adaptive.coalescePartitions.enabled Configuration Property"
    With [spark.sql.adaptive.coalescePartitions.enabled](../configuration-properties.md#spark.sql.adaptive.coalescePartitions.enabled) configuration property disabled (`false`), `apply` does nothing and simply gives the input [SparkPlan](../physical-operators/SparkPlan.md) back unmodified.

`apply` makes sure that one of the following holds or does nothing (and simply gives the input [SparkPlan](../physical-operators/SparkPlan.md) back unmodified):

1. All the leaves in the query plan are [QueryStageExec](../adaptive-query-execution/QueryStageExec.md) physical operators
1. No [CustomShuffleReaderExec](../physical-operators/CustomShuffleReaderExec.md) physical operator in the given query plan

`apply` collects all [ShuffleQueryStageExec](../adaptive-query-execution/ShuffleQueryStageExec.md) leaf physical operators in the input [physical query plan](../physical-operators/SparkPlan.md).

`apply` makes sure that the following holds or does nothing (and simply gives the input [SparkPlan](../physical-operators/SparkPlan.md) back unmodified):

1. All [ShuffleQueryStageExec](../adaptive-query-execution/ShuffleQueryStageExec.md) leaf physical operators use [ShuffleExchangeExec](../physical-operators/ShuffleExchangeExec.md) unary physical operators that have [canChangeNumPartitions](../physical-operators/ShuffleExchangeExec.md#canChangeNumPartitions) flag enabled

`apply`...FIXME

`apply` is part of the [Rule](../catalyst/Rule.md#apply) abstraction.
