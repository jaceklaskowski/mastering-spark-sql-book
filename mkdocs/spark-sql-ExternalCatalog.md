title: ExternalCatalog

# ExternalCatalog -- External Catalog (Metastore) of Permanent Relational Entities

`ExternalCatalog` is the <<contract, contract>> of an *external system catalog* (aka _metadata registry_ or _metastore_) of permanent relational entities, i.e. databases, tables, partitions, and functions.

[[features]]
.ExternalCatalog's Features per Relational Entity
[cols="2,^1,^1,^1,^1",options="header",width="100%"]
|===
| Feature
| Database
| Function
| Partition
| Table

| Alter
| <<alterDatabase, alterDatabase>>
| <<alterFunction, alterFunction>>
| <<alterPartitions, alterPartitions>>
| <<alterTable, alterTable>>, <<alterTableDataSchema, alterTableDataSchema>>, <<alterTableStats, alterTableStats>>

| Create
| <<createDatabase, createDatabase>>
| <<createFunction, createFunction>>
| <<createPartitions, createPartitions>>
| <<createTable, createTable>>

| Drop
| <<dropDatabase, dropDatabase>>
| <<dropFunction, dropFunction>>
| <<dropPartitions, dropPartitions>>
| <<dropTable, dropTable>>

| Get
| <<getDatabase, getDatabase>>
| <<getFunction, getFunction>>
| <<getPartition, getPartition>>, <<getPartitionOption, getPartitionOption>>
| <<getTable, getTable>>

| List
| <<listDatabases, listDatabases>>
| <<listFunctions, listFunctions>>
| <<listPartitionNames, listPartitionNames>>, <<listPartitions, listPartitions>>, <<listPartitionsByFilter, listPartitionsByFilter>>
| <<listTables, listTables>>

| Load
|
|
| <<loadDynamicPartitions, loadDynamicPartitions>>, <<loadPartition, loadPartition>>
| <<loadTable, loadTable>>

| Rename
|
| <<renameFunction, renameFunction>>
| <<renamePartitions, renamePartitions>>
| <<renameTable, renameTable>>

| Check Existence
| <<databaseExists, databaseExists>>
| <<functionExists, functionExists>>
|
| <<tableExists, tableExists>>

| Set
|
|
|
| <<setCurrentDatabase, setCurrentDatabase>>
|===

[[contract]]
.ExternalCatalog Contract (incl. Protected Methods)
[cols="1m,2",options="header",width="100%"]
|===
| Method
| Description

| alterPartitions
a| [[alterPartitions]]

[source, scala]
----
alterPartitions(
  db: String,
  table: String,
  parts: Seq[CatalogTablePartition]): Unit
----

| createPartitions
a| [[createPartitions]]

[source, scala]
----
createPartitions(
  db: String,
  table: String,
  parts: Seq[CatalogTablePartition],
  ignoreIfExists: Boolean): Unit
----

| databaseExists
a| [[databaseExists]]

[source, scala]
----
databaseExists(db: String): Boolean
----

| doAlterDatabase
a| [[doAlterDatabase]]

[source, scala]
----
doAlterDatabase(dbDefinition: CatalogDatabase): Unit
----

| doAlterFunction
a| [[doAlterFunction]]

[source, scala]
----
doAlterFunction(db: String, funcDefinition: CatalogFunction): Unit
----

| doAlterTable
a| [[doAlterTable]]

[source, scala]
----
doAlterTable(tableDefinition: CatalogTable): Unit
----

| doAlterTableDataSchema
a| [[doAlterTableDataSchema]]

[source, scala]
----
doAlterTableDataSchema(db: String, table: String, newDataSchema: StructType): Unit
----

| doAlterTableStats
a| [[doAlterTableStats]]

[source, scala]
----
doAlterTableStats(db: String, table: String, stats: Option[CatalogStatistics]): Unit
----

| doCreateDatabase
a| [[doCreateDatabase]]

[source, scala]
----
doCreateDatabase(dbDefinition: CatalogDatabase, ignoreIfExists: Boolean): Unit
----

| doCreateFunction
a| [[doCreateFunction]]

[source, scala]
----
doCreateFunction(db: String, funcDefinition: CatalogFunction): Unit
----

| doCreateTable
a| [[doCreateTable]]

[source, scala]
----
doCreateTable(tableDefinition: CatalogTable, ignoreIfExists: Boolean): Unit
----

| doDropDatabase
a| [[doDropDatabase]]

[source, scala]
----
doDropDatabase(db: String, ignoreIfNotExists: Boolean, cascade: Boolean): Unit
----

| doDropFunction
a| [[doDropFunction]]

[source, scala]
----
doDropFunction(db: String, funcName: String): Unit
----

