title: CacheManager

# CacheManager -- In-Memory Cache for Tables and Views

`CacheManager` is an in-memory cache (_registry_) for structured queries (by their link:spark-sql-LogicalPlan.adoc[logical plans]).

`CacheManager` is shared across `SparkSessions` through link:spark-sql-SparkSession.adoc#sharedState[SharedState].

[source, scala]
----
val spark: SparkSession = ...
spark.sharedState.cacheManager
----

NOTE: A Spark developer can use `CacheManager` to cache ``Dataset``s using link:spark-sql-caching-and-persistence.adoc#cache[cache] or link:spark-sql-caching-and-persistence.adoc#persist[persist] operators.

`CacheManager` uses the <<cachedData, cachedData>> internal registry to manage cached structured queries and their link:spark-sql-LogicalPlan-InMemoryRelation.adoc[InMemoryRelation] cached representation.

`CacheManager` <<isEmpty, can be empty>>.

[[CachedData]]
[[plan]]
[[cachedRepresentation]]
`CacheManager` uses `CachedData` data structure for managing <<cachedData, cached structured queries>> with the <<spark-sql-LogicalPlan.adoc#, LogicalPlan>> (of a structured query) and a corresponding <<spark-sql-LogicalPlan-InMemoryRelation.adoc#, InMemoryRelation>> leaf logical operator.

