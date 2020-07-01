title: StaticSQLConf

# StaticSQLConf -- Cross-Session, Immutable and Static SQL Configuration

`StaticSQLConf` holds <<properties, cross-session, immutable and static SQL configuration properties>>.

[[properties]]
.StaticSQLConf's Configuration Properties
[cols="1a",options="header",width="100%"]
|===
| Configuration Property

| [[spark.sql.catalogImplementation]][[CATALOG_IMPLEMENTATION]] *spark.sql.catalogImplementation*

(internal) Configures `in-memory` (default) or ``hive``-related link:spark-sql-BaseSessionStateBuilder.adoc[BaseSessionStateBuilder] and link:spark-sql-ExternalCatalog.adoc[ExternalCatalog]

link:spark-sql-SparkSession-Builder.adoc#enableHiveSupport[Builder.enableHiveSupport] is used to enable link:hive/index.adoc[Hive support] for a link:spark-sql-SparkSession.adoc[SparkSession].

Used when:

* `SparkSession` utility is requested for the link:spark-sql-SparkSession.adoc#sessionStateClassName[name of the BaseSessionStateBuilder implementation] (when `SparkSession` is requested for a link:spark-sql-SparkSession.adoc#sessionState[SessionState])

* `SharedState` utility is requested for the link:spark-sql-SharedState.adoc#externalCatalogClassName[name of the ExternalCatalog implementation] (when `SharedState` is requested for an link:spark-sql-SharedState.adoc#externalCatalog[ExternalCatalog])

* `SparkSession.Builder` is requested to link:spark-sql-SparkSession-Builder.adoc#enableHiveSupport[enable Hive support]

* `spark-shell` is executed

* `SetCommand` is executed (with `hive.` keys)

| [[spark.sql.debug]][[DEBUG_MODE]] *spark.sql.debug*

(internal) Only used for internal debugging when `HiveExternalCatalog` is requested to link:hive/HiveExternalCatalog.adoc#restoreTableMetadata[restoreTableMetadata].

Default: `false`

Not all functions are supported when enabled.

| [[spark.sql.extensions]][[SPARK_SESSION_EXTENSIONS]] *spark.sql.extensions*

Name of the *SQL extension configuration class* that is used to configure `SparkSession` extensions (when `Builder` is requested to <<spark-sql-SparkSession-Builder.adoc#getOrCreate, get or create a SparkSession>>). The class should implement `Function1[SparkSessionExtensions, Unit]`, and must have a no-args constructor.

Default: (empty)

| [[spark.sql.filesourceTableRelationCacheSize]][[FILESOURCE_TABLE_RELATION_CACHE_SIZE]] *spark.sql.filesourceTableRelationCacheSize*

(internal) The maximum size of the cache that maps qualified table names to table relation plans. Must not be negative.

Default: `1000`

| [[spark.sql.globalTempDatabase]][[GLOBAL_TEMP_DATABASE]] *spark.sql.globalTempDatabase*

(internal) Name of the Spark-owned internal database of global temporary views

Default: `global_temp`

Used exclusively to create a <<spark-sql-GlobalTempViewManager.adoc#creating-instance, GlobalTempViewManager>> when `SharedState` is first requested for the <<spark-sql-SharedState.adoc#globalTempViewManager, GlobalTempViewManager>>.

NOTE: The name of the internal database cannot conflict with the names of any database that is already available in <<spark-sql-SharedState.adoc#externalCatalog, ExternalCatalog>>.

| [[spark.sql.hive.thriftServer.singleSession]][[HIVE_THRIFT_SERVER_SINGLESESSION]] *spark.sql.hive.thriftServer.singleSession*

When enabled (`true`), Hive Thrift server is running in a single session mode. All the JDBC/ODBC connections share the temporary views, function registries, SQL configuration and the current database.

Default: `false`

| [[spark.sql.queryExecutionListeners]][[QUERY_EXECUTION_LISTENERS]] *spark.sql.queryExecutionListeners*

List of class names that implement <<spark-sql-QueryExecutionListener.adoc#, QueryExecutionListener>> that will be automatically <<spark-sql-ExecutionListenerManager.adoc#register, registered>> to new `SparkSessions`.

Default: (empty)

The classes should have either a no-arg constructor, or a constructor that expects a `SparkConf` argument.

| [[spark.sql.sources.schemaStringLengthThreshold]][[SCHEMA_STRING_LENGTH_THRESHOLD]] *spark.sql.sources.schemaStringLengthThreshold*

(internal) The maximum length allowed in a single cell when storing additional schema information in Hive's metastore

Default: `4000`

| [[spark.sql.ui.retainedExecutions]][[UI_RETAINED_EXECUTIONS]] *spark.sql.ui.retainedExecutions*

Number of executions to retain in the Spark UI.

Default: `1000`

| [[spark.sql.warehouse.dir]][[WAREHOUSE_PATH]] *spark.sql.warehouse.dir*

Directory of a Spark warehouse

Default: `spark-warehouse`

|===

The <<properties, properties>> in `StaticSQLConf` can only be queried and can never be changed once the first `SparkSession` is created.

[source, scala]
----
import org.apache.spark.sql.internal.StaticSQLConf
scala> val metastoreName = spark.conf.get(StaticSQLConf.CATALOG_IMPLEMENTATION.key)
metastoreName: String = hive

scala> spark.conf.set(StaticSQLConf.CATALOG_IMPLEMENTATION.key, "hive")
org.apache.spark.sql.AnalysisException: Cannot modify the value of a static config: spark.sql.catalogImplementation;
  at org.apache.spark.sql.RuntimeConfig.requireNonStaticConf(RuntimeConfig.scala:144)
  at org.apache.spark.sql.RuntimeConfig.set(RuntimeConfig.scala:41)
  ... 50 elided
----
