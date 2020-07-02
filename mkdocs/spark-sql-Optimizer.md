title: Optimizer

# Catalyst Optimizer -- Generic Logical Query Plan Optimizer

`Optimizer` (aka *Catalyst Optimizer*) is the base of <<extensions, logical query plan optimizers>> that defines the <<batches, rule batches of logical optimizations>> (i.e. logical optimizations that are the rules that transform the query plan of a structured query to produce the <<spark-sql-QueryExecution.adoc#optimizedPlan, optimized logical plan>>).

[[extensions]]
NOTE: <<spark-sql-SparkOptimizer.adoc#, SparkOptimizer>> is the one and only direct implementation of the `Optimizer` Contract in Spark SQL.

`Optimizer` is a <<spark-sql-catalyst-RuleExecutor.adoc#, RuleExecutor>> of <<spark-sql-LogicalPlan.adoc#, LogicalPlan>> (i.e. `RuleExecutor[LogicalPlan]`).

```
Optimizer: Analyzed Logical Plan ==> Optimized Logical Plan
```

`Optimizer` is available as the <<spark-sql-SessionState.adoc#optimizer, optimizer>> property of a session-specific `SessionState`.

[source, scala]
----
scala> :type spark
org.apache.spark.sql.SparkSession

scala> :type spark.sessionState.optimizer
org.apache.spark.sql.catalyst.optimizer.Optimizer
----

You can access the optimized logical plan of a structured query (as a <<spark-sql-Dataset.adoc#, Dataset>>) using <<spark-sql-dataset-operators.adoc#explain, Dataset.explain>> basic action (with `extended` flag enabled) or SQL's `EXPLAIN EXTENDED` SQL command.

[source, scala]
----
// sample structured query
val inventory = spark
  .range(5)
  .withColumn("new_column", 'id + 5 as "plus5")

// Using explain operator (with extended flag enabled)
scala> inventory.explain(extended = true)
== Parsed Logical Plan ==
'Project [id#0L, ('id + 5) AS plus5#2 AS new_column#3]
+- AnalysisBarrier
      +- Range (0, 5, step=1, splits=Some(8))

== Analyzed Logical Plan ==
id: bigint, new_column: bigint
Project [id#0L, (id#0L + cast(5 as bigint)) AS new_column#3L]
+- Range (0, 5, step=1, splits=Some(8))

== Optimized Logical Plan ==
Project [id#0L, (id#0L + 5) AS new_column#3L]
+- Range (0, 5, step=1, splits=Some(8))

== Physical Plan ==
*(1) Project [id#0L, (id#0L + 5) AS new_column#3L]
+- *(1) Range (0, 5, step=1, splits=8)
----

Alternatively, you can access the analyzed logical plan using `QueryExecution` and its <<spark-sql-QueryExecution.adoc#optimizedPlan, optimizedPlan>> property  (that together with `numberedTreeString` method is a very good "debugging" tool).

[source, scala]
----
val optimizedPlan = inventory.queryExecution.optimizedPlan
scala> println(optimizedPlan.numberedTreeString)
00 Project [id#0L, (id#0L + 5) AS new_column#3L]
01 +- Range (0, 5, step=1, splits=Some(8))
----

`Optimizer` defines the <<defaultBatches, default rule batches>> that are considered the base rule batches that can be further refined (extended or <<excludedRules, with some rules excluded>>).

[[defaultBatches]]
.Optimizer's Default Optimization Rule Batches (in the order of execution)
[cols="2,1,3,3",options="header",width="100%"]
|===
^.^| Batch Name
^.^| Strategy
| Rules
| Description

^.^| [[Eliminate_Distinct]] Eliminate Distinct
^.^| `Once`
| [[EliminateDistinct]] EliminateDistinct
|

.7+^.^| [[Finish_Analysis]] Finish Analysis
.7+^.^| `Once`
| [[EliminateSubqueryAliases]] <<spark-sql-Optimizer-EliminateSubqueryAliases.adoc#, EliminateSubqueryAliases>>
| Removes (eliminates) <<spark-sql-LogicalPlan-SubqueryAlias.adoc#, SubqueryAlias>> unary logical operators from a <<spark-sql-LogicalPlan.adoc#, logical plan>>

| [[EliminateView]] <<spark-sql-Optimizer-EliminateView.adoc#, EliminateView>>
| Removes (eliminates) <<spark-sql-LogicalPlan-View.adoc#, View>> unary logical operators from a <<spark-sql-LogicalPlan.adoc#, logical plan>> and replaces them with their <<spark-sql-LogicalPlan-View.adoc#child, child>> logical operator

| [[ReplaceExpressions]] <<spark-sql-Optimizer-ReplaceExpressions.adoc#, ReplaceExpressions>>
| Replaces <<spark-sql-Expression-RuntimeReplaceable.adoc#, RuntimeReplaceable>> expressions with their single child expression

| [[ComputeCurrentTime]] <<spark-sql-Optimizer-ComputeCurrentTime.adoc#, ComputeCurrentTime>>
|

| [[GetCurrentDatabase]] <<spark-sql-Optimizer-GetCurrentDatabase.adoc#, GetCurrentDatabase>>
|

| [[RewriteDistinctAggregates]] RewriteDistinctAggregates
|

| [[ReplaceDeduplicateWithAggregate]] ReplaceDeduplicateWithAggregate
|

^.^| [[Union]] Union
^.^| `Once`
| [[CombineUnions]] <<spark-sql-Optimizer-CombineUnions.adoc#, CombineUnions>>
|

.2+^.^| [[LocalRelation-early]] LocalRelation early
.2+^.^| <<fixedPoint, FixedPoint>>
| [[ConvertToLocalRelation]] ConvertToLocalRelation
|

| [[PropagateEmptyRelation]] <<spark-sql-Optimizer-PropagateEmptyRelation.adoc#, PropagateEmptyRelation>>
|

^.^| [[Pullup-Correlated-Expressions]] Pullup Correlated Expressions
^.^| `Once`
| [[PullupCorrelatedPredicates]] link:spark-sql-Optimizer-PullupCorrelatedPredicates.adoc[PullupCorrelatedPredicates]
|

^.^| [[Subquery]] Subquery
^.^| `Once`
| [[OptimizeSubqueries]] link:spark-sql-Optimizer-OptimizeSubqueries.adoc[OptimizeSubqueries]
|

.6+^.^| [[Replace-Operators]] Replace Operators
.6+^.^| <<fixedPoint, FixedPoint>>
| link:spark-sql-Optimizer-RewriteExceptAll.adoc[RewriteExceptAll]
| [[RewriteExceptAll]]

| RewriteIntersectAll
| [[RewriteIntersectAll]]

| ReplaceIntersectWithSemiJoin
| [[ReplaceIntersectWithSemiJoin]]

| link:spark-sql-Optimizer-ReplaceExceptWithFilter.adoc[ReplaceExceptWithFilter]
| [[ReplaceExceptWithFilter]]

| link:spark-sql-Optimizer-ReplaceExceptWithAntiJoin.adoc[ReplaceExceptWithAntiJoin]
| [[ReplaceExceptWithAntiJoin]]

| ReplaceDistinctWithAggregate
| [[ReplaceDistinctWithAggregate]]

.2+^.^| [[Aggregate]] Aggregate
.2+^.^| <<fixedPoint, FixedPoint>>
| RemoveLiteralFromGroupExpressions
|

| RemoveRepetitionFromGroupExpressions
|

^.^| <<operatorOptimizationBatch, operatorOptimizationBatch>>
^.^|
|
|

^.^| [[Join-Reorder]][[Join_Reorder]] Join Reorder
^.^| `Once`
| [[CostBasedJoinReorder]] <<spark-sql-Optimizer-CostBasedJoinReorder.adoc#, CostBasedJoinReorder>>
| Reorders <<spark-sql-LogicalPlan-Join.adoc#, Join>> logical operators

^.^| [[Remove-Redundant-Sorts]] Remove Redundant Sorts
^.^| `Once`
| [[RemoveRedundantSorts]] RemoveRedundantSorts
|

^.^| [[Decimal-Optimizations]][[Decimal_Optimizations]] Decimal Optimizations
^.^| <<fixedPoint, FixedPoint>>
| [[DecimalAggregates]] link:spark-sql-Optimizer-DecimalAggregates.adoc[DecimalAggregates]
|

.2+^.^| [[Object_Expressions_Optimization]] Object Expressions Optimization
.2+^.^| <<fixedPoint, FixedPoint>>
| EliminateMapObjects
|

| [[CombineTypedFilters]] link:spark-sql-Optimizer-CombineTypedFilters.adoc[CombineTypedFilters]
|

.2+^.^| [[LocalRelation]] LocalRelation
.2+^.^| <<fixedPoint, FixedPoint>>
| ConvertToLocalRelation
|

| link:spark-sql-Optimizer-PropagateEmptyRelation.adoc[PropagateEmptyRelation]
|

^.^| [[Extract-PythonUDF-From-JoinCondition]] Extract PythonUDF From JoinCondition
^.^| `Once`
| PullOutPythonUDFInJoinCondition
|

^.^| [[Check_Cartesian_Products]] Check Cartesian Products
^.^| `Once`
| CheckCartesianProducts
|

.4+^.^| [[RewriteSubquery]] RewriteSubquery
.4+^.^| `Once`
| [[RewritePredicateSubquery]] link:spark-sql-Optimizer-RewritePredicateSubquery.adoc[RewritePredicateSubquery]
|

| [[ColumnPruning]] link:spark-sql-Optimizer-ColumnPruning.adoc[ColumnPruning]
|

| [[CollapseProject]] CollapseProject
|

| [[RemoveRedundantProject]] RemoveRedundantProject
|

^.^| [[UpdateAttributeReferences]] UpdateAttributeReferences
^.^| `Once`
| UpdateNullabilityInAttributeReferences
|

|===

TIP: Consult the https://github.com/apache/spark/blob/v2.4.0/sql/catalyst/src/main/scala/org/apache/spark/sql/catalyst/optimizer/Optimizer.scala#L56-L181[sources] of the `Optimizer` class for the up-to-date list of the <<defaultBatches, default optimization rule batches>>.

`Optimizer` defines the <<operatorOptimizationRuleSet, operator optimization rules>> with the <<extendedOperatorOptimizationRules, extendedOperatorOptimizationRules>> extension point for additional optimizations in the *Operator Optimization* batch.

[[operatorOptimizationRuleSet]]
.Optimizer's Operator Optimization Rules (in the order of execution)
[cols="1,1",options="header",width="100%"]
|===
| Rule Name
| Description

| PushProjectionThroughUnion
| [[PushProjectionThroughUnion]]

| ReorderJoin
| [[ReorderJoin]]

| EliminateOuterJoin
| [[EliminateOuterJoin]]

| PushPredicateThroughJoin
| [[PushPredicateThroughJoin]]

| PushDownPredicate
| [[PushDownPredicate]]

| LimitPushDown
| [[LimitPushDown]]

| ColumnPruning
| [[ColumnPruning]]

| CollapseRepartition
| [[CollapseRepartition]]

| CollapseProject
| [[CollapseProject]]

| <<spark-sql-Optimizer-CollapseWindow.adoc#, CollapseWindow>>
| [[CollapseWindow]] Collapses two adjacent Window logical operators

| CombineFilters
| [[CombineFilters]]

| CombineLimits
| [[CombineLimits]]

| CombineUnions
| [[CombineUnions]]

| NullPropagation
| [[NullPropagation]]

| ConstantPropagation
| [[ConstantPropagation]]

| FoldablePropagation
| [[FoldablePropagation]]

| OptimizeIn
| [[OptimizeIn]]

| ConstantFolding
| [[ConstantFolding]]

| ReorderAssociativeOperator
| [[ReorderAssociativeOperator]]

| LikeSimplification
| [[LikeSimplification]]

| BooleanSimplification
| [[BooleanSimplification]]

| SimplifyConditionals
| [[SimplifyConditionals]]

| RemoveDispensableExpressions
| [[RemoveDispensableExpressions]]

| SimplifyBinaryComparison
| [[SimplifyBinaryComparison]]

| PruneFilters
| [[PruneFilters]]

| EliminateSorts
| [[EliminateSorts]]

| SimplifyCasts
| [[SimplifyCasts]]

| SimplifyCaseConversionExpressions
| [[SimplifyCaseConversionExpressions]]

| RewriteCorrelatedScalarSubquery
| [[RewriteCorrelatedScalarSubquery]]

| EliminateSerialization
| [[EliminateSerialization]]

| RemoveRedundantAliases
| [[RemoveRedundantAliases]]

| RemoveRedundantProject
| [[RemoveRedundantProject]]

| SimplifyExtractValueOps
| [[SimplifyExtractValueOps]]

| CombineConcats
| [[CombineConcats]]
|===

`Optimizer` defines <<operatorOptimizationBatch, Operator Optimization Batch>> that is simply a collection of rule batches with the <<operatorOptimizationRuleSet, operator optimization rules>> before and after `InferFiltersFromConstraints` logical rule.

[[operatorOptimizationBatch]]
.Optimizer's Operator Optimization Batch (in the order of execution)
[cols="2,1,3",options="header",width="100%"]
|===
^.^| Batch Name
^.^| Strategy
| Rules

| [[Operator-Optimization-before-Inferring-Filters]] Operator Optimization before Inferring Filters
| <<fixedPoint, FixedPoint>>
| <<operatorOptimizationRuleSet, Operator optimization rules>>

| [[Infer-Filters]] Infer Filters
| `Once`
| link:spark-sql-Optimizer-InferFiltersFromConstraints.adoc[InferFiltersFromConstraints]

| [[Operator-Optimization-after-Inferring-Filters]] Operator Optimization after Inferring Filters
| <<fixedPoint, FixedPoint>>
| <<operatorOptimizationRuleSet, Operator optimization rules>>
|===

[[excludedRules]]
[[spark.sql.optimizer.excludedRules]]
`Optimizer` uses <<spark-sql-properties.adoc#spark.sql.optimizer.excludedRules, spark.sql.optimizer.excludedRules>> configuration property to control what optimization rules in the <<defaultBatches, defaultBatches>> should be excluded (default: none).

[[sessionCatalog]]
[[creating-instance]]
`Optimizer` takes a <<spark-sql-SessionCatalog.adoc#, SessionCatalog>> when created.

NOTE: `Optimizer` is a Scala abstract class and cannot be <<creating-instance, created>> directly. It is created indirectly when the <<extensions, concrete Optimizers>> are.

[[nonExcludableRules]]
`Optimizer` considers some optimization rules as *non-excludable*. The non-excludable optimization rules are considered critical for query optimization and are not recommended to be excluded (even if they are specified in <<spark.sql.optimizer.excludedRules, spark.sql.optimizer.excludedRules>> configuration property).

. <<ComputeCurrentTime, ComputeCurrentTime>>
. <<EliminateDistinct, EliminateDistinct>>
. <<EliminateSubqueryAliases, EliminateSubqueryAliases>>
. <<EliminateView, EliminateView>>
. <<GetCurrentDatabase, GetCurrentDatabase>>
. <<PullOutPythonUDFInJoinCondition, PullOutPythonUDFInJoinCondition>>
. <<PullupCorrelatedPredicates, PullupCorrelatedPredicates>>
. <<ReplaceDeduplicateWithAggregate, ReplaceDeduplicateWithAggregate>>
. <<ReplaceDistinctWithAggregate, ReplaceDistinctWithAggregate>>
. <<ReplaceExceptWithAntiJoin, ReplaceExceptWithAntiJoin>>
. <<ReplaceExceptWithFilter, ReplaceExceptWithFilter>>
. <<ReplaceExpressions, ReplaceExpressions>>
. <<ReplaceIntersectWithSemiJoin, ReplaceIntersectWithSemiJoin>>
. <<RewriteCorrelatedScalarSubquery, RewriteCorrelatedScalarSubquery>>
. <<RewriteDistinctAggregates, RewriteDistinctAggregates>>
. <<RewriteExceptAll, RewriteExceptAll>>
. <<RewriteIntersectAll, RewriteIntersectAll>>
. <<RewritePredicateSubquery, RewritePredicateSubquery>>

[[internal-properties]]
.Optimizer's Internal Registries and Counters
[cols="1,1,2",options="header",width="100%"]
|===
| Name
| Initial Value
| Description

| [[fixedPoint]] `fixedPoint`
| `FixedPoint` with the number of iterations as defined by link:spark-sql-CatalystConf.adoc#optimizerMaxIterations[spark.sql.optimizer.maxIterations]
| Used in <<Replace-Operators, Replace Operators>>, <<Aggregate, Aggregate>>, <<Operator-Optimizations, Operator Optimizations>>, <<Decimal-Optimizations, Decimal Optimizations>>, <<Typed-Filter-Optimization, Typed Filter Optimization>> and <<LocalRelation, LocalRelation>> batches (and also indirectly in the User Provided Optimizers rule batch in link:spark-sql-SparkOptimizer.adoc#User-Provided-Optimizers[SparkOptimizer]).
|===

=== [[extendedOperatorOptimizationRules]] Additional Operator Optimization Rules -- `extendedOperatorOptimizationRules` Extension Point

[source, scala]
----
extendedOperatorOptimizationRules: Seq[Rule[LogicalPlan]]
----

`extendedOperatorOptimizationRules` extension point defines additional rules for the Operator Optimization batch.

NOTE: `extendedOperatorOptimizationRules` rules are executed right after <<Operator-Optimization-before-Inferring-Filters, Operator Optimization before Inferring Filters>> and <<Operator-Optimization-after-Inferring-Filters, Operator Optimization after Inferring Filters>>.

=== [[batches]] `batches` Final Method

[source, scala]
----
batches: Seq[Batch]
----

NOTE: `batches` is part of the <<spark-sql-catalyst-RuleExecutor.adoc#batches, RuleExecutor Contract>> to define the rule batches to use when executed.

`batches`...FIXME
