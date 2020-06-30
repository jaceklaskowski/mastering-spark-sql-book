title: Cost-Based Optimization

# Cost-Based Optimization (CBO) of Logical Query Plan

*Cost-Based Optimization* (aka *Cost-Based Query Optimization* or *CBO Optimizer*) is an optimization technique in Spark SQL that uses <<statistics, table statistics>> to determine the most efficient query execution plan of a structured query (given the logical query plan).

Cost-based optimization is disabled by default. Spark SQL uses <<spark.sql.cbo.enabled, spark.sql.cbo.enabled>> configuration property to control whether the CBO should be enabled and used for query optimization or not.

Cost-Based Optimization uses <<optimizations, logical optimization rules>> (e.g. link:spark-sql-Optimizer-CostBasedJoinReorder.adoc[CostBasedJoinReorder]) to optimize the logical plan of a structured query based on statistics.

You first use <<ANALYZE-TABLE, ANALYZE TABLE COMPUTE STATISTICS>> SQL command to compute <<statistics, table statistics>>. Use <<DESCRIBE-EXTENDED, DESCRIBE EXTENDED>> SQL command to inspect the statistics.

Logical operators have <<LogicalPlanStats, statistics support>> that is used for query planning.

There is also support for <<column-histograms, equi-height column histograms>>.

=== [[statistics]] Table Statistics

The table statistics can be computed for tables, partitions and columns and are as follows:

. [[total-size-stat]] *Total size* (in bytes) of a link:spark-sql-LogicalPlan-AnalyzeTableCommand.adoc[table] or link:spark-sql-LogicalPlan-AnalyzePartitionCommand.adoc[table partitions]

. [[row-count-stat]][[rowCount]] *Row count* of a link:spark-sql-LogicalPlan-AnalyzeTableCommand.adoc[table] or link:spark-sql-LogicalPlan-AnalyzePartitionCommand.adoc[table partitions]

. [[column-stats]] link:spark-sql-LogicalPlan-AnalyzeColumnCommand.adoc[Column statistics], i.e. *min*, *max*, *num_nulls*, *distinct_count*, *avg_col_len*, *max_col_len*, *histogram*

=== [[spark.sql.cbo.enabled]] spark.sql.cbo.enabled Spark SQL Configuration Property

Cost-based optimization is enabled when link:spark-sql-properties.adoc#spark.sql.cbo.enabled[spark.sql.cbo.enabled] configuration property is turned on, i.e. `true`.

NOTE: link:spark-sql-properties.adoc#spark.sql.cbo.enabled[spark.sql.cbo.enabled] configuration property is turned off, i.e. `false`, by default.

TIP: Use link:spark-sql-SQLConf.adoc#cboEnabled[SQLConf.cboEnabled] to access the current value of `spark.sql.cbo.enabled` property.

[source, scala]
----
// CBO is disabled by default
val sqlConf = spark.sessionState.conf
scala> println(sqlConf.cboEnabled)
false

// Create a new SparkSession with CBO enabled
// You could spark-submit -c spark.sql.cbo.enabled=true
val sparkCboEnabled = spark.newSession
import org.apache.spark.sql.internal.SQLConf.CBO_ENABLED
sparkCboEnabled.conf.set(CBO_ENABLED.key, true)
val isCboEnabled = sparkCboEnabled.conf.get(CBO_ENABLED.key)
println(s"Is CBO enabled? $isCboEnabled")
----

NOTE: CBO is disabled explicitly in Spark Structured Streaming.

=== [[ANALYZE-TABLE]] ANALYZE TABLE COMPUTE STATISTICS SQL Command

Cost-Based Optimization uses the statistics stored in a metastore (aka _external catalog_) using link:spark-sql-SparkSqlAstBuilder.adoc#ANALYZE-TABLE[ANALYZE TABLE] SQL command.

