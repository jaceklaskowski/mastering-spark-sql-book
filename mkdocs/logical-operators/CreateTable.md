title: CreateTable

# CreateTable Logical Operator

`CreateTable` is a xref:spark-sql-LogicalPlan.adoc[logical operator] that represents (is <<creating-instance, created>> for) the following:

* `DataFrameWriter` is requested to xref:spark-sql-DataFrameWriter.adoc#createTable[create a table] (for xref:spark-sql-DataFrameWriter.adoc#saveAsTable[DataFrameWriter.saveAsTable] operator)

* `SparkSqlAstBuilder` is requested to xref:spark-sql-SparkSqlAstBuilder.adoc#visitCreateTable[visitCreateTable] (for `CREATE TABLE` SQL command) or xref:spark-sql-SparkSqlAstBuilder.adoc#visitCreateHiveTable[visitCreateHiveTable] (for `CREATE EXTERNAL TABLE` SQL command)

* `CatalogImpl` is requested to xref:spark-sql-CatalogImpl.adoc#createTable[create a table] (for xref:spark-sql-Catalog.adoc#createTable[Catalog.createTable] operator)

`CreateTable` requires that the <<spark-sql-CatalogTable.adoc#provider, table provider>> of the <<tableDesc, CatalogTable>> is defined or throws an `AssertionError`:

```
assertion failed: The table to be created must have a provider.
```

The optional <<query, AS query>> is defined when used for the following:

* `DataFrameWriter` is requested to xref:spark-sql-DataFrameWriter.adoc#createTable[create a table] (for xref:spark-sql-DataFrameWriter.adoc#saveAsTable[DataFrameWriter.saveAsTable] operator)

* `SparkSqlAstBuilder` is requested to xref:spark-sql-SparkSqlAstBuilder.adoc#visitCreateTable[visitCreateTable] (for `CREATE TABLE` SQL command) or xref:spark-sql-SparkSqlAstBuilder.adoc#visitCreateHiveTable[visitCreateHiveTable] (for `CREATE EXTERNAL TABLE` SQL command) with an AS clause

[[resolved]]
`CreateTable` can never be <<spark-sql-Expression.adoc#resolved, resolved>> and is replaced (_resolved_) with a logical command at analysis phase in the following rules:

* (for non-hive data source tables) <<spark-sql-Analyzer-DataSourceAnalysis.adoc#, DataSourceAnalysis>> posthoc logical resolution rule to a <<spark-sql-LogicalPlan-CreateDataSourceTableCommand.adoc#, CreateDataSourceTableCommand>> or a <<spark-sql-LogicalPlan-CreateDataSourceTableAsSelectCommand.adoc#, CreateDataSourceTableAsSelectCommand>> logical command (when the <<query, query>> was defined or not, respectively)

* (for hive tables) link:hive/HiveAnalysis.adoc[HiveAnalysis] post-hoc logical resolution rule to a <<spark-sql-LogicalPlan-CreateTableCommand.adoc#, CreateTableCommand>> or a link:hive/CreateHiveTableAsSelectCommand.adoc[CreateHiveTableAsSelectCommand] logical command (when <<query, query>> was defined or not, respectively)

=== [[creating-instance]] Creating CreateTable Instance

`CreateTable` takes the following to be created:

* [[tableDesc]] xref:spark-sql-CatalogTable.adoc[Table metadata]
* [[mode]] xref:spark-sql-DataFrameWriter.adoc#SaveMode[SaveMode]
* [[query]] Optional AS query (xref:spark-sql-LogicalPlan.adoc[Logical query plan])

When created, `CreateTable` makes sure that the optional <<query, logical query plan>> is undefined only when the <<mode, mode>> is `ErrorIfExists` or `Ignore`. `CreateTable` throws an `AssertionError` otherwise:

```
assertion failed: create table without data insertion can only use ErrorIfExists or Ignore as SaveMode.
```
