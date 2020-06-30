title: Basic Aggregation

# Basic Aggregation -- Typed and Untyped Grouping Operators

You can calculate aggregates over a group of rows in a link:spark-sql-Dataset.adoc[Dataset] using <<aggregate-operators, aggregate operators>> (possibly with link:spark-sql-functions.adoc#aggregate-functions[aggregate functions]).

[[aggregate-operators]]
.Aggregate Operators
[width="100%",cols="1,1,2",options="header"]
|===
| Operator
| Return Type
| Description

| <<agg, agg>>
| link:spark-sql-RelationalGroupedDataset.adoc[RelationalGroupedDataset]
| Aggregates with or without grouping (i.e. over an entire Dataset)

| <<groupBy, groupBy>>
| link:spark-sql-RelationalGroupedDataset.adoc[RelationalGroupedDataset]
| Used for *untyped aggregates* using DataFrames. Grouping is described using link:spark-sql-Column.adoc[column expressions] or column names.

| <<groupByKey, groupByKey>>
| link:spark-sql-KeyValueGroupedDataset.adoc[KeyValueGroupedDataset]
| Used for *typed aggregates* using Datasets with records grouped by a key-defining discriminator function.
|===

NOTE: Aggregate functions without aggregate operators return a single value. If you want to find the aggregate values for each unique value (in a column), you should <<groupBy, groupBy>> first (over this column) to build the groups.

[NOTE]
====
You can also use link:spark-sql-SparkSession.adoc#sql[SparkSession] to execute _good ol'_ SQL with `GROUP BY` should you prefer.

[source, scala]
----
val spark: SparkSession = ???
spark.sql("SELECT COUNT(*) FROM sales GROUP BY city")
----

SQL or Dataset API's operators go through the same query planning and optimizations, and have the same performance characteristic in the end.
====

=== [[agg]] Aggregates Over Subset Of or Whole Dataset -- `agg` Operator

[source, scala]
----
agg(expr: Column, exprs: Column*): DataFrame
agg(exprs: Map[String, String]): DataFrame
agg(aggExpr: (String, String), aggExprs: (String, String)*): DataFrame
----

<<agg, agg>> applies an aggregate function on a subset or the entire `Dataset` (i.e. considering the entire data set as one group).

NOTE: `agg` on a `Dataset` is simply a shortcut for <<groupBy, groupBy().agg(...)>>.

[source, scala]
----
scala> spark.range(10).agg(sum('id) as "sum").show
+---+
|sum|
+---+
| 45|
+---+
----

`agg` can compute aggregate expressions on all the records in a `Dataset`.

=== [[groupBy]] Untyped Grouping -- `groupBy` Operator

[source, scala]
----
groupBy(cols: Column*): RelationalGroupedDataset
groupBy(col1: String, cols: String*): RelationalGroupedDataset
----

`groupBy` operator groups the rows in a `Dataset` by columns (as link:spark-sql-Column.adoc[Column expressions] or names).

`groupBy` gives a link:spark-sql-RelationalGroupedDataset.adoc[RelationalGroupedDataset] to execute aggregate functions or operators.

[source, scala]
----
// 10^3-record large data set
val ints = 1 to math.pow(10, 3).toInt
val nms = ints.toDF("n").withColumn("m", 'n % 2)
scala> nms.count
res0: Long = 1000

val q = nms.
  groupBy('m).
  agg(sum('n) as "sum").
  orderBy('m)
scala> q.show
+---+------+
|  m|   sum|
+---+------+
|  0|250500|
|  1|250000|
+---+------+
----

Internally, `groupBy` link:spark-sql-Dataset.adoc#resolve[resolves column names] (possibly quoted) and link:spark-sql-RelationalGroupedDataset.adoc#creating-instance[creates] a `RelationalGroupedDataset` (with link:spark-sql-RelationalGroupedDataset.adoc#groupType[groupType] being `GroupByType`).

NOTE: The following uses the data setup as described in <<test-setup, Test Setup>> section below.

[source, scala]
----
scala> tokens.show
+----+---------+-----+
|name|productId|score|
+----+---------+-----+
| aaa|      100| 0.12|
| aaa|      200| 0.29|
| bbb|      200| 0.53|
| bbb|      300| 0.42|
+----+---------+-----+

scala> tokens.groupBy('name).avg().show
+----+--------------+----------+
|name|avg(productId)|avg(score)|
+----+--------------+----------+
| aaa|         150.0|     0.205|
| bbb|         250.0|     0.475|
+----+--------------+----------+

scala> tokens.groupBy('name, 'productId).agg(Map("score" -> "avg")).show
+----+---------+----------+
|name|productId|avg(score)|
+----+---------+----------+
| aaa|      200|      0.29|
| bbb|      200|      0.53|
| bbb|      300|      0.42|
| aaa|      100|      0.12|
+----+---------+----------+

scala> tokens.groupBy('name).count.show
+----+-----+
|name|count|
+----+-----+
| aaa|    2|
| bbb|    2|
+----+-----+

scala> tokens.groupBy('name).max("score").show
+----+----------+
|name|max(score)|
+----+----------+
| aaa|      0.29|
| bbb|      0.53|
+----+----------+

scala> tokens.groupBy('name).sum("score").show
+----+----------+
|name|sum(score)|
+----+----------+
| aaa|      0.41|
| bbb|      0.95|
+----+----------+

scala> tokens.groupBy('productId).sum("score").show
+---------+------------------+
|productId|        sum(score)|
+---------+------------------+
|      300|              0.42|
|      100|              0.12|
|      200|0.8200000000000001|
+---------+------------------+
----

=== [[groupByKey]] Typed Grouping -- `groupByKey` Operator

[source, scala]
----
groupByKey[K: Encoder](func: T => K): KeyValueGroupedDataset[K, T]
----

`groupByKey` groups records (of type `T`) by the input `func` and in the end returns a link:spark-sql-KeyValueGroupedDataset.adoc[KeyValueGroupedDataset] to apply aggregation to.

NOTE: `groupByKey` is ``Dataset``'s experimental API.

```
scala> tokens.groupByKey(_.productId).count.orderBy($"value").show
+-----+--------+
|value|count(1)|
+-----+--------+
|  100|       1|
|  200|       2|
|  300|       1|
+-----+--------+

import org.apache.spark.sql.expressions.scalalang._
val q = tokens.
  groupByKey(_.productId).
  agg(typed.sum[Token](_.score)).
  toDF("productId", "sum").
  orderBy('productId)
scala> q.show
+---------+------------------+
|productId|               sum|
+---------+------------------+
|      100|              0.12|
|      200|0.8200000000000001|
|      300|              0.42|
+---------+------------------+
```

=== [[test-setup]] Test Setup

This is a setup for learning `GroupedData`. Paste it into Spark Shell using `:paste`.

[source, scala]
----
import spark.implicits._

case class Token(name: String, productId: Int, score: Double)
val data = Seq(
  Token("aaa", 100, 0.12),
  Token("aaa", 200, 0.29),
  Token("bbb", 200, 0.53),
  Token("bbb", 300, 0.42))
val tokens = data.toDS.cache  // <1>
----
<1> Cache the dataset so the following queries won't load/recompute data over and over again.
