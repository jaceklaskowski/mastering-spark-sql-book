# RunnableCommand -- Generic Logical Command with Side Effects

`RunnableCommand` is the generic link:spark-sql-LogicalPlan-Command.adoc[logical command] that is <<run, executed>> eagerly for its side effects.

[[contract]]
[[run]]
`RunnableCommand` defines one abstract method `run` that computes a collection of link:spark-sql-Row.adoc[Row] records with the side effect, i.e. the result of executing a command.

[source, scala]
----
run(sparkSession: SparkSession): Seq[Row]
----

NOTE: `RunnableCommand` logical operator is resolved to link:spark-sql-SparkPlan-ExecutedCommandExec.adoc[ExecutedCommandExec] physical operator in link:spark-sql-SparkStrategy-BasicOperators.adoc#RunnableCommand[BasicOperators] execution planning strategy.

[NOTE]
====
`run` is executed when:

* `ExecutedCommandExec` link:spark-sql-SparkPlan-ExecutedCommandExec.adoc#sideEffectResult[executes logical RunnableCommand and caches the result as InternalRows]

* `InsertIntoHadoopFsRelationCommand` is link:spark-sql-LogicalPlan-InsertIntoHadoopFsRelationCommand.adoc#run[executed]

* `QueryExecution` is requested to link:spark-sql-QueryExecution.adoc#hiveResultString[transform the result of executing DescribeTableCommand to a Hive-compatible output format]
====

[[available-commands]]
.Available RunnableCommands
[width="100%",cols="1,2",options="header"]
|===
| RunnableCommand
| Description

| AddFileCommand
|

| AddJarCommand
|

| AlterDatabasePropertiesCommand
|

| AlterTableAddPartitionCommand
| [[AlterTableAddPartitionCommand]]

| AlterTableChangeColumnCommand
|

| AlterTableDropPartitionCommand
|

| AlterTableRecoverPartitionsCommand
|

| AlterTableRenameCommand
|

| AlterTableRenamePartitionCommand
|

| AlterTableSerDePropertiesCommand
|

| AlterTableSetLocationCommand
|

| AlterTableSetPropertiesCommand
|

| AlterTableUnsetPropertiesCommand
|

| AlterViewAsCommand
|

| link:spark-sql-LogicalPlan-AnalyzeColumnCommand.adoc[AnalyzeColumnCommand]
| [[AnalyzeColumnCommand]]

| link:spark-sql-LogicalPlan-AnalyzePartitionCommand.adoc[AnalyzePartitionCommand]
| [[AnalyzePartitionCommand]]

| link:spark-sql-LogicalPlan-AnalyzeTableCommand.adoc[AnalyzeTableCommand]
| [[AnalyzeTableCommand]]

| CacheTableCommand
a| [[CacheTableCommand]] When <<run, executed>>, `CacheTableCommand` link:spark-sql-Dataset.adoc#ofRows[creates a DataFrame] followed by link:spark-sql-dataset-operators.adoc#createTempView[registering a temporary view] for the optional `query`.

[source, scala]
----
CACHE LAZY? TABLE [table] (AS? [query])?
----

`CacheTableCommand` requests the session-specific `Catalog` to link:spark-sql-Catalog.adoc#cacheTable[cache the table].

NOTE: `CacheTableCommand` uses `SparkSession` link:spark-sql-SparkSession.adoc#catalog[to access the `Catalog`].

If the caching is not `LAZY` (which is not by default), `CacheTableCommand` link:spark-sql-SparkSession.adoc#table[creates a DataFrame for the table] and link:spark-sql-dataset-operators.adoc#count[counts the rows] (that will trigger the caching).

NOTE: `CacheTableCommand` uses a Spark SQL pattern to trigger DataFrame caching by executing `count` operation.

[source, scala]
----
val q = "CACHE TABLE ids AS SELECT * from range(5)"
scala> println(sql(q).queryExecution.logical.numberedTreeString)
00 CacheTableCommand `ids`, false
01    +- 'Project [*]
02       +- 'UnresolvedTableValuedFunction range, [5]

