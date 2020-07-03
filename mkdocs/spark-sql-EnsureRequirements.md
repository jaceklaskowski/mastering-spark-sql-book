# EnsureRequirements Physical Query Optimization

[[apply]]
`EnsureRequirements` is a *physical query optimization* (aka _physical query preparation rule_ or simply _preparation rule_) that `QueryExecution` link:spark-sql-QueryExecution.adoc#preparations[uses] to optimize the physical plan of a structured query by transforming the following physical operators (up the plan tree):

. Removes two adjacent link:spark-sql-SparkPlan-ShuffleExchangeExec.adoc[ShuffleExchangeExec] physical operators if the child partitioning scheme guarantees the parent's partitioning

. For other non-``ShuffleExchangeExec`` physical operators, <<ensureDistributionAndOrdering, ensures partition distribution and ordering>> (possibly adding new physical operators, e.g. link:spark-sql-SparkPlan-BroadcastExchangeExec.adoc[BroadcastExchangeExec] and link:spark-sql-SparkPlan-ShuffleExchangeExec.adoc[ShuffleExchangeExec] for distribution or link:spark-sql-SparkPlan-SortExec.adoc[SortExec] for sorting)

Technically, `EnsureRequirements` is just a link:spark-sql-catalyst-Rule.adoc[Catalyst rule] for transforming link:spark-sql-SparkPlan.adoc[physical query plans], i.e. `Rule[SparkPlan]`.

`EnsureRequirements` is part of link:spark-sql-QueryExecution.adoc#preparations[preparations] batch of physical query plan rules and is executed when `QueryExecution` is requested for the link:spark-sql-QueryExecution.adoc#executedPlan[optimized physical query plan] (i.e. in *executedPlan* phase of a query execution).

[[conf]]
`EnsureRequirements` takes a link:spark-sql-SQLConf.adoc[SQLConf] when created.

[source, scala]
----
val q = ??? // FIXME
val sparkPlan = q.queryExecution.sparkPlan

import org.apache.spark.sql.execution.exchange.EnsureRequirements
val plan = EnsureRequirements(spark.sessionState.conf).apply(sparkPlan)
----

=== [[createPartitioning]] `createPartitioning` Internal Method

CAUTION: FIXME

=== [[defaultNumPreShufflePartitions]] `defaultNumPreShufflePartitions` Internal Method

CAUTION: FIXME

=== [[ensureDistributionAndOrdering]] Enforcing Partition Requirements (Distribution and Ordering) of Physical Operator -- `ensureDistributionAndOrdering` Internal Method

[source, scala]
----
ensureDistributionAndOrdering(operator: SparkPlan): SparkPlan
----

Internally, `ensureDistributionAndOrdering` takes the following from the input physical `operator`:

* link:spark-sql-SparkPlan.adoc#requiredChildDistribution[required partition requirements] for the children

* link:spark-sql-SparkPlan.adoc#requiredChildOrdering[required sort ordering] per the required partition requirements per child

* child physical plans

NOTE: The number of requirements for partitions and their sort ordering has to match the number and the order of the child physical plans.

`ensureDistributionAndOrdering` matches the operator's required partition requirements of children (`requiredChildDistributions`) to the children's link:spark-sql-SparkPlan.adoc#outputPartitioning[output partitioning] and (in that order):

. If the child satisfies the requested distribution, the child is left unchanged

. For link:spark-sql-Distribution-BroadcastDistribution.adoc[BroadcastDistribution], the child becomes the child of link:spark-sql-SparkPlan-BroadcastExchangeExec.adoc[BroadcastExchangeExec] unary operator for link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc[broadcast hash joins]

