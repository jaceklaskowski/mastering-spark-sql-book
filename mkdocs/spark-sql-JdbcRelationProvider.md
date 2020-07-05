# JdbcRelationProvider

[[shortName]]
`JdbcRelationProvider` is a <<spark-sql-DataSourceRegister.adoc#, DataSourceRegister>> and registers itself to handle *jdbc* data source format.

NOTE: `JdbcRelationProvider` uses `META-INF/services/org.apache.spark.sql.sources.DataSourceRegister` file for the registration which is available in the https://github.com/apache/spark/blob/master/sql/core/src/main/resources/META-INF/services/org.apache.spark.sql.sources.DataSourceRegister[source code] of Apache Spark.

`JdbcRelationProvider` is a <<createRelation-RelationProvider, RelationProvider>> and a <<createRelation-CreatableRelationProvider, CreatableRelationProvider>>.

`JdbcRelationProvider` is used when `DataFrameReader` is requested to load data from [jdbc](DataFrameReader.md#jdbc) data source.

[source, scala]
----
val table = spark.read.jdbc(...)

// or in a more verbose way
val table = spark.read.format("jdbc").load(...)
----

=== [[createRelation-RelationProvider]] Loading Data from Table Using JDBC -- `createRelation` Method (from RelationProvider)

[source, scala]
----
createRelation(
  sqlContext: SQLContext,
  parameters: Map[String, String]): BaseRelation
----

NOTE: `createRelation` is part of <<spark-sql-RelationProvider.adoc#createRelation, RelationProvider Contract>> to create a <<spark-sql-BaseRelation.adoc#, BaseRelation>> for reading.

`createRelation` creates a `JDBCPartitioningInfo` (using link:spark-sql-JDBCOptions.adoc[JDBCOptions] and the input `parameters` that correspond to the link:spark-sql-JDBCOptions.adoc#options[Options for JDBC Data Source]).

NOTE: `createRelation` uses [partitionColumn](DataFrameReader.md#partitionColumn), [lowerBound](DataFrameReader.md#lowerBound), [upperBound](DataFrameReader.md#upperBound) and [numPartitions](DataFrameReader.md#numPartitions).

In the end, `createRelation` creates a link:spark-sql-JDBCRelation.adoc#creating-instance[JDBCRelation] with link:spark-sql-JDBCRelation.adoc#columnPartition[column partitions] (and link:spark-sql-JDBCOptions.adoc[JDBCOptions]).

=== [[createRelation-CreatableRelationProvider]] Writing Rows of Structured Query (DataFrame) to Table Using JDBC -- `createRelation` Method (from CreatableRelationProvider)

[source, scala]
----
createRelation(
  sqlContext: SQLContext,
  mode: SaveMode,
  parameters: Map[String, String],
  df: DataFrame): BaseRelation
----

NOTE: `createRelation` is part of the <<spark-sql-CreatableRelationProvider.adoc#createRelation, CreatableRelationProvider Contract>> to write the rows of a structured query (a DataFrame) to an external data source.

Internally, `createRelation` creates a link:spark-sql-JDBCOptions.adoc#creating-instance[JDBCOptions] (from the input `parameters`).

`createRelation` reads link:spark-sql-CatalystConf.adoc#caseSensitiveAnalysis[caseSensitiveAnalysis] (using the input `sqlContext`).

`createRelation` checks whether the table (given `dbtable` and `url` link:spark-sql-JDBCOptions.adoc#options[options] in the input `parameters`) exists.

NOTE: `createRelation` uses a database-specific `JdbcDialect` to link:spark-sql-JdbcDialect.adoc#getTableExistsQuery[check whether a table exists].

`createRelation` branches off per whether the table already exists in the database or not.

If the table *does not* exist, `createRelation` creates the table (by executing `CREATE TABLE` with <<spark-sql-JDBCOptions.adoc#createTableColumnTypes, createTableColumnTypes>> and <<spark-sql-JDBCOptions.adoc#createTableOptions, createTableOptions>> options from the input `parameters`) and writes the rows to the database in a single transaction.

If however the table *does* exist, `createRelation` branches off per link:spark-sql-DataFrameWriter.adoc#SaveMode[SaveMode] (see the following <<createRelation-CreatableRelationProvider-SaveMode, createRelation and SaveMode>>).

[[createRelation-CreatableRelationProvider-SaveMode]]
.createRelation and SaveMode
[cols="1,2",options="header",width="100%"]
|===
| Name
| Description

| <<spark-sql-DataFrameWriter.adoc#Append, Append>>
| Saves the records to the table.

| <<spark-sql-DataFrameWriter.adoc#ErrorIfExists, ErrorIfExists>>
a| Throws a `AnalysisException` with the message:

```
Table or view '[table]' already exists. SaveMode: ErrorIfExists.
```

| <<spark-sql-DataFrameWriter.adoc#Ignore, Ignore>>
| Does nothing.

| <<spark-sql-DataFrameWriter.adoc#Overwrite, Overwrite>>
a| Truncates or drops the table

NOTE: `createRelation` truncates the table only when link:spark-sql-JDBCOptions.adoc#truncate[truncate] JDBC option is enabled and link:spark-sql-JdbcDialect.adoc#isCascadingTruncateTable[isCascadingTruncateTable] is disabled.
|===

In the end, `createRelation` closes the JDBC connection to the database and <<createRelation-RelationProvider, creates a JDBCRelation>>.
