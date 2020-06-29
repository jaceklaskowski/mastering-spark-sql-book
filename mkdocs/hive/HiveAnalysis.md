title: HiveAnalysis

# HiveAnalysis PostHoc Logical Resolution Rule

`HiveAnalysis` is a link:HiveSessionStateBuilder.adoc#postHocResolutionRules[logical posthoc resolution rule] that the link:HiveSessionStateBuilder.adoc#analyzer[Hive-specific logical query plan analyzer] uses to <<apply, FIXME>>.

Technically, `HiveAnalysis` is a link:../spark-sql-catalyst-Rule.adoc[Catalyst rule] for transforming link:../spark-sql-LogicalPlan.adoc[logical plans], i.e. `Rule[LogicalPlan]`.

[source, scala]
----
// FIXME Example of HiveAnalysis
----

=== [[apply]] Applying HiveAnalysis Rule to Logical Plan (Executing HiveAnalysis) -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of link:../spark-sql-catalyst-Rule.adoc#apply[Rule Contract] to apply a rule to a link:../spark-sql-LogicalPlan.adoc[logical plan].

`apply`...FIXME