. Any other pair of child and distribution leads to link:spark-sql-SparkPlan-ShuffleExchangeExec.adoc[ShuffleExchangeExec] unary physical operator (with proper <<createPartitioning, partitioning>> for distribution and with link:spark-sql-properties.adoc#spark.sql.shuffle.partitions[spark.sql.shuffle.partitions] number of partitions, i.e. `200` by default)

NOTE: link:spark-sql-SparkPlan-ShuffleExchangeExec.adoc[ShuffleExchangeExec] can appear in the physical plan when the children's output partitioning cannot satisfy the physical operator's required child distribution.

If the input `operator` has multiple children and specifies child output distributions, then the children's link:spark-sql-SparkPlan.adoc#outputPartitioning[output partitionings] have to be compatible.

If the children's output partitionings are not all compatible, then...FIXME

`ensureDistributionAndOrdering` <<withExchangeCoordinator, adds ExchangeCoordinator>> (only when link:spark-sql-adaptive-query-execution.adoc[adaptive query execution] is enabled which is not by default).

NOTE: At this point in `ensureDistributionAndOrdering` the required child distributions are already handled.

`ensureDistributionAndOrdering` matches the operator's required sort ordering of children (`requiredChildOrderings`) to the children's link:spark-sql-SparkPlan.adoc#outputPartitioning[output partitioning] and if the orderings do not match, link:spark-sql-SparkPlan-SortExec.adoc#creating-instance[SortExec] unary physical operator is created as a new child.

In the end, `ensureDistributionAndOrdering` link:spark-sql-catalyst-TreeNode.adoc#withNewChildren[sets the new children] for the input `operator`.

NOTE: `ensureDistributionAndOrdering` is used exclusively when `EnsureRequirements` is <<apply, executed>> (i.e. applied to a physical plan).

=== [[withExchangeCoordinator]] Adding ExchangeCoordinator (Adaptive Query Execution) -- `withExchangeCoordinator` Internal Method

[source, scala]
----
withExchangeCoordinator(
  children: Seq[SparkPlan],
  requiredChildDistributions: Seq[Distribution]): Seq[SparkPlan]
----

`withExchangeCoordinator` adds link:spark-sql-ExchangeCoordinator.adoc[ExchangeCoordinator] to link:spark-sql-SparkPlan-ShuffleExchangeExec.adoc[ShuffleExchangeExec] operators if adaptive query execution is enabled (per link:spark-sql-properties.adoc#spark.sql.adaptive.enabled[spark.sql.adaptive.enabled] property) and partitioning scheme of the `ShuffleExchangeExec` operators support `ExchangeCoordinator`.

NOTE: link:spark-sql-properties.adoc#spark.sql.adaptive.enabled[spark.sql.adaptive.enabled] property is disabled by default.

[[supportsCoordinator]]
Internally, `withExchangeCoordinator` checks if the input `children` operators support `ExchangeCoordinator` which is that either holds:

* If there is at least one link:spark-sql-SparkPlan-ShuffleExchangeExec.adoc[ShuffleExchangeExec] operator, all children are either `ShuffleExchangeExec` with link:spark-sql-SparkPlan-Partitioning.adoc#HashPartitioning[HashPartitioning] or their link:spark-sql-SparkPlan.adoc#outputPartitioning[output partitioning] is link:spark-sql-SparkPlan-Partitioning.adoc#HashPartitioning[HashPartitioning] (even inside link:spark-sql-SparkPlan-Partitioning.adoc#PartitioningCollection[PartitioningCollection])

* There are at least two `children` operators and the input `requiredChildDistributions` are all link:spark-sql-Distribution-ClusteredDistribution.adoc[ClusteredDistribution]

With link:spark-sql-adaptive-query-execution.adoc[adaptive query execution] (i.e. when link:spark-sql-adaptive-query-execution.adoc#spark.sql.adaptive.enabled[spark.sql.adaptive.enabled] configuration property is `true`) and the <<supportsCoordinator, operator supports ExchangeCoordinator>>, `withExchangeCoordinator` creates a `ExchangeCoordinator` and:

* For every `ShuffleExchangeExec`, link:spark-sql-SparkPlan-ShuffleExchangeExec.adoc#coordinator[registers the `ExchangeCoordinator`]

* <<createPartitioning, Creates HashPartitioning partitioning scheme>> with the link:spark-sql-SQLConf.adoc#numShufflePartitions[default number of partitions to use when shuffling data for joins or aggregations] (as link:spark-sql-properties.adoc#spark.sql.shuffle.partitions[spark.sql.shuffle.partitions] which is `200` by default) and adds `ShuffleExchangeExec` to the final result (for the current physical operator)

Otherwise (when adaptive query execution is disabled or `children` do not support `ExchangeCoordinator`), `withExchangeCoordinator` returns the input `children` unchanged.

NOTE: `withExchangeCoordinator` is used exclusively for <<ensureDistributionAndOrdering, enforcing partition requirements of a physical operator>>.

=== [[reorderJoinPredicates]] `reorderJoinPredicates` Internal Method

[source, scala]
----
reorderJoinPredicates(plan: SparkPlan): SparkPlan
----

`reorderJoinPredicates`...FIXME

NOTE: `reorderJoinPredicates` is used when...FIXME
