# DDLUtils Utility

`DDLUtils` is a helper object that...FIXME

[[HIVE_PROVIDER]]
`DDLUtils` uses `hive` value to denote link:hive/index.adoc[Hive data source] (_Hive provider_).

=== [[verifyPartitionProviderIsHive]] `verifyPartitionProviderIsHive` Utility

[source, scala]
----
verifyPartitionProviderIsHive(
  spark: SparkSession,
  table: CatalogTable,
  action: String): Unit
----

`verifyPartitionProviderIsHive` requests the given link:spark-sql-CatalogTable.adoc[CatalogTable] for the link:spark-sql-CatalogTable.adoc#identifier[TableIdentifier] that is in turn requested for the table name.

`verifyPartitionProviderIsHive` throws an `AnalysisException` when link:hive/configuration-properties.adoc#spark.sql.hive.manageFilesourcePartitions[spark.sql.hive.manageFilesourcePartitions] configuration property is disabled (`false`) and the input `CatalogTable` is a <<isDatasourceTable, data source table>>:

```
[action] is not allowed on [tableName] since filesource partition management is disabled (spark.sql.hive.manageFilesourcePartitions = false).
```

`verifyPartitionProviderIsHive` throws an `AnalysisException` when the link:spark-sql-CatalogTable.adoc#tracksPartitionsInCatalog[tracksPartitionsInCatalog] of the given `CatalogTable` is disabled (`false`) and the input `CatalogTable` is a <<isDatasourceTable, data source table>>:

```
[action] is not allowed on [tableName] since its partition metadata is not stored in the Hive metastore. To import this information into the metastore, run `msck repair table [tableName]`
```

NOTE: `verifyPartitionProviderIsHive` is used when `AlterTableAddPartitionCommand`, `AlterTableRenamePartitionCommand`, `AlterTableDropPartitionCommand`, `AlterTableSetLocationCommand`, link:spark-sql-LogicalPlan-TruncateTableCommand.adoc[TruncateTableCommand], link:spark-sql-LogicalPlan-DescribeTableCommand.adoc[DescribeTableCommand], and `ShowPartitionsCommand` commands are executed.

=== [[isDatasourceTable]] `isDatasourceTable` Utility

[source, scala]
----
isDatasourceTable(
  table: CatalogTable): Boolean
----

`isDatasourceTable` is positive (`true`) when the link:spark-sql-CatalogTable.adoc#provider[provider] of the input link:spark-sql-CatalogTable.adoc[CatalogTable] is not <<HIVE_PROVIDER, hive>> when defined. Otherwise, `isDatasourceTable` is negative (`false`).

[NOTE]
====
`isDatasourceTable` is used when:

* `HiveExternalCatalog` is requested to link:hive/HiveExternalCatalog.adoc#createTable[createTable] (and link:hive/HiveExternalCatalog.adoc#saveTableIntoHive[saveTableIntoHive])

* `HiveUtils` utility is used to link:hive/HiveUtils.adoc#inferSchema[inferSchema]

* `AlterTableSerDePropertiesCommand`, `AlterTableAddColumnsCommand`, `LoadDataCommand`, link:spark-sql-LogicalPlan-ShowCreateTableCommand.adoc[ShowCreateTableCommand] commands are executed

* `DDLUtils` utility is used to <<verifyPartitionProviderIsHive, verifyPartitionProviderIsHive>>

* link:spark-sql-Analyzer-DataSourceAnalysis.adoc[DataSourceAnalysis] and link:spark-sql-Analyzer-FindDataSourceTable.adoc[FindDataSourceTable] logical rules are executed
====

=== [[isHiveTable]] `isHiveTable` Utility

[source, scala]
----
isHiveTable(
  provider: Option[String]): Boolean
----

`isHiveTable` is positive (`true`) when the link:spark-sql-CatalogTable.adoc#provider[provider] is <<HIVE_PROVIDER, hive>> when defined. Otherwise, `isHiveTable` is negative (`false`).

[NOTE]
====
`isHiveTable` is used when:

* link:hive/HiveAnalysis.adoc[HiveAnalysis] logical resolution rule is executed

* `DDLUtils` utility is used to <<isHiveTable, isHiveTable>>

* `HiveOnlyCheck` extended check rule is executed
====

=== [[verifyNotReadPath]] `verifyNotReadPath` Utility

[source, scala]
----
verifyNotReadPath(
  query: LogicalPlan,
  outputPath: Path) : Unit
----

`verifyNotReadPath`...FIXME

[NOTE]
====
`verifyNotReadPath` is used when...FIXME
====
