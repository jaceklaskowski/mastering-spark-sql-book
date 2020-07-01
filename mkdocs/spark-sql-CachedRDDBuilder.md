# CachedRDDBuilder

`CachedRDDBuilder` is <<creating-instance, created>> exclusively when <<spark-sql-LogicalPlan-InMemoryRelation.adoc#, InMemoryRelation>> leaf logical operator is created.

[[cachedColumnBuffers]]
`CachedRDDBuilder` uses a `RDD` of <<CachedBatch, CachedBatches>> that is either <<_cachedColumnBuffers, given>> or <<buildBuffers, built internally>>.

[[CachedBatch]]
`CachedRDDBuilder` uses `CachedBatch` data structure with the following attributes:

* [[numRows]] Number of rows
* [[buffers]] Buffers (`Array[Array[Byte]]`)
* [[stats]] Statistics (<<spark-sql-InternalRow.adoc#, InternalRow>>)

[[isCachedColumnBuffersLoaded]]
`CachedRDDBuilder` uses `isCachedColumnBuffersLoaded` flag that is enabled (`true`) when the <<_cachedColumnBuffers, _cachedColumnBuffers>> is defined (not `null`). `isCachedColumnBuffersLoaded` is used exclusively when `CacheManager` is requested to <<spark-sql-CacheManager.adoc#recacheByCondition, recacheByCondition>>.

[[sizeInBytesStats]]
`CachedRDDBuilder` uses `sizeInBytesStats` metric (`LongAccumulator`) to <<buildBuffers, buildBuffers>> and when `InMemoryRelation` is requested to <<spark-sql-LogicalPlan-InMemoryRelation.adoc#computeStats, computeStats>>.

=== [[creating-instance]] Creating CachedRDDBuilder Instance

`CachedRDDBuilder` takes the following to be created:

* [[useCompression]] `useCompression` flag
* [[batchSize]] Batch size
* [[storageLevel]] `StorageLevel`
* [[cachedPlan]] <<spark-sql-SparkPlan.adoc#, Physical operator>>
* [[tableName]] Table name
* [[_cachedColumnBuffers]] `RDD[CachedBatch]` (default: `null`)

`CachedRDDBuilder` initializes the <<internal-registries, internal registries and counters>>.

=== [[buildBuffers]] `buildBuffers` Internal Method

[source, scala]
----
buildBuffers(): RDD[CachedBatch]
----

`buildBuffers`...FIXME

NOTE: `buildBuffers` is used exclusively when `CachedRDDBuilder` is requested to <<cachedColumnBuffers, cachedColumnBuffers>>.

=== [[clearCache]] `clearCache` Method

[source, scala]
----
clearCache(blocking: Boolean = true): Unit
----

`clearCache`...FIXME

NOTE: `clearCache` is used exclusively when `CacheManager` is requested to <<spark-sql-CacheManager.adoc#clearCache, clearCache>>, <<spark-sql-CacheManager.adoc#uncacheQuery, uncacheQuery>>, and <<spark-sql-CacheManager.adoc#recacheByCondition, recacheByCondition>>.
