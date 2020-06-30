# Adaptive Query Execution

*Adaptive Query Execution* (aka *Adaptive Query Optimisation* or *Adaptive Optimisation*) is an optimisation of a link:spark-sql-SparkPlan.adoc[query execution plan] that link:spark-sql-SparkPlanner.adoc[Spark Planner] uses for allowing alternative execution plans at runtime that would be optimized better based on runtime statistics.

Quoting the description of a <<i-want-more, talk>> by the authors of Adaptive Query Execution:

> At runtime, the adaptive execution mode can change shuffle join to broadcast join if it finds the size of one table is less than the broadcast threshold. It can also handle skewed input data for join and change the partition number of the next stage to better fit the data scale. In general, adaptive execution decreases the effort involved in tuning SQL query parameters and improves the execution performance by choosing a better execution plan and parallelism at runtime.

Adaptive Query Execution is disabled by default. Set <<spark.sql.adaptive.enabled, spark.sql.adaptive.enabled>> configuration property to `true` to enable it.

NOTE: Adaptive query execution is not supported for streaming Datasets and is disabled at their execution.

=== [[spark.sql.adaptive.enabled]] spark.sql.adaptive.enabled Configuration Property

link:spark-sql-properties.adoc#spark.sql.adaptive.enabled[spark.sql.adaptive.enabled] configuration property turns adaptive query execution on.

TIP: Use link:spark-sql-SQLConf.adoc#adaptiveExecutionEnabled[adaptiveExecutionEnabled] method to access the current value.

=== [[EnsureRequirements]] EnsureRequirements

link:spark-sql-EnsureRequirements.adoc[EnsureRequirements] is...FIXME

=== [[i-want-more]] Further Reading and Watching

. (video) https://youtu.be/FZgojLWdjaw[An Adaptive Execution Engine For Apache Spark SQL &mdash; Carson Wang]

. https://conferences.oreilly.com/strata/strata-sg/public/schedule/detail/62938[An adaptive execution mode for Spark SQL] by Carson Wang (Intel), Yucai Yu (Intel) at Strata Data Conference in Singapore, December 7, 2017
