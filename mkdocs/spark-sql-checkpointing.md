# Checkpointing

*Dataset Checkpointing* is a feature of Spark SQL to truncate a logical query plan that could specifically be useful for highly iterative data algorithms (e.g. Spark MLlib that uses Spark SQL's `Dataset` API for data manipulation).

[NOTE]
====
Checkpointing is actually a feature of Spark Core (that Spark SQL uses for distributed computations) that allows a driver to be restarted on failure with previously computed state of a distributed computation described as an `RDD`. That has been successfully used in Spark Streaming - the now-obsolete Spark module for stream processing based on RDD API.

Checkpointing truncates the lineage of a RDD to be checkpointed. That has been successfully used in Spark MLlib in iterative machine learning algorithms like ALS.

Dataset checkpointing in Spark SQL uses checkpointing to truncate the lineage of the underlying RDD of a `Dataset` being checkpointed.
====

Checkpointing can be eager or lazy per `eager` flag of <<spark-sql-Dataset-untyped-transformations.adoc#checkpoint, checkpoint>> operator. *Eager checkpointing* is the default checkpointing and happens immediately when requested. *Lazy checkpointing* does not and will only happen when an action is executed.

[[checkpoint-directory]]
Using Dataset checkpointing requires that you specify the *checkpoint directory*. The directory stores the checkpoint files for RDDs to be checkpointed. Use <<sparkcontext-setCheckpointDir, SparkContext.setCheckpointDir>> to set the path to a checkpoint directory.

Checkpointing can be <<spark-sql-Dataset-untyped-transformations.adoc#localCheckpoint, local>> or <<spark-sql-Dataset-untyped-transformations.adoc#checkpoint, reliable>> which defines how reliable the <<checkpoint-directory, checkpoint directory>> is. *Local checkpointing* uses executor storage to write checkpoint files to and due to the executor lifecycle is considered unreliable. *Reliable checkpointing* uses a reliable data storage like Hadoop HDFS.

.Dataset Checkpointing Types
[cols="1,^1,^2",options="header",width="100%"]
|===
|
| Eager
| Lazy

^| *Reliable*
| <<spark-sql-Dataset-untyped-transformations.adoc#checkpoint, checkpoint>>
| <<spark-sql-Dataset-untyped-transformations.adoc#checkpoint, checkpoint(eager = false)>>

^| *Local*
| <<spark-sql-Dataset-untyped-transformations.adoc#localCheckpoint, localCheckpoint>>
| <<spark-sql-Dataset-untyped-transformations.adoc#localCheckpoint, localCheckpoint(eager = false)>>
|===

A RDD can be recovered from a checkpoint files using <<sparkcontext-checkpointFile, SparkContext.checkpointFile>>. You can use link:spark-sql-SparkSession.adoc#internalCreateDataFrame[SparkSession.internalCreateDataFrame] method to (re)create the DataFrame from the RDD of internal binary rows.

[[logging]]
[TIP]
====
Enable `INFO` logging level for `org.apache.spark.rdd.ReliableRDDCheckpointData` logger to see what happens while an RDD is checkpointed.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.rdd.ReliableRDDCheckpointData=INFO
```

Refer to link:spark-logging.adoc[Logging].
====

```
import org.apache.spark.sql.functions.rand
val nums = spark.range(5).withColumn("random", rand()).filter($"random" > 0.5)
scala> nums.show
+---+------------------+
| id|            random|
+---+------------------+
|  0| 0.752877642067488|
|  1|0.5271005540026181|
+---+------------------+

scala> println(nums.queryExecution.toRdd.toDebugString)
(8) MapPartitionsRDD[7] at toRdd at <console>:27 []
 |  MapPartitionsRDD[6] at toRdd at <console>:27 []
 |  ParallelCollectionRDD[5] at toRdd at <console>:27 []

// Remember to set the checkpoint directory
scala> nums.checkpoint
org.apache.spark.SparkException: Checkpoint directory has not been set in the SparkContext
  at org.apache.spark.rdd.RDD.checkpoint(RDD.scala:1548)
  at org.apache.spark.sql.Dataset.checkpoint(Dataset.scala:594)
  at org.apache.spark.sql.Dataset.checkpoint(Dataset.scala:539)
  ... 49 elided

spark.sparkContext.setCheckpointDir("/tmp/checkpoints")

val checkpointDir = spark.sparkContext.getCheckpointDir.get
scala> println(checkpointDir)
file:/tmp/checkpoints/b1f413dc-3eaf-46a0-99de-d795252035e0

val numsCheckpointed = nums.checkpoint
scala> println(numsCheckpointed.queryExecution.toRdd.toDebugString)
(8) MapPartitionsRDD[11] at toRdd at <console>:27 []
 |  MapPartitionsRDD[9] at checkpoint at <console>:26 []
 |  ReliableCheckpointRDD[10] at checkpoint at <console>:26 []

// Set org.apache.spark.rdd.ReliableRDDCheckpointData logger to INFO
// to see what happens while an RDD is checkpointed
// Let's use log4j API
import org.apache.log4j.{Level, Logger}
Logger.getLogger("org.apache.spark.rdd.ReliableRDDCheckpointData").setLevel(Level.INFO)

scala> nums.checkpoint
18/03/23 00:05:15 INFO ReliableRDDCheckpointData: Done checkpointing RDD 12 to file:/tmp/checkpoints/b1f413dc-3eaf-46a0-99de-d795252035e0/rdd-12, new parent is RDD 13
res7: org.apache.spark.sql.Dataset[org.apache.spark.sql.Row] = [id: bigint, random: double]

// Save the schema as it is going to use to reconstruct nums dataset from a RDD
val schema = nums.schema

// Recover nums dataset from the checkpoint files
// Start from recovering the underlying RDD
// And create a Dataset based on the RDD

// Get the path to the checkpoint files of the checkpointed RDD of the Dataset
import org.apache.spark.sql.execution.LogicalRDD
val logicalRDD = numsCheckpointed.queryExecution.optimizedPlan.asInstanceOf[LogicalRDD]
val checkpointFiles = logicalRDD.rdd.getCheckpointFile.get
scala> println(checkpointFiles)
file:/tmp/checkpoints/b1f413dc-3eaf-46a0-99de-d795252035e0/rdd-9

// SparkContext.checkpointFile is a `protected[spark]` method
// Use :paste -raw mode in Spark shell and define a helper object to "escape" the package lock-in
scala> :paste -raw
// Entering paste mode (ctrl-D to finish)

package org.apache.spark
object my {
  import scala.reflect.ClassTag
  import org.apache.spark.rdd.RDD
  def recover[T: ClassTag](sc: SparkContext, path: String): RDD[T] = {
    sc.checkpointFile[T](path)
  }
}

// Exiting paste mode, now interpreting.

// Make sure to use the same checkpoint directory

import org.apache.spark.my
import org.apache.spark.sql.catalyst.InternalRow
val numsRddRecovered = my.recover[InternalRow](spark.sparkContext, checkpointFiles)
scala> :type numsRddRecovered
org.apache.spark.rdd.RDD[org.apache.spark.sql.catalyst.InternalRow]

// We have to convert RDD[InternalRow] to DataFrame

// Use :paste -raw again as we use `private[sql]` method
scala> :pa -raw
// Entering paste mode (ctrl-D to finish)

package org.apache.spark.sql
object my2 {
  import org.apache.spark.rdd.RDD
  import org.apache.spark.sql.{DataFrame, SparkSession}
  import org.apache.spark.sql.catalyst.InternalRow
  import org.apache.spark.sql.types.StructType
  def createDataFrame(spark: SparkSession, catalystRows: RDD[InternalRow], schema: StructType): DataFrame = {
    spark.internalCreateDataFrame(catalystRows, schema)
  }
}

// Exiting paste mode, now interpreting.

import org.apache.spark.sql.my2
val numsRecovered = my2.createDataFrame(spark, numsRddRecovered, schema)
scala> numsRecovered.show
+---+------------------+
| id|            random|
+---+------------------+
|  0| 0.752877642067488|
|  1|0.5271005540026181|
+---+------------------+
```

=== [[sparkcontext-setCheckpointDir]] Specifying Checkpoint Directory -- `SparkContext.setCheckpointDir` Method

[source, scala]
----
SparkContext.setCheckpointDir(directory: String)
----

`setCheckpointDir` sets the <<checkpoint-directory, checkpoint directory>>.

Internally, `setCheckpointDir`...FIXME

=== [[sparkcontext-checkpointFile]] Recovering RDD From Checkpoint Files -- `SparkContext.checkpointFile` Method

[source, scala]
----
SparkContext.checkpointFile(directory: String)
----

`checkpointFile` reads (_recovers_) a RDD from a checkpoint directory.

NOTE: `SparkContext.checkpointFile` is a `protected[spark]` method so the code to access it has to be in `org.apache.spark` package.

Internally, `checkpointFile` creates a `ReliableCheckpointRDD` in a scope.
