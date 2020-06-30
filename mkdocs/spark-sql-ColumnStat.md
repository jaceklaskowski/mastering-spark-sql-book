# ColumnStat

[[creating-instance]]
`ColumnStat` holds the <<statistics, statistics>> of a table column (as part of the link:spark-sql-CatalogStatistics.adoc[table statistics] in a metastore).

[[statistics]]
.Column Statistics
[cols="1,2",options="header",width="100%"]
|===
| Name
| Description

| [[distinctCount]] `distinctCount`
| Number of distinct values

| [[min]] `min`
| Minimum value

| [[max]] `max`
| Maximum value

| [[nullCount]] `nullCount`
| Number of `null` values

| [[avgLen]] `avgLen`
| Average length of the values

| [[maxLen]] `maxLen`
| Maximum length of the values

| [[histogram]] `histogram`
| Histogram of values (as `Histogram` which is empty by default)
|===

`ColumnStat` is computed (and <<rowToColumnStat, created from the result row>>) using link:spark-sql-cost-based-optimization.adoc#ANALYZE-TABLE[ANALYZE TABLE COMPUTE STATISTICS FOR COLUMNS] SQL command (that `SparkSqlAstBuilder` link:spark-sql-SparkSqlAstBuilder.adoc#ANALYZE-TABLE[translates] to link:spark-sql-LogicalPlan-AnalyzeColumnCommand.adoc[AnalyzeColumnCommand] logical command).

[source, scala]
----
val cols = "id, p1, p2"
val analyzeTableSQL = s"ANALYZE TABLE t1 COMPUTE STATISTICS FOR COLUMNS $cols"
spark.sql(analyzeTableSQL)
----

`ColumnStat` may optionally hold the <<histogram, histogram of values>> which is empty by default. With link:spark-sql-properties.adoc#spark.sql.statistics.histogram.enabled[spark.sql.statistics.histogram.enabled] configuration property turned on `ANALYZE TABLE COMPUTE STATISTICS FOR COLUMNS` SQL command generates column (equi-height) histograms.

NOTE: `spark.sql.statistics.histogram.enabled` is off by default.

You can inspect the column statistics using link:spark-sql-cost-based-optimization.adoc#DESCRIBE-EXTENDED[DESCRIBE EXTENDED] SQL command.

```
scala> sql("DESC EXTENDED t1 id").show
+--------------+----------+
|info_name     |info_value|
+--------------+----------+
|col_name      |id        |
|data_type     |int       |
|comment       |NULL      |
|min           |0         |
|max           |1         |
|num_nulls     |0         |
|distinct_count|2         |
|avg_col_len   |4         |
|max_col_len   |4         |
|histogram     |NULL      | <-- no histogram (spark.sql.statistics.histogram.enabled off)
+--------------+----------+
```

`ColumnStat` is part of the link:spark-sql-CatalogStatistics.adoc#colStats[statistics of a table].

[source, scala]
----
// Make sure that you ran ANALYZE TABLE (as described above)
val db = spark.catalog.currentDatabase
val tableName = "t1"
val metadata = spark.sharedState.externalCatalog.getTable(db, tableName)
val stats = metadata.stats.get

scala> :type stats
org.apache.spark.sql.catalyst.catalog.CatalogStatistics

val colStats = stats.colStats
scala> :type colStats
Map[String,org.apache.spark.sql.catalyst.plans.logical.ColumnStat]
----

`ColumnStat` is <<toMap, converted to properties>> (serialized) while persisting the table (statistics) to a metastore.

[source, scala]
----
scala> :type colStats
Map[String,org.apache.spark.sql.catalyst.plans.logical.ColumnStat]

val colName = "p1"

val p1stats = colStats(colName)
scala> :type p1stats
org.apache.spark.sql.catalyst.plans.logical.ColumnStat

import org.apache.spark.sql.types.DoubleType
val props = p1stats.toMap(colName, dataType = DoubleType)
scala> println(props)
Map(distinctCount -> 2, min -> 0.0, version -> 1, max -> 1.4, maxLen -> 8, avgLen -> 8, nullCount -> 0)
----

`ColumnStat` is <<fromMap, re-created from properties>> (deserialized) when `HiveExternalCatalog` is requested for link:hive/HiveExternalCatalog.adoc#statsFromProperties[restoring table statistics from properties] (from a Hive Metastore).

[source, scala]
----
scala> :type props
Map[String,String]

scala> println(props)
Map(distinctCount -> 2, min -> 0.0, version -> 1, max -> 1.4, maxLen -> 8, avgLen -> 8, nullCount -> 0)

import org.apache.spark.sql.types.StructField
val p1 = $"p1".double

import org.apache.spark.sql.catalyst.plans.logical.ColumnStat
val colStatsOpt = ColumnStat.fromMap(table = "t1", field = p1, map = props)

scala> :type colStatsOpt
Option[org.apache.spark.sql.catalyst.plans.logical.ColumnStat]
----

`ColumnStat` is also <<creating-instance, created>> when `JoinEstimation` is requested to link:spark-sql-JoinEstimation.adoc#estimateInnerOuterJoin[estimateInnerOuterJoin] for `Inner`, `Cross`, `LeftOuter`, `RightOuter` and `FullOuter` joins.

[source, scala]
----
val tableName = "t1"

// Make the example reproducible
import org.apache.spark.sql.catalyst.TableIdentifier
val tid = TableIdentifier(tableName)
val sessionCatalog = spark.sessionState.catalog
sessionCatalog.dropTable(tid, ignoreIfNotExists = true, purge = true)

// CREATE TABLE t1
Seq((0, 0, "zero"), (1, 1, "one")).
  toDF("id", "p1", "p2").
  write.
  saveAsTable(tableName)

