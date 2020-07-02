title: InsertIntoDataSourceCommand

# InsertIntoDataSourceCommand Logical Command

`InsertIntoDataSourceCommand` is a <<spark-sql-LogicalPlan-RunnableCommand.adoc#, RunnableCommand>> that <<run, inserts or overwrites data in an InsertableRelation>> (per <<overwrite, overwrite>> flag).

`InsertIntoDataSourceCommand` is <<creating-instance, created>> exclusively when `DataSourceAnalysis` logical resolution is <<spark-sql-Analyzer-DataSourceAnalysis.adoc#apply, executed>> (and <<spark-sql-Analyzer-DataSourceAnalysis.adoc#InsertIntoTable-InsertableRelation, resolves>> an <<InsertIntoTable.adoc#, InsertIntoTable>> unary logical operator with a <<spark-sql-LogicalPlan-LogicalRelation.adoc#, LogicalRelation>> on an <<spark-sql-InsertableRelation.adoc#, InsertableRelation>>).

[source, plaintext]
----
sql("DROP TABLE IF EXISTS t2")
sql("CREATE TABLE t2(id long)")

val query = "SELECT * FROM RANGE(1)"
// Using INSERT INTO SQL statement so we can access QueryExecution
// DataFrameWriter.insertInto returns no value
val q = sql("INSERT INTO TABLE t2 " + query)
val logicalPlan = q.queryExecution.logical
scala> println(logicalPlan.numberedTreeString)
00 'InsertIntoTable 'UnresolvedRelation `t2`, false, false
01 +- 'Project [*]
02    +- 'UnresolvedTableValuedFunction RANGE, [1]

val analyzedPlan = q.queryExecution.analyzed
scala> println(analyzedPlan.numberedTreeString)
00 InsertIntoHiveTable `default`.`t2`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, false, false, [id#6L]
01 +- Project [id#6L]
02    +- Range (0, 1, step=1, splits=None)
----

[[innerChildren]]
`InsertIntoDataSourceCommand` returns the <<query, logical query plan>> when requested for the <<spark-sql-catalyst-TreeNode.adoc#innerChildren, inner nodes>> (that should be shown as an inner nested tree of this node).

[source, plaintext]
----
val query = "SELECT * FROM RANGE(1)"
val sqlText = "INSERT INTO TABLE t2 " + query
val plan = spark.sessionState.sqlParser.parsePlan(sqlText)
scala> println(plan.numberedTreeString)
00 'InsertIntoTable 'UnresolvedRelation `t2`, false, false
01 +- 'Project [*]
02    +- 'UnresolvedTableValuedFunction RANGE, [1]
----

=== [[creating-instance]] Creating InsertIntoDataSourceCommand Instance

`InsertIntoDataSourceCommand` takes the following to be created:

* [[logicalRelation]] <<spark-sql-LogicalPlan-LogicalRelation.adoc#, LogicalRelation>> leaf logical operator
* [[query]] <<spark-sql-LogicalPlan.adoc#, Logical query plan>>
* [[overwrite]] `overwrite` flag

=== [[run]] Executing Logical Command (Inserting or Overwriting Data in InsertableRelation) -- `run` Method

[source, scala]
----
run(
  session: SparkSession): Seq[Row]
----

NOTE: `run` is part of <<spark-sql-LogicalPlan-RunnableCommand.adoc#run, RunnableCommand Contract>> to execute (run) a logical command.

`run` takes the <<spark-sql-InsertableRelation.adoc#, InsertableRelation>> (that is the <<spark-sql-LogicalPlan-LogicalRelation.adoc#relation, relation>> of the <<logicalRelation, LogicalRelation>>).

`run` then <<spark-sql-Dataset.adoc#ofRows, creates a DataFrame>> for the <<query, logical query plan>> and the input `SparkSession`.

`run` requests the `DataFrame` for the <<spark-sql-Dataset.adoc#queryExecution, QueryExecution>> that in turn is requested for the <<spark-sql-QueryExecution.adoc#toRdd, RDD>> (of the structured query). `run` requests the <<logicalRelation, LogicalRelation>> for the <<spark-sql-catalyst-QueryPlan.adoc#schema, output schema>>.

With the RDD and the output schema, `run` creates <<spark-sql-SparkSession.adoc#internalCreateDataFrame, another DataFrame>> that is the `RDD[InternalRow]` with the schema applied.

`run` requests the `InsertableRelation` to <<spark-sql-InsertableRelation.adoc#insert, insert or overwrite data>>.

In the end, since the data in the `InsertableRelation` has changed, `run` requests the `CacheManager` to <<spark-sql-CacheManager.adoc#recacheByPlan, recacheByPlan>> with the <<logicalRelation, LogicalRelation>>.

NOTE: `run` requests the `SparkSession` for <<spark-sql-SparkSession.adoc#sharedState, SharedState>> that is in turn requested for the <<spark-sql-SharedState.adoc#cacheManager, CacheManager>>.
