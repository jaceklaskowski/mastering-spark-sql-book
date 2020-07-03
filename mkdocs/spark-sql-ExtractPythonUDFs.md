# ExtractPythonUDFs Physical Query Optimization

[[apply]]
`ExtractPythonUDFs` is a *physical query optimization* (aka _physical query preparation rule_ or simply _preparation rule_) that `QueryExecution` link:spark-sql-QueryExecution.adoc#preparations[uses] to optimize the physical plan of a structured query by <<extract, extracting Python UDFs from a physical query plan>> (excluding `FlatMapGroupsInPandasExec` operators that it simply skips over).

Technically, `ExtractPythonUDFs` is just a link:spark-sql-catalyst-Rule.adoc[Catalyst rule] for transforming link:spark-sql-SparkPlan.adoc[physical query plans], i.e. `Rule[SparkPlan]`.

`ExtractPythonUDFs` is part of link:spark-sql-QueryExecution.adoc#preparations[preparations] batch of physical query plan rules and is executed when `QueryExecution` is requested for the link:spark-sql-QueryExecution.adoc#executedPlan[optimized physical query plan] (i.e. in *executedPlan* phase of a query execution).

=== [[extract]] Extracting Python UDFs from Physical Query Plan -- `extract` Internal Method

[source, scala]
----
extract(plan: SparkPlan): SparkPlan
----

`extract`...FIXME

NOTE: `extract` is used exclusively when `ExtractPythonUDFs` is requested to <<apply, optimize a physical query plan>>.

=== [[trySplitFilter]] `trySplitFilter` Internal Method

[source, scala]
----
trySplitFilter(plan: SparkPlan): SparkPlan
----

`trySplitFilter`...FIXME

NOTE: `trySplitFilter` is used exclusively when `ExtractPythonUDFs` is requested to <<extract, extract>>.