| doDropTable
a| [[doDropTable]]

[source, scala]
----
doDropTable(
  db: String,
  table: String,
  ignoreIfNotExists: Boolean,
  purge: Boolean): Unit
----

| doRenameFunction
a| [[doRenameFunction]]

[source, scala]
----
doRenameFunction(db: String, oldName: String, newName: String): Unit
----

| doRenameTable
a| [[doRenameTable]]

[source, scala]
----
doRenameTable(db: String, oldName: String, newName: String): Unit
----

| dropPartitions
a| [[dropPartitions]]

[source, scala]
----
dropPartitions(
  db: String,
  table: String,
  parts: Seq[TablePartitionSpec],
  ignoreIfNotExists: Boolean,
  purge: Boolean,
  retainData: Boolean): Unit
----

| functionExists
a| [[functionExists]]

[source, scala]
----
functionExists(db: String, funcName: String): Boolean
----

| getDatabase
a| [[getDatabase]]

[source, scala]
----
getDatabase(db: String): CatalogDatabase
----

| getFunction
a| [[getFunction]]

[source, scala]
----
getFunction(db: String, funcName: String): CatalogFunction
----

| getPartition
a| [[getPartition]]

[source, scala]
----
getPartition(db: String, table: String, spec: TablePartitionSpec): CatalogTablePartition
----

| getPartitionOption
a| [[getPartitionOption]]

[source, scala]
----
getPartitionOption(
  db: String,
  table: String,
  spec: TablePartitionSpec): Option[CatalogTablePartition]
----

| getTable
a| [[getTable]]

[source, scala]
----
getTable(db: String, table: String): CatalogTable
----

| listDatabases
a| [[listDatabases]]

[source, scala]
----
listDatabases(): Seq[String]
listDatabases(pattern: String): Seq[String]
----

| listFunctions
a| [[listFunctions]]

[source, scala]
----
listFunctions(db: String, pattern: String): Seq[String]
----

| listPartitionNames
a| [[listPartitionNames]]

[source, scala]
----
listPartitionNames(
  db: String,
  table: String,
  partialSpec: Option[TablePartitionSpec] = None): Seq[String]
----

| listPartitions
a| [[listPartitions]]

[source, scala]
----
listPartitions(
  db: String,
  table: String,
  partialSpec: Option[TablePartitionSpec] = None): Seq[CatalogTablePartition]
----

| listPartitionsByFilter
a| [[listPartitionsByFilter]]

[source, scala]
----
listPartitionsByFilter(
  db: String,
  table: String,
  predicates: Seq[Expression],
  defaultTimeZoneId: String): Seq[CatalogTablePartition]
----

| listTables
a| [[listTables]]

[source, scala]
----
listTables(db: String): Seq[String]
listTables(db: String, pattern: String): Seq[String]
----

| loadDynamicPartitions
a| [[loadDynamicPartitions]]

[source, scala]
----
loadDynamicPartitions(
  db: String,
  table: String,
  loadPath: String,
  partition: TablePartitionSpec,
  replace: Boolean,
  numDP: Int): Unit
----

| loadPartition
a| [[loadPartition]]

[source, scala]
----
loadPartition(
  db: String,
  table: String,
  loadPath: String,
  partition: TablePartitionSpec,
  isOverwrite: Boolean,
  inheritTableSpecs: Boolean,
  isSrcLocal: Boolean): Unit
----

| loadTable
a| [[loadTable]]

[source, scala]
----
loadTable(
  db: String,
  table: String,
  loadPath: String,
  isOverwrite: Boolean,
  isSrcLocal: Boolean): Unit
----

| renamePartitions
a| [[renamePartitions]]

[source, scala]
----
renamePartitions(
  db: String,
  table: String,
  specs: Seq[TablePartitionSpec],
  newSpecs: Seq[TablePartitionSpec]): Unit
----

| setCurrentDatabase
a| [[setCurrentDatabase]]

[source, scala]
----
setCurrentDatabase(db: String): Unit
----

| tableExists
a| [[tableExists]]

[source, scala]
----
tableExists(db: String, table: String): Boolean
----
|===

`ExternalCatalog` is available as link:spark-sql-SharedState.adoc#externalCatalog[externalCatalog] of link:spark-sql-SparkSession.adoc#sharedState[SharedState] (in `SparkSession`).

[source, scala]
----
scala> :type spark
org.apache.spark.sql.SparkSession

scala> :type spark.sharedState.externalCatalog
org.apache.spark.sql.catalyst.catalog.ExternalCatalog
----

`ExternalCatalog` is available as ephemeral <<in-memory, in-memory>> or persistent <<hive, hive-aware>>.

