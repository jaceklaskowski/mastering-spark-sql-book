title: WriteToDataSourceV2Exec

# WriteToDataSourceV2Exec Physical Operator

`WriteToDataSourceV2Exec` is a <<spark-sql-SparkPlan.adoc#, physical operator>> that represents an <<spark-sql-LogicalPlan-AppendData.adoc#, AppendData>> logical operator (and a deprecated <<spark-sql-LogicalPlan-WriteToDataSourceV2.adoc#, WriteToDataSourceV2>> logical operator) at execution time.

`WriteToDataSourceV2Exec` is <<creating-instance, created>> exclusively when <<spark-sql-SparkStrategy-DataSourceV2Strategy.adoc#, DataSourceV2Strategy>> execution planning strategy is requested to plan an <<spark-sql-SparkStrategy-DataSourceV2Strategy.adoc#apply-AppendData, AppendData>> logical operator (and a deprecated <<spark-sql-SparkStrategy-DataSourceV2Strategy.adoc#apply-WriteToDataSourceV2, WriteToDataSourceV2>>).

NOTE: Although <<spark-sql-LogicalPlan-WriteToDataSourceV2.adoc#, WriteToDataSourceV2>> logical operator is deprecated since Spark SQL 2.4.0 (for <<spark-sql-LogicalPlan-AppendData.adoc#, AppendData>> logical operator), the `AppendData` logical operator is currently used in tests only. That makes `WriteToDataSourceV2` logical operator still relevant.

[[creating-instance]]
`WriteToDataSourceV2Exec` takes the following to be created:

* [[writer]] <<spark-sql-DataSourceWriter.adoc#, DataSourceWriter>>
* [[query]] Child <<spark-sql-SparkPlan.adoc#, physical plan>>

[[children]]
When requested for the <<spark-sql-catalyst-TreeNode.adoc#children, child operators>>, `WriteToDataSourceV2Exec` gives the one <<query, child physical plan>>.

[[output]]
When requested for the <<spark-sql-catalyst-QueryPlan.adoc#output, output attributes>>, `WriteToDataSourceV2Exec` gives no attributes (an empty collection).

[[logging]]
[TIP]
====
Enable `INFO` logging level for `org.apache.spark.sql.execution.datasources.v2.WriteToDataSourceV2Exec` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.execution.datasources.v2.WriteToDataSourceV2Exec=INFO
```

Refer to <<spark-logging.adoc#, Logging>>.
====

=== [[doExecute]] Executing Physical Operator (Generating RDD[InternalRow]) -- `doExecute` Method

[source, scala]
----
doExecute(): RDD[InternalRow]
----

NOTE: `doExecute` is part of <<spark-sql-SparkPlan.adoc#doExecute, SparkPlan Contract>> to generate the runtime representation of a structured query as a distributed computation over <<spark-sql-InternalRow.adoc#, internal binary rows>> on Apache Spark (i.e. `RDD[InternalRow]`).

`doExecute` requests the <<writer, DataSourceWriter>> to <<spark-sql-DataSourceWriter.adoc#createWriterFactory, create a DataWriterFactory>> for the writing task.

`doExecute` requests the <<writer, DataSourceWriter>> to <<spark-sql-DataSourceWriter.adoc#useCommitCoordinator, use a CommitCoordinator or not>>.

`doExecute` requests the <<query, child physical plan>> to <<spark-sql-SparkPlan.adoc#execute, execute>> (that triggers physical query planning and in the end generates an `RDD` of <<spark-sql-InternalRow.adoc#, internal binary rows>>).

`doExecute` prints out the following INFO message to the logs:

```
Start processing data source writer: [writer]. The input RDD has [length] partitions.
```

[[doExecute-runJob]]
`doExecute` requests the <<spark-sql-SparkPlan.adoc#sparkContext, SparkContext>> to run a Spark job with the following:

* The `RDD[InternalRow]` of the <<query, child physical plan>>

* A partition processing function that requests the `DataWritingSparkTask` object to <<spark-sql-DataWritingSparkTask.adoc#run, run>> the writing task (of the <<writer, DataSourceWriter>>) with or with no commit coordinator

* A result handler function that records the result `WriterCommitMessage` from a successful data writer and requests the <<writer, DataSourceWriter>> to <<spark-sql-DataSourceWriter.adoc#onDataWriterCommit, handle the commit message>> (which does nothing by default)

`doExecute` prints out the following INFO message to the logs:

```
Data source writer [writer] is committing.
```

`doExecute` requests the <<writer, DataSourceWriter>> to <<spark-sql-DataSourceWriter.adoc#commit, commit>> (passing on with the commit messages).

In the end, `doExecute` prints out the following INFO message to the logs:

```
Data source writer [writer] committed.
```

In case of any error (`Throwable`), `doExecute`...FIXME
