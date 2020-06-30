title: Subexpression Elimination

# Subexpression Elimination In Code-Generated Expression Evaluation (Common Expression Reuse)

*Subexpression Elimination* (aka *Common Expression Reuse*) is an optimisation of a link:spark-sql-LogicalPlan.adoc[logical query plan] that link:spark-sql-CodegenContext.adoc#subexpressionElimination[eliminates expressions in code-generated (non-interpreted) expression evaluation].

Subexpression Elimination is enabled by default. Use the internal <<spark.sql.subexpressionElimination.enabled, spark.sql.subexpressionElimination.enabled>> configuration property control whether the feature is enabled (`true`) or not (`false`).

Subexpression Elimination is used (by means of link:spark-sql-SparkPlan.adoc#subexpressionEliminationEnabled[subexpressionEliminationEnabled] flag of `SparkPlan`) when the following physical operators are requested to execute (i.e. moving away from queries to an RDD of internal rows to describe a distributed computation):

* link:spark-sql-SparkPlan-ProjectExec.adoc#doExecute[ProjectExec]

* link:spark-sql-SparkPlan-HashAggregateExec.adoc#doExecute[HashAggregateExec] (and for link:spark-sql-SparkPlan-HashAggregateExec.adoc#finishAggregate[finishAggregate])

* link:spark-sql-SparkPlan-ObjectHashAggregateExec.adoc#doExecute[ObjectHashAggregateExec]

* link:spark-sql-SparkPlan-SortAggregateExec.adoc#doExecute[SortAggregateExec]

* link:spark-sql-SparkPlan-WindowExec.adoc#doExecute[WindowExec] (and creates a link:spark-sql-SparkPlan-WindowExec.adoc#windowFrameExpressionFactoryPairs[lookup table for WindowExpressions and factory functions for WindowFunctionFrame])

Internally, subexpression elimination happens when `CodegenContext` is requested for link:spark-sql-CodegenContext.adoc#subexpressionElimination[subexpressionElimination] (when `CodegenContext` is requested to <<generateExpressions, generateExpressions>> with <<spark.sql.subexpressionElimination.enabled, subexpression elimination>> enabled).

=== [[spark.sql.subexpressionElimination.enabled]] spark.sql.subexpressionElimination.enabled Configuration Property

link:spark-sql-properties.adoc#spark.sql.subexpressionElimination.enabled[spark.sql.subexpressionElimination.enabled] internal configuration property controls whether the subexpression elimination optimization is enabled or not.

TIP: Use link:spark-sql-SQLConf.adoc#subexpressionEliminationEnabled[subexpressionEliminationEnabled] method to access the current value.

[source, scala]
----
scala> import spark.sessionState.conf
import spark.sessionState.conf

scala> conf.subexpressionEliminationEnabled
res1: Boolean = true
----
