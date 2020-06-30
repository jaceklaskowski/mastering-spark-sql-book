title: Aggregate Functions

# Standard Aggregate Functions

[[functions]]
.Standard Aggregate Functions
[align="center",cols="1m,2",width="100%",options="header"]
|===
| Name
| Description

| approx_count_distinct
a| [[approx_count_distinct]]

[source, scala]
----
approx_count_distinct(e: Column): Column
approx_count_distinct(columnName: String): Column
approx_count_distinct(e: Column, rsd: Double): Column
approx_count_distinct(columnName: String, rsd: Double): Column
----

| avg
a| [[avg]]

[source, scala]
----
avg(e: Column): Column
avg(columnName: String): Column
----

| collect_list
a| [[collect_list]]

[source, scala]
----
collect_list(e: Column): Column
collect_list(columnName: String): Column
----

| collect_set
a| [[collect_set]]

[source, scala]
----
collect_set(e: Column): Column
collect_set(columnName: String): Column
----

| corr
a| [[corr]]

[source, scala]
----
corr(column1: Column, column2: Column): Column
corr(columnName1: String, columnName2: String): Column
----

| count
a| [[count]]

[source, scala]
----
count(e: Column): Column
count(columnName: String): TypedColumn[Any, Long]
----

| countDistinct
a| [[countDistinct]]

[source, scala]
----
countDistinct(expr: Column, exprs: Column*): Column
countDistinct(columnName: String, columnNames: String*): Column
----

| covar_pop
a| [[covar_pop]]

[source, scala]
----
covar_pop(column1: Column, column2: Column): Column
covar_pop(columnName1: String, columnName2: String): Column
----

| covar_samp
a| [[covar_samp]]

[source, scala]
----
covar_samp(column1: Column, column2: Column): Column
covar_samp(columnName1: String, columnName2: String): Column
----

| first
a| [[first]]

[source, scala]
----
first(e: Column): Column
first(e: Column, ignoreNulls: Boolean): Column
first(columnName: String): Column
first(columnName: String, ignoreNulls: Boolean): Column
----

Returns the first value in a group. Returns the first non-null value when `ignoreNulls` flag on. If all values are null, then returns null.

| grouping
a| [[grouping]]

[source, scala]
----
grouping(e: Column): Column
grouping(columnName: String): Column
----

Indicates whether a given column is aggregated or not

| grouping_id
a| [[grouping_id]]

[source, scala]
----
grouping_id(cols: Column*): Column
grouping_id(colName: String, colNames: String*): Column
----

Computes the level of grouping

| kurtosis
a| [[kurtosis]]

[source, scala]
----
kurtosis(e: Column): Column
kurtosis(columnName: String): Column
----

| last
a| [[last]]

[source, scala]
----
last(e: Column, ignoreNulls: Boolean): Column
last(columnName: String, ignoreNulls: Boolean): Column
last(e: Column): Column
last(columnName: String): Column
----

| max
a| [[max]]

[source, scala]
----
max(e: Column): Column
max(columnName: String): Column
----

| mean
a| [[mean]]

[source, scala]
----
mean(e: Column): Column
mean(columnName: String): Column
----

| min
a| [[min]]

[source, scala]
----
min(e: Column): Column
min(columnName: String): Column
----

| skewness
a| [[skewness]]

[source, scala]
----
skewness(e: Column): Column
skewness(columnName: String): Column
----

| stddev
a| [[stddev]]

[source, scala]
----
stddev(e: Column): Column
stddev(columnName: String): Column
----

| stddev_pop
a| [[stddev_pop]]

[source, scala]
----
stddev_pop(e: Column): Column
stddev_pop(columnName: String): Column
----

| stddev_samp
a| [[stddev_samp]]

[source, scala]
----
stddev_samp(e: Column): Column
stddev_samp(columnName: String): Column
----

| sum
a| [[sum]]

[source, scala]
----
sum(e: Column): Column
sum(columnName: String): Column
----

| sumDistinct
a| [[sumDistinct]]

[source, scala]
----
sumDistinct(e: Column): Column
sumDistinct(columnName: String): Column
----

| variance
a| [[variance]]

[source, scala]
----
variance(e: Column): Column
variance(columnName: String): Column
----

