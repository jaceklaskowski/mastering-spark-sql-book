# BasicWriteJobStatsTracker

`BasicWriteJobStatsTracker` is a concrete <<spark-sql-WriteJobStatsTracker.adoc#, WriteJobStatsTracker>>.

`BasicWriteJobStatsTracker` is <<creating-instance, created>> when <<spark-sql-LogicalPlan-DataWritingCommand.adoc#basicWriteJobStatsTracker, DataWritingCommand>> and Spark Structured Streaming's `FileStreamSink` are requested for one.

[[newTaskInstance]]
When requested for a new <<spark-sql-WriteJobStatsTracker.adoc#newTaskInstance, WriteTaskStatsTracker>>, `BasicWriteJobStatsTracker` creates a new <<spark-sql-BasicWriteTaskStatsTracker.adoc#, BasicWriteTaskStatsTracker>>.

=== [[creating-instance]] Creating BasicWriteJobStatsTracker Instance

`BasicWriteJobStatsTracker` takes the following when created:

* [[serializableHadoopConf]] Serializable Hadoop `Configuration`
* [[metrics]] Metrics (`Map[String, SQLMetric]`)
