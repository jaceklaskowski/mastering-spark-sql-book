title: RuleExecutor

# RuleExecutor -- Tree Transformation Rule Executor

`RuleExecutor` is the <<contract, base>> of <<extensions, rule executors>> that are responsible for <<execute, executing>> a collection of <<batches, batches (of rules)>> to transform a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>>.

[[contract]]
[source, scala]
----
package org.apache.spark.sql.catalyst.rules

abstract class RuleExecutor[TreeType <: TreeNode[_]] {
  // only required properties (vals and methods) that have no implementation
  // the others follow
  protected def batches: Seq[Batch]
}
----

.RuleExecutor Contract
[cols="1m,2",options="header",width="100%"]
|===
| Property
| Description

| batches
a| [[batches]]

[source, scala]
----
batches: Seq[Batch]
----

Collection of <<Batch, rule batches>>, i.e. a sequence of a collection of <<spark-sql-catalyst-Rule.adoc#, rules>> with a name and a <<Strategy, strategy>> that `RuleExecutor` uses when <<execute, executed>>
|===

[[TreeType]]
NOTE: `TreeType` is the type of the <<spark-sql-catalyst-TreeNode.adoc#implementations, TreeNode implementation>> that a `RuleExecutor` can be <<execute, executed>> on, i.e. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>, <<spark-sql-SparkPlan.adoc#, SparkPlan>>, <<spark-sql-Expression.adoc#, Expression>> or a combination thereof.

[[extensions]]
.RuleExecutors (Direct Implementations)
[cols="1,2",options="header",width="100%"]
|===
| RuleExecutor
| Description

| <<spark-sql-Analyzer.adoc#, Analyzer>>
| [[Analyzer]] Logical query plan analyzer

| `ExpressionCanonicalizer`
| [[ExpressionCanonicalizer]]

| <<spark-sql-Optimizer.adoc#, Optimizer>>
| [[Optimizer]] Generic logical query plan optimizer
|===

=== [[execute]] Applying Rule Batches to TreeNode -- `execute` Method

[source, scala]
----
execute(plan: TreeType): TreeType
----

`execute` iterates over <<batches, rule batches>> and applies rules sequentially to the input `plan`.

`execute` tracks the number of iterations and the time of executing each rule (with a plan).

When a rule changes a plan, you should see the following TRACE message in the logs:

```
TRACE HiveSessionStateBuilder$$anon$1:
=== Applying Rule [ruleName] ===
[currentAndModifiedPlansSideBySide]
```

After the number of iterations has reached the number of iterations for the batch's `Strategy` it stops execution and prints out the following WARN message to the logs:

```
WARN HiveSessionStateBuilder$$anon$1: Max iterations ([iteration]) reached for batch [batchName]
```

When the plan has not changed (after applying rules), you should see the following TRACE message in the logs and `execute` moves on to applying the rules in the next batch. The moment is called *fixed point* (i.e. when the execution *converges*).

```
TRACE HiveSessionStateBuilder$$anon$1: Fixed point reached for batch [batchName] after [iteration] iterations.
```

After the batch finishes, if the plan has been changed by the rules, you should see the following DEBUG message in the logs:

```
DEBUG HiveSessionStateBuilder$$anon$1:
=== Result of Batch [batchName] ===
[currentAndModifiedPlansSideBySide]
```

Otherwise, when the rules had no changes to a plan, you should see the following TRACE message in the logs:

```
TRACE HiveSessionStateBuilder$$anon$1: Batch [batchName] has no effect.
```

=== [[Batch]] Rule Batch -- Collection of Rules

`Batch` is a named collection of <<spark-sql-catalyst-Rule.adoc#, rules>> with a <<Strategy, strategy>>.

[[Batch-creating-instance]]
`Batch` takes the following when created:

* [[name]] Batch name
* [[strategy]] <<Strategy, Strategy>>
* [[rules]] Collection of <<spark-sql-catalyst-Rule.adoc#, rules>>

=== [[Strategy]] Batch Execution Strategy

`Strategy` is the base of the batch execution strategies that indicate the maximum number of executions (aka _maxIterations_).

[source, scala]
----
abstract class Strategy {
  def maxIterations: Int
}
----

.Strategies
[cols="1,2",options="header",width="100%"]
|===
| Strategy
| Description

| `Once`
| [[Once]] A strategy that runs only once (with `maxIterations` as `1`)

| `FixedPoint`
| [[FixedPoint]] A strategy that runs until fix point (i.e. converge) or `maxIterations` times, whichever comes first
|===

=== [[isPlanIntegral]] `isPlanIntegral` Method

[source, scala]
----
isPlanIntegral(plan: TreeType): Boolean
----

`isPlanIntegral` simply returns `true`.

NOTE: `isPlanIntegral` is used exclusively when `RuleExecutor` is requested to <<execute, execute>>.
