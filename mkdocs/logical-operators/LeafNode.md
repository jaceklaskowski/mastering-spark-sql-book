title: LeafNode

# LeafNode -- Base Logical Operator with No Child Operators and Optional Statistics

`LeafNode` is the base of <<extensions, logical operators>> that have no <<spark-sql-catalyst-TreeNode.adoc#children, child>> operators.

`LeafNode` that wants to survive analysis has to define <<computeStats, computeStats>> as it throws an `UnsupportedOperationException` by default.

[[extensions]]
.LeafNodes (Direct Implementations)
[cols="1,2",options="header",width="100%"]
|===
| LeafNode
| Description

| <<spark-sql-LogicalPlan-AnalysisBarrier.adoc#, AnalysisBarrier>>
| [[AnalysisBarrier]]

| <<spark-sql-LogicalPlan-DataSourceV2Relation.adoc#, DataSourceV2Relation>>
| [[DataSourceV2Relation]]

| <<spark-sql-LogicalPlan-ExternalRDD.adoc#, ExternalRDD>>
| [[ExternalRDD]]

| link:hive/HiveTableRelation.adoc[HiveTableRelation]
| [[HiveTableRelation]]

| <<spark-sql-LogicalPlan-InMemoryRelation.adoc#, InMemoryRelation>>
| [[InMemoryRelation]]

| <<spark-sql-LogicalPlan-LocalRelation.adoc#, LocalRelation>>
| [[LocalRelation]]

| <<spark-sql-LogicalPlan-LogicalRDD.adoc#, LogicalRDD>>
| [[LogicalRDD]]

| <<spark-sql-LogicalPlan-LogicalRelation.adoc#, LogicalRelation>>
| [[LogicalRelation]]

| <<spark-sql-LogicalPlan-OneRowRelation.adoc#, OneRowRelation>>
| [[OneRowRelation]]

| <<spark-sql-LogicalPlan-Range.adoc#, Range>>
| [[Range]]

| <<spark-sql-LogicalPlan-UnresolvedCatalogRelation.adoc#, UnresolvedCatalogRelation>>
| [[UnresolvedCatalogRelation]]

| <<spark-sql-LogicalPlan-UnresolvedInlineTable.adoc#, UnresolvedInlineTable>>
| [[UnresolvedInlineTable]]

| <<spark-sql-LogicalPlan-UnresolvedRelation.adoc#, UnresolvedRelation>>
| [[UnresolvedRelation]]

| <<spark-sql-LogicalPlan-UnresolvedTableValuedFunction.adoc#, UnresolvedTableValuedFunction>>
| [[UnresolvedTableValuedFunction]]
|===

=== [[computeStats]] Computing Statistics -- `computeStats` Method

[source, scala]
----
computeStats(): Statistics
----

`computeStats` simply throws an `UnsupportedOperationException`.

NOTE: Logical operators, e.g. link:spark-sql-LogicalPlan-ExternalRDD.adoc[ExternalRDD], link:spark-sql-LogicalPlan-LogicalRDD.adoc[LogicalRDD] and `DataSourceV2Relation`, or relations, e.g. `HadoopFsRelation` or `BaseRelation`, use link:spark-sql-properties.adoc#spark.sql.defaultSizeInBytes[spark.sql.defaultSizeInBytes] internal property for the default estimated size if the statistics could not be computed.

NOTE: `computeStats` is used exclusively when `SizeInBytesOnlyStatsPlanVisitor` uses the link:spark-sql-SizeInBytesOnlyStatsPlanVisitor.adoc#default[default case] to compute the size statistic (in bytes) for a link:spark-sql-LogicalPlan.adoc[logical operator].
