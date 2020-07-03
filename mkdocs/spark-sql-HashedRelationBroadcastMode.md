# HashedRelationBroadcastMode

`HashedRelationBroadcastMode` is a link:spark-sql-BroadcastMode.adoc[BroadcastMode] that `BroadcastHashJoinExec` uses for the link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc#requiredChildDistribution[required output distribution of child operators].

[[creating-instance]]
[[key]]
`HashedRelationBroadcastMode` takes build-side join keys (as link:spark-sql-Expression.adoc[Catalyst expressions]) when created.

[[canonicalized]]
`HashedRelationBroadcastMode` gives a copy of itself with <<key, keys>> canonicalized when requested for a link:spark-sql-BroadcastMode.adoc#canonicalized[canonicalized] version.

=== [[transform]] `transform` Method

[source, scala]
----
transform(rows: Array[InternalRow]): HashedRelation // <1>
transform(
  rows: Iterator[InternalRow],
  sizeHint: Option[Long]): HashedRelation
----
<1> Uses the other `transform` with the size of `rows` as `sizeHint`

NOTE: `transform` is part of link:spark-sql-BroadcastMode.adoc#transform[BroadcastMode Contract] to...FIXME.

`transform`...FIXME
