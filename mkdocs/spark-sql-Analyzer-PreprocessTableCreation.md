# PreprocessTableCreation PostHoc Logical Resolution Rule

`PreprocessTableCreation` is a <<spark-sql-Analyzer.adoc#postHocResolutionRules, posthoc logical resolution rule>> that <<apply, resolves a logical query plan>> with <<spark-sql-LogicalPlan-CreateTable.adoc#, CreateTable>> logical operators.

`PreprocessTableCreation` is part of the <<spark-sql-Analyzer.adoc#Post-Hoc-Resolution, Post-Hoc Resolution>> once-executed batch of the link:hive/HiveSessionStateBuilder.adoc#analyzer[Hive-specific] and the <<spark-sql-BaseSessionStateBuilder.adoc#analyzer, default>> logical analyzers.

`PreprocessTableCreation` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

[[sparkSession]]
[[creating-instance]]
`PreprocessTableCreation` takes a <<spark-sql-SparkSession.adoc#, SparkSession>> when created.

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply`...FIXME
