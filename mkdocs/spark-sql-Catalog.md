title: Catalog

# Catalog -- Metastore Management Interface

`Catalog` is the <<contract, interface>> for managing a *metastore* (aka _metadata catalog_) of relational entities (e.g. database(s), tables, functions, table columns and temporary views).

`Catalog` is available using link:spark-sql-SparkSession.adoc#catalog[SparkSession.catalog] property.

[source, scala]
----
scala> :type spark
org.apache.spark.sql.SparkSession

scala> :type spark.catalog
org.apache.spark.sql.catalog.Catalog
----

[[contract]]
.Catalog Contract
[cols="1m,2",options="header",width="100%"]
|===
| Method
| Description

| cacheTable
a| [[cacheTable]]

[source, scala]
----
cacheTable(tableName: String): Unit
cacheTable(tableName: String, storageLevel: StorageLevel): Unit
----

Caches the specified table in memory

Used for SQL's link:spark-sql-caching-and-persistence.adoc#cache-table[CACHE TABLE] and `AlterTableRenameCommand` command.

| clearCache
a| [[clearCache]]

[source, scala]
----
clearCache(): Unit
----

| createTable
a| [[createTable]]

[source, scala]
----
createTable(tableName: String, path: String): DataFrame
createTable(
  tableName: String,
  source: String,
  options: java.util.Map[String, String]): DataFrame
createTable(
  tableName: String,
  source: String,
  options: Map[String, String]): DataFrame
createTable(tableName: String, path: String, source: String): DataFrame
createTable(
  tableName: String,
  source: String,
  schema: StructType,
  options: java.util.Map[String, String]): DataFrame
createTable(
  tableName: String,
  source: String,
  schema: StructType,
  options: Map[String, String]): DataFrame
----

| currentDatabase
a| [[currentDatabase]]

[source, scala]
----
currentDatabase: String
----

| databaseExists
a| [[databaseExists]]

[source, scala]
----
databaseExists(dbName: String): Boolean
----

| dropGlobalTempView
a| [[dropGlobalTempView]]

[source, scala]
----
dropGlobalTempView(viewName: String): Boolean
----

| dropTempView
a| [[dropTempView]]

[source, scala]
----
dropTempView(viewName: String): Boolean
----

| functionExists
a| [[functionExists]]

[source, scala]
----
functionExists(functionName: String): Boolean
functionExists(dbName: String, functionName: String): Boolean
----

| getDatabase
a| [[getDatabase]]

[source, scala]
----
getDatabase(dbName: String): Database
----

| getFunction
a| [[getFunction]]

[source, scala]
----
getFunction(functionName: String): Function
getFunction(dbName: String, functionName: String): Function
----

| getTable
a| [[getTable]]

[source, scala]
----
getTable(tableName: String): Table
getTable(dbName: String, tableName: String): Table
----

| isCached
a| [[isCached]]

[source, scala]
----
isCached(tableName: String): Boolean
----

| listColumns
a| [[listColumns]]

[source, scala]
----
listColumns(tableName: String): Dataset[Column]
listColumns(dbName: String, tableName: String): Dataset[Column]
----

| listDatabases
a| [[listDatabases]]

[source, scala]
----
listDatabases(): Dataset[Database]
----

| listFunctions
a| [[listFunctions]]

[source, scala]
----
listFunctions(): Dataset[Function]
listFunctions(dbName: String): Dataset[Function]
----

| listTables
a| [[listTables]]

[source, scala]
----
listTables(): Dataset[Table]
listTables(dbName: String): Dataset[Table]
----

| recoverPartitions
a| [[recoverPartitions]]

[source, scala]
----
recoverPartitions(tableName: String): Unit
----

| refreshByPath
a| [[refreshByPath]]

[source, scala]
----
refreshByPath(path: String): Unit
----

| refreshTable
a| [[refreshTable]]

[source, scala]
----
refreshTable(tableName: String): Unit
----

| setCurrentDatabase
a| [[setCurrentDatabase]]

[source, scala]
----
setCurrentDatabase(dbName: String): Unit
----

| tableExists
a| [[tableExists]]

[source, scala]
----
tableExists(tableName: String): Boolean
tableExists(dbName: String, tableName: String): Boolean
----

| uncacheTable
a| [[uncacheTable]]

[source, scala]
----
uncacheTable(
  tableName: String): Unit
----

|===

NOTE: <<spark-sql-CatalogImpl.adoc#, CatalogImpl>> is the one and only known implementation of the <<contract, Catalog Contract>> in Apache Spark.
