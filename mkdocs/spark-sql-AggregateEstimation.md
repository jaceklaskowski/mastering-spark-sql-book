# AggregateEstimation

`AggregateEstimation` is...FIXME

=== [[estimate]] Estimating Statistics and Query Hints of Aggregate Logical Operator -- `estimate` Method

[source, scala]
----
estimate(agg: Aggregate): Option[Statistics]
----

`estimate`...FIXME

NOTE: `estimate` is used exclusively when `BasicStatsPlanVisitor` is requested to link:spark-sql-BasicStatsPlanVisitor.adoc#visitAggregate[estimate statistics and query hints of a Aggregate logical operator].
