title: SharedState

# SharedState -- State Shared Across SparkSessions

`SharedState` holds the <<attributes, shared state>> across multiple link:spark-sql-SparkSession.adoc#newSession[SparkSessions].

[[attributes]]
.SharedState's Properties
[cols="1,1,2",options="header",width="100%"]
|===
| Name
| Type
| Description

| `cacheManager`
| link:spark-sql-CacheManager.adoc[CacheManager]
| [[cacheManager]]

| <<externalCatalog-indepth, externalCatalog>>
| link:spark-sql-ExternalCatalog.adoc[ExternalCatalog]
a| [[externalCatalog]] Metastore of permanent relational entities, i.e. databases, tables, partitions, and functions.

NOTE: `externalCatalog` is initialized lazily on the first access.

| <<globalTempViewManager-indepth, globalTempViewManager>>
| <<spark-sql-GlobalTempViewManager.adoc#, GlobalTempViewManager>>
| [[globalTempViewManager]] Management interface of global temporary views

| `jarClassLoader`
| `NonClosableMutableURLClassLoader`
| [[jarClassLoader]]

| `sparkContext`
| `SparkContext`
| [[sparkContext]] Spark Core's `SparkContext`

| `statusStore`
| link:spark-sql-SQLAppStatusStore.adoc[SQLAppStatusStore]
| [[statusStore]]

| <<warehousePath-indepth, warehousePath>>
| `String`
| [[warehousePath]] Warehouse path
|===

`SharedState` is available as the <<spark-sql-SparkSession.adoc#sharedState, sharedState>> property of a `SparkSession`.

[source, scala]
----
scala> :type spark
org.apache.spark.sql.SparkSession

scala> :type spark.sharedState
org.apache.spark.sql.internal.SharedState
----

`SharedState` is shared across `SparkSessions`.

[source, scala]
----
scala> spark.newSession.sharedState == spark.sharedState
res1: Boolean = true
----

[[creating-instance]]
`SharedState` is created exclusively when accessed using link:spark-sql-SparkSession.adoc#sharedState[sharedState] property of `SparkSession`.

