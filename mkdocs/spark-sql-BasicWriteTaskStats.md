# BasicWriteTaskStats

[[creating-instance]]
`BasicWriteTaskStats` is a basic <<spark-sql-WriteTaskStats.adoc#, WriteTaskStats>> that carries the following statistics:

* [[numPartitions]] `numPartitions`
* [[numFiles]] `numFiles`
* [[numBytes]] `numBytes`
* [[numRows]] `numRows`

`BasicWriteTaskStats` is <<creating-instance, created>> exclusively when `BasicWriteTaskStatsTracker` is requested for <<spark-sql-BasicWriteTaskStatsTracker.adoc#getFinalStats, getFinalStats>>.
