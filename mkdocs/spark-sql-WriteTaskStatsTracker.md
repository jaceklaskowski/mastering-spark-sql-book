# WriteTaskStatsTracker

`WriteTaskStatsTracker` is the <<contract, abstraction>> of <<implementations, WriteTaskStatsTrackers>> that collect the statistics of the number of <<newBucket, buckets>>, <<newFile, files>>, <<newPartition, partitions>> and <<newRow, rows>> processed.

[[contract]]
.WriteTaskStatsTracker Contract
[cols="1m,2",options="header",width="100%"]
|===
| Method
| Description

| getFinalStats
a| [[getFinalStats]]

[source, scala]
----
getFinalStats(): WriteTaskStats
----

The final <<spark-sql-WriteTaskStats.adoc#, WriteTaskStats>> statistics computed so far

Used when `EmptyDirectoryWriteTask`, `SingleDirectoryWriteTask` and `DynamicPartitionWriteTask` are requested to execute

| newBucket
a| [[newBucket]]

[source, scala]
----
newBucket(bucketId: Int): Unit
----

Used when...FIXME

| newFile
a| [[newFile]]

[source, scala]
----
newFile(filePath: String): Unit
----

Used when...FIXME

| newPartition
a| [[newPartition]]

[source, scala]
----
newPartition(partitionValues: InternalRow): Unit
----

Used when...FIXME

| newRow
a| [[newRow]]

[source, scala]
----
newRow(row: InternalRow): Unit
----

Used when...FIXME
|===

[[implementations]]
NOTE: <<spark-sql-BasicWriteTaskStatsTracker.adoc#, BasicWriteTaskStatsTracker>> is the one and only known implementation of the <<contract, WriteTaskStatsTracker Contract>> in Apache Spark.
