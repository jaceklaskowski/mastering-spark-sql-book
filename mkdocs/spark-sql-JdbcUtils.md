title: JdbcUtils

# JdbcUtils Helper Object

`JdbcUtils` is a Scala object with <<methods, methods>> to support link:spark-sql-JDBCRDD.adoc[JDBCRDD], link:spark-sql-JDBCRelation.adoc[JDBCRelation] and link:spark-sql-JdbcRelationProvider.adoc[JdbcRelationProvider].

[[methods]]
.JdbcUtils API
[cols="1,2",options="header",width="100%"]
|===
| Name
| Description

| <<createConnectionFactory, createConnectionFactory>>
a| Used when:

* `JDBCRDD` is requested to link:spark-sql-JDBCRDD.adoc#scanTable[scanTable] and link:spark-sql-JDBCRDD.adoc#resolveTable[resolveTable]

* `JdbcRelationProvider` is requested to <<spark-sql-JdbcRelationProvider.adoc#createRelation-CreatableRelationProvider, write the rows of a structured query (a DataFrame) to a table>>

| <<createTable, createTable>>
|

| <<dropTable, dropTable>>
|

| <<getCommonJDBCType, getCommonJDBCType>>
|

| <<getCustomSchema, getCustomSchema>>
| Replaces data types in a table schema

Used exclusively when `JDBCRelation` is link:spark-sql-JDBCRelation.adoc#schema[created] (and link:spark-sql-JDBCOptions.adoc#customSchema[customSchema] JDBC option was defined)

| <<getInsertStatement, getInsertStatement>>
|

| <<getSchema, getSchema>>
| Used when `JDBCRDD` is requested to link:spark-sql-JDBCRDD.adoc#resolveTable[resolveTable]

| <<getSchemaOption, getSchemaOption>>
| Used when `JdbcRelationProvider` is requested to <<spark-sql-JdbcRelationProvider.adoc#createRelation-CreatableRelationProvider, write the rows of a structured query (a DataFrame) to a table>>

| <<resultSetToRows, resultSetToRows>>
| Used when...FIXME

| <<resultSetToSparkInternalRows, resultSetToSparkInternalRows>>
| Used when `JDBCRDD` is requested to link:spark-sql-JDBCRDD.adoc#compute[compute a partition]

| <<schemaString, schemaString>>
|

| <<saveTable, saveTable>>
|

| <<tableExists, tableExists>>
| Used when `JdbcRelationProvider` is requested to <<spark-sql-JdbcRelationProvider.adoc#createRelation-CreatableRelationProvider, write the rows of a structured query (a DataFrame) to a table>>

| <<truncateTable, truncateTable>>
| Used when...FIXME
|===

=== [[createConnectionFactory]] `createConnectionFactory` Method

[source, scala]
----
createConnectionFactory(options: JDBCOptions): () => Connection
----

`createConnectionFactory`...FIXME

[NOTE]
====
`createConnectionFactory` is used when:

* `JDBCRDD` is requested to link:spark-sql-JDBCRDD.adoc#scanTable[scanTable] (and in turn creates a link:spark-sql-JDBCRDD.adoc#creating-instance[JDBCRDD]) and link:spark-sql-JDBCRDD.adoc#resolveTable[resolveTable]

* `JdbcRelationProvider` is requested to link:spark-sql-JdbcRelationProvider.adoc#createRelation[create a BaseRelation]

* `JdbcUtils` is requested to <<saveTable, saveTable>>
====

=== [[getCommonJDBCType]] `getCommonJDBCType` Method

[source, scala]
----
getCommonJDBCType(dt: DataType): Option[JdbcType]
----

`getCommonJDBCType`...FIXME

NOTE: `getCommonJDBCType` is used when...FIXME

=== [[getCatalystType]] `getCatalystType` Internal Method

[source, scala]
----
getCatalystType(
  sqlType: Int,
  precision: Int,
  scale: Int,
  signed: Boolean): DataType
----

`getCatalystType`...FIXME

NOTE: `getCatalystType` is used when...FIXME

=== [[getSchemaOption]] `getSchemaOption` Method

[source, scala]
----
getSchemaOption(conn: Connection, options: JDBCOptions): Option[StructType]
----

`getSchemaOption`...FIXME

NOTE: `getSchemaOption` is used when...FIXME

