# BasicWriteTaskStatsTracker

`BasicWriteTaskStatsTracker` is a concrete <<spark-sql-WriteTaskStatsTracker.adoc#, WriteTaskStatsTracker>>.

`BasicWriteTaskStatsTracker` is <<creating-instance, created>> exclusively when `BasicWriteJobStatsTracker` is requested for <<spark-sql-BasicWriteJobStatsTracker.adoc#newTaskInstance, one>>.

[[creating-instance]]
[[hadoopConf]]
`BasicWriteTaskStatsTracker` takes a Hadoop `Configuration` when created.

=== [[getFinalStats]] Getting Final WriteTaskStats -- `getFinalStats` Method

[source, scala]
----
getFinalStats(): WriteTaskStats
----

NOTE: `getFinalStats` is part of the <<spark-sql-WriteTaskStatsTracker.adoc#getFinalStats, WriteTaskStatsTracker Contract>> to get the final <<spark-sql-WriteTaskStats.adoc#, WriteTaskStats>> statistics computed so far.

`getFinalStats`...FIXME