[[NOSCAN]]
```
ANALYZE TABLE tableIdentifier partitionSpec?
COMPUTE STATISTICS (NOSCAN | FOR COLUMNS identifierSeq)?
```

Depending on the variant, `ANALYZE TABLE` computes different <<statistics, statistics>>, i.e. of a table, partitions or columns.

. `ANALYZE TABLE` with neither `PARTITION` specification nor `FOR COLUMNS` clause

. `ANALYZE TABLE` with `PARTITION` specification (but no `FOR COLUMNS` clause)

. `ANALYZE TABLE` with `FOR COLUMNS` clause (but no `PARTITION` specification)

[[spark.sql.statistics.histogram.enabled]]
[TIP]
====
Use link:spark-sql-properties.adoc#spark.sql.statistics.histogram.enabled[spark.sql.statistics.histogram.enabled] configuration property to enable column (equi-height) histograms that can provide better estimation accuracy but cause an extra table scan).

`spark.sql.statistics.histogram.enabled` is off by default.
====

[NOTE]
====
`ANALYZE TABLE` with `PARTITION` specification and `FOR COLUMNS` clause is incorrect.

```
// !!! INCORRECT !!!
ANALYZE TABLE t1 PARTITION (p1, p2) COMPUTE STATISTICS FOR COLUMNS id, p1
```

In such a case, `SparkSqlAstBuilder` reports a WARN message to the logs and simply ignores the partition specification.

```
WARN Partition specification is ignored when collecting column statistics: [partitionSpec]
```
====

When executed, the above `ANALYZE TABLE` variants are link:spark-sql-SparkSqlAstBuilder.adoc#ANALYZE-TABLE[translated] to the following logical commands (in a logical query plan), respectively:

. link:spark-sql-LogicalPlan-AnalyzeTableCommand.adoc[AnalyzeTableCommand]

. link:spark-sql-LogicalPlan-AnalyzePartitionCommand.adoc[AnalyzePartitionCommand]

. link:spark-sql-LogicalPlan-AnalyzeColumnCommand.adoc[AnalyzeColumnCommand]

=== [[DESCRIBE-EXTENDED]] DESCRIBE EXTENDED SQL Command

You can view the statistics of a table, partitions or a column (stored in a metastore) using link:spark-sql-SparkSqlAstBuilder.adoc#DESCRIBE[DESCRIBE EXTENDED] SQL command.

```
(DESC | DESCRIBE) TABLE? (EXTENDED | FORMATTED)?
tableIdentifier partitionSpec? describeColName?
```

Table-level statistics are in *Statistics* row while partition-level statistics are in *Partition Statistics* row.

TIP: Use `DESC EXTENDED tableName` for table-level statistics and `DESC EXTENDED tableName PARTITION (p1, p2, ...)` for partition-level statistics only.

[source, scala]
----
// table-level statistics are in Statistics row
scala> sql("DESC EXTENDED t1").show(numRows = 30, truncate = false)
+----------------------------+--------------------------------------------------------------+-------+
|col_name                    |data_type                                                     |comment|
+----------------------------+--------------------------------------------------------------+-------+
|id                          |int                                                           |null   |
|p1                          |int                                                           |null   |
|p2                          |string                                                        |null   |
|# Partition Information     |                                                              |       |
|# col_name                  |data_type                                                     |comment|
|p1                          |int                                                           |null   |
|p2                          |string                                                        |null   |
|                            |                                                              |       |
|# Detailed Table Information|                                                              |       |
|Database                    |default                                                       |       |
|Table                       |t1                                                            |       |
|Owner                       |jacek                                                         |       |
|Created Time                |Wed Dec 27 14:10:44 CET 2017                                  |       |
|Last Access                 |Thu Jan 01 01:00:00 CET 1970                                  |       |
|Created By                  |Spark 2.3.0                                                   |       |
|Type                        |MANAGED                                                       |       |
|Provider                    |parquet                                                       |       |
|Table Properties            |[transient_lastDdlTime=1514453141]                            |       |
|Statistics                  |714 bytes, 2 rows                                             |       |
|Location                    |file:/Users/jacek/dev/oss/spark/spark-warehouse/t1            |       |
|Serde Library               |org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe   |       |
|InputFormat                 |org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |       |
|OutputFormat                |org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat|       |
|Storage Properties          |[serialization.format=1]                                      |       |
|Partition Provider          |Catalog                                                       |       |
+----------------------------+--------------------------------------------------------------+-------+

