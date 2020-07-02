title: CreateTempViewUsing

# CreateTempViewUsing Logical Command

`CreateTempViewUsing` is a <<spark-sql-LogicalPlan-RunnableCommand.adoc#, logical command>> for <<run, creating or replacing a temporary view>> (global or not) using a <<provider, data source>>.

`CreateTempViewUsing` is <<creating-instance, created>> to represent <<spark-sql-SparkSqlAstBuilder.adoc#visitCreateTempViewUsing, CREATE TEMPORARY VIEW &hellip; USING>> SQL statements.

[source, scala]
----
val sqlText = s"""
    |CREATE GLOBAL TEMPORARY VIEW myTempCsvView
    |(id LONG, name STRING)
    |USING csv
  """.stripMargin
// Logical commands are executed at analysis
scala> sql(sqlText)
res4: org.apache.spark.sql.DataFrame = []

scala> spark.catalog.listTables(spark.sharedState.globalTempViewManager.database).show
+-------------+-----------+-----------+---------+-----------+
|         name|   database|description|tableType|isTemporary|
+-------------+-----------+-----------+---------+-----------+
|mytempcsvview|global_temp|       null|TEMPORARY|       true|
+-------------+-----------+-----------+---------+-----------+
----

=== [[run]] Executing Logical Command -- `run` Method

[source, scala]
----
run(sparkSession: SparkSession): Seq[Row]
----

NOTE: `run` is part of <<spark-sql-LogicalPlan-RunnableCommand.adoc#run, RunnableCommand Contract>> to execute (run) a logical command.

`run` creates a <<spark-sql-DataSource.adoc#apply, DataSource>> and requests it to <<spark-sql-DataSource.adoc#resolveRelation, resolve itself>> (i.e. create a <<spark-sql-BaseRelation.adoc#, BaseRelation>>).

`run` then requests the input `SparkSession` to <<spark-sql-SparkSession.adoc#baseRelationToDataFrame, create a DataFrame from the BaseRelation>> that is used to <<spark-sql-Dataset.adoc#logicalPlan, get the analyzed logical plan>> (that is the view definition of the temporary table).

Depending on the <<global, global>> flag, `run` requests the `SessionCatalog` to <<spark-sql-SessionCatalog.adoc#createGlobalTempView, createGlobalTempView>> (`global` flag is on) or <<spark-sql-SessionCatalog.adoc#createTempView, createTempView>> (`global` flag is off).

`run` throws an `AnalysisException` when executed with `hive` <<provider, provider>>.

```
Hive data source can only be used with tables, you can't use it with CREATE TEMP VIEW USING
```
=== [[creating-instance]] Creating CreateTempViewUsing Instance

`CreateTempViewUsing` takes the following when created:

* [[tableIdent]] `TableIdentifier`
* [[userSpecifiedSchema]] Optional user-defined schema (as <<spark-sql-StructType.adoc#, StructType>>)
* [[replace]] `replace` flag
* [[global]] `global` flag
* [[provider]] Name of the <<spark-sql-DataSource.adoc#, data source provider>>
* [[options]] Options (as `Map[String, String]`)

=== [[argString]] `argString` Method

[source, scala]
----
argString: String
----

NOTE: `argString` is part of the <<spark-sql-catalyst-TreeNode.adoc#argString, TreeNode Contract>> to...FIXME.

`argString`...FIXME
