title: SizeInBytesOnlyStatsPlanVisitor

# SizeInBytesOnlyStatsPlanVisitor -- LogicalPlanVisitor for Total Size (in Bytes) Statistic Only

`SizeInBytesOnlyStatsPlanVisitor` is a link:spark-sql-LogicalPlanVisitor.adoc[LogicalPlanVisitor] that computes a single dimension for link:spark-sql-Statistics.adoc[plan statistics], i.e. the total size (in bytes).

=== [[default]] `default` Method

[source, scala]
----
default(p: LogicalPlan): Statistics
----

NOTE: `default` is part of link:spark-sql-LogicalPlanVisitor.adoc#default[LogicalPlanVisitor Contract] to compute the size statistic (in bytes) of a logical operator.

`default` requests a link:spark-sql-LogicalPlan-LeafNode.adoc[leaf logical operator] for the statistics or creates a link:spark-sql-Statistics.adoc[Statistics] with the product of the `sizeInBytes` statistic of every link:spark-sql-catalyst-TreeNode.adoc#children[child operator].

NOTE: `default` uses the cache of the estimated statistics of a logical operator so the statistics of an operator is link:spark-sql-LogicalPlanStats.adoc#stats[computed] once until it is link:spark-sql-LogicalPlanStats.adoc#invalidateStatsCache[invalidated].

=== [[visitIntersect]] `visitIntersect` Method

[source, scala]
----
visitIntersect(p: Intersect): Statistics
----

NOTE: `visitIntersect` is part of link:spark-sql-LogicalPlanVisitor.adoc#visitIntersect[LogicalPlanVisitor Contract] to...FIXME.

`visitIntersect`...FIXME

=== [[visitJoin]] `visitJoin` Method

[source, scala]
----
visitJoin(p: Join): Statistics
----

NOTE: `visitJoin` is part of link:spark-sql-LogicalPlanVisitor.adoc#visitJoin[LogicalPlanVisitor Contract] to...FIXME.

`visitJoin`...FIXME
