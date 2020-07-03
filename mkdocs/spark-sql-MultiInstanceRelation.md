# MultiInstanceRelation

`MultiInstanceRelation` is a <<contact, contact>> of link:spark-sql-LogicalPlan.adoc[logical operators] which a <<newInstance, single instance>> might appear multiple times in a logical query plan.

[[newInstance]]
[[contract]]
[source, scala]
----
package org.apache.spark.sql.catalyst.analysis

trait MultiInstanceRelation {
  def newInstance(): LogicalPlan
}
----

When `ResolveReferences` logical evaluation is link:spark-sql-Analyzer-ResolveReferences.adoc#apply[executed], every `MultiInstanceRelation` in a logical query plan is requested to <<newInstance, produce a new version of itself with globally unique expression ids>>.

[[implementations]]
.MultiInstanceRelations
[cols="1,2",options="header",width="100%"]
|===
| MultiInstanceRelation
| Description

| `ContinuousExecutionRelation`
| [[ContinuousExecutionRelation]] Used in Spark Structured Streaming

| link:spark-sql-LogicalPlan-DataSourceV2Relation.adoc[DataSourceV2Relation]
| [[DataSourceV2Relation]]

| link:spark-sql-LogicalPlan-ExternalRDD.adoc[ExternalRDD]
| [[ExternalRDD]]

| link:hive/HiveTableRelation.adoc[HiveTableRelation]
| [[HiveTableRelation]]

| link:spark-sql-LogicalPlan-InMemoryRelation.adoc[InMemoryRelation]
| [[InMemoryRelation]]

| link:spark-sql-LogicalPlan-LocalRelation.adoc[LocalRelation]
| [[LocalRelation]]

| link:spark-sql-LogicalPlan-LogicalRDD.adoc[LogicalRDD]
| [[LogicalRDD]]

| link:spark-sql-LogicalPlan-LogicalRelation.adoc[LogicalRelation]
| [[LogicalRelation]]

| link:spark-sql-LogicalPlan-Range.adoc[Range]
| [[Range]]

| link:spark-sql-LogicalPlan-View.adoc[View]
| [[View]]

| `StreamingExecutionRelation`
| [[StreamingExecutionRelation]] Used in Spark Structured Streaming

| `StreamingRelation`
| [[StreamingRelation]] Used in Spark Structured Streaming

| `StreamingRelationV2`
| [[StreamingRelationV2]] Used in Spark Structured Streaming
|===