[[logging]]
[TIP]
====
Enable `ALL` logging level for `org.apache.spark.sql.execution.CacheManager` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.execution.CacheManager=ALL
```

Refer to <<spark-logging.adoc#, Logging>>.
====

=== [[cachedData]] Cached Structured Queries -- `cachedData` Internal Registry

[source, scala]
----
cachedData: LinkedList[CachedData]
----

`cachedData` is a collection of <<CachedData, CachedData>>.

A new `CachedData` added when `CacheManager` is requested to:

* <<cacheQuery, cacheQuery>>

* <<recacheByCondition, recacheByCondition>>

A `CachedData` removed when `CacheManager` is requested to:

* <<uncacheQuery, uncacheQuery>>

* <<recacheByCondition, recacheByCondition>>

All `CachedData` removed (cleared) when `CacheManager` is requested to <<clearCache, clearCache>>

=== [[lookupCachedData]] `lookupCachedData` Method

[source, scala]
----
lookupCachedData(query: Dataset[_]): Option[CachedData]
lookupCachedData(plan: LogicalPlan): Option[CachedData]
----

`lookupCachedData`...FIXME

[NOTE]
====
`lookupCachedData` is used when:

* <<spark-sql-dataset-operators.adoc#storageLevel, Dataset.storageLevel>> basic action is used

* `CatalogImpl` is requested to <<spark-sql-CatalogImpl.adoc#isCached, isCached>>

* `CacheManager` is requested to <<cacheQuery, cacheQuery>> and <<useCachedData, useCachedData>>
====

=== [[uncacheQuery]] Un-caching Dataset -- `uncacheQuery` Method

[source, scala]
----
uncacheQuery(
  query: Dataset[_],
  cascade: Boolean,
  blocking: Boolean = true): Unit
uncacheQuery(
  spark: SparkSession,
  plan: LogicalPlan,
  cascade: Boolean,
  blocking: Boolean): Unit
----

`uncacheQuery`...FIXME

[NOTE]
====
`uncacheQuery` is used when:

* <<spark-sql-dataset-operators.adoc#unpersist, Dataset.unpersist>> basic action is used

* <<spark-sql-LogicalPlan-DropTableCommand.adoc#, DropTableCommand>> and `TruncateTableCommand` logical commands are executed

* `CatalogImpl` is requested to <<spark-sql-CatalogImpl.adoc#uncacheTable, uncache>> and <<spark-sql-CatalogImpl.adoc#refreshTable, refresh>> a table or view, <<spark-sql-CatalogImpl.adoc#dropTempView, dropTempView>> and <<spark-sql-CatalogImpl.adoc#dropGlobalTempView, dropGlobalTempView>>
====

=== [[isEmpty]] `isEmpty` Method

[source, scala]
----
isEmpty: Boolean
----

`isEmpty` simply says whether there are any <<CachedData, CachedData>> entries in the <<cachedData, cachedData>> internal registry.

=== [[cacheQuery]] Caching Dataset -- `cacheQuery` Method

[source, scala]
----
cacheQuery(
  query: Dataset[_],
  tableName: Option[String] = None,
  storageLevel: StorageLevel = MEMORY_AND_DISK): Unit
----

`cacheQuery` adds the link:spark-sql-Dataset.adoc#logicalPlan[analyzed logical plan] of the input <<spark-sql-Dataset.adoc#, Dataset>> to the <<cachedData, cachedData>> internal registry of cached queries.

Internally, `cacheQuery` requests the `Dataset` for the link:spark-sql-Dataset.adoc#logicalPlan[analyzed logical plan] and creates a link:spark-sql-LogicalPlan-InMemoryRelation.adoc#apply[InMemoryRelation] with the following properties:

* link:spark-sql-properties.adoc#spark.sql.inMemoryColumnarStorage.compressed[spark.sql.inMemoryColumnarStorage.compressed] (enabled by default)

* link:spark-sql-properties.adoc#spark.sql.inMemoryColumnarStorage.batchSize[spark.sql.inMemoryColumnarStorage.batchSize] (default: `10000`)

* Input `storageLevel` storage level (default: `MEMORY_AND_DISK`)

* link:spark-sql-QueryExecution.adoc#executedPlan[Optimized physical query plan] (after requesting `SessionState` to link:spark-sql-SessionState.adoc#executePlan[execute] the analyzed logical plan)

* Input `tableName`

* link:spark-sql-LogicalPlanStats.adoc#stats[Statistics] of the analyzed query plan

`cacheQuery` then creates a <<CachedData, CachedData>> (for the analyzed query plan and the `InMemoryRelation`) and adds it to the <<cachedData, cachedData>> internal registry.

If the input `query` <<lookupCachedData, has already been cached>>, `cacheQuery` simply prints the following WARN message to the logs and exits (i.e. does nothing but prints out the WARN message):

```
Asked to cache already cached data.
```

[NOTE]
====
`cacheQuery` is used when:

* <<spark-sql-dataset-operators.adoc#persist, Dataset.persist>> basic action is used

* `CatalogImpl` is requested to <<spark-sql-CatalogImpl.adoc#cacheTable, cache>> and <<spark-sql-CatalogImpl.adoc#refreshTable, refresh>> a table or view in-memory
====

=== [[clearCache]] Removing All Cached Logical Plans -- `clearCache` Method

[source, scala]
----
clearCache(): Unit
----

`clearCache` takes every `CachedData` from the <<cachedData, cachedData>> internal registry and requests it for the <<cachedRepresentation, InMemoryRelation>> to access the <<spark-sql-LogicalPlan-InMemoryRelation.adoc#cacheBuilder, CachedRDDBuilder>>. `clearCache` requests the `CachedRDDBuilder` to <<spark-sql-CachedRDDBuilder.adoc#clearCache, clearCache>>.

In the end, `clearCache` removes all `CachedData` entries from the <<cachedData, cachedData>> internal registry.

NOTE: `clearCache` is used exclusively when `CatalogImpl` is requested to <<spark-sql-CatalogImpl.adoc#clearCache, clear the cache>>.

=== [[recacheByCondition]] Re-Caching Structured Query -- `recacheByCondition` Internal Method

[source, scala]
----
recacheByCondition(spark: SparkSession, condition: LogicalPlan => Boolean): Unit
----

`recacheByCondition`...FIXME

NOTE: `recacheByCondition` is used when `CacheManager` is requested to <<uncacheQuery, uncache a structured query>>, <<recacheByPlan, recacheByPlan>>, and <<recacheByPath, recacheByPath>>.

=== [[recacheByPlan]] `recacheByPlan` Method

[source, scala]
----
recacheByPlan(spark: SparkSession, plan: LogicalPlan): Unit
----

`recacheByPlan`...FIXME

NOTE: `recacheByPlan` is used exclusively when `InsertIntoDataSourceCommand` logical command is <<spark-sql-LogicalPlan-InsertIntoDataSourceCommand.adoc#run, executed>>.

=== [[recacheByPath]] `recacheByPath` Method

[source, scala]
----
recacheByPath(spark: SparkSession, resourcePath: String): Unit
----

`recacheByPath`...FIXME

NOTE: `recacheByPath` is used exclusively when `CatalogImpl` is requested to link:spark-sql-CatalogImpl.adoc#refreshByPath[refreshByPath].

=== [[useCachedData]] Replacing Segments of Logical Query Plan With Cached Data -- `useCachedData` Method

[source, scala]
----
useCachedData(plan: LogicalPlan): LogicalPlan
----

`useCachedData`...FIXME

NOTE: `useCachedData` is used exclusively when `QueryExecution` is requested for a link:spark-sql-QueryExecution.adoc#withCachedData[cached logical query plan].

=== [[lookupAndRefresh]] `lookupAndRefresh` Internal Method

[source, scala]
----
lookupAndRefresh(
  plan: LogicalPlan,
  fs: FileSystem,
  qualifiedPath: Path): Boolean
----

`lookupAndRefresh`...FIXME

NOTE: `lookupAndRefresh` is used exclusively when `CacheManager` is requested to <<recacheByPath, recacheByPath>>.