scala> spark.table("t1").show
+---+---+----+
| id| p1|  p2|
+---+---+----+
|  0|  0|zero|
|  1|  1| one|
+---+---+----+

// partition-level statistics are in Partition Statistics row
scala> sql("DESC EXTENDED t1 PARTITION (p1=0, p2='zero')").show(numRows = 30, truncate = false)
+--------------------------------+---------------------------------------------------------------------------------+-------+
|col_name                        |data_type                                                                        |comment|
+--------------------------------+---------------------------------------------------------------------------------+-------+
|id                              |int                                                                              |null   |
|p1                              |int                                                                              |null   |
|p2                              |string                                                                           |null   |
|# Partition Information         |                                                                                 |       |
|# col_name                      |data_type                                                                        |comment|
|p1                              |int                                                                              |null   |
|p2                              |string                                                                           |null   |
|                                |                                                                                 |       |
|# Detailed Partition Information|                                                                                 |       |
|Database                        |default                                                                          |       |
|Table                           |t1                                                                               |       |
|Partition Values                |[p1=0, p2=zero]                                                                  |       |
|Location                        |file:/Users/jacek/dev/oss/spark/spark-warehouse/t1/p1=0/p2=zero                  |       |
|Serde Library                   |org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe                      |       |
|InputFormat                     |org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat                    |       |
|OutputFormat                    |org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat                   |       |
|Storage Properties              |[path=file:/Users/jacek/dev/oss/spark/spark-warehouse/t1, serialization.format=1]|       |
|Partition Parameters            |{numFiles=1, transient_lastDdlTime=1514469540, totalSize=357}                    |       |
|Partition Statistics            |357 bytes, 1 rows                                                                |       |
|                                |                                                                                 |       |
|# Storage Information           |                                                                                 |       |
|Location                        |file:/Users/jacek/dev/oss/spark/spark-warehouse/t1                               |       |
|Serde Library                   |org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe                      |       |
|InputFormat                     |org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat                    |       |
|OutputFormat                    |org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat                   |       |
|Storage Properties              |[serialization.format=1]                                                         |       |
+--------------------------------+---------------------------------------------------------------------------------+-------+
----

You can view the statistics of a single column using `DESC EXTENDED tableName columnName` that are in a Dataset with two columns, i.e. `info_name` and `info_value`.

[source, scala]
----
scala> sql("DESC EXTENDED t1 id").show
+--------------+----------+
|info_name     |info_value|
+--------------+----------+
|col_name      |id        |
|data_type     |int       |
|comment       |NULL      |
|min           |0         |
|max           |1         |
|num_nulls     |0         |
|distinct_count|2         |
|avg_col_len   |4         |
|max_col_len   |4         |
|histogram     |NULL      |
+--------------+----------+


scala> sql("DESC EXTENDED t1 p1").show
+--------------+----------+
|info_name     |info_value|
+--------------+----------+
|col_name      |p1        |
|data_type     |int       |
|comment       |NULL      |
|min           |0         |
|max           |1         |
|num_nulls     |0         |
|distinct_count|2         |
|avg_col_len   |4         |
|max_col_len   |4         |
|histogram     |NULL      |
+--------------+----------+


scala> sql("DESC EXTENDED t1 p2").show
+--------------+----------+
|info_name     |info_value|
+--------------+----------+
|col_name      |p2        |
|data_type     |string    |
|comment       |NULL      |
|min           |NULL      |
|max           |NULL      |
|num_nulls     |0         |
|distinct_count|2         |
|avg_col_len   |4         |
|max_col_len   |4         |
|histogram     |NULL      |
+--------------+----------+
----

