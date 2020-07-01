title: BaseRelation

# BaseRelation -- Collection of Tuples with Schema

`BaseRelation` is the <<contract, contract>> of <<implementations, relations>> (aka _collections of tuples_) with a known <<schema, schema>>.

NOTE: "Data source", "relation" and "table" are often used as synonyms.

[[contract]]
[source, scala]
----
package org.apache.spark.sql.sources

abstract class BaseRelation {
  // only required properties (vals and methods) that have no implementation
  // the others follow
  def schema: StructType
  def sqlContext: SQLContext
}
----

.(Subset of) BaseRelation Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| `schema`
| [[schema]] link:spark-sql-StructType.adoc[StructType] that describes the schema of tuples

| `sqlContext`
| [[sqlContext]] link:spark-sql-SQLContext.adoc[SQLContext]
|===

`BaseRelation` is "created" when `DataSource` is requested to link:spark-sql-DataSource.adoc#resolveRelation[resolve a relation].

`BaseRelation` is transformed into a `DataFrame` when `SparkSession` is requested to link:spark-sql-SparkSession.adoc#baseRelationToDataFrame[create a DataFrame].

`BaseRelation` uses <<needConversion, needConversion>> flag to control type conversion of objects inside link:spark-sql-Row.adoc[Rows] to Catalyst types, e.g. `java.lang.String` to `UTF8String`.

NOTE: It is recommended that custom data sources (outside Spark SQL) should leave <<needConversion, needConversion>> flag enabled, i.e. `true`.

`BaseRelation` can optionally give an <<sizeInBytes, estimated size>> (in bytes).

[[implementations]]
.BaseRelations
[width="100%",cols="1,2",options="header"]
|===
| BaseRelation
| Description

| `ConsoleRelation`
| [[ConsoleRelation]] Used in Spark Structured Streaming

| link:spark-sql-BaseRelation-HadoopFsRelation.adoc[HadoopFsRelation]
| [[HadoopFsRelation]]

| link:spark-sql-JDBCRelation.adoc[JDBCRelation]
| [[JDBCRelation]]

| <<spark-sql-KafkaRelation.adoc#, KafkaRelation>>
| [[KafkaRelation]] Datasets with records from Apache Kafka
|===

=== [[needConversion]] Should JVM Objects Inside Rows Be Converted to Catalyst Types? -- `needConversion` Method

[source, scala]
----
needConversion: Boolean
----

`needConversion` flag is enabled (`true`) by default.

NOTE: It is recommended to leave `needConversion` enabled for data sources outside Spark SQL.

NOTE: `needConversion` is used exclusively when `DataSourceStrategy` execution planning strategy is link:spark-sql-SparkStrategy-DataSourceStrategy.adoc#apply[executed] (and link:spark-sql-SparkStrategy-DataSourceStrategy.adoc#toCatalystRDD[does the RDD conversion] from `RDD[Row]` to `RDD[InternalRow]`).

=== [[unhandledFilters]] Finding Unhandled Filter Predicates -- `unhandledFilters` Method

[source, scala]
----
unhandledFilters(filters: Array[Filter]): Array[Filter]
----

`unhandledFilters` returns <<spark-sql-Filter.adoc#, Filter predicates>> that the data source does not support (handle) natively.

NOTE: `unhandledFilters` returns the input `filters` by default as it is considered safe to double evaluate filters regardless whether they could be supported or not.

NOTE: `unhandledFilters` is used exclusively when `DataSourceStrategy` execution planning strategy is requested to <<spark-sql-SparkStrategy-DataSourceStrategy.adoc#selectFilters, selectFilters>>.

=== [[sizeInBytes]] Estimated Size Of Relation (In Bytes) -- `sizeInBytes` Method

[source, scala]
----
sizeInBytes: Long
----

`sizeInBytes` is the estimated size of a relation (used in query planning).

NOTE: `sizeInBytes` defaults to link:spark-sql-properties.adoc#spark.sql.defaultSizeInBytes[spark.sql.defaultSizeInBytes] internal property (i.e. infinite).

NOTE: `sizeInBytes` is used exclusively when `LogicalRelation` is requested to link:spark-sql-LogicalPlan-LogicalRelation.adoc#computeStats[computeStats] (and they are not available in link:spark-sql-LogicalPlan-LogicalRelation.adoc#catalogTable[CatalogTable]).
