title: SaveIntoDataSourceCommand

# SaveIntoDataSourceCommand Logical Command

`SaveIntoDataSourceCommand` is a <<spark-sql-LogicalPlan-RunnableCommand.adoc#, logical command>> that, when <<run, executed>>, FIXME.

`SaveIntoDataSourceCommand` is <<creating-instance, created>> exclusively when `DataSource` is requested to <<spark-sql-DataSource.adoc#planForWriting, create a logical command for writing>> (to a <<spark-sql-CreatableRelationProvider.adoc#implementations, CreatableRelationProvider>> data source).

[[innerChildren]]
`SaveIntoDataSourceCommand` returns the <<query, logical query plan>> when requested for the <<spark-sql-catalyst-TreeNode.adoc#innerChildren, inner nodes (that should be shown as an inner nested tree of this node)>>.

[source, scala]
----
// DEMO Example with inner nodes that should be shown as an inner nested tree of this node

val lines = Seq("SaveIntoDataSourceCommand").toDF("line")

// NOTE: There are two CreatableRelationProviders: jdbc and kafka
// jdbc is simpler to use in spark-shell as it does not need --packages
val url = "jdbc:derby:memory:;databaseName=/tmp/test;create=true"
val requiredOpts = Map("url" -> url, "dbtable" -> "lines")
// Use overwrite SaveMode to make the demo reproducible
import org.apache.spark.sql.SaveMode.Overwrite
lines.write.options(requiredOpts).format("jdbc").mode(Overwrite).save

// Go to web UI's SQL tab and see the last executed query
----

[[simpleString]]
`SaveIntoDataSourceCommand` <<spark-sql-SQLConf.adoc#redactOptions, redacts>> the <<options, options>> for the <<spark-sql-catalyst-QueryPlan.adoc#simpleString, simple description with state prefix>>.

```
SaveIntoDataSourceCommand [dataSource], [redacted], [mode]
```

=== [[run]] Executing Logical Command -- `run` Method

[source, scala]
----
run(
  sparkSession: SparkSession): Seq[Row]
----

NOTE: `run` is part of <<spark-sql-LogicalPlan-RunnableCommand.adoc#run, RunnableCommand Contract>> to execute (run) a logical command.

`run` simply requests the <<dataSource, CreatableRelationProvider data source>> to <<spark-sql-CreatableRelationProvider.adoc#createRelation, save the rows of a structured query (a DataFrame)>>.

In the end, `run` returns an empty `Seq[Row]` (just to follow the signature and please the Scala compiler).

=== [[creating-instance]] Creating SaveIntoDataSourceCommand Instance

`SaveIntoDataSourceCommand` takes the following when created:

* [[query]] <<spark-sql-LogicalPlan.adoc#, Logical query plan>>
* [[dataSource]] <<spark-sql-CreatableRelationProvider.adoc#, CreatableRelationProvider>> data source
* [[options]] Options (as `Map[String, String]`)
* [[mode]] <<spark-sql-DataFrameWriter.adoc#SaveMode, SaveMode>>
