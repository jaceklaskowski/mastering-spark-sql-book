# CatalogImpl

`CatalogImpl` is the link:spark-sql-Catalog.adoc[Catalog] in Spark SQL that...FIXME

.CatalogImpl uses SessionCatalog (through SparkSession)
image::images/spark-sql-CatalogImpl.png[align="center"]

NOTE: `CatalogImpl` is in `org.apache.spark.sql.internal` package.

=== [[createTable]] Creating Table -- `createTable` Method

[source, scala]
----
createTable(
  tableName: String,
  source: String,
  schema: StructType,
  options: Map[String, String]): DataFrame
----

NOTE: `createTable` is part of link:spark-sql-Catalog.adoc#createTable[Catalog Contract] to...FIXME.

`createTable`...FIXME

=== [[getTable]] `getTable` Method

[source, scala]
----
getTable(tableName: String): Table
getTable(dbName: String, tableName: String): Table
----

NOTE: `getTable` is part of link:spark-sql-Catalog.adoc#getTable[Catalog Contract] to...FIXME.

`getTable`...FIXME

=== [[getFunction]] `getFunction` Method

[source, scala]
----
getFunction(
  functionName: String): Function
getFunction(
  dbName: String,
  functionName: String): Function
----

NOTE: `getFunction` is part of link:spark-sql-Catalog.adoc#getFunction[Catalog Contract] to...FIXME.

`getFunction`...FIXME

=== [[functionExists]] `functionExists` Method

[source, scala]
----
functionExists(
  functionName: String): Boolean
functionExists(
  dbName: String,
  functionName: String): Boolean
----

NOTE: `functionExists` is part of link:spark-sql-Catalog.adoc#functionExists[Catalog Contract] to...FIXME.

`functionExists`...FIXME

=== [[cacheTable]] Caching Table or View In-Memory -- `cacheTable` Method

[source, scala]
----
cacheTable(tableName: String): Unit
----

Internally, `cacheTable` first link:spark-sql-SparkSession.adoc#table[creates a DataFrame for the table] followed by requesting `CacheManager` to link:spark-sql-CacheManager.adoc#cacheQuery[cache it].

NOTE: `cacheTable` uses the link:spark-sql-SparkSession.adoc#sharedState[session-scoped SharedState] to access the `CacheManager`.

NOTE: `cacheTable` is part of link:spark-sql-Catalog.adoc#contract[Catalog contract].

=== [[clearCache]] Removing All Cached Tables From In-Memory Cache -- `clearCache` Method

[source, scala]
----
clearCache(): Unit
----

`clearCache` requests `CacheManager` to link:spark-sql-CacheManager.adoc#clearCache[remove all cached tables from in-memory cache].

NOTE: `clearCache` is part of link:spark-sql-Catalog.adoc#contract[Catalog contract].

=== [[createExternalTable]] Creating External Table From Path -- `createExternalTable` Method

[source, scala]
----
createExternalTable(tableName: String, path: String): DataFrame
createExternalTable(tableName: String, path: String, source: String): DataFrame
createExternalTable(
  tableName: String,
  source: String,
  options: Map[String, String]): DataFrame
createExternalTable(
  tableName: String,
  source: String,
  schema: StructType,
  options: Map[String, String]): DataFrame
----

`createExternalTable` creates an external table `tableName` from the given `path` and returns the corresponding link:spark-sql-DataFrame.adoc[DataFrame].

[source, scala]
----
import org.apache.spark.sql.SparkSession
val spark: SparkSession = ...

val readmeTable = spark.catalog.createExternalTable("readme", "README.md", "text")
readmeTable: org.apache.spark.sql.DataFrame = [value: string]

scala> spark.catalog.listTables.filter(_.name == "readme").show
+------+--------+-----------+---------+-----------+
|  name|database|description|tableType|isTemporary|
+------+--------+-----------+---------+-----------+
|readme| default|       null| EXTERNAL|      false|
+------+--------+-----------+---------+-----------+

