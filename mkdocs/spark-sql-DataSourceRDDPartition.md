# DataSourceRDDPartition

`DataSourceRDDPartition` is a Spark Core `Partition` of link:spark-sql-DataSourceRDD.adoc[DataSourceRDD] and Spark Structured Streaming's `ContinuousDataSourceRDD` RDDs.

`DataSourceRDDPartition` is <<creating-instance, created>> when:

* link:spark-sql-DataSourceRDD.adoc#getPartitions[DataSourceRDD] and Spark Structured Streaming's `ContinuousDataSourceRDD` are requested for partitions

* link:spark-sql-DataSourceRDD.adoc#compute[DataSourceRDD] and Spark Structured Streaming's `ContinuousDataSourceRDD` are requested to compute a partition

* link:spark-sql-DataSourceRDD.adoc#getPreferredLocations[DataSourceRDD] and Spark Structured Streaming's `ContinuousDataSourceRDD` are requested for preferred locations

[[creating-instance]]
`DataSourceRDDPartition` takes the following when created:

* [[index]] Partition index
* [[inputPartition]] <<spark-sql-InputPartition.adoc#, InputPartition>>
