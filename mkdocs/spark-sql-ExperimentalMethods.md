# ExperimentalMethods

`ExperimentalMethods` holds extra <<extraOptimizations, optimizations>> and <<extraStrategies, strategies>> that are used in <<spark-sql-SparkOptimizer.adoc#User-Provided-Optimizers, SparkOptimizer>> and <<spark-sql-SparkPlanner.adoc#, SparkPlanner>>, respectively.

[[attributes]]
.ExperimentalMethods' Attributes
[width="100%",cols="1m,2",options="header"]
|===
| Name
| Description

| extraOptimizations
a| [[extraOptimizations]] Collection of link:spark-sql-catalyst-Rule.adoc[rules] to optimize link:spark-sql-LogicalPlan.adoc[LogicalPlans] (i.e. `Rule[LogicalPlan]` objects)

[source, scala]
----
extraOptimizations: Seq[Rule[LogicalPlan]]
----

Used when `SparkOptimizer` is requested for the <<spark-sql-SparkOptimizer.adoc#User-Provided-Optimizers, User Provided Optimizers>>

| extraStrategies
a| [[extraStrategies]] Collection of <<spark-sql-SparkStrategy.adoc#, SparkStrategies>>

[source, scala]
----
extraStrategies: Seq[Strategy]
----

Used when `SessionState` is requested for the link:spark-sql-SessionState.adoc#planner[SparkPlanner]
|===

`ExperimentalMethods` is available as the <<spark-sql-SparkSession.adoc#experimental, experimental>> property of a `SparkSession`.

[source, scala]
----
scala> :type spark
org.apache.spark.sql.SparkSession

scala> :type spark.experimental
org.apache.spark.sql.ExperimentalMethods
----

=== Example

[source, scala]
----
import org.apache.spark.sql.catalyst.rules.Rule
import org.apache.spark.sql.catalyst.plans.logical.LogicalPlan

object SampleRule extends Rule[LogicalPlan] {
  def apply(p: LogicalPlan): LogicalPlan = p
}

scala> :type spark
org.apache.spark.sql.SparkSession

spark.experimental.extraOptimizations = Seq(SampleRule)

// extraOptimizations is used in Spark Optimizer
val rule = spark.sessionState.optimizer.batches.flatMap(_.rules).filter(_ == SampleRule).head
scala> rule.ruleName
res0: String = SampleRule
----
