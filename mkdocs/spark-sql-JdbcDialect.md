# JdbcDialect

`JdbcDialect` is the <<contract, base>> of <<extensions, JDBC dialects>> that <<canHandle, handle a specific JDBC URL>> (and handle necessary type-related conversions to properly load a data from a table into a `DataFrame`).

[[contract]]
[source, scala]
----
package org.apache.spark.sql.jdbc

abstract class JdbcDialect extends Serializable {
  // only required properties (vals and methods) that have no implementation
  // the others follow
  def canHandle(url : String): Boolean
}
----

.(Subset of) JdbcDialect Contract
[cols="1,2",options="header",width="100%"]
|===
| Property
| Description

| `canHandle`
| [[canHandle]] Used when...FIXME
|===

[[extensions]]
.JdbcDialects
[cols="1,2",options="header",width="100%"]
|===
| JdbcDialect
| Description

| `AggregatedDialect`
| [[AggregatedDialect]]

| `DB2Dialect`
| [[DB2Dialect]]

| `DerbyDialect`
| [[DerbyDialect]]

| `MsSqlServerDialect`
| [[MsSqlServerDialect]]

| `MySQLDialect`
| [[MySQLDialect]]

| `NoopDialect`
| [[NoopDialect]]

| `OracleDialect`
| [[OracleDialect]]

| `PostgresDialect`
| [[PostgresDialect]]

| `TeradataDialect`
| [[TeradataDialect]]
|===

=== [[getCatalystType]] `getCatalystType` Method

[source, scala]
----
getCatalystType(
  sqlType: Int,
  typeName: String,
  size: Int,
  md: MetadataBuilder): Option[DataType]
----

`getCatalystType`...FIXME

NOTE: `getCatalystType` is used when...FIXME

=== [[getJDBCType]] `getJDBCType` Method

[source, scala]
----
getJDBCType(dt: DataType): Option[JdbcType]
----

`getJDBCType`...FIXME

NOTE: `getJDBCType` is used when...FIXME

=== [[quoteIdentifier]] `quoteIdentifier` Method

[source, scala]
----
quoteIdentifier(colName: String): String
----

`quoteIdentifier`...FIXME

NOTE: `quoteIdentifier` is used when...FIXME

=== [[getTableExistsQuery]] `getTableExistsQuery` Method

[source, scala]
----
getTableExistsQuery(table: String): String
----

`getTableExistsQuery`...FIXME

NOTE: `getTableExistsQuery` is used when...FIXME

=== [[getSchemaQuery]] `getSchemaQuery` Method

[source, scala]
----
getSchemaQuery(table: String): String
----

`getSchemaQuery`...FIXME

NOTE: `getSchemaQuery` is used when...FIXME

=== [[getTruncateQuery]] `getTruncateQuery` Method

[source, scala]
----
getTruncateQuery(table: String): String
----

`getTruncateQuery`...FIXME

NOTE: `getTruncateQuery` is used when...FIXME

=== [[beforeFetch]] `beforeFetch` Method

[source, scala]
----
beforeFetch(connection: Connection, properties: Map[String, String]): Unit
----

`beforeFetch`...FIXME

NOTE: `beforeFetch` is used when...FIXME

=== [[escapeSql]] `escapeSql` Internal Method

[source, scala]
----
escapeSql(value: String): String
----

`escapeSql`...FIXME

NOTE: `escapeSql` is used when...FIXME

=== [[compileValue]] `compileValue` Method

[source, scala]
----
compileValue(value: Any): Any
----

`compileValue`...FIXME

NOTE: `compileValue` is used when...FIXME

=== [[isCascadingTruncateTable]] `isCascadingTruncateTable` Method

[source, scala]
----
isCascadingTruncateTable(): Option[Boolean]
----

`isCascadingTruncateTable`...FIXME

NOTE: `isCascadingTruncateTable` is used when...FIXME