=== [[optimizations]] Cost-Based Optimizations

The link:spark-sql-Optimizer.adoc[Spark Optimizer] uses heuristics (rules) that are applied to a logical query plan for cost-based optimization.

Among the optimization rules are the following:

1. link:spark-sql-Optimizer-CostBasedJoinReorder.adoc[CostBasedJoinReorder] logical optimization rule for join reordering with 2 or more consecutive inner or cross joins (possibly separated by `Project` operators) when link:spark-sql-properties.adoc#spark.sql.cbo.enabled[spark.sql.cbo.enabled] and link:spark-sql-properties.adoc#spark.sql.cbo.joinReorder.enabled[spark.sql.cbo.joinReorder.enabled] configuration properties are both enabled.

=== [[commands]] Logical Commands for Altering Table Statistics

The following are the logical commands that link:spark-sql-SessionCatalog.adoc#alterTableStats[alter table statistics in a metastore] (aka _external catalog_):

. link:spark-sql-LogicalPlan-AnalyzeTableCommand.adoc[AnalyzeTableCommand]

. link:spark-sql-LogicalPlan-AnalyzeColumnCommand.adoc[AnalyzeColumnCommand]

. `AlterTableAddPartitionCommand`

. `AlterTableDropPartitionCommand`

. `AlterTableSetLocationCommand`

. `TruncateTableCommand`

. link:hive/InsertIntoHiveTable.adoc[InsertIntoHiveTable]

. link:spark-sql-LogicalPlan-InsertIntoHadoopFsRelationCommand.adoc[InsertIntoHadoopFsRelationCommand]

. `LoadDataCommand`

=== [[EXPLAIN-COST]] EXPLAIN COST SQL Command

CAUTION: FIXME See link:spark-sql-LogicalPlanStats.adoc[LogicalPlanStats]

=== [[LogicalPlanStats]] LogicalPlanStats -- Statistics Estimates of Logical Operator

link:spark-sql-LogicalPlanStats.adoc[LogicalPlanStats] adds statistics support to logical operators and is used for query planning (with or without cost-based optimization, e.g. link:spark-sql-Optimizer-CostBasedJoinReorder.adoc[CostBasedJoinReorder] or link:spark-sql-SparkStrategy-JoinSelection.adoc[JoinSelection], respectively).

=== [[column-histograms]] Equi-Height Histograms for Columns

From https://issues.apache.org/jira/browse/SPARK-17074[SPARK-17074 generate equi-height histogram for column]:

[quote]
____
Equi-height histogram is effective in handling skewed data distribution.

For equi-height histogram, the heights of all bins(intervals) are the same. The default number of bins we use is 254.

Now we use a two-step method to generate an equi-height histogram:
1. use percentile_approx to get percentiles (end points of the equi-height bin intervals);
2. use a new aggregate function to get distinct counts in each of these bins.

Note that this method takes two table scans. In the future we may provide other algorithms which need only one table scan.
____


From https://github.com/apache/spark/pull/19479[++[SPARK-17074] [SQL] Generate equi-height histogram in column statistics #19479++]:

[quote]
____
Equi-height histogram is effective in cardinality estimation, and more accurate than basic column stats (min, max, ndv, etc) especially in skew distribution.

For equi-height histogram, all buckets (intervals) have the same height (frequency).

we use a two-step method to generate an equi-height histogram:

1. use ApproximatePercentile to get percentiles p(0), p(1/n), p(2/n) ... p((n-1)/n), p(1);

2. construct range values of buckets, e.g. [p(0), p(1/n)], [p(1/n), p(2/n)] ... [p((n-1)/n), p(1)], and use ApproxCountDistinctForIntervals to count ndv in each bucket. Each bucket is of the form: (lowerBound, higherBound, ndv).
____

