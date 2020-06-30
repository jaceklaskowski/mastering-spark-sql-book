# Caching and Persistence

One of the optimizations in Spark SQL is *Dataset caching* (aka *Dataset persistence*) which is available using the <<spark-sql-dataset-operators.adoc#, Dataset API>> using the following basic actions:

* [[cache]] <<spark-sql-dataset-operators.adoc#cache, cache>>

* [[persist]] <<spark-sql-dataset-operators.adoc#persist, persist>>

* [[unpersist]] <<spark-sql-dataset-operators.adoc#unpersist, unpersist>>

`cache` is simply `persist` with `MEMORY_AND_DISK` storage level.

[source, scala]
----
// Cache Dataset -- it is lazy and so nothing really happens
val data = spark.range(1).cache

// Trigger caching by executing an action
// The common idiom is to execute count since it's fairly cheap
data.count
----

At this point you could use web UI's *Storage* tab to review the Datasets persisted. Visit http://localhost:4040/storage.

.web UI's Storage tab
image::images/spark-webui-storage.png[align="center"]

`persist` uses <<spark-sql-CacheManager.adoc#, CacheManager>> for an in-memory cache of structured queries (and <<spark-sql-LogicalPlan-InMemoryRelation.adoc#, InMemoryRelation>> logical operators), and is used to <<spark-sql-CacheManager.adoc#cacheQuery, cache structured queries>> (which simply registers the structured queries as <<spark-sql-LogicalPlan-InMemoryRelation.adoc#, InMemoryRelation>> leaf logical operators).

At <<spark-sql-QueryExecution.adoc#withCachedData, withCachedData>> phase (of execution of a structured query), `QueryExecution` requests the `CacheManager` to <<spark-sql-CacheManager.adoc#useCachedData, replace segments of a logical query plan with their cached data>> (including <<spark-sql-subqueries.adoc#, subqueries>>).

```
scala> println(data.queryExecution.withCachedData.numberedTreeString)
00 InMemoryRelation [id#9L], StorageLevel(disk, memory, deserialized, 1 replicas)
01    +- *(1) Range (0, 1, step=1, splits=8)
```

```
// Use the cached Dataset in another query
// Notice InMemoryRelation in use for cached queries
scala> df.withColumn("newId", 'id).explain(extended = true)
== Parsed Logical Plan ==
'Project [*, 'id AS newId#16]
+- Range (0, 1, step=1, splits=Some(8))

== Analyzed Logical Plan ==
id: bigint, newId: bigint
Project [id#0L, id#0L AS newId#16L]
+- Range (0, 1, step=1, splits=Some(8))

== Optimized Logical Plan ==
Project [id#0L, id#0L AS newId#16L]
+- InMemoryRelation [id#0L], true, 10000, StorageLevel(disk, memory, deserialized, 1 replicas)
      +- *Range (0, 1, step=1, splits=Some(8))

== Physical Plan ==
*Project [id#0L, id#0L AS newId#16L]
+- InMemoryTableScan [id#0L]
      +- InMemoryRelation [id#0L], true, 10000, StorageLevel(disk, memory, deserialized, 1 replicas)
            +- *Range (0, 1, step=1, splits=Some(8))

// Clear in-memory cache using SQL
// Equivalent to spark.catalog.clearCache
scala> sql("CLEAR CACHE").collect
res1: Array[org.apache.spark.sql.Row] = Array()

// Visit http://localhost:4040/storage to confirm the cleaning
```

[NOTE]
====
You can also use SQL's `CACHE TABLE [tableName]` to cache `tableName` table in memory. Unlike <<cache, cache>> and <<spark-sql-dataset-operators.adoc#persist, persist>> operators, `CACHE TABLE` is an eager operation which is executed as soon as the statement is executed.

[source,scala]
----
sql("CACHE TABLE [tableName]")
----

You could however use `LAZY` keyword to make caching lazy.

[source,scala]
----
sql("CACHE LAZY TABLE [tableName]")
----

Use SQL's `REFRESH TABLE [tableName]` to refresh a cached table.

Use SQL's `UNCACHE TABLE (IF EXISTS)? [tableName]` to remove a table from cache.

Use SQL's `CLEAR CACHE` to remove all tables from cache.
====

[NOTE]
====
Be careful what you cache, i.e. what Dataset is cached, as it gives different queries cached.

[source, scala]
----
// cache after range(5)
val q1 = spark.range(5).cache.filter($"id" % 2 === 0).select("id")
scala> q1.explain
== Physical Plan ==
*Filter ((id#0L % 2) = 0)
+- InMemoryTableScan [id#0L], [((id#0L % 2) = 0)]
      +- InMemoryRelation [id#0L], true, 10000, StorageLevel(disk, memory, deserialized, 1 replicas)
            +- *Range (0, 5, step=1, splits=8)

// cache at the end
val q2 = spark.range(1).filter($"id" % 2 === 0).select("id").cache
scala> q2.explain
== Physical Plan ==
InMemoryTableScan [id#17L]
   +- InMemoryRelation [id#17L], true, 10000, StorageLevel(disk, memory, deserialized, 1 replicas)
         +- *Filter ((id#17L % 2) = 0)
            +- *Range (0, 1, step=1, splits=8)
----
====

[TIP]
====
You can check whether a Dataset was cached or not using the following code:

[source, scala]
----
scala> :type q2
org.apache.spark.sql.Dataset[org.apache.spark.sql.Row]

val cache = spark.sharedState.cacheManager
scala> cache.lookupCachedData(q2.queryExecution.logical).isDefined
res0: Boolean = false
----
====

=== [[cache-table]] SQL's CACHE TABLE

SQL's `CACHE TABLE` corresponds to requesting the session-specific `Catalog` to link:spark-sql-Catalog.adoc#cacheTable[caching the table].

Internally, `CACHE TABLE` becomes link:spark-sql-LogicalPlan-RunnableCommand.adoc#CacheTableCommand[CacheTableCommand] runnable command that...FIXME
