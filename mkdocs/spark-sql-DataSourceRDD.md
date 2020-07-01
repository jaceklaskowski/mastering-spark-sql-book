title: DataSourceRDD

# DataSourceRDD -- Input RDD Of DataSourceV2ScanExec Physical Operator

`DataSourceRDD` acts as a thin adapter between Spark SQL's <<spark-sql-data-source-api-v2.adoc#, Data Source API V2>> and Spark Core's RDD API.

`DataSourceRDD` is an `RDD` that is <<creating-instance, created>> exclusively when `DataSourceV2ScanExec` physical operator is requested for the link:spark-sql-SparkPlan-DataSourceV2ScanExec.adoc#inputRDD[input RDD] (when `WholeStageCodegenExec` physical operator is link:spark-sql-SparkPlan-WholeStageCodegenExec.adoc#doExecute[executed]).

`DataSourceRDD` uses <<spark-sql-DataSourceRDDPartition.adoc#, DataSourceRDDPartition>> when requested for the <<getPartitions, partitions>> (that is a mere wrapper of the <<inputPartitions, InputPartitions>>).

[[creating-instance]]
`DataSourceRDD` takes the following to be created:

* [[sc]] Spark Core's `SparkContext`
* [[inputPartitions]] <<spark-sql-InputPartition.adoc#, InputPartitions>> (`Seq[InputPartition[T]]`)

`DataSourceRDD` as a Spark Core `RDD` overrides the following methods:

* <<getPartitions, getPartitions>>

* <<compute, compute>>

* <<getPreferredLocations, getPreferredLocations>>

=== [[getPreferredLocations]] Requesting Preferred Locations (For Partition) -- `getPreferredLocations` Method

[source, scala]
----
getPreferredLocations(split: Partition): Seq[String]
----

NOTE: `getPreferredLocations` is part of Spark Core's `RDD` Contract to...FIXME.

`getPreferredLocations` simply requests the given `split` <<spark-sql-DataSourceRDDPartition.adoc#, DataSourceRDDPartition>> for the <<spark-sql-DataSourceRDDPartition.adoc#inputPartition, InputPartition>> that in turn is requested for the <<spark-sql-InputPartition.adoc#preferredLocations, preferred locations>>.

=== [[getPartitions]] RDD Partitions -- `getPartitions` Method

[source, scala]
----
getPartitions: Array[Partition]
----

NOTE: `getPartitions` is part of Spark Core's `RDD` Contract to...FIXME

`getPartitions` simply creates a <<spark-sql-DataSourceRDDPartition.adoc#, DataSourceRDDPartition>> for every <<inputPartitions, InputPartition>>.

=== [[compute]] Computing Partition (in TaskContext) -- `compute` Method

[source, scala]
----
compute(split: Partition, context: TaskContext): Iterator[T]
----

NOTE: `compute` is part of Spark Core's `RDD` Contract to compute a partition (in a `TaskContext`).

`compute` requests the input <<spark-sql-DataSourceRDDPartition.adoc#, DataSourceRDDPartition>> (the `split` partition) for the <<spark-sql-DataSourceRDDPartition.adoc#inputPartition, InputPartition>> that in turn is requested to <<spark-sql-InputPartition.adoc#createPartitionReader, create an InputPartitionReader>>.

`compute` registers a Spark Core `TaskCompletionListener` that requests the `InputPartitionReader` to close when a task completes.

`compute` returns a Spark Core `InterruptibleIterator` that requests the `InputPartitionReader` to <<spark-sql-InputPartitionReader.adoc#next, proceed to the next record>> (when requested to `hasNext`) and <<spark-sql-InputPartitionReader.adoc#get, return the current record>> (when `next`).