| var_pop
a| [[var_pop]]

[source, scala]
----
var_pop(e: Column): Column
var_pop(columnName: String): Column
----

| var_samp
a| [[var_samp]]

[source, scala]
----
var_samp(e: Column): Column
var_samp(columnName: String): Column
----
|===

=== [[grouping]] `grouping` Aggregate Function

[source, scala]
----
grouping(e: Column): Column
grouping(columnName: String): Column  // <1>
----
<1> Calls the first `grouping` with `columnName` as a `Column`

`grouping` is an aggregate function that indicates whether a specified column is aggregated or not and:

* returns `1` if the column is in a subtotal and is `NULL`
* returns `0` if the underlying value is `NULL` or any other value

NOTE: `grouping` can only be used with link:spark-sql-multi-dimensional-aggregation.adoc#cube[cube], link:spark-sql-multi-dimensional-aggregation.adoc#rollup[rollup] or `GROUPING SETS` multi-dimensional aggregate operators (and is verified when link:spark-sql-Analyzer-CheckAnalysis.adoc#Grouping[`Analyzer` does check analysis]).

From https://cwiki.apache.org/confluence/display/Hive/Enhanced&#43;Aggregation%2C&#43;Cube%2C&#43;Grouping&#43;and&#43;Rollup#EnhancedAggregation,Cube,GroupingandRollup-Grouping\_\_IDfunction[Hive's documentation about Grouping__ID function] (that can somehow help to understand `grouping`):

> When aggregates are displayed for a column its value is `null`. This may conflict in case the column itself has some `null` values. There needs to be some way to identify `NULL` in column, which means aggregate and `NULL` in column, which means value. `GROUPING__ID` function is the solution to that.

[source, scala]
----
val tmpWorkshops = Seq(
  ("Warsaw", 2016, 2),
  ("Toronto", 2016, 4),
  ("Toronto", 2017, 1)).toDF("city", "year", "count")

// there seems to be a bug with nulls
// and so the need for the following union
val cityNull = Seq(
  (null.asInstanceOf[String], 2016, 2)).toDF("city", "year", "count")

val workshops = tmpWorkshops union cityNull

scala> workshops.show
+-------+----+-----+
|   city|year|count|
+-------+----+-----+
| Warsaw|2016|    2|
|Toronto|2016|    4|
|Toronto|2017|    1|
|   null|2016|    2|
+-------+----+-----+

val q = workshops
  .cube("city", "year")
  .agg(grouping("city"), grouping("year")) // <-- grouping here
  .sort($"city".desc_nulls_last, $"year".desc_nulls_last)

scala> q.show
+-------+----+--------------+--------------+
|   city|year|grouping(city)|grouping(year)|
+-------+----+--------------+--------------+
| Warsaw|2016|             0|             0|
| Warsaw|null|             0|             1|
|Toronto|2017|             0|             0|
|Toronto|2016|             0|             0|
|Toronto|null|             0|             1|
|   null|2017|             1|             0|
|   null|2016|             1|             0|
|   null|2016|             0|             0|  <-- null is city
|   null|null|             0|             1|  <-- null is city
|   null|null|             1|             1|
+-------+----+--------------+--------------+
----

Internally, `grouping` creates a link:spark-sql-Column.adoc[Column] with `Grouping` expression.

```
val q = workshops.cube("city", "year").agg(grouping("city"))
scala> println(q.queryExecution.logical)
'Aggregate [cube(city#182, year#183)], [city#182, year#183, grouping('city) AS grouping(city)#705]
+- Union
   :- Project [_1#178 AS city#182, _2#179 AS year#183, _3#180 AS count#184]
   :  +- LocalRelation [_1#178, _2#179, _3#180]
   +- Project [_1#192 AS city#196, _2#193 AS year#197, _3#194 AS count#198]
      +- LocalRelation [_1#192, _2#193, _3#194]

scala> println(q.queryExecution.analyzed)
Aggregate [city#724, year#725, spark_grouping_id#721], [city#724, year#725, cast((shiftright(spark_grouping_id#721, 1) & 1) as tinyint) AS grouping(city)#720]
+- Expand [List(city#182, year#183, count#184, city#722, year#723, 0), List(city#182, year#183, count#184, city#722, null, 1), List(city#182, year#183, count#184, null, year#723, 2), List(city#182, year#183, count#184, null, null, 3)], [city#182, year#183, count#184, city#724, year#725, spark_grouping_id#721]
   +- Project [city#182, year#183, count#184, city#182 AS city#722, year#183 AS year#723]
      +- Union
         :- Project [_1#178 AS city#182, _2#179 AS year#183, _3#180 AS count#184]
         :  +- LocalRelation [_1#178, _2#179, _3#180]
         +- Project [_1#192 AS city#196, _2#193 AS year#197, _3#194 AS count#198]
            +- LocalRelation [_1#192, _2#193, _3#194]
```

