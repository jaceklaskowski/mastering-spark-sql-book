# ResolveTimeZone Logical Resolution Rule

`ResolveTimeZone` is a logical resolution rule that the link:spark-sql-Analyzer.adoc#ResolveRelations[logical query plan analyzer] uses to <<apply, FIXME>>.

Technically, `ResolveTimeZone` is just a link:spark-sql-catalyst-Rule.adoc[Catalyst rule] for transforming link:spark-sql-LogicalPlan.adoc[logical plans], i.e. `Rule[LogicalPlan]`.

`ResolveTimeZone` is part of link:spark-sql-Analyzer.adoc#Resolution[Resolution] fixed-point batch of rules.

=== [[apply]] Applying ResolveTimeZone to Logical Plan -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of link:spark-sql-catalyst-Rule.adoc#apply[Rule Contract] to apply a rule to a link:spark-sql-LogicalPlan.adoc[logical plan].

`apply`...FIXME
