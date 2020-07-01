title: CatalogTable

# CatalogTable -- Table Specification (Metadata)

`CatalogTable` is the *specification* (_metadata_) of a table.

`CatalogTable` is stored in a xref:spark-sql-SessionCatalog.adoc[SessionCatalog] (session-scoped catalog of relational entities).

[source, scala]
----
scala> :type spark.sessionState.catalog
org.apache.spark.sql.catalyst.catalog.SessionCatalog

// Using high-level user-friendly catalog interface
scala> spark.catalog.listTables.filter($"name" === "t1").show
+----+--------+-----------+---------+-----------+
|name|database|description|tableType|isTemporary|
+----+--------+-----------+---------+-----------+
|  t1| default|       null|  MANAGED|      false|
+----+--------+-----------+---------+-----------+

// Using low-level internal SessionCatalog interface to access CatalogTables
val t1Tid = spark.sessionState.sqlParser.parseTableIdentifier("t1")
val t1Metadata = spark.sessionState.catalog.getTempViewOrPermanentTableMetadata(t1Tid)
scala> :type t1Metadata
org.apache.spark.sql.catalyst.catalog.CatalogTable
----

`CatalogTable` is <<creating-instance, created>> when:

* `SessionCatalog` is requested for a link:spark-sql-SessionCatalog.adoc#getTempViewOrPermanentTableMetadata[table metadata]

* `HiveClientImpl` is requested for link:hive/HiveClientImpl.adoc#getTableOption[looking up a table in a metastore]

* `DataFrameWriter` is requested to link:spark-sql-DataFrameWriter.adoc#createTable[create a table]

* link:hive/InsertIntoHiveDirCommand.adoc[InsertIntoHiveDirCommand] logical command is executed

* `SparkSqlAstBuilder` does link:spark-sql-SparkSqlAstBuilder.adoc#visitCreateTable[visitCreateTable] and link:spark-sql-SparkSqlAstBuilder.adoc#visitCreateHiveTable[visitCreateHiveTable]

* `CreateTableLikeCommand` logical command is executed

* `CreateViewCommand` logical command is <<spark-sql-LogicalPlan-CreateViewCommand.adoc#run, executed>> (and <<spark-sql-LogicalPlan-CreateViewCommand.adoc#prepareTable, prepareTable>>)

* `CatalogImpl` is requested to link:spark-sql-CatalogImpl.adoc#createTable[createTable]

[[simpleString]]
The *readable text representation* of a `CatalogTable` (aka `simpleString`) is...FIXME

NOTE: `simpleString` is used exclusively when `ShowTablesCommand` logical command is <<spark-sql-LogicalPlan-ShowTablesCommand.adoc#run, executed>> (with a partition specification).

[[toString]]
`CatalogTable` uses the following *text representation* (i.e. `toString`)...FIXME

`CatalogTable` is <<creating-instance, created>> with the optional <<bucketSpec, bucketing specification>> that is used for the following:

* `CatalogImpl` is requested to <<spark-sql-CatalogImpl.adoc#listColumns-internal, list the columns of a table>>

* `FindDataSourceTable` logical evaluation rule is requested to <<spark-sql-Analyzer-FindDataSourceTable.adoc#readDataSourceTable, readDataSourceTable>> (when <<spark-sql-Analyzer-FindDataSourceTable.adoc#apply, executed>> for data source tables)

* `CreateTableLikeCommand` logical command is executed

* `DescribeTableCommand` logical command is requested to <<spark-sql-LogicalPlan-DescribeTableCommand.adoc#run, describe detailed partition and storage information>> (when <<spark-sql-LogicalPlan-DescribeTableCommand.adoc#run, executed>>)

* <<spark-sql-LogicalPlan-ShowCreateTableCommand.adoc#, ShowCreateTableCommand>> logical command is executed

* <<spark-sql-LogicalPlan-CreateDataSourceTableCommand.adoc#run, CreateDataSourceTableCommand>> and <<spark-sql-LogicalPlan-CreateDataSourceTableAsSelectCommand.adoc#run, CreateDataSourceTableAsSelectCommand>> logical commands are executed

* `CatalogTable` is requested to <<toLinkedHashMap, convert itself to LinkedHashMap>>

* `HiveExternalCatalog` is requested to link:hive/HiveExternalCatalog.adoc#doCreateTable[doCreateTable], link:hive/HiveExternalCatalog.adoc#tableMetaToTableProps[tableMetaToTableProps], link:hive/HiveExternalCatalog.adoc#doAlterTable[doAlterTable], link:hive/HiveExternalCatalog.adoc#restoreHiveSerdeTable[restoreHiveSerdeTable] and link:hive/HiveExternalCatalog.adoc#restoreDataSourceTable[restoreDataSourceTable]

