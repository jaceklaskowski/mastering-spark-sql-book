title: Untyped Transformations

# Dataset API -- Untyped Transformations

*Untyped transformations* are part of the Dataset API for transforming a `Dataset` to a <<spark-sql-DataFrame.adoc#, DataFrame>>, a <<spark-sql-Column.adoc#, Column>>, a <<spark-sql-RelationalGroupedDataset.adoc#, RelationalGroupedDataset>>, a <<spark-sql-DataFrameNaFunctions.adoc#, DataFrameNaFunctions>> or a <<spark-sql-DataFrameStatFunctions.adoc#, DataFrameStatFunctions>> (and hence _untyped_).

NOTE: Untyped transformations are the methods in the `Dataset` Scala class that are grouped in `untypedrel` group name, i.e. `@group untypedrel`.

[[methods]]
.Dataset API's Untyped Transformations
[cols="1,2",options="header",width="100%"]
|===
| Transformation
| Description

| <<agg, agg>>
a|

[source, scala]
----
agg(aggExpr: (String, String), aggExprs: (String, String)*): DataFrame
agg(expr: Column, exprs: Column*): DataFrame
agg(exprs: Map[String, String]): DataFrame
----

| <<apply, apply>>
a| Selects a column based on the column name (i.e. maps a `Dataset` onto a `Column`)

[source, scala]
----
apply(colName: String): Column
----

| <<col, col>>
a| Selects a column based on the column name (i.e. maps a `Dataset` onto a `Column`)

[source, scala]
----
col(colName: String): Column
----

| <<colRegex, colRegex>>
a|

[source, scala]
----
colRegex(colName: String): Column
----

Selects a column based on the column name specified as a regex (i.e. maps a `Dataset` onto a `Column`)

| <<crossJoin, crossJoin>>
a|

[source, scala]
----
crossJoin(right: Dataset[_]): DataFrame
----

| <<cube, cube>>
a|

[source, scala]
----
cube(cols: Column*): RelationalGroupedDataset
cube(col1: String, cols: String*): RelationalGroupedDataset
----

| <<drop, drop>>
a|

[source, scala]
----
drop(colName: String): DataFrame
drop(colNames: String*): DataFrame
drop(col: Column): DataFrame
----

| <<groupBy, groupBy>>
a|

[source, scala]
----
groupBy(cols: Column*): RelationalGroupedDataset
groupBy(col1: String, cols: String*): RelationalGroupedDataset
----

| <<join, join>>
a|

[source, scala]
----
join(right: Dataset[_]): DataFrame
join(right: Dataset[_], usingColumn: String): DataFrame
join(right: Dataset[_], usingColumns: Seq[String]): DataFrame
join(right: Dataset[_], usingColumns: Seq[String], joinType: String): DataFrame
join(right: Dataset[_], joinExprs: Column): DataFrame
join(right: Dataset[_], joinExprs: Column, joinType: String): DataFrame
----

| <<na, na>>
a|

[source, scala]
----
na: DataFrameNaFunctions
----

| <<rollup, rollup>>
a|

[source, scala]
----
rollup(cols: Column*): RelationalGroupedDataset
rollup(col1: String, cols: String*): RelationalGroupedDataset
----

| <<select, select>>
a|

[source, scala]
----
select(cols: Column*): DataFrame
select(col: String, cols: String*): DataFrame
----

| <<selectExpr, selectExpr>>
a|

[source, scala]
----
selectExpr(exprs: String*): DataFrame
----

| <<stat, stat>>
a|

[source, scala]
----
stat: DataFrameStatFunctions
----

| <<withColumn, withColumn>>
a|

[source, scala]
----
withColumn(colName: String, col: Column): DataFrame
----

| <<withColumnRenamed, withColumnRenamed>>
a|

[source, scala]
----
withColumnRenamed(existingName: String, newName: String): DataFrame
----
|===

=== [[agg]] `agg` Untyped Transformation

[source, scala]
----
agg(aggExpr: (String, String), aggExprs: (String, String)*): DataFrame
agg(expr: Column, exprs: Column*): DataFrame
agg(exprs: Map[String, String]): DataFrame
----

`agg`...FIXME

=== [[apply]] `apply` Untyped Transformation

[source, scala]
----
apply(colName: String): Column
----

`apply` selects a column based on the column name (i.e. maps a `Dataset` onto a `Column`).

=== [[col]] `col` Untyped Transformation

[source, scala]
----
col(colName: String): Column
----

`col` selects a column based on the column name (i.e. maps a `Dataset` onto a `Column`).

Internally, `col` branches off per the input column name.

If the column name is `*` (a star), `col` simply creates a <<spark-sql-Column.adoc#apply, Column>> with <<spark-sql-Expression-ResolvedStar.adoc#, ResolvedStar>> expression (with the <<spark-sql-catalyst-QueryPlan.adoc#output, schema output attributes>> of the <<spark-sql-QueryExecution.adoc#analyzed, analyzed logical plan>> of the <<spark-sql-Dataset.adoc#queryExecution, QueryExecution>>).

Otherwise, `col` uses <<colRegex, colRegex>> untyped transformation when <<spark-sql-properties.adoc#spark.sql.parser.quotedRegexColumnNames, spark.sql.parser.quotedRegexColumnNames>> configuration property is enabled.

