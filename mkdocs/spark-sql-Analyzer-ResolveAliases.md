# ResolveAliases Logical Resolution Rule

`ResolveAliases` is a logical resolution rule that the link:spark-sql-Analyzer.adoc#ResolveAliases[logical query plan analyzer] uses to <<apply, FIXME>> in an entire logical query plan.

Technically, `ResolveAliases` is just a link:spark-sql-catalyst-Rule.adoc[Catalyst rule] for transforming link:spark-sql-LogicalPlan.adoc[logical plans], i.e. `Rule[LogicalPlan]`.

`ResolveAliases` is part of link:spark-sql-Analyzer.adoc#Resolution[Resolution] fixed-point batch of rules.

NOTE: `ResolveAliases` is a Scala object inside link:spark-sql-Analyzer.adoc[Analyzer] class.

[[example]]
[source, scala]
----
import spark.sessionState.analyzer.ResolveAliases

// FIXME Using ResolveAliases rule
----

=== [[apply]] Applying ResolveAliases to Logical Plan -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of link:spark-sql-catalyst-Rule.adoc#apply[Rule Contract] to apply a rule to a link:spark-sql-LogicalPlan.adoc[logical plan].

`apply`...FIXME

=== [[assignAliases]] `assignAliases` Internal Method

[source, scala]
----
assignAliases(exprs: Seq[NamedExpression]): Seq[NamedExpression]
----

`assignAliases`...FIXME

NOTE: `assignAliases` is used when...FIXME
