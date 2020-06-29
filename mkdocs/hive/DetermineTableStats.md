title: DetermineTableStats

# DetermineTableStats Logical PostHoc Resolution Rule -- Computing Total Size Table Statistic for HiveTableRelations

`DetermineTableStats` is a link:HiveSessionStateBuilder.adoc#postHocResolutionRules[logical posthoc resolution rule] that the link:HiveSessionStateBuilder.adoc#analyzer[Hive-specific logical query plan analyzer] uses to <<apply, compute total size table statistic for HiveTableRelations with no statistics>>.

Technically, `DetermineTableStats` is a link:../spark-sql-catalyst-Rule.adoc[Catalyst rule] for transforming link:../spark-sql-LogicalPlan.adoc[logical plans], i.e. `Rule[LogicalPlan]`.

=== [[apply]] `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of link:../spark-sql-catalyst-Rule.adoc#apply[Rule Contract] to apply a rule to a link:../spark-sql-LogicalPlan.adoc[logical plan] (aka _execute a rule_).

`apply`...FIXME
