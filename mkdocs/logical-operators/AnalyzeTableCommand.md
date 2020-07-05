title: AnalyzeTableCommand

# AnalyzeTableCommand Logical Command -- Computing Table-Level Statistics

`AnalyzeTableCommand` is a link:spark-sql-LogicalPlan-RunnableCommand.adoc[logical command] that <<run, computes statistics>> (i.e. <<total-size-stat, total size>> and <<row-count-stat, row count>>) for a <<tableIdent, table>> and stores the stats in a metastore.

`AnalyzeTableCommand` is <<creating-instance, created>> exclusively for link:spark-sql-SparkSqlAstBuilder.adoc#AnalyzeTableCommand[ANALYZE TABLE] with no `PARTITION` specification and `FOR COLUMNS` clause.

[source, scala]
----
// Seq((0, 0, "zero"), (1, 1, "one")).toDF("id", "p1", "p2").write.partitionBy("p1", "p2").saveAsTable("t1")
val sqlText = "ANALYZE TABLE t1 COMPUTE STATISTICS NOSCAN"
val plan = spark.sql(sqlText).queryExecution.logical
import org.apache.spark.sql.execution.command.AnalyzeTableCommand
val cmd = plan.asInstanceOf[AnalyzeTableCommand]
scala> println(cmd)
AnalyzeTableCommand `t1`, false
----

=== [[run]] Executing Logical Command (Computing Table-Level Statistics and Altering Metastore) -- `run` Method

[source, scala]
----
run(sparkSession: SparkSession): Seq[Row]
----

NOTE: `run` is part of <<spark-sql-LogicalPlan-RunnableCommand.adoc#run, RunnableCommand Contract>> to execute (run) a logical command.

`run` requests the session-specific `SessionCatalog` for the link:spark-sql-SessionCatalog.adoc#getTableMetadata[metadata] of the <<tableIdent, table>> and makes sure that it is not a view (aka _temporary table_).

NOTE: `run` uses the input `SparkSession` to access the session-specific link:spark-sql-SparkSession.adoc#sessionState[SessionState] that in turn gives access to the current link:spark-sql-SessionState.adoc#catalog[SessionCatalog].

[[total-size-stat]][[row-count-stat]]
`run` computes the link:spark-sql-CommandUtils.adoc#calculateTotalSize[total size] and, without <<noscan, NOSCAN>> flag, the link:spark-sql-dataset-operators.adoc#count[row count] statistics of the table.

NOTE: `run` uses `SparkSession` to link:spark-sql-SparkSession.adoc#table[find the table] in a metastore.

In the end, `run` link:spark-sql-SessionCatalog.adoc#alterTableStats[alters table statistics] if link:spark-sql-CommandUtils.adoc#compareAndGetNewStats[different from the existing table statistics in metastore].

`run` throws a `AnalysisException` when executed on a view.

```
ANALYZE TABLE is not supported on views.
```

[NOTE]
====
Row count statistics triggers a Spark job to count the number of rows in a table (that happens with `ANALYZE TABLE` with no `NOSCAN` flag).

[source, scala]
----
// Seq((0, 0, "zero"), (1, 1, "one")).toDF("id", "p1", "p2").write.partitionBy("p1", "p2").saveAsTable("t1")
val sqlText = "ANALYZE TABLE t1 COMPUTE STATISTICS"
val plan = spark.sql(sqlText).queryExecution.logical
import org.apache.spark.sql.execution.command.AnalyzeTableCommand
val cmd = plan.asInstanceOf[AnalyzeTableCommand]
scala> println(cmd)
AnalyzeTableCommand `t1`, false

// Execute ANALYZE TABLE
// Check out web UI's Jobs tab for the number of Spark jobs
// http://localhost:4040/jobs/
spark.sql(sqlText).show
----
====

=== [[creating-instance]] Creating AnalyzeTableCommand Instance

`AnalyzeTableCommand` takes the following when created:

* [[tableIdent]] `TableIdentifier`
* [[noscan]] `noscan` flag (enabled by default) that indicates whether link:spark-sql-cost-based-optimization.adoc#NOSCAN[NOSCAN] option was used or not
