title: FileScanRDD

# FileScanRDD -- Input RDD of FileSourceScanExec Physical Operator

`FileScanRDD` is an `RDD` of link:spark-sql-InternalRow.adoc[internal binary rows] (i.e. `RDD[InternalRow]`) that is the input RDD of a link:spark-sql-SparkPlan-FileSourceScanExec.adoc[FileSourceScanExec] physical operator (in <<spark-sql-whole-stage-codegen.adoc#, Whole-Stage Java Code Generation>>).

`FileScanRDD` is <<creating-instance, created>> exclusively when `FileSourceScanExec` physical operator is requested to link:spark-sql-SparkPlan-FileSourceScanExec.adoc#createBucketedReadRDD[createBucketedReadRDD] or link:spark-sql-SparkPlan-FileSourceScanExec.adoc#createNonBucketedReadRDD[createNonBucketedReadRDD] (when `FileSourceScanExec` operator is requested for the link:spark-sql-SparkPlan-FileSourceScanExec.adoc#inputRDD[input RDD] when `WholeStageCodegenExec` physical operator is link:spark-sql-SparkPlan-WholeStageCodegenExec.adoc#doExecute[executed]).

[source, scala]
----
val q = spark.read.text("README.md")

val sparkPlan = q.queryExecution.executedPlan
import org.apache.spark.sql.execution.FileSourceScanExec
val scan = sparkPlan.collectFirst { case exec: FileSourceScanExec => exec }.get
val inputRDD = scan.inputRDDs.head

import org.apache.spark.sql.execution.datasources.FileScanRDD
assert(inputRDD.isInstanceOf[FileScanRDD])

val rdd = scan.execute
scala> println(rdd.toDebugString)
(1) MapPartitionsRDD[1] at execute at <console>:27 []
 |  FileScanRDD[0] at inputRDDs at <console>:26 []

val fileScanRDD = rdd.dependencies.head.rdd
assert(fileScanRDD.isInstanceOf[FileScanRDD])
----

[[FilePartition]]
[[files]]
[[index]]
When <<creating-instance, created>>, `FileScanRDD` is given <<filePartitions, FilePartitions>> that are custom RDD partitions with <<spark-sql-PartitionedFile.adoc#, PartitionedFiles>> (_file blocks_).

`FileScanRDD` uses the following properties when requested to <<compute, compute a partition>>:

* [[ignoreCorruptFiles]] <<spark-sql-properties.adoc#spark.sql.files.ignoreCorruptFiles, spark.sql.files.ignoreCorruptFiles>>

* [[ignoreMissingFiles]] <<spark-sql-properties.adoc#spark.sql.files.ignoreMissingFiles, spark.sql.files.ignoreMissingFiles>>

[[creating-instance]]
`FileScanRDD` takes the following to be created:

* [[sparkSession]] link:spark-sql-SparkSession.adoc[SparkSession]
* [[readFunction]] Read function that takes a link:spark-sql-PartitionedFile.adoc[PartitionedFile] and gives link:spark-sql-InternalRow.adoc[internal rows] back (`(PartitionedFile) => Iterator[InternalRow]`)
* [[filePartitions]] <<FilePartition, FilePartitions>> (_file blocks_)

[[logging]]
[TIP]
====
Enable `ALL` logging level for `org.apache.spark.sql.execution.datasources.FileScanRDD` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.execution.datasources.FileScanRDD=ALL
```

Refer to <<spark-logging.adoc#, Logging>>.
====

=== [[getPreferredLocations]] Placement Preferences of Partition (Preferred Locations) -- `getPreferredLocations` Method

[source, scala]
----
getPreferredLocations(split: RDDPartition): Seq[String]
----

[NOTE]
====
`getPreferredLocations` is part of the RDD Contract to specify *placement preferences* (aka _preferred locations_) of a partition.

Find out more in https://jaceklaskowski.gitbooks.io/mastering-apache-spark/spark-rdd.html#getPreferredLocations[The Internals of Apache Spark].
====

`getPreferredLocations` requests the given <<FilePartition, FilePartition>> (`split`) for <<spark-sql-PartitionedFile.adoc#, PartitionedFiles>>.

For every `PartitionedFile`, `getPreferredLocations` adds the size of the file(s) to the host (location) it is available at.

In the end, `getPreferredLocations` gives the top 3 hosts with the most data available (file blocks).

=== [[getPartitions]] RDD Partitions -- `getPartitions` Method

[source, scala]
----
getPartitions: Array[RDDPartition]
----

[NOTE]
====
`getPartitions` is part of the RDD Contract to specify the partitions of a distributed computation.

Find out more in https://jaceklaskowski.gitbooks.io/mastering-apache-spark/spark-rdd.html#getPartitions[The Internals of Apache Spark].
====

`getPartitions` simply returns the <<filePartitions, FilePartitions>> (the `FileScanRDD` was created with).

=== [[compute]] Computing Partition (in TaskContext) -- `compute` Method

[source, scala]
----
compute(split: RDDPartition, context: TaskContext): Iterator[InternalRow]
----

NOTE: `compute` is part of Spark Core's `RDD` Contract to compute a partition (in a `TaskContext`).

`compute` creates a Scala https://www.scala-lang.org/api/2.11.12/#scala.collection.Iterator[Iterator] (of Java `Objects`) that...FIXME

NOTE: The given `RDDPartition` is actually a <<FilePartition, FilePartition>> with one or more <<spark-sql-PartitionedFile.adoc#, PartitionedFiles>> (_file blocks_).

`compute` then requests the input `TaskContext` to register a completion listener to be executed when a task completes (i.e. `addTaskCompletionListener`) that simply closes the iterator.

In the end, `compute` returns the iterator.

==== [[compute-next]] Getting Next Element -- `next` Method

[source, scala]
----
next(): Object
----

NOTE: `next` is part of the <<https://www.scala-lang.org/api/2.12.x/scala/collection/Iterator.html#next, Iterator Contract>> to produce the next element of this iterator.

`next` takes the next element of the current iterator over elements of a file block (<<spark-sql-PartitionedFile.adoc#, PartitionedFile>>).

`next` increments the metrics of bytes and number of rows read (that could be the number of rows in a <<spark-sql-ColumnarBatch.adoc#, ColumnarBatch>> for vectorized reads).

==== [[compute-nextIterator]] Getting Next Iterator -- `nextIterator` Internal Method

[source, scala]
----
nextIterator(): Boolean
----

`nextIterator`...FIXME

==== [[compute-readCurrentFile]] Getting Iterator (of Elements) of Current File Block -- `readCurrentFile` Internal Method

[source, scala]
----
readCurrentFile(): Iterator[InternalRow]
----

`readCurrentFile`...FIXME