NOTE: `grouping` was added to Spark SQL in https://issues.apache.org/jira/browse/SPARK-12706[[SPARK-12706\] support grouping/grouping_id function together group set].

=== [[grouping_id]] `grouping_id` Aggregate Function

[source, scala]
----
grouping_id(cols: Column*): Column
grouping_id(colName: String, colNames: String*): Column // <1>
----
<1> Calls the first `grouping_id` with `colName` and `colNames` as objects of type `Column`

`grouping_id` is an aggregate function that computes the level of grouping:

* `0` for combinations of each column
* `1` for subtotals of column 1
* `2` for subtotals of column 2
* And so on&hellip;

[source, scala]
----
val tmpWorkshops = Seq(
  ("Warsaw", 2016, 2),
  ("Toronto", 2016, 4),
  ("Toronto", 2017, 1)).toDF("city", "year", "count")

// there seems to be a bug with nulls
// and so the need for the following union
val cityNull = Seq(
  (null.asInstanceOf[String], 2016, 2)).toDF("city", "year", "count")

val workshops = tmpWorkshops union cityNull

scala> workshops.show
+-------+----+-----+
|   city|year|count|
+-------+----+-----+
| Warsaw|2016|    2|
|Toronto|2016|    4|
|Toronto|2017|    1|
|   null|2016|    2|
+-------+----+-----+

val query = workshops
  .cube("city", "year")
  .agg(grouping_id()) // <-- all grouping columns used
  .sort($"city".desc_nulls_last, $"year".desc_nulls_last)
scala> query.show
+-------+----+-------------+
|   city|year|grouping_id()|
+-------+----+-------------+
| Warsaw|2016|            0|
| Warsaw|null|            1|
|Toronto|2017|            0|
|Toronto|2016|            0|
|Toronto|null|            1|
|   null|2017|            2|
|   null|2016|            2|
|   null|2016|            0|
|   null|null|            1|
|   null|null|            3|
+-------+----+-------------+

scala> spark.catalog.listFunctions.filter(_.name.contains("grouping_id")).show(false)
+-----------+--------+-----------+----------------------------------------------------+-----------+
|name       |database|description|className                                           |isTemporary|
+-----------+--------+-----------+----------------------------------------------------+-----------+
|grouping_id|null    |null       |org.apache.spark.sql.catalyst.expressions.GroupingID|true       |
+-----------+--------+-----------+----------------------------------------------------+-----------+

// bin function gives the string representation of the binary value of the given long column
scala> query.withColumn("bitmask", bin($"grouping_id()")).show
+-------+----+-------------+-------+
|   city|year|grouping_id()|bitmask|
+-------+----+-------------+-------+
| Warsaw|2016|            0|      0|
| Warsaw|null|            1|      1|
|Toronto|2017|            0|      0|
|Toronto|2016|            0|      0|
|Toronto|null|            1|      1|
|   null|2017|            2|     10|
|   null|2016|            2|     10|
|   null|2016|            0|      0|  <-- null is city
|   null|null|            3|     11|
|   null|null|            1|      1|
+-------+----+-------------+-------+
----

The list of columns of `grouping_id` should match grouping columns (in `cube` or `rollup`) exactly, or empty which means all the grouping columns (which is exactly what the function expects).

NOTE: `grouping_id` can only be used with link:spark-sql-multi-dimensional-aggregation.adoc#cube[cube], link:spark-sql-multi-dimensional-aggregation.adoc#rollup[rollup] or `GROUPING SETS` multi-dimensional aggregate operators (and is verified when link:spark-sql-Analyzer-CheckAnalysis.adoc#GroupingID[`Analyzer` does check analysis]).