=== [[getSchema]] `getSchema` Method

[source, scala]
----
getSchema(
  resultSet: ResultSet,
  dialect: JdbcDialect,
  alwaysNullable: Boolean = false): StructType
----

`getSchema`...FIXME

NOTE: `getSchema` is used when...FIXME

=== [[resultSetToRows]] `resultSetToRows` Method

[source, scala]
----
resultSetToRows(resultSet: ResultSet, schema: StructType): Iterator[Row]
----

`resultSetToRows`...FIXME

NOTE: `resultSetToRows` is used when...FIXME

=== [[resultSetToSparkInternalRows]] `resultSetToSparkInternalRows` Method

[source, scala]
----
resultSetToSparkInternalRows(
  resultSet: ResultSet,
  schema: StructType,
  inputMetrics: InputMetrics): Iterator[InternalRow]
----

`resultSetToSparkInternalRows`...FIXME

NOTE: `resultSetToSparkInternalRows` is used when...FIXME

=== [[schemaString]] `schemaString` Method

[source, scala]
----
schemaString(
  df: DataFrame,
  url: String,
  createTableColumnTypes: Option[String] = None): String
----

`schemaString`...FIXME

NOTE: `schemaString` is used exclusively when `JdbcUtils` is requested to <<createTable, create a table>>.

=== [[parseUserSpecifiedCreateTableColumnTypes]] `parseUserSpecifiedCreateTableColumnTypes` Internal Method

[source, scala]
----
parseUserSpecifiedCreateTableColumnTypes(
  df: DataFrame,
  createTableColumnTypes: String): Map[String, String]
----

`parseUserSpecifiedCreateTableColumnTypes`...FIXME

NOTE: `parseUserSpecifiedCreateTableColumnTypes` is used exclusively when `JdbcUtils` is requested to <<schemaString, schemaString>>.

=== [[saveTable]] `saveTable` Method

[source, scala]
----
saveTable(
  df: DataFrame,
  tableSchema: Option[StructType],
  isCaseSensitive: Boolean,
  options: JDBCOptions): Unit
----

`saveTable` takes the <<spark-sql-JDBCOptions.adoc#url, url>>, <<spark-sql-JDBCOptions.adoc#table, table>>, <<spark-sql-JDBCOptions.adoc#batchSize, batchSize>>, <<spark-sql-JDBCOptions.adoc#isolationLevel, isolationLevel>> options and <<createConnectionFactory, createConnectionFactory>>.

`saveTable` <<getInsertStatement, getInsertStatement>>.

`saveTable` takes the <<spark-sql-JDBCOptions.adoc#numPartitions, numPartitions>> option and applies <<spark-sql-dataset-operators.adoc#coalesce, coalesce>> operator to the input `DataFrame` if the number of partitions of its <<spark-sql-Dataset.adoc#rdd, RDD>> is less than the `numPartitions` option.

In the end, `saveTable` requests the possibly-repartitioned `DataFrame` for its <<spark-sql-Dataset.adoc#rdd, RDD>> (it may have changed after the <<spark-sql-dataset-operators.adoc#coalesce, coalesce>> operator) and executes <<savePartition, savePartition>> for every partition (using `RDD.foreachPartition`).

NOTE: `saveTable` is used exclusively when `JdbcRelationProvider` is requested to <<spark-sql-JdbcRelationProvider.adoc#createRelation-CreatableRelationProvider, write the rows of a structured query (a DataFrame) to a table>>.

=== [[getCustomSchema]] Replacing Data Types In Table Schema -- `getCustomSchema` Method

[source, scala]
----
getCustomSchema(
  tableSchema: StructType,
  customSchema: String,
  nameEquality: Resolver): StructType
----

`getCustomSchema` replaces the data type of the fields in the input `tableSchema` link:spark-sql-StructType.adoc[schema] that are included in the input `customSchema` (if defined).

Internally, `getCustomSchema` branches off per the input `customSchema`.

If the input `customSchema` is undefined or empty, `getCustomSchema` simply returns the input `tableSchema` unchanged.

Otherwise, if the input `customSchema` is not empty, `getCustomSchema` requests `CatalystSqlParser` to link:spark-sql-AbstractSqlParser.adoc#parseTableSchema[parse it] (i.e. create a new link:spark-sql-StructType.adoc[StructType] for the given `customSchema` canonical schema representation).

