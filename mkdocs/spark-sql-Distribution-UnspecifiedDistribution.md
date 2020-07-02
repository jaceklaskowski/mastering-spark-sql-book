# UnspecifiedDistribution

`UnspecifiedDistribution` is a link:spark-sql-Distribution.adoc[Distribution] that...FIXME

[[requiredNumPartitions]]
`UnspecifiedDistribution` specifies `None` for the link:spark-sql-Distribution.adoc#requiredNumPartitions[required number of partitions].

NOTE: `None` for the required number of partitions indicates to use any number of partitions (possibly link:spark-sql-properties.adoc#spark.sql.shuffle.partitions[spark.sql.shuffle.partitions] configuration property with the default of `200` partitions).

=== [[createPartitioning]] `createPartitioning` Method

[source, scala]
----
createPartitioning(numPartitions: Int): Partitioning
----

NOTE: `createPartitioning` is part of link:spark-sql-Distribution.adoc#createPartitioning[Distribution Contract] to create a link:spark-sql-SparkPlan-Partitioning.adoc[Partitioning] for a given number of partitions.

`createPartitioning`...FIXME