scala> sql("select count(*) as count from readme").show(false)
+-----+
|count|
+-----+
|99   |
+-----+
----

The `source` input parameter is the name of the data source provider for the table, e.g. parquet, json, text. If not specified, `createExternalTable` uses link:spark-sql-properties.adoc#spark.sql.sources.default[spark.sql.sources.default] setting to know the data source format.

NOTE: `source` input parameter must not be `hive` as it leads to a `AnalysisException`.

`createExternalTable` sets the mandatory `path` option when specified explicitly in the input parameter list.

`createExternalTable` parses `tableName` into `TableIdentifier` (using link:spark-sql-SparkSqlParser.adoc[SparkSqlParser]). It creates a link:spark-sql-CatalogTable.adoc[CatalogTable] and then link:spark-sql-SessionState.adoc#executePlan[executes] (by link:spark-sql-QueryExecution.adoc#toRdd[toRDD]) a link:spark-sql-LogicalPlan-CreateTable.adoc[CreateTable] logical plan. The result link:spark-sql-DataFrame.adoc[DataFrame] is a `Dataset[Row]` with the link:spark-sql-QueryExecution.adoc[QueryExecution] after executing link:spark-sql-LogicalPlan-SubqueryAlias.adoc[SubqueryAlias] logical plan and link:spark-sql-RowEncoder.adoc[RowEncoder].

.CatalogImpl.createExternalTable
image::images/spark-sql-CatalogImpl-createExternalTable.png[align="center"]

NOTE: `createExternalTable` is part of link:spark-sql-Catalog.adoc#contract[Catalog contract].

=== [[listTables]] Listing Tables in Database (as Dataset) -- `listTables` Method

[source, scala]
----
listTables(): Dataset[Table]
listTables(dbName: String): Dataset[Table]
----

NOTE: `listTables` is part of link:spark-sql-Catalog.adoc#listTables[Catalog Contract] to get a list of tables in the specified database.

Internally, `listTables` requests <<sessionCatalog, SessionCatalog>> to link:spark-sql-SessionCatalog.adoc#listTables[list all tables] in the specified `dbName` database and <<makeTable, converts them to Tables>>.

In the end, `listTables` <<makeDataset, creates a Dataset>> with the tables.

=== [[listColumns]] Listing Columns of Table (as Dataset) -- `listColumns` Method

[source, scala]
----
listColumns(tableName: String): Dataset[Column]
listColumns(dbName: String, tableName: String): Dataset[Column]
----

NOTE: `listColumns` is part of link:spark-sql-Catalog.adoc#listColumns[Catalog Contract] to...FIXME.

`listColumns` requests <<sessionCatalog, SessionCatalog>> for the link:spark-sql-SessionCatalog.adoc#getTempViewOrPermanentTableMetadata[table metadata].

`listColumns` takes the link:spark-sql-CatalogTable.adoc#schema[schema] from the table metadata and creates a `Column` for every field (with the optional comment as the description).

In the end, `listColumns` <<makeDataset, creates a Dataset>> with the columns.

=== [[makeTable]] Converting TableIdentifier to Table -- `makeTable` Internal Method

[source, scala]
----
makeTable(tableIdent: TableIdentifier): Table
----

`makeTable` creates a `Table` using the input `TableIdentifier` and the link:spark-sql-SessionCatalog.adoc#getTempViewOrPermanentTableMetadata[table metadata] (from the current link:spark-sql-SessionCatalog.adoc[SessionCatalog]) if available.

NOTE: `makeTable` uses <<sparkSession, SparkSession>> to access link:spark-sql-SessionState.adoc#sessionState[SessionState] that is then used to access link:spark-sql-SessionState.adoc#catalog[SessionCatalog].

NOTE: `makeTable` is used when `CatalogImpl` is requested to <<listTables, listTables>> or <<getTable, getTable>>.

=== [[makeDataset]] Creating Dataset from DefinedByConstructorParams Data -- `makeDataset` Method