// ids table is already cached but let's use it anyway (and see what happens)
val q2 = "CACHE LAZY TABLE ids"
scala> println(sql(q2).queryExecution.logical.numberedTreeString)
17/05/17 06:16:39 WARN CacheManager: Asked to cache already cached data.
00 CacheTableCommand `ids`, true
----

| ClearCacheCommand
|

| CreateDatabaseCommand
|

| link:spark-sql-LogicalPlan-CreateDataSourceTableAsSelectCommand.adoc[CreateDataSourceTableAsSelectCommand]
| [[CreateDataSourceTableAsSelectCommand]] When <<run, executed>>, ...FIXME

Used exclusively when link:spark-sql-Analyzer-DataSourceAnalysis.adoc[DataSourceAnalysis] posthoc logical resolution rule resolves a link:spark-sql-LogicalPlan-CreateTable.adoc[CreateTable] logical operator with queries using non-Hive table providers (which is when `DataFrameWriter` link:spark-sql-DataFrameWriter.adoc#saveAsTable[saves a DataFrame to a non-Hive table] or for link:spark-sql-SparkSqlAstBuilder.adoc#visitCreateTable[Create Table As Select] SQL statements)

| link:spark-sql-LogicalPlan-CreateDataSourceTableCommand.adoc[CreateDataSourceTableCommand]
| [[CreateDataSourceTableCommand]]

| CreateFunctionCommand
|

| <<spark-sql-LogicalPlan-CreateTableCommand.adoc#, CreateTableCommand>>
| [[CreateTableCommand]]

| CreateTableLikeCommand
|

| <<spark-sql-LogicalPlan-CreateTempViewUsing.adoc#, CreateTempViewUsing>>
| [[CreateTempViewUsing]]

| <<spark-sql-LogicalPlan-CreateViewCommand.adoc#, CreateViewCommand>>
| [[CreateViewCommand]]

| link:spark-sql-LogicalPlan-DescribeColumnCommand.adoc[DescribeColumnCommand]
| [[DescribeColumnCommand]]

| DescribeDatabaseCommand
|

| DescribeFunctionCommand
|

| link:spark-sql-LogicalPlan-DescribeTableCommand.adoc[DescribeTableCommand]
| [[DescribeTableCommand]]

| DropDatabaseCommand
|

| DropFunctionCommand
|

| DropTableCommand
|

| ExplainCommand
|

| <<spark-sql-LogicalPlan-InsertIntoDataSourceCommand.adoc#, InsertIntoDataSourceCommand>>
| [[InsertIntoDataSourceCommand]]

| link:link:spark-sql-LogicalPlan-InsertIntoHadoopFsRelationCommand.adoc[InsertIntoHadoopFsRelationCommand]
| [[InsertIntoHadoopFsRelationCommand]]

| link:hive/InsertIntoHiveTable.adoc[InsertIntoHiveTable]
| [[InsertIntoHiveTable]]

| ListFilesCommand
|

| ListJarsCommand
|

| LoadDataCommand
|

| RefreshResource
|

| RefreshTable
|

| ResetCommand
|

| SaveIntoDataSourceCommand
| [[SaveIntoDataSourceCommand]] When <<run, executed>>, requests `DataSource` to link:spark-sql-DataSource.adoc#write[write a DataFrame to a data source per save mode].

Used exclusively when `DataFrameWriter` is requested to link:spark-sql-DataFrameWriter.adoc#save[save a DataFrame to a data source].

| SetCommand
| [[SetCommand]]

| SetDatabaseCommand
|

| ShowColumnsCommand
|

| <<spark-sql-LogicalPlan-ShowCreateTableCommand.adoc#, ShowCreateTableCommand>>
| [[ShowCreateTableCommand]]

| ShowDatabasesCommand
|

| ShowFunctionsCommand
|

| ShowPartitionsCommand
|

| ShowTablePropertiesCommand
|

| <<spark-sql-LogicalPlan-ShowTablesCommand.adoc#, ShowTablesCommand>>
| [[ShowTablesCommand]]

| StreamingExplainCommand
|

| xref:spark-sql-LogicalPlan-TruncateTableCommand.adoc[TruncateTableCommand]
| [[TruncateTableCommand]]

| UncacheTableCommand
|
|===