* `HiveClientImpl` is requested to link:hive/HiveClientImpl.adoc#getTableOption[retrieve a table metadata if available]>> and link:hive/HiveClientImpl.adoc#toHiveTable[toHiveTable]

* link:hive/InsertIntoHiveTable.adoc[InsertIntoHiveTable] logical command is executed

* `DataFrameWriter` is requested to <<spark-sql-DataFrameWriter.adoc#createTable, create a table>> (via <<spark-sql-DataFrameWriter.adoc#saveAsTable, saveAsTable>>)

* `SparkSqlAstBuilder` is requested to <<spark-sql-SparkSqlAstBuilder.adoc#visitCreateTable, visitCreateTable>> and <<spark-sql-SparkSqlAstBuilder.adoc#visitCreateHiveTable, visitCreateHiveTable>>

=== [[creating-instance]] Creating CatalogTable Instance

`CatalogTable` takes the following to be created:

* [[identifier]] `TableIdentifier`
* [[tableType]] <<CatalogTableType, Table type>>
* [[storage]] link:spark-sql-CatalogStorageFormat.adoc[CatalogStorageFormat]
* [[schema]] link:spark-sql-StructType.adoc[Schema]
* [[provider]] Name of the table provider (optional)
* [[partitionColumnNames]] Partition column names
* [[bucketSpec]] Optional <<spark-sql-BucketSpec.adoc#, Bucketing specification>> (default: `None`)
* [[owner]] Owner
* [[createTime]] Create time
* [[lastAccessTime]] Last access time
* [[createVersion]] Create version
* [[properties]] Properties
* [[stats]] Optional link:spark-sql-CatalogStatistics.adoc[table statistics]
* [[viewText]] Optional view text
* [[comment]] Optional comment
* [[unsupportedFeatures]] Unsupported features
* [[tracksPartitionsInCatalog]] `tracksPartitionsInCatalog` flag
* [[schemaPreservesCase]] `schemaPreservesCase` flag
* [[ignoredProperties]] Ignored properties

=== [[CatalogTableType]] Table Type

The type of a table (`CatalogTableType`) can be one of the following:

