# HintInfo

[[creating-instance]]
`HintInfo` takes a single <<broadcast, broadcast>> flag when created.

`HintInfo` is <<creating-instance, created>> when:

. link:spark-sql-functions.adoc#broadcast[Dataset.broadcast] function is used

. `ResolveBroadcastHints` logical resolution rule is link:spark-sql-Analyzer-ResolveBroadcastHints.adoc#apply[executed] (and resolves link:spark-sql-LogicalPlan-UnresolvedHint.adoc[UnresolvedHint] logical operators)

. link:spark-sql-LogicalPlan-ResolvedHint.adoc#creating-instance[ResolvedHint] and link:spark-sql-Statistics.adoc#creating-instance[Statistics] are created

. `InMemoryRelation` is requested for link:spark-sql-LogicalPlan-InMemoryRelation.adoc#computeStats[computeStats] (when link:spark-sql-LogicalPlan-InMemoryRelation.adoc#sizeInBytesStats[sizeInBytesStats] is `0`)

. `HintInfo` is requested to <<resetForJoin, resetForJoin>>

[[broadcast]]
`broadcast` is used to...FIXME

`broadcast` is off (i.e. `false`) by default.

[source, scala]
----
import org.apache.spark.sql.catalyst.plans.logical.HintInfo
val broadcastOff = HintInfo()

scala> println(broadcastOff.broadcast)
false

val broadcastOn = broadcastOff.copy(broadcast = true)
scala> println(broadcastOn)
(broadcast)

val broadcastOff = broadcastOn.resetForJoin
scala> println(broadcastOff.broadcast)
false
----

=== [[resetForJoin]] `resetForJoin` Method

[source, scala]
----
resetForJoin(): HintInfo
----

`resetForJoin`...FIXME

NOTE: `resetForJoin` is used when `SizeInBytesOnlyStatsPlanVisitor` is requested to link:spark-sql-SizeInBytesOnlyStatsPlanVisitor.adoc#visitIntersect[visitIntersect] and link:spark-sql-SizeInBytesOnlyStatsPlanVisitor.adoc#visitJoin[visitJoin].