In the case when the column name is not `*` and <<spark-sql-properties.adoc#spark.sql.parser.quotedRegexColumnNames, spark.sql.parser.quotedRegexColumnNames>> configuration property is disabled, `col` creates a <<spark-sql-Column.adoc#apply, Column>> with the column name <<spark-sql-Dataset.adoc#resolve, resolved>> (as a <<spark-sql-Expression-NamedExpression.adoc#, NamedExpression>>).

=== [[colRegex]] `colRegex` Untyped Transformation

[source, scala]
----
colRegex(colName: String): Column
----

`colRegex` selects a column based on the column name specified as a regex (i.e. maps a `Dataset` onto a `Column`).

NOTE: `colRegex` is used in <<col, col>> when <<spark-sql-properties.adoc#spark.sql.parser.quotedRegexColumnNames, spark.sql.parser.quotedRegexColumnNames>> configuration property is enabled (and the column name is not `*`).

Internally, `colRegex` matches the input column name to different regular expressions (in the order):

. For column names with quotes without a qualifier, `colRegex` simply creates a <<spark-sql-Column.adoc#apply, Column>> with a <<spark-sql-Expression-UnresolvedRegex.adoc#, UnresolvedRegex>> (with no table)

. For column names with quotes with a qualifier, `colRegex` simply creates a <<spark-sql-Column.adoc#apply, Column>> with a <<spark-sql-Expression-UnresolvedRegex.adoc#, UnresolvedRegex>> (with a table specified)

. For other column names, `colRegex` (behaves like <<col, col>> and) creates a <<spark-sql-Column.adoc#apply, Column>> with the column name <<spark-sql-Dataset.adoc#resolve, resolved>> (as a <<spark-sql-Expression-NamedExpression.adoc#, NamedExpression>>)

=== [[crossJoin]] `crossJoin` Untyped Transformation

[source, scala]
----
crossJoin(right: Dataset[_]): DataFrame
----

`crossJoin`...FIXME

=== [[cube]] `cube` Untyped Transformation

[source, scala]
----
cube(cols: Column*): RelationalGroupedDataset
cube(col1: String, cols: String*): RelationalGroupedDataset
----

`cube`...FIXME

=== [[drop]] Dropping One or More Columns -- `drop` Untyped Transformation

[source, scala]
----
drop(colName: String): DataFrame
drop(colNames: String*): DataFrame
drop(col: Column): DataFrame
----

`drop`...FIXME

=== [[groupBy]] `groupBy` Untyped Transformation

[source, scala]
----
groupBy(cols: Column*): RelationalGroupedDataset
groupBy(col1: String, cols: String*): RelationalGroupedDataset
----

`groupBy`...FIXME

=== [[join]] `join` Untyped Transformation

[source, scala]
----
join(right: Dataset[_]): DataFrame
join(right: Dataset[_], usingColumn: String): DataFrame
join(right: Dataset[_], usingColumns: Seq[String]): DataFrame
join(right: Dataset[_], usingColumns: Seq[String], joinType: String): DataFrame
join(right: Dataset[_], joinExprs: Column): DataFrame
join(right: Dataset[_], joinExprs: Column, joinType: String): DataFrame
----

`join`...FIXME

=== [[na]] `na` Untyped Transformation

[source, scala]
----
na: DataFrameNaFunctions
----

`na` simply creates a <<spark-sql-DataFrameNaFunctions.adoc#, DataFrameNaFunctions>> to work with missing data.

=== [[rollup]] `rollup` Untyped Transformation

[source, scala]
----
rollup(cols: Column*): RelationalGroupedDataset
rollup(col1: String, cols: String*): RelationalGroupedDataset
----

`rollup`...FIXME

=== [[select]] `select` Untyped Transformation

[source, scala]
----
select(cols: Column*): DataFrame
select(col: String, cols: String*): DataFrame
----

`select`...FIXME

=== [[selectExpr]] Projecting Columns using SQL Statements -- `selectExpr` Untyped Transformation

[source, scala]
----
selectExpr(exprs: String*): DataFrame
----

`selectExpr` is like `select`, but accepts SQL statements.

[source, scala]
----
val ds = spark.range(5)

scala> ds.selectExpr("rand() as random").show
16/04/14 23:16:06 INFO HiveSqlParser: Parsing command: rand() as random
+-------------------+
|             random|
+-------------------+
|  0.887675894185651|
|0.36766085091074086|
| 0.2700020856675186|
| 0.1489033635529543|
| 0.5862990791950973|
+-------------------+
----

Internally, it executes `select` with every expression in `exprs` mapped to link:spark-sql-Column.adoc[Column] (using link:spark-sql-SparkSqlParser.adoc[SparkSqlParser.parseExpression]).

[source, scala]
----
scala> ds.select(expr("rand() as random")).show
+------------------+
|            random|
+------------------+
|0.5514319279894851|
|0.2876221510433741|
|0.4599999092045741|
|0.5708558868374893|
|0.6223314406247136|
+------------------+
----

=== [[stat]] `stat` Untyped Transformation

[source, scala]
----
stat: DataFrameStatFunctions
----

`stat` simply creates a <<spark-sql-DataFrameStatFunctions.adoc#, DataFrameStatFunctions>> to work with statistic functions.

=== [[withColumn]] `withColumn` Untyped Transformation

[source, scala]
----
withColumn(colName: String, col: Column): DataFrame
----

`withColumn`...FIXME

=== [[withColumnRenamed]] `withColumnRenamed` Untyped Transformation

[source, scala]
----
withColumnRenamed(existingName: String, newName: String): DataFrame
----

`withColumnRenamed`...FIXME
