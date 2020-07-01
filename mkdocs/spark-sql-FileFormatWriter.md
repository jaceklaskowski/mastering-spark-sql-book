# FileFormatWriter Utility

`FileFormatWriter` utility is used to <<write, write the result of a structured query>>.

[[logging]]
[TIP]
====
Enable `ALL` logging level for `org.apache.spark.sql.execution.datasources.FileFormatWriter` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.execution.datasources.FileFormatWriter=ALL
```

Refer to <<spark-logging.adoc#, Logging>>.
====

=== [[write]] Writing Result of Structured Query (Query Result) -- `write` Method

[source, scala]
----
write(
  sparkSession: SparkSession,
  plan: SparkPlan,
  fileFormat: FileFormat,
  committer: FileCommitProtocol,
  outputSpec: OutputSpec,
  hadoopConf: Configuration,
  partitionColumns: Seq[Attribute],
  bucketSpec: Option[BucketSpec],
  statsTrackers: Seq[WriteJobStatsTracker],
  options: Map[String, String]): Set[String]
----

`write` creates a Hadoop https://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/mapreduce/Job.html[Job] instance (with the given Hadoop https://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/conf/Configuration.html[Configuration]) and uses the following job output classes:

* `Void` for keys

* `InternalRow` for values

`write` sets the output directory (for the map-reduce job) to be the `outputPath` of the given `OutputSpec`.

[[write-outputWriterFactory]]
`write` requests the given `FileFormat` to <<spark-sql-FileFormat.adoc#prepareWrite, prepareWrite>>.

[[write-description]]
`write` creates a `WriteJobDescription` with the following:

* `maxRecordsPerFile` based on the `maxRecordsPerFile` option (from the given options) if available or <<spark-sql-properties.adoc#spark.sql.files.maxRecordsPerFile, spark.sql.files.maxRecordsPerFile>>

* `timeZoneId` based on the `timeZone` option (from the given options) if available or <<spark-sql-properties.adoc#spark.sql.session.timeZone, spark.sql.session.timeZone>>

`write` requests the given `FileCommitProtocol` committer to `setupJob`.

[[write-rdd]]
`write` executes the given <<spark-sql-SparkPlan.adoc#, SparkPlan>> (and generates an RDD). The execution can be directly on the given physical operator if ordering matches the requirements or uses <<spark-sql-SparkPlan-SortExec.adoc#, SortExec>> physical operator (with `global` flag off).

[[write-runJob]]
`write` runs a Spark job (action) on the <<write-rdd, RDD>> with <<executeTask, executeTask>> as the partition function. The result task handler simply requests the given `FileCommitProtocol` committer to `onTaskCommit` (with the `TaskCommitMessage` of a `WriteTaskResult`) and saves the `WriteTaskResult`.

[[write-commitJob]]
`write` requests the given `FileCommitProtocol` committer to `commitJob` (with the Hadoop `Job` instance and the `TaskCommitMessage` of all write tasks).

`write` prints out the following INFO message to the logs:

```
Write Job [uuid] committed.
```

[[write-processStats]]
`write` <<processStats, processStats>>.

`write` prints out the following INFO message to the logs:

```
Finished processing stats for write job [uuid].
```

In the end, `write` returns all the partition paths that were updated during this write job.

[NOTE]
====
`write` is used when:

* <<spark-sql-LogicalPlan-InsertIntoHadoopFsRelationCommand.adoc#, InsertIntoHadoopFsRelationCommand>> logical command is executed

* `SaveAsHiveFile` is requested to <<hive/SaveAsHiveFile.adoc#saveAsHiveFile, saveAsHiveFile>> (when link:hive/InsertIntoHiveDirCommand.adoc[InsertIntoHiveDirCommand] and link:hive/InsertIntoHiveTable.adoc[InsertIntoHiveTable] logical commands are executed)

* Structured Streaming's `FileStreamSink` is requested to add a streaming batch (`addBatch`)
====

==== [[write-Throwable]] `write` Method And Throwables

In case of any `Throwable`, `write` prints out the following ERROR message to the logs:

```
Aborting job [uuid].
```

[[write-abortJob]]
`write` requests the given `FileCommitProtocol` committer to `abortJob` (with the Hadoop `Job` instance).

In the end, `write` throws a `SparkException`.

=== [[executeTask]] `executeTask` Internal Method

[source, scala]
----
executeTask(
  description: WriteJobDescription,
  sparkStageId: Int,
  sparkPartitionId: Int,
  sparkAttemptNumber: Int,
  committer: FileCommitProtocol,
  iterator: Iterator[InternalRow]): WriteTaskResult
----

`executeTask`...FIXME

NOTE: `executeTask` is used exclusively when `FileFormatWriter` is requested to <<write, write the result of a structured query>>.

=== [[processStats]] `processStats` Internal Method

[source, scala]
----
processStats(
  statsTrackers: Seq[WriteJobStatsTracker],
  statsPerTask: Seq[Seq[WriteTaskStats]]): Unit
----

`processStats`...FIXME

NOTE: `processStats` is used exclusively when `FileFormatWriter` is requested to <<write, write the result of a structured query>>.