NOTE: Spark SQL's `grouping_id` function is known as `grouping__id` in Hive.

From https://cwiki.apache.org/confluence/display/Hive/Enhanced&#43;Aggregation%2C&#43;Cube%2C&#43;Grouping&#43;and&#43;Rollup#EnhancedAggregation,Cube,GroupingandRollup-Grouping\_\_IDfunction[Hive's documentation about Grouping__ID function]:

> When aggregates are displayed for a column its value is `null`. This may conflict in case the column itself has some `null` values. There needs to be some way to identify `NULL` in column, which means aggregate and `NULL` in column, which means value. `GROUPING__ID` function is the solution to that.

Internally, `grouping_id()` creates a link:spark-sql-Column.adoc[Column] with `GroupingID` unevaluable expression.

NOTE: link:spark-sql-Expression.adoc#Unevaluable[Unevaluable expressions] are expressions replaced by some other expressions during link:spark-sql-Analyzer.adoc[analysis] or link:spark-sql-Optimizer.adoc[optimization].

```
// workshops dataset was defined earlier
val q = workshops
  .cube("city", "year")
  .agg(grouping_id())

// grouping_id function is spark_grouping_id virtual column internally
// that is resolved during analysis - see Analyzed Logical Plan
scala> q.explain(true)
== Parsed Logical Plan ==
'Aggregate [cube(city#182, year#183)], [city#182, year#183, grouping_id() AS grouping_id()#742]
+- Union
   :- Project [_1#178 AS city#182, _2#179 AS year#183, _3#180 AS count#184]
   :  +- LocalRelation [_1#178, _2#179, _3#180]
   +- Project [_1#192 AS city#196, _2#193 AS year#197, _3#194 AS count#198]
      +- LocalRelation [_1#192, _2#193, _3#194]

== Analyzed Logical Plan ==
city: string, year: int, grouping_id(): int
Aggregate [city#757, year#758, spark_grouping_id#754], [city#757, year#758, spark_grouping_id#754 AS grouping_id()#742]
+- Expand [List(city#182, year#183, count#184, city#755, year#756, 0), List(city#182, year#183, count#184, city#755, null, 1), List(city#182, year#183, count#184, null, year#756, 2), List(city#182, year#183, count#184, null, null, 3)], [city#182, year#183, count#184, city#757, year#758, spark_grouping_id#754]
   +- Project [city#182, year#183, count#184, city#182 AS city#755, year#183 AS year#756]
      +- Union
         :- Project [_1#178 AS city#182, _2#179 AS year#183, _3#180 AS count#184]
         :  +- LocalRelation [_1#178, _2#179, _3#180]
         +- Project [_1#192 AS city#196, _2#193 AS year#197, _3#194 AS count#198]
            +- LocalRelation [_1#192, _2#193, _3#194]

== Optimized Logical Plan ==
Aggregate [city#757, year#758, spark_grouping_id#754], [city#757, year#758, spark_grouping_id#754 AS grouping_id()#742]
+- Expand [List(city#755, year#756, 0), List(city#755, null, 1), List(null, year#756, 2), List(null, null, 3)], [city#757, year#758, spark_grouping_id#754]
   +- Union
      :- LocalRelation [city#755, year#756]
      +- LocalRelation [city#755, year#756]

== Physical Plan ==
*HashAggregate(keys=[city#757, year#758, spark_grouping_id#754], functions=[], output=[city#757, year#758, grouping_id()#742])
+- Exchange hashpartitioning(city#757, year#758, spark_grouping_id#754, 200)
   +- *HashAggregate(keys=[city#757, year#758, spark_grouping_id#754], functions=[], output=[city#757, year#758, spark_grouping_id#754])
      +- *Expand [List(city#755, year#756, 0), List(city#755, null, 1), List(null, year#756, 2), List(null, null, 3)], [city#757, year#758, spark_grouping_id#754]
         +- Union
            :- LocalTableScan [city#755, year#756]
            +- LocalTableScan [city#755, year#756]
```

NOTE: `grouping_id` was added to Spark SQL in https://issues.apache.org/jira/browse/SPARK-12706[[SPARK-12706\] support grouping/grouping_id function together group set].