* `EXTERNAL` for external tables (link:hive/HiveClientImpl.adoc#getTableOption[EXTERNAL_TABLE] in Hive)
* `MANAGED` for managed tables (link:hive/HiveClientImpl.adoc#getTableOption[MANAGED_TABLE] in Hive)
* `VIEW` for views (link:hive/HiveClientImpl.adoc#getTableOption[VIRTUAL_VIEW] in Hive)

`CatalogTableType` is included when a `TreeNode` is requested for a xref:spark-sql-catalyst-TreeNode.adoc#shouldConvertToJson[JSON representation] for...FIXME

=== [[stats-metadata]] Table Statistics for Query Planning (Auto Broadcast Joins and Cost-Based Optimization)

You manage a table metadata using the link:spark-sql-Catalog.adoc[catalog] interface (aka _metastore_). Among the management tasks is to get the <<stats, statistics>> of a table (that are used for link:spark-sql-cost-based-optimization.adoc[cost-based query optimization]).

[source, scala]
----
scala> t1Metadata.stats.foreach(println)
CatalogStatistics(714,Some(2),Map(p1 -> ColumnStat(2,Some(0),Some(1),0,4,4,None), id -> ColumnStat(2,Some(0),Some(1),0,4,4,None)))

scala> t1Metadata.stats.map(_.simpleString).foreach(println)
714 bytes, 2 rows
----

NOTE: The <<stats, CatalogStatistics>> are optional when `CatalogTable` is <<creating-instance, created>>.

CAUTION: FIXME When are stats specified? What if there are not?

Unless <<stats, CatalogStatistics>> are available in a table metadata (in a catalog) for a non-streaming link:spark-sql-FileFormat.adoc[file data source table], `DataSource` link:spark-sql-DataSource.adoc#resolveRelation[creates] a `HadoopFsRelation` with the table size specified by link:spark-sql-properties.adoc#spark.sql.defaultSizeInBytes[spark.sql.defaultSizeInBytes] internal property (default: `Long.MaxValue`) for query planning of joins (and possibly to auto broadcast the table).

Internally, Spark alters table statistics using link:spark-sql-ExternalCatalog.adoc#doAlterTableStats[ExternalCatalog.doAlterTableStats].

Unless <<stats, CatalogStatistics>> are available in a table metadata (in a catalog) for `HiveTableRelation` (and `hive` provider) `DetermineTableStats` logical resolution rule can compute the table size using HDFS (if link:spark-sql-properties.adoc#spark.sql.statistics.fallBackToHdfs[spark.sql.statistics.fallBackToHdfs] property is turned on) or assume link:spark-sql-properties.adoc#spark.sql.defaultSizeInBytes[spark.sql.defaultSizeInBytes] (that effectively disables table broadcasting).

When requested to link:hive/HiveClientImpl.adoc#getTableOption[look up a table in a metastore], `HiveClientImpl` link:hive/HiveClientImpl.adoc#readHiveStats[reads table or partition statistics directly from a Hive metastore].

You can use link:spark-sql-LogicalPlan-AnalyzeColumnCommand.adoc[AnalyzeColumnCommand], link:spark-sql-LogicalPlan-AnalyzePartitionCommand.adoc[AnalyzePartitionCommand], link:spark-sql-LogicalPlan-AnalyzeTableCommand.adoc[AnalyzeTableCommand] commands to record statistics in a catalog.

The table statistics can be link:spark-sql-CommandUtils.adoc#updateTableStats[automatically updated] (after executing commands like `AlterTableAddPartitionCommand`) when link:spark-sql-properties.adoc#spark.sql.statistics.size.autoUpdate.enabled[spark.sql.statistics.size.autoUpdate.enabled] property is turned on.

You can use `DESCRIBE` SQL command to show the histogram of a column if stored in a catalog.

=== [[dataSchema]] `dataSchema` Method

[source, scala]
----
dataSchema: StructType
----

`dataSchema`...FIXME

NOTE: `dataSchema` is used when...FIXME

=== [[partitionSchema]] `partitionSchema` Method

[source, scala]
----
partitionSchema: StructType
----

`partitionSchema`...FIXME

NOTE: `partitionSchema` is used when...FIXME

=== [[toLinkedHashMap]] Converting Table Specification to LinkedHashMap -- `toLinkedHashMap` Method

[source, scala]
----
toLinkedHashMap: mutable.LinkedHashMap[String, String]
----

`toLinkedHashMap` converts the table specification to a collection of pairs (`LinkedHashMap[String, String]`) with the following fields and their values:

* *Database* with the database of the <<identifier, TableIdentifier>>

* *Table* with the table of the <<identifier, TableIdentifier>>

* *Owner* with the <<owner, owner>> (if defined)

* *Created Time* with the <<createTime, createTime>>

* *Created By* with `Spark` and the <<createVersion, createVersion>>

* *Type* with the name of the <<tableType, CatalogTableType>>

* *Provider* with the <<provider, provider>> (if defined)

* <<spark-sql-BucketSpec.adoc#toLinkedHashMap, Bucket specification>> (of the <<bucketSpec, BucketSpec>> if defined)

* *Comment* with the <<comment, comment>> (if defined)

* *View Text*, *View Default Database* and *View Query Output Columns* for <<tableType, VIEW table type>>

* *Table Properties* with the <<tableProperties, tableProperties>> (if not empty)

* *Statistics* with the <<stats, CatalogStatistics>> (if defined)

* <<spark-sql-CatalogStorageFormat.adoc#toLinkedHashMap, Storage specification>> (of the <<storage, CatalogStorageFormat>> if defined)

* *Partition Provider* with *Catalog* if the <<tracksPartitionsInCatalog, tracksPartitionsInCatalog>> flag is on

* *Partition Columns* with the <<partitionColumns, partitionColumns>> (if not empty)

* *Schema* with the <<schema, schema>> (if not empty)

[NOTE]
====
`toLinkedHashMap` is used when:

* `DescribeTableCommand` is requested to link:spark-sql-LogicalPlan-DescribeTableCommand.adoc#describeFormattedTableInfo[describeFormattedTableInfo] (when `DescribeTableCommand` is requested to link:spark-sql-LogicalPlan-DescribeTableCommand.adoc#run[run] for a non-temporary table and the link:spark-sql-LogicalPlan-DescribeTableCommand.adoc#isExtended[isExtended] flag on)

* `CatalogTable` is requested for either a <<simpleString, simple>> or a <<toString, catalog>> text representation
====

=== [[database]] `database` Method

[source, scala]
----
database: String
----

`database` simply returns the database (of the <<identifier, TableIdentifier>>) or throws an `AnalysisException`:

```
table [identifier] did not specify database
```

NOTE: `database` is used when...FIXME