[TIP]
====
Enable `INFO` logging level for `org.apache.spark.sql.internal.SharedState` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.internal.SharedState=INFO
```

Refer to link:spark-logging.adoc[Logging].
====

=== [[warehousePath-indepth]] `warehousePath` Property

[source, scala]
----
warehousePath: String
----

`warehousePath` is the warehouse path with the value of:

. link:spark-sql-hive-metastore.adoc#hive.metastore.warehouse.dir[hive.metastore.warehouse.dir] if defined and link:spark-sql-StaticSQLConf.adoc#spark.sql.warehouse.dir[spark.sql.warehouse.dir] is not

. link:spark-sql-StaticSQLConf.adoc#spark.sql.warehouse.dir[spark.sql.warehouse.dir] if `hive.metastore.warehouse.dir` is undefined

You should see the following INFO message in the logs when `SharedState` is <<creating-instance, created>>:

```
INFO Warehouse path is '[warehousePath]'.
```

`warehousePath` is used exclusively when `SharedState` initializes <<externalCatalog, ExternalCatalog>> (and creates the default database in the metastore).

While initialized, `warehousePath` does the following:

. Loads `hive-site.xml` if available on CLASSPATH, i.e. adds it as a configuration resource to Hadoop's http://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/conf/Configuration.html[Configuration] (of `SparkContext`).

. Removes `hive.metastore.warehouse.dir` from `SparkConf` (of `SparkContext`) and leaves it off if defined using any of the Hadoop configuration resources.

. [[hive.metastore.warehouse.dir]] Sets link:spark-sql-StaticSQLConf.adoc#spark.sql.warehouse.dir[spark.sql.warehouse.dir] or link:spark-sql-hive-metastore.adoc#hive.metastore.warehouse.dir[hive.metastore.warehouse.dir] in the Hadoop configuration (of `SparkContext`)

a. If `hive.metastore.warehouse.dir` has been defined in any of the Hadoop configuration resources but link:spark-sql-StaticSQLConf.adoc#spark.sql.warehouse.dir[spark.sql.warehouse.dir] has not, `spark.sql.warehouse.dir` becomes the value of `hive.metastore.warehouse.dir`.
+
You should see the following INFO message in the logs:
+
```
spark.sql.warehouse.dir is not set, but hive.metastore.warehouse.dir is set. Setting spark.sql.warehouse.dir to the value of hive.metastore.warehouse.dir ('[hiveWarehouseDir]').
```

b. Otherwise, the Hadoop configuration's `hive.metastore.warehouse.dir` is set to `spark.sql.warehouse.dir`
+
You should see the following INFO message in the logs:
+
```
Setting hive.metastore.warehouse.dir ('[hiveWarehouseDir]') to the value of spark.sql.warehouse.dir ('[sparkWarehouseDir]').
```

=== [[externalCatalog-indepth]] `externalCatalog` Property

[source, scala]
----
externalCatalog: ExternalCatalog
----

`externalCatalog` is created reflectively per <<externalCatalogClassName, spark.sql.catalogImplementation>> internal configuration property (with the current Hadoop's http://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/conf/Configuration.html[Configuration] as `SparkContext.hadoopConfiguration`):

* link:hive/HiveExternalCatalog.adoc[HiveExternalCatalog] for `hive`
* link:spark-sql-InMemoryCatalog.adoc[InMemoryCatalog] for `in-memory` (default)

While initialized:

. link:spark-sql-ExternalCatalog.adoc#createDatabase[Creates] the *default* database (with `default database` description and <<warehousePath, warehousePath>> location) if link:spark-sql-ExternalCatalog.adoc#databaseExists[it doesn't exist].

. link:spark-sql-ExternalCatalog.adoc#addListener[Registers] a `ExternalCatalogEventListener` that propagates external catalog events to the Spark listener bus.

=== [[externalCatalogClassName]] `externalCatalogClassName` Internal Method

[source, scala]
----
externalCatalogClassName(
  conf: SparkConf): String
----

`externalCatalogClassName` gives the name of the class of the link:spark-sql-ExternalCatalog.adoc#implementations[ExternalCatalog] implementation per link:spark-sql-StaticSQLConf.adoc#spark.sql.catalogImplementation[spark.sql.catalogImplementation], i.e.

* link:hive/HiveExternalCatalog.adoc[org.apache.spark.sql.hive.HiveExternalCatalog] for `hive`
* link:spark-sql-InMemoryCatalog.adoc[org.apache.spark.sql.catalyst.catalog.InMemoryCatalog] for `in-memory`

NOTE: `externalCatalogClassName` is used exclusively when `SharedState` is requested for the <<externalCatalog, ExternalCatalog>>.

=== [[globalTempViewManager-indepth]] Accessing Management Interface of Global Temporary Views -- `globalTempViewManager` Property

[source, scala]
----
globalTempViewManager: GlobalTempViewManager
----

When accessed for the very first time, `globalTempViewManager` gets the name of the global temporary view database (as the value of <<spark-sql-StaticSQLConf.adoc#spark.sql.globalTempDatabase, spark.sql.globalTempDatabase>> internal static configuration property).

In the end, `globalTempViewManager` creates a new <<spark-sql-GlobalTempViewManager.adoc#creating-instance, GlobalTempViewManager>> (with the database name).

`globalTempViewManager` throws a `SparkException` when the global temporary view database <<spark-sql-ExternalCatalog.adoc#databaseExists, exist>> in the <<externalCatalog, ExternalCatalog>>.

```
[globalTempDB] is a system preserved database, please rename your existing database to resolve the name conflict, or set a different value for spark.sql.globalTempDatabase, and launch your Spark application again.
```

NOTE: `globalTempViewManager` is used when <<spark-sql-BaseSessionStateBuilder.adoc#catalog, BaseSessionStateBuilder>> and link:hive/HiveSessionStateBuilder.adoc#catalog[HiveSessionStateBuilder] are requested for the <<spark-sql-SessionCatalog.adoc#, SessionCatalog>>.
