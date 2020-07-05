title: SparkSqlParser

# SparkSqlParser -- Default SQL Parser

`SparkSqlParser` is the default link:spark-sql-AbstractSqlParser.adoc[SQL parser] of the SQL statements supported in Spark SQL.

`SparkSqlParser` supports <<VariableSubstitution, variable substitution>>.

[[astBuilder]]
`SparkSqlParser` uses link:spark-sql-SparkSqlAstBuilder.adoc[SparkSqlAstBuilder] (as link:spark-sql-AbstractSqlParser.adoc#astBuilder[AstBuilder]).

NOTE: Spark SQL supports SQL statements as described in https://github.com/apache/spark/blob/master/sql/catalyst/src/main/antlr4/org/apache/spark/sql/catalyst/parser/SqlBase.g4[SqlBase.g4] ANTLR grammar.

`SparkSqlParser` is available as link:spark-sql-SessionState.adoc#sqlParser[sqlParser] of a `SessionState`.

[source, scala]
----
val spark: SparkSession = ...
spark.sessionState.sqlParser
----

`SparkSqlParser` is used to translate an expression to the corresponding link:spark-sql-Column.adoc[Column] in the following:

* link:spark-sql-functions.adoc#expr[expr] function
* link:spark-sql-Dataset.adoc#selectExpr[Dataset.selectExpr] operator
* link:spark-sql-Dataset.adoc#filter[Dataset.filter] operator
* link:spark-sql-Dataset.adoc#where[Dataset.where] operator

[source, scala]
----
scala> expr("token = 'hello'")
16/07/07 18:32:53 INFO SparkSqlParser: Parsing command: token = 'hello'
res0: org.apache.spark.sql.Column = (token = hello)
----

`SparkSqlParser` is used to parse table strings into their corresponding table identifiers in the following:

* `table` methods in [DataFrameReader](DataFrameReader.md#table) and [SparkSession](spark-sql-SparkSession.md#table)
* [insertInto](spark-sql-DataFrameWriter.md#insertInto) and [saveAsTable](spark-sql-DataFrameWriter.md#saveAsTable) methods of `DataFrameWriter`
* `createExternalTable` and `refreshTable` methods of [Catalog](spark-sql-Catalog.md) (and [SessionState](spark-sql-SessionState.md#refreshTable))

`SparkSqlParser` is used to translate a SQL text to its corresponding link:spark-sql-LogicalPlan.adoc[logical operator] in link:spark-sql-SparkSession.adoc#sql[SparkSession.sql] method.

[[logging]]
[TIP]
====
Enable `INFO` logging level for `org.apache.spark.sql.execution.SparkSqlParser` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.execution.SparkSqlParser=INFO
```

Refer to link:spark-logging.adoc[Logging].
====

=== [[VariableSubstitution]] Variable Substitution

CAUTION: FIXME See `SparkSqlParser` and `substitutor`.