`getCustomSchema` then uses `SchemaUtils` to link:spark-sql-SchemaUtils.adoc#checkColumnNameDuplication[checkColumnNameDuplication] (in the column names of the user-defined `customSchema` schema with the input `nameEquality`).

In the end, `getCustomSchema` replaces the data type of the fields in the input `tableSchema` that are included in the input `userSchema`.

NOTE: `getCustomSchema` is used exclusively when `JDBCRelation` is link:spark-sql-JDBCRelation.adoc#schema[created] (and link:spark-sql-JDBCOptions.adoc#customSchema[customSchema] JDBC option was defined).

=== [[dropTable]] `dropTable` Method

[source, scala]
----
dropTable(conn: Connection, table: String): Unit
----

`dropTable`...FIXME

NOTE: `dropTable` is used when...FIXME

=== [[createTable]] Creating Table Using JDBC -- `createTable` Method

[source, scala]
----
createTable(
  conn: Connection,
  df: DataFrame,
  options: JDBCOptions): Unit
----

`createTable` <<schemaString, builds the table schema>> (given the input `DataFrame` with the <<spark-sql-JDBCOptions.adoc#url, url>> and <<spark-sql-JDBCOptions.adoc#createTableColumnTypes, createTableColumnTypes>> options).

`createTable` uses the <<spark-sql-JDBCOptions.adoc#table, table>> and <<spark-sql-JDBCOptions.adoc#createTableOptions, createTableOptions>> options.

In the end, `createTable` concatenates all the above texts into a `CREATE TABLE [table] ([strSchema]) [createTableOptions]` SQL DDL statement followed by executing it (using the input JDBC `Connection`).

NOTE: `createTable` is used exclusively when `JdbcRelationProvider` is requested to <<spark-sql-JdbcRelationProvider.adoc#createRelation-CreatableRelationProvider, write the rows of a structured query (a DataFrame) to a table>>.

=== [[getInsertStatement]] `getInsertStatement` Method

[source, scala]
----
getInsertStatement(
  table: String,
  rddSchema: StructType,
  tableSchema: Option[StructType],
  isCaseSensitive: Boolean,
  dialect: JdbcDialect): String
----

`getInsertStatement`...FIXME

NOTE: `getInsertStatement` is used when...FIXME

=== [[getJdbcType]] `getJdbcType` Internal Method

[source, scala]
----
getJdbcType(dt: DataType, dialect: JdbcDialect): JdbcType
----

`getJdbcType`...FIXME

NOTE: `getJdbcType` is used when...FIXME

=== [[tableExists]] `tableExists` Method

[source, scala]
----
tableExists(conn: Connection, options: JDBCOptions): Boolean
----

`tableExists`...FIXME

NOTE: `tableExists` is used exclusively when `JdbcRelationProvider` is requested to <<spark-sql-JdbcRelationProvider.adoc#createRelation-CreatableRelationProvider, write the rows of a structured query (a DataFrame) to a table>>.

=== [[truncateTable]] `truncateTable` Method

[source, scala]
----
truncateTable(conn: Connection, options: JDBCOptions): Unit
----

`truncateTable`...FIXME

NOTE: `truncateTable` is used exclusively when `JdbcRelationProvider` is requested to <<spark-sql-JdbcRelationProvider.adoc#createRelation-CreatableRelationProvider, write the rows of a structured query (a DataFrame) to a table>>.

=== [[savePartition]] Saving Rows (Per Partition) to Table -- `savePartition` Method

[source, scala]
----
savePartition(
  getConnection: () => Connection,
  table: String,
  iterator: Iterator[Row],
  rddSchema: StructType,
  insertStmt: String,
  batchSize: Int,
  dialect: JdbcDialect,
  isolationLevel: Int): Iterator[Byte]
----

`savePartition` creates a JDBC `Connection` using the input `getConnection` function.

`savePartition` tries to set the input `isolationLevel` if it is different than `TRANSACTION_NONE` and the database supports transactions.

`savePartition` then writes rows (in the input `Iterator[Row]`) using batches that are submitted after `batchSize` rows where added.

NOTE: `savePartition` is used exclusively when `JdbcUtils` is requested to <<saveTable, saveTable>>.
