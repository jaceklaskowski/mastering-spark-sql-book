# ResolveHiveSerdeTable Logical Resolution Rule

`ResolveHiveSerdeTable` is a logical resolution rule (i.e. `Rule[LogicalPlan]`) that the link:HiveSessionStateBuilder.adoc#analyzer[Hive-specific logical query plan analyzer] uses to <<apply, resolve the metadata of a hive table for CreateTable logical operators>>.

`ResolveHiveSerdeTable` is part of link:../spark-sql-Analyzer.adoc#extendedResolutionRules[additional rules] in link:../spark-sql-Analyzer.adoc#Resolution[Resolution] fixed-point batch of rules.

[source, scala]
----
// FIXME Example of ResolveHiveSerdeTable
----

=== [[apply]] Applying ResolveHiveSerdeTable Rule to Logical Plan -- `apply` Method

[source, scala]
----
apply(
  plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of link:../spark-sql-catalyst-Rule.adoc#apply[Rule Contract] to apply a rule to a link:../spark-sql-LogicalPlan.adoc[logical plan].

`apply`...FIXME