[[implementations]]
.ExternalCatalogs
[cols="1,2,2",options="header",width="100%"]
|===
| ExternalCatalog
| Alias
| Description

| link:hive/HiveExternalCatalog.adoc[HiveExternalCatalog]
| [[hive]] `hive`
| A persistent system catalog using a Hive metastore.

| link:spark-sql-InMemoryCatalog.adoc[InMemoryCatalog]
| [[in-memory]] `in-memory`
| An in-memory (ephemeral) system catalog that does not require setting up external systems (like a Hive metastore).

It is intended for testing or exploration purposes only and therefore should not be used in production.
|===

The <<implementations, concrete>> `ExternalCatalog` is chosen using link:spark-sql-SparkSession-Builder.adoc#enableHiveSupport[Builder.enableHiveSupport] that enables the Hive support (and sets link:spark-sql-StaticSQLConf.adoc#spark.sql.catalogImplementation[spark.sql.catalogImplementation] configuration property to <<hive, hive>> when the Hive classes are available).

[source, scala]
----
import org.apache.spark.sql.internal.StaticSQLConf
val catalogType = spark.conf.get(StaticSQLConf.CATALOG_IMPLEMENTATION.key)
scala> println(catalogType)
hive

scala> spark.sessionState.conf.getConf(StaticSQLConf.CATALOG_IMPLEMENTATION)
res1: String = hive
----

[TIP]
====
Set `spark.sql.catalogImplementation` to `in-memory` when starting `spark-shell` to use link:spark-sql-InMemoryCatalog.adoc[InMemoryCatalog] external catalog.

[source, scala]
----
// spark-shell --conf spark.sql.catalogImplementation=in-memory

import org.apache.spark.sql.internal.StaticSQLConf
scala> spark.sessionState.conf.getConf(StaticSQLConf.CATALOG_IMPLEMENTATION)
res0: String = in-memory
----
====

[IMPORTANT]
====
You cannot change `ExternalCatalog` after `SparkSession` has been created using link:spark-sql-StaticSQLConf.adoc#spark.sql.catalogImplementation[spark.sql.catalogImplementation] configuration property as it is a static configuration.

[source, scala]
----
import org.apache.spark.sql.internal.StaticSQLConf
scala> spark.conf.set(StaticSQLConf.CATALOG_IMPLEMENTATION.key, "hive")
org.apache.spark.sql.AnalysisException: Cannot modify the value of a static config: spark.sql.catalogImplementation;
  at org.apache.spark.sql.RuntimeConfig.requireNonStaticConf(RuntimeConfig.scala:144)
  at org.apache.spark.sql.RuntimeConfig.set(RuntimeConfig.scala:41)
  ... 49 elided
----
====

[[addListener]]
`ExternalCatalog` is a `ListenerBus` of `ExternalCatalogEventListener` listeners that handle `ExternalCatalogEvent` events.

[TIP]
====
Use `addListener` and `removeListener` to register and de-register `ExternalCatalogEventListener` listeners, accordingly.

Read https://jaceklaskowski.gitbooks.io/mastering-apache-spark/spark-SparkListenerBus.html#ListenerBus[ListenerBus Event Bus Contract] in Mastering Apache Spark 2 gitbook to learn more about Spark Core's `ListenerBus` interface.
====

=== [[alterTableStats]] Altering Table Statistics -- `alterTableStats` Method

[source, scala]
----
alterTableStats(db: String, table: String, stats: Option[CatalogStatistics]): Unit
----

`alterTableStats`...FIXME

NOTE: `alterTableStats` is used exclusively when `SessionCatalog` is requested for link:spark-sql-SessionCatalog.adoc#alterTableStats[altering the statistics of a table in a metastore] (that can happen when any logical command is executed that could change the table statistics).

=== [[alterTable]] Altering Table -- `alterTable` Method

[source, scala]
----
alterTable(tableDefinition: CatalogTable): Unit
----

`alterTable`...FIXME

NOTE: `alterTable` is used exclusively when `SessionCatalog` is requested for link:spark-sql-SessionCatalog.adoc#alterTable[altering the statistics of a table in a metastore].

=== [[createTable]] `createTable` Method

[source, scala]
----
createTable(tableDefinition: CatalogTable, ignoreIfExists: Boolean): Unit
----

`createTable`...FIXME

NOTE: `createTable` is used when...FIXME

=== [[alterTableDataSchema]] `alterTableDataSchema` Method

[source, scala]
----
alterTableDataSchema(db: String, table: String, newDataSchema: StructType): Unit
----

`alterTableDataSchema`...FIXME

NOTE: `alterTableDataSchema` is used exclusively when `SessionCatalog` is requested to <<spark-sql-SessionCatalog.adoc#alterTableDataSchema, alterTableDataSchema>>.