// As we drop and create immediately we may face problems with unavailable partition files
// Invalidate cache
spark.sql(s"REFRESH TABLE $tableName")

// Use ANALYZE TABLE...FOR COLUMNS to compute column statistics
// that saves them in a metastore (aka an external catalog)
val df = spark.table(tableName)
val allCols = df.columns.mkString(",")
val analyzeTableSQL = s"ANALYZE TABLE t1 COMPUTE STATISTICS FOR COLUMNS $allCols"
spark.sql(analyzeTableSQL)

// Fetch the table metadata (with column statistics) from a metastore
val metastore = spark.sharedState.externalCatalog
val db = spark.catalog.currentDatabase
val tableMeta = metastore.getTable(db, table = tableName)

// The column statistics are part of the table statistics
val colStats = tableMeta.stats.get.colStats

scala> :type colStats
Map[String,org.apache.spark.sql.catalyst.plans.logical.ColumnStat]

scala> colStats.map { case (name, cs) => s"$name: $cs" }.foreach(println)
// the output may vary
id: ColumnStat(2,Some(0),Some(1),0,4,4,None)
p1: ColumnStat(2,Some(0),Some(1),0,4,4,None)
p2: ColumnStat(2,None,None,0,4,4,None)
----

NOTE: `ColumnStat` does not support <<min, minimum>> and <<max, maximum>> metrics for binary (i.e. `Array[Byte]`) and string types.

=== [[toExternalString]] Converting Value to External/Java Representation (per Catalyst Data Type) -- `toExternalString` Internal Method

[source, scala]
----
toExternalString(v: Any, colName: String, dataType: DataType): String
----

`toExternalString`...FIXME

NOTE: `toExternalString` is used exclusively when `ColumnStat` is requested for <<toMap, statistic properties>>.

=== [[supportsHistogram]] `supportsHistogram` Method

[source, scala]
----
supportsHistogram(dataType: DataType): Boolean
----

`supportsHistogram`...FIXME

NOTE: `supportsHistogram` is used when...FIXME

=== [[toMap]] Converting ColumnStat to Properties (ColumnStat Serialization) -- `toMap` Method

[source, scala]
----
toMap(colName: String, dataType: DataType): Map[String, String]
----

`toMap` converts <<statistics, ColumnStat>> to the <<toMap-properties, properties>>.

[[properties]]
.ColumnStat.toMap's Properties
[cols="1,2",options="header",width="100%"]
|===
| Key
| Value

| `version`
| `1`

| `distinctCount`
| <<distinctCount, distinctCount>>

| `nullCount`
| <<nullCount, nullCount>>

| `avgLen`
| <<avgLen, avgLen>>

| `maxLen`
| <<maxLen, maxLen>>

| `min`
| <<toExternalString, External/Java representation>> of <<min, min>>

| `max`
| <<toExternalString, External/Java representation>> of <<max, max>>

| `histogram`
| Serialized version of <<histogram, Histogram>> (using `HistogramSerializer.serialize`)
|===

NOTE: `toMap` adds `min`, `max`, `histogram` entries only if they are available.

NOTE: Interestingly, `colName` and `dataType` input parameters bring no value to `toMap` itself, but merely allow for a more user-friendly error reporting when <<toExternalString, converting>> `min` and `max` column statistics.

NOTE: `toMap` is used exclusively when `HiveExternalCatalog` is requested for link:hive/HiveExternalCatalog.adoc#statsToProperties[converting table statistics to properties] (before persisting them as part of table metadata in a Hive metastore).

=== [[fromMap]] Re-Creating Column Statistics from Properties (ColumnStat Deserialization) -- `fromMap` Method

[source, scala]
----
fromMap(table: String, field: StructField, map: Map[String, String]): Option[ColumnStat]
----

`fromMap` creates a `ColumnStat` by fetching <<properties, properties>> of every <<statistics, column statistic>> from the input `map`.

`fromMap` returns `None` when recovering column statistics fails for whatever reason.

```
WARN Failed to parse column statistics for column [fieldName] in table [table]
```

NOTE: Interestingly, `table` input parameter brings no value to `fromMap` itself, but merely allows for a more user-friendly error reporting when parsing column statistics fails.

NOTE: `fromMap` is used exclusively when `HiveExternalCatalog` is requested for link:hive/HiveExternalCatalog.adoc#statsFromProperties[restoring table statistics from properties] (from a Hive Metastore).

=== [[rowToColumnStat]] Creating Column Statistics from InternalRow (Result of Computing Column Statistics) -- `rowToColumnStat` Method

[source, scala]
----
rowToColumnStat(
  row: InternalRow,
  attr: Attribute,
  rowCount: Long,
  percentiles: Option[ArrayData]): ColumnStat
----

`rowToColumnStat` <<creating-instance, creates>> a `ColumnStat` from the input `row` and the following positions:

[start=0]
. <<distinctCount, distinctCount>>
. <<min, min>>
. <<max, max>>
. <<nullCount, nullCount>>
. <<avgLen, avgLen>>
. <<maxLen, maxLen>>

If the ``6``th field is not empty, `rowToColumnStat` uses it to create <<histogram, histogram>>.

NOTE: `rowToColumnStat` is used exclusively when `AnalyzeColumnCommand` is link:spark-sql-LogicalPlan-AnalyzeColumnCommand.adoc#run[executed] (to link:spark-sql-LogicalPlan-AnalyzeColumnCommand.adoc#computeColumnStats[compute the statistics for specified columns]).

=== [[statExprs]] `statExprs` Method

[source, scala]
----
statExprs(
  col: Attribute,
  conf: SQLConf,
  colPercentiles: AttributeMap[ArrayData]): CreateNamedStruct
----

`statExprs`...FIXME

NOTE: `statExprs` is used when...FIXME
