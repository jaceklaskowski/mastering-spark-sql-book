title: AnalyzeColumnCommand

# AnalyzeColumnCommand Logical Command

`AnalyzeColumnCommand` is a link:spark-sql-LogicalPlan-RunnableCommand.adoc[logical command] for link:spark-sql-SparkSqlAstBuilder.adoc#AnalyzeColumnCommand[ANALYZE TABLE] with `FOR COLUMNS` clause (and no `PARTITION` specification).

```
ANALYZE TABLE tableName COMPUTE STATISTICS FOR COLUMNS columnNames
```

[source, scala]
----
// Make the example reproducible
val tableName = "t1"
import org.apache.spark.sql.catalyst.TableIdentifier
val tableId = TableIdentifier(tableName)

val sessionCatalog = spark.sessionState.catalog
sessionCatalog.dropTable(tableId, ignoreIfNotExists = true, purge = true)

val df = Seq((0, 0.0, "zero"), (1, 1.4, "one")).toDF("id", "p1", "p2")
df.write.saveAsTable("t1")

// AnalyzeColumnCommand represents ANALYZE TABLE...FOR COLUMNS SQL command
val allCols = df.columns.mkString(",")
val analyzeTableSQL = s"ANALYZE TABLE $tableName COMPUTE STATISTICS FOR COLUMNS $allCols"
val plan = spark.sql(analyzeTableSQL).queryExecution.logical
import org.apache.spark.sql.execution.command.AnalyzeColumnCommand
val cmd = plan.asInstanceOf[AnalyzeColumnCommand]
scala> println(cmd)
AnalyzeColumnCommand `t1`, [id, p1, p2]

spark.sql(analyzeTableSQL)
val stats = sessionCatalog.getTableMetadata(tableId).stats.get
scala> println(stats.simpleString)
1421 bytes, 2 rows

scala> stats.colStats.map { case (c, ss) => s"$c: $ss" }.foreach(println)
id: ColumnStat(2,Some(0),Some(1),0,4,4,None)
p1: ColumnStat(2,Some(0.0),Some(1.4),0,8,8,None)
p2: ColumnStat(2,None,None,0,4,4,None)

// Use DESC EXTENDED for friendlier output
scala> sql(s"DESC EXTENDED $tableName id").show
+--------------+----------+
|     info_name|info_value|
+--------------+----------+
|      col_name|        id|
|     data_type|       int|
|       comment|      NULL|
|           min|         0|
|           max|         1|
|     num_nulls|         0|
|distinct_count|         2|
|   avg_col_len|         4|
|   max_col_len|         4|
|     histogram|      NULL|
+--------------+----------+
----

`AnalyzeColumnCommand` can <<computeColumnStats, generate column histograms>> when link:spark-sql-properties.adoc#spark.sql.statistics.histogram.enabled[spark.sql.statistics.histogram.enabled] configuration property is turned on (which is disabled by default). `AnalyzeColumnCommand` supports column histograms for the following link:spark-sql-DataType.adoc[data types]:

* `IntegralType`
* `DecimalType`
* `DoubleType`
* `FloatType`
* `DateType`
* `TimestampType`

NOTE: Histograms can provide better estimation accuracy. Currently, Spark only supports equi-height histogram. Note that collecting histograms takes extra cost. For example, collecting column statistics usually takes only one table scan, but generating equi-height histogram will cause an extra table scan.

[source, scala]
----
// ./bin/spark-shell --conf spark.sql.statistics.histogram.enabled=true
// Use the above example to set up the environment
// Make sure that ANALYZE TABLE COMPUTE STATISTICS FOR COLUMNS was run with histogram enabled

// There are 254 bins by default
// Use spark.sql.statistics.histogram.numBins to control the bins
val descExtSQL = s"DESC EXTENDED $tableName p1"
scala> spark.sql(descExtSQL).show(truncate = false)
+--------------+-----------------------------------------------------+
|info_name     |info_value                                           |
+--------------+-----------------------------------------------------+
|col_name      |p1                                                   |
|data_type     |double                                               |
|comment       |NULL                                                 |
|min           |0.0                                                  |
|max           |1.4                                                  |
|num_nulls     |0                                                    |
|distinct_count|2                                                    |
|avg_col_len   |8                                                    |
|max_col_len   |8                                                    |
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

NOTE: `AnalyzeColumnCommand` is described by `analyze` labeled alternative in `statement` expression in `SqlBase.g4` and parsed using link:spark-sql-SparkSqlAstBuilder.adoc#visitAnalyze[SparkSqlAstBuilder].

NOTE: `AnalyzeColumnCommand` is not supported on views.

=== [[run]] Executing Logical Command -- `run` Method

[source, scala]
----
run(sparkSession: SparkSession): Seq[Row]
----

NOTE: `run` is part of <<spark-sql-LogicalPlan-RunnableCommand.adoc#run, RunnableCommand Contract>> to execute (run) a logical command.

`run` calculates the following statistics:

* sizeInBytes
* stats for each column

CAUTION: FIXME

=== [[computeColumnStats]] Computing Statistics for Specified Columns -- `computeColumnStats` Internal Method

[source, scala]
----
computeColumnStats(
  sparkSession: SparkSession,
  tableIdent: TableIdentifier,
  columnNames: Seq[String]): (Long, Map[String, ColumnStat])
----

`computeColumnStats`...FIXME

NOTE: `computeColumnStats` is used exclusively when `AnalyzeColumnCommand` is <<run, executed>>.

=== [[computePercentiles]] `computePercentiles` Internal Method

[source, scala]
----
computePercentiles(
  attributesToAnalyze: Seq[Attribute],
  sparkSession: SparkSession,
  relation: LogicalPlan): AttributeMap[ArrayData]
----

`computePercentiles`...FIXME

NOTE: `computePercentiles` is used exclusively when `AnalyzeColumnCommand` is <<run, executed>> (and <<computeColumnStats, computes column statistics>>).

=== [[creating-instance]] Creating AnalyzeColumnCommand Instance

`AnalyzeColumnCommand` takes the following when created:

* [[tableIdent]] `TableIdentifier`
* [[columnNames]] Column names
