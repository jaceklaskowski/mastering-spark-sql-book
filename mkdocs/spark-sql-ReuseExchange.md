# ReuseExchange Physical Query Optimization

`ReuseExchange` is a *physical query optimization* (aka _physical query preparation rule_ or simply _preparation rule_) that `QueryExecution` link:spark-sql-QueryExecution.adoc#preparations[uses] to optimize the physical plan of a structured query by <<apply, FIXME>>.

Technically, `ReuseExchange` is just a link:spark-sql-catalyst-Rule.adoc[Catalyst rule] for transforming link:spark-sql-SparkPlan.adoc[physical query plans], i.e. `Rule[SparkPlan]`.

`ReuseExchange` is part of link:spark-sql-QueryExecution.adoc#preparations[preparations] batch of physical query plan rules and is executed when `QueryExecution` is requested for the link:spark-sql-QueryExecution.adoc#executedPlan[optimized physical query plan] (i.e. in *executedPlan* phase of a query execution).

=== [[apply]] `apply` Method

[source, scala]
----
apply(plan: SparkPlan): SparkPlan
----

NOTE: `apply` is part of link:spark-sql-catalyst-Rule.adoc#apply[Rule Contract] to apply a rule to a link:spark-sql-SparkPlan.adoc[physical plan].

`apply` finds all link:spark-sql-SparkPlan-Exchange.adoc[Exchange] unary operators and...FIXME

`apply` does nothing and simply returns the input physical `plan` if link:spark-sql-properties.adoc#spark.sql.exchange.reuse[spark.sql.exchange.reuse] internal configuration property is off (i.e. `false`).

NOTE: link:spark-sql-properties.adoc#spark.sql.exchange.reuse[spark.sql.exchange.reuse] internal configuration property is on (i.e. `true`) by default.
