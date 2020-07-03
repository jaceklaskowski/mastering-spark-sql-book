# DataSourceAnalysis PostHoc Logical Resolution Rule

`DataSourceAnalysis` is a <<spark-sql-Analyzer.adoc#postHocResolutionRules, posthoc logical resolution rule>> that the link:spark-sql-BaseSessionStateBuilder.adoc#analyzer[default] and link:hive/HiveSessionStateBuilder.adoc#analyzer[Hive-specific] logical query plan analyzers use to <<apply, FIXME>>.

[[resolutions]]
.DataSourceAnalysis's Logical Resolutions (Conversions)
[cols="1,1,2",options="header",width="100%"]
|===
| Source Operator
| Target Operator
| Description

| <<spark-sql-LogicalPlan-CreateTable.adoc#, CreateTable>> [small]#(isDatasourceTable + no query)#
| <<spark-sql-LogicalPlan-CreateDataSourceTableCommand.adoc#, CreateDataSourceTableCommand>>
| [[CreateTable-no-query]]

| <<spark-sql-LogicalPlan-CreateTable.adoc#, CreateTable>> [small]#(isDatasourceTable + a resolved query)#
| <<spark-sql-LogicalPlan-CreateDataSourceTableAsSelectCommand.adoc#, CreateDataSourceTableAsSelectCommand>>
| [[CreateTable-query]]

| <<InsertIntoTable.adoc#, InsertIntoTable>> with <<spark-sql-InsertableRelation.adoc#, InsertableRelation>>
| <<spark-sql-LogicalPlan-InsertIntoDataSourceCommand.adoc#, InsertIntoDataSourceCommand>>
| [[InsertIntoTable-InsertableRelation]]

| link:InsertIntoDir.adoc[InsertIntoDir] [small]#(non-hive provider)#
| <<spark-sql-LogicalPlan-InsertIntoDataSourceDirCommand.adoc#, InsertIntoDataSourceDirCommand>>
| [[InsertIntoDir]]

| <<InsertIntoTable.adoc#, InsertIntoTable>> with <<spark-sql-BaseRelation-HadoopFsRelation.adoc#, HadoopFsRelation>>
| <<spark-sql-LogicalPlan-InsertIntoHadoopFsRelationCommand.adoc#, InsertIntoHadoopFsRelationCommand>>
| [[InsertIntoTable-HadoopFsRelation]]
|===

Technically, `DataSourceAnalysis` is a link:spark-sql-catalyst-Rule.adoc[Catalyst rule] for transforming link:spark-sql-LogicalPlan.adoc[logical plans], i.e. `Rule[LogicalPlan]`.

[source, scala]
----
// FIXME Example of DataSourceAnalysis
import org.apache.spark.sql.execution.datasources.DataSourceAnalysis
val rule = DataSourceAnalysis(spark.sessionState.conf)

val plan = FIXME

rule(plan)
----

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply`...FIXME