Spark SQL uses link:spark-sql-ColumnStat.adoc[column statistics] that may optionally hold the link:spark-sql-ColumnStat.adoc#histogram[histogram of values] (which is empty by default). With link:spark-sql-properties.adoc#spark.sql.statistics.histogram.enabled[spark.sql.statistics.histogram.enabled] configuration property turned on <<ANALYZE-TABLE, ANALYZE TABLE COMPUTE STATISTICS FOR COLUMNS>> SQL command generates column (equi-height) histograms.

NOTE: `spark.sql.statistics.histogram.enabled` is off by default.

[source, scala]
----
// Computing column statistics with histogram
// ./bin/spark-shell --conf spark.sql.statistics.histogram.enabled=true
scala> spark.sessionState.conf.histogramEnabled
res1: Boolean = true

val tableName = "t1"

// Make the example reproducible
import org.apache.spark.sql.catalyst.TableIdentifier
val tid = TableIdentifier(tableName)
val sessionCatalog = spark.sessionState.catalog
sessionCatalog.dropTable(tid, ignoreIfNotExists = true, purge = true)

// CREATE TABLE t1
Seq((0, 0, "zero"), (1, 1, "one")).
  toDF("id", "p1", "p2").
  write.
  saveAsTable(tableName)

// As we drop and create immediately we may face problems with unavailable partition files
// Invalidate cache
spark.sql(s"REFRESH TABLE $tableName")

// Use ANALYZE TABLE...FOR COLUMNS to compute column statistics
// that saves them in a metastore (aka an external catalog)
val df = spark.table(tableName)
val allCols = df.columns.mkString(",")
val analyzeTableSQL = s"ANALYZE TABLE t1 COMPUTE STATISTICS FOR COLUMNS $allCols"
spark.sql(analyzeTableSQL)

// Column statistics with histogram should be in the external catalog (metastore)
----

You can inspect the column statistics using <<DESCRIBE-EXTENDED, DESCRIBE EXTENDED>> SQL command.

[source, scala]
----
// Inspecting column statistics with column histogram
// See the above example for how to compute the stats
val colName = "id"
val descExtSQL = s"DESC EXTENDED $tableName $colName"

// 254 bins by default --> num_of_bins in histogram row below
scala> sql(descExtSQL).show(truncate = false)
+--------------+-----------------------------------------------------+
|info_name     |info_value                                           |
+--------------+-----------------------------------------------------+
|col_name      |id                                                   |
|data_type     |int                                                  |
|comment       |NULL                                                 |
|min           |0                                                    |
|max           |1                                                    |
|num_nulls     |0                                                    |
|distinct_count|2                                                    |
|avg_col_len   |4                                                    |
|max_col_len   |4                                                    |
|histogram     |height: 0.007874015748031496, num_of_bins: 254       |
|bin_0         |lower_bound: 0.0, upper_bound: 0.0, distinct_count: 1|
|bin_1         |lower_bound: 0.0, upper_bound: 0.0, distinct_count: 1|
|bin_2         |lower_bound: 0.0, upper_bound: 0.0, distinct_count: 1|
|bin_3         |lower_bound: 0.0, upper_bound: 0.0, distinct_count: 1|
|bin_4         |lower_bound: 0.0, upper_bound: 0.0, distinct_count: 1|
|bin_5         |lower_bound: 0.0, upper_bound: 0.0, distinct_count: 1|
|bin_6         |lower_bound: 0.0, upper_bound: 0.0, distinct_count: 1|
|bin_7         |lower_bound: 0.0, upper_bound: 0.0, distinct_count: 1|
|bin_8         |lower_bound: 0.0, upper_bound: 0.0, distinct_count: 1|
|bin_9         |lower_bound: 0.0, upper_bound: 0.0, distinct_count: 1|
+--------------+-----------------------------------------------------+
only showing top 20 rows
----
