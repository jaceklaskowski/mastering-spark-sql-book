title: Filter

# Data Source Filter Predicate (For Filter Pushdown)

`Filter` is the <<contract, contract>> for <<implementations, filter predicates>> that can be pushed down to a relation (aka _data source_).

`Filter` is used when:

* (Data Source API V1) `BaseRelation` is requested for link:spark-sql-BaseRelation.adoc#unhandledFilters[unhandled filter predicates] (and hence `BaseRelation` implementations, i.e. link:spark-sql-JDBCRelation.adoc#unhandledFilters[JDBCRelation])

* (Data Source API V1) `PrunedFilteredScan` is requested for link:spark-sql-PrunedFilteredScan.adoc#buildScan[build a scan] (and hence `PrunedFilteredScan` implementations, i.e. link:spark-sql-JDBCRelation.adoc#buildScan[JDBCRelation])

* `FileFormat` is requested to link:spark-sql-FileFormat.adoc#buildReader[buildReader] (and hence `FileFormat` implementations, i.e. link:spark-sql-OrcFileFormat.adoc#buildReader[OrcFileFormat], link:spark-sql-CSVFileFormat.adoc#buildReader[CSVFileFormat], link:spark-sql-JsonFileFormat.adoc#buildReader[JsonFileFormat], link:spark-sql-TextFileFormat.adoc#buildReader[TextFileFormat] and Spark MLlib's `LibSVMFileFormat`)

* `FileFormat` is requested to link:spark-sql-FileFormat.adoc#buildReaderWithPartitionValues[build a Data Reader with partition column values appended] (and hence `FileFormat` implementations, i.e. link:spark-sql-OrcFileFormat.adoc#buildReaderWithPartitionValues[OrcFileFormat], link:spark-sql-ParquetFileFormat.adoc#buildReaderWithPartitionValues[ParquetFileFormat])

* `RowDataSourceScanExec` is link:spark-sql-SparkPlan-RowDataSourceScanExec.adoc#creating-instance[created] (for a link:spark-sql-SparkPlan-DataSourceScanExec.adoc#simpleString[simple text representation (in a query plan tree)])

* `DataSourceStrategy` execution planning strategy is requested to link:spark-sql-SparkStrategy-DataSourceStrategy.adoc#pruneFilterProject[pruneFilterProject] (when link:spark-sql-SparkStrategy-DataSourceStrategy.adoc#apply[executed] for link:spark-sql-LogicalPlan-LogicalRelation.adoc[LogicalRelation] logical operators with a link:spark-sql-PrunedFilteredScan.adoc[PrunedFilteredScan] or a link:spark-sql-PrunedScan.adoc[PrunedScan])

* `DataSourceStrategy` execution planning strategy is requested to link:spark-sql-SparkStrategy-DataSourceStrategy.adoc#selectFilters[selectFilters]

* `JDBCRDD` is link:spark-sql-JDBCRDD.adoc#filters[created] and requested to link:spark-sql-JDBCRDD.adoc#scanTable[scanTable]

* (Data Source API V2) `SupportsPushDownFilters` is requested to link:spark-sql-SupportsPushDownFilters.adoc#pushFilters[pushFilters] and for link:spark-sql-SupportsPushDownFilters.adoc#pushedFilters[pushedFilters]

[[contract]]
[source, scala]
----
package org.apache.spark.sql.sources

abstract class Filter {
  // only required methods that have no implementation
  // the others follow
  def references: Array[String]
}
----

.Filter Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| `references`
a| [[references]] *Column references*, i.e. list of column names that are referenced by a filter

Used when:

* `Filter` is requested to <<findReferences, find the column references in a value>>

* <<And, And>>, <<Or, Or>> and <<Not, Not>> filters are requested for the <<references, column references>>
|===

[[implementations]]
.Filters
[cols="1,2",options="header",width="100%"]
|===
| Filter
| Description

| `And`
| [[And]]

| `EqualNullSafe`
| [[EqualNullSafe]]

| `EqualTo`
| [[EqualTo]]

| `GreaterThan`
| [[GreaterThan]]

| `GreaterThanOrEqual`
| [[GreaterThanOrEqual]]

| `In`
| [[In]]

| `IsNotNull`
| [[IsNotNull]]

| `IsNull`
| [[IsNull]]

| `LessThan`
| [[LessThan]]

| `LessThanOrEqual`
| [[LessThanOrEqual]]

| `Not`
| [[Not]]

| `Or`
| [[Or]]

| `StringContains`
| [[StringContains]]

| `StringEndsWith`
| [[StringEndsWith]]

| `StringStartsWith`
| [[StringStartsWith]]
|===

=== [[findReferences]] Finding Column References in Any Value -- `findReferences` Method

[source, scala]
----
findReferences(value: Any): Array[String]
----

`findReferences` takes the <<references, references>> from the `value` filter is it is one or returns an empty array.

NOTE: `findReferences` is used when <<EqualTo, EqualTo>>, <<EqualNullSafe, EqualNullSafe>>, <<GreaterThan, GreaterThan>>, <<GreaterThanOrEqual, GreaterThanOrEqual>>, <<LessThan, LessThan>>, <<LessThanOrEqual, LessThanOrEqual>> and <<In, In>> filters are requested for their <<references, column references>>.
