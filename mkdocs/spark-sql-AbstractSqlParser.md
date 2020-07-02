title: AbstractSqlParser

# AbstractSqlParser -- Base SQL Parsing Infrastructure

`AbstractSqlParser` is the <<contract, base>> of <<implementations, ParserInterfaces>> that use an <<astBuilder, AstBuilder>> to parse SQL statements and convert them to Spark SQL entities, i.e. link:spark-sql-DataType.adoc[DataType], link:spark-sql-StructType.adoc[StructType], link:spark-sql-Expression.adoc[Expression], link:spark-sql-LogicalPlan.adoc[LogicalPlan] and `TableIdentifier`.

`AbstractSqlParser` is the foundation of the SQL parsing infrastructure.

[[contract]]
[source, scala]
----
package org.apache.spark.sql.catalyst.parser

abstract class AbstractSqlParser extends ParserInterface {
  // only required properties (vals and methods) that have no implementation
  // the others follow
  def astBuilder: AstBuilder
}
----

.AbstractSqlParser Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| [[astBuilder]] `astBuilder`
a| link:spark-sql-AstBuilder.adoc[AstBuilder] for parsing SQL statements.

Used in all the `parse` methods, i.e. <<parseDataType, parseDataType>>, <<parseExpression, parseExpression>>, <<parsePlan, parsePlan>>, <<parseTableIdentifier, parseTableIdentifier>>, and <<parseTableSchema, parseTableSchema>>.
|===

[[implementations]]
.AbstractSqlParser's Implementations
[width="100%",cols="1,2",options="header"]
|===
| Name
| Description

| link:spark-sql-SparkSqlParser.adoc[SparkSqlParser]
a| [[SparkSqlParser]] The default SQL parser in link:spark-sql-SessionState.adoc#sqlParser[SessionState] available as `sqlParser` property.

[source, scala]
----
val spark: SparkSession = ...
spark.sessionState.sqlParser
----

| link:spark-sql-CatalystSqlParser.adoc[CatalystSqlParser]
| [[CatalystSqlParser]] Creates a link:spark-sql-DataType.adoc[DataType] or a link:spark-sql-StructType.adoc[StructType] (schema) from their canonical string representation.
|===

=== [[parse]] Setting Up SqlBaseLexer and SqlBaseParser for Parsing -- `parse` Method

[source, scala]
----
parse[T](command: String)(toResult: SqlBaseParser => T): T
----

`parse` sets up a proper ANTLR parsing infrastructure with `SqlBaseLexer` and `SqlBaseParser` (which are the ANTLR-specific classes of Spark SQL that are auto-generated at build time from the `SqlBase.g4` grammar).

TIP: Review the definition of ANTLR grammar for Spark SQL in https://github.com/apache/spark/blob/master/sql/catalyst/src/main/antlr4/org/apache/spark/sql/catalyst/parser/SqlBase.g4[sql/catalyst/src/main/antlr4/org/apache/spark/sql/catalyst/parser/SqlBase.g4].

Internally, `parse` first prints out the following INFO message to the logs:

```
INFO SparkSqlParser: Parsing command: [command]
```

TIP: Enable `INFO` logging level for the custom `AbstractSqlParser`, i.e. link:spark-sql-SparkSqlParser.adoc#logging[SparkSqlParser] or link:spark-sql-CatalystSqlParser.adoc#logging[CatalystSqlParser], to see the above INFO message.

`parse` then creates and sets up a `SqlBaseLexer` and `SqlBaseParser` that in turn passes the latter on to the input `toResult` function where the parsing finally happens.

NOTE: `parse` uses `SLL` prediction mode for parsing first before falling back to `LL` mode.

In case of parsing errors, `parse` reports a `ParseException`.

NOTE: `parse` is used in all the `parse` methods, i.e. <<parseDataType, parseDataType>>, <<parseExpression, parseExpression>>, <<parsePlan, parsePlan>>, <<parseTableIdentifier, parseTableIdentifier>>, and <<parseTableSchema, parseTableSchema>>.

=== [[parseDataType]] `parseDataType` Method

[source, scala]
----
parseDataType(sqlText: String): DataType
----

NOTE: `parseDataType` is part of link:spark-sql-ParserInterface.adoc#parseDataType[ParserInterface Contract] to...FIXME.

`parseDataType`...FIXME

=== [[parseExpression]] `parseExpression` Method

[source, scala]
----
parseExpression(sqlText: String): Expression
----

NOTE: `parseExpression` is part of link:spark-sql-ParserInterface.adoc#parseExpression[ParserInterface Contract] to...FIXME.

`parseExpression`...FIXME

=== [[parseFunctionIdentifier]] `parseFunctionIdentifier` Method

[source, scala]
----
parseFunctionIdentifier(sqlText: String): FunctionIdentifier
----

NOTE: `parseFunctionIdentifier` is part of link:spark-sql-ParserInterface.adoc#parseFunctionIdentifier[ParserInterface Contract] to...FIXME.

`parseFunctionIdentifier`...FIXME

=== [[parseTableIdentifier]] `parseTableIdentifier` Method

[source, scala]
----
parseTableIdentifier(sqlText: String): TableIdentifier
----

NOTE: `parseTableIdentifier` is part of link:spark-sql-ParserInterface.adoc#parseTableIdentifier[ParserInterface Contract] to...FIXME.

`parseTableIdentifier`...FIXME

=== [[parseTableSchema]] `parseTableSchema` Method

[source, scala]
----
parseTableSchema(sqlText: String): StructType
----

NOTE: `parseTableSchema` is part of link:spark-sql-ParserInterface.adoc#parseTableSchema[ParserInterface Contract] to...FIXME.

`parseTableSchema`...FIXME

=== [[parsePlan]] `parsePlan` Method

[source, scala]
----
parsePlan(sqlText: String): LogicalPlan
----

NOTE: `parsePlan` is part of link:spark-sql-ParserInterface.adoc#parsePlan[ParserInterface Contract] to...FIXME.

`parsePlan` creates a link:spark-sql-LogicalPlan.adoc[LogicalPlan] for a given SQL textual statement.

Internally, `parsePlan` <<parse, builds a `SqlBaseParser`>> and requests <<astBuilder, AstBuilder>> to link:spark-sql-AstBuilder.adoc#visitSingleStatement[parse a single SQL statement].

If a SQL statement could not be parsed, `parsePlan` throws a `ParseException`:

```
Unsupported SQL statement
```
