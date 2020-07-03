# SizeEstimator

`SizeEstimator` is...FIXME

=== [[estimate]] `estimate` Method

[source, scala]
----
estimate(obj: AnyRef): Long // <1>
// Internal
estimate(obj: AnyRef, visited: IdentityHashMap[AnyRef, AnyRef]): Long
----
<1> Uses `estimate` with an empty `IdentityHashMap`

`estimate`...FIXME

[NOTE]
====
`estimate` is used when:

* `SizeEstimator` is requested to <<sampleArray, sampleArray>>

* FIXME
====

=== [[sampleArray]] `sampleArray` Internal Method

[source, scala]
----
sampleArray(
  array: AnyRef,
  state: SearchState,
  rand: Random,
  drawn: OpenHashSet[Int],
  length: Int): Long
----

`sampleArray`...FIXME

NOTE: `sampleArray` is used when...FIXME

=== [[visitSingleObject]] `visitSingleObject` Internal Method

[source, scala]
----
visitSingleObject(obj: AnyRef, state: SearchState): Unit
----

`visitSingleObject`...FIXME

NOTE: `visitSingleObject` is used exclusively when `SizeEstimator` is requested to <<estimate, estimate>>.