[source, scala]
----
makeDataset[T <: DefinedByConstructorParams](
  data: Seq[T],
  sparkSession: SparkSession): Dataset[T]
----

`makeDataset` creates an link:spark-sql-ExpressionEncoder.adoc#apply[ExpressionEncoder] (from link:spark-sql-ExpressionEncoder.adoc#DefinedByConstructorParams[DefinedByConstructorParams]) and link:spark-sql-ExpressionEncoder.adoc#toRow[encodes] elements of the input `data` to <<spark-sql-InternalRow.adoc#, internal binary rows>>.

`makeDataset` then creates a link:spark-sql-LogicalPlan-LocalRelation.adoc#creating-instance[LocalRelation] logical operator. `makeDataset` requests `SessionState` to link:spark-sql-SessionState.adoc#executePlan[execute the plan] and link:spark-sql-Dataset.adoc#creating-instance[creates] the result `Dataset`.

NOTE: `makeDataset` is used when `CatalogImpl` is requested to <<listDatabases, list databases>>, <<listTables, tables>>, <<listFunctions, functions>> and <<listColumns, columns>>

=== [[refreshTable]] Refreshing Analyzed Logical Plan of Table Query and Re-Caching It -- `refreshTable` Method

[source, scala]
----
refreshTable(tableName: String): Unit
----

NOTE: `refreshTable` is part of link:spark-sql-Catalog.adoc#refreshTable[Catalog Contract] to...FIXME.

`refreshTable` requests `SessionState` for the link:spark-sql-SessionState.adoc#sqlParser[SQL parser] to link:spark-sql-ParserInterface.adoc#parseTableIdentifier[parse a TableIdentifier given the table name].

NOTE: `refreshTable` uses <<sparkSession, SparkSession>> to access the link:spark-sql-SparkSession.adoc#sessionState[SessionState].

`refreshTable` requests <<sessionCatalog, SessionCatalog>> for the link:spark-sql-SessionCatalog.adoc#getTempViewOrPermanentTableMetadata[table metadata].

`refreshTable` then link:spark-sql-SparkSession.adoc#table[creates a DataFrame for the table name].

For a temporary or persistent `VIEW` table, `refreshTable` requests the link:spark-sql-QueryExecution.adoc#analyzed[analyzed] logical plan of the DataFrame (for the table) to link:spark-sql-LogicalPlan.adoc#refresh[refresh] itself.

For other types of table, `refreshTable` requests <<sessionCatalog, SessionCatalog>> for link:spark-sql-SessionCatalog.adoc#refreshTable[refreshing the table metadata] (i.e. invalidating the table).

If the table <<isCached, has been cached>>, `refreshTable` requests `CacheManager` to link:spark-sql-CacheManager.adoc#uncacheQuery[uncache] and link:spark-sql-CacheManager.adoc#cacheQuery[cache] the table `DataFrame` again.

NOTE: `refreshTable` uses <<sparkSession, SparkSession>> to access the link:spark-sql-SparkSession.adoc#sharedState[SharedState] that is used to access link:spark-sql-SharedState.adoc#cacheManager[CacheManager].

=== [[refreshByPath]] `refreshByPath` Method

[source, scala]
----
refreshByPath(resourcePath: String): Unit
----

NOTE: `refreshByPath` is part of link:spark-sql-Catalog.adoc#refreshByPath[Catalog Contract] to...FIXME.

`refreshByPath`...FIXME

=== [[dropGlobalTempView]] `dropGlobalTempView` Method

[source, scala]
----
dropGlobalTempView(
  viewName: String): Boolean
----

NOTE: `dropGlobalTempView` is part of link:spark-sql-Catalog.adoc#dropGlobalTempView[Catalog] contract].

`dropGlobalTempView`...FIXME

=== [[listColumns-internal]] `listColumns` Internal Method

[source, scala]
----
listColumns(tableIdentifier: TableIdentifier): Dataset[Column]
----

`listColumns`...FIXME

NOTE: `listColumns` is used exclusively when `CatalogImpl` is requested to <<listColumns, listColumns>>.
