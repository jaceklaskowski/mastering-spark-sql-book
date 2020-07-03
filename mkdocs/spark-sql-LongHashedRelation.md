# LongHashedRelation

`LongHashedRelation` is a link:spark-sql-HashedRelation.adoc[HashedRelation] that is used when `HashedRelation` is requested for a link:spark-sql-HashedRelation.adoc#apply[concrete HashedRelation instance] when the single key is of type long.

`LongHashedRelation` is also a Java `Externalizable`, i.e. when persisted, only the identity is written in the serialization stream and it is the responsibility of the class to <<writeExternal, save>> and <<readExternal, restore>> the contents of its instances.

`LongHashedRelation` is <<creating-instance, created>> when:

. `HashedRelation` is requested for a link:spark-sql-HashedRelation.adoc#apply[concrete HashedRelation] (and <<apply, apply>> factory method is used)

. `LongHashedRelation` is requested for a <<asReadOnlyCopy, read-only copy>> (when `BroadcastHashJoinExec` is requested to link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc#doExecute[execute])

=== [[writeExternal]] `writeExternal` Method

[source, scala]
----
writeExternal(out: ObjectOutput): Unit
----

NOTE: `writeExternal` is part of Java's link:++https://docs.oracle.com/javase/8/docs/api/java/io/Externalizable.html#writeExternal-java.io.ObjectOutput-++[Externalizable Contract] to...FIXME.

`writeExternal`...FIXME

NOTE: `writeExternal` is used when...FIXME

=== [[readExternal]] `readExternal` Method

[source, scala]
----
readExternal(in: ObjectInput): Unit
----

NOTE: `readExternal` is part of Java's link:++https://docs.oracle.com/javase/8/docs/api/java/io/Externalizable.html#readExternal-java.io.ObjectInput-++[Externalizable Contract] to...FIXME.

`readExternal`...FIXME

NOTE: `readExternal` is used when...FIXME

=== [[creating-instance]] Creating LongHashedRelation Instance

`LongHashedRelation` takes the following when created:

* [[nFields]] Number of fields
* [[map]] `LongToUnsafeRowMap`

`LongHashedRelation` initializes the <<internal-registries, internal registries and counters>>.

=== [[asReadOnlyCopy]] Creating Read-Only Copy of LongHashedRelation -- `asReadOnlyCopy` Method

[source, scala]
----
asReadOnlyCopy(): LongHashedRelation
----

NOTE: `asReadOnlyCopy` is part of link:spark-sql-HashedRelation.adoc#asReadOnlyCopy[HashedRelation Contract] to...FIXME.

`asReadOnlyCopy`...FIXME

=== [[getValue]] Getting Value Row for Given Key -- `getValue` Method

[source, scala]
----
getValue(key: InternalRow): InternalRow
----

NOTE: `getValue` is part of link:spark-sql-HashedRelation.adoc#getValue[HashedRelation Contract] to give the value internal row for a given key.

`getValue` checks if the input `key` is null at `0` position and if so gives `null`. Otherwise, `getValue` takes the long value at position `0` and <<getValue, gets the value>>.

=== [[apply]] Creating LongHashedRelation Instance -- `apply` Factory Method

[source, scala]
----
apply(
  input: Iterator[InternalRow],
  key: Seq[Expression],
  sizeEstimate: Int,
  taskMemoryManager: TaskMemoryManager): LongHashedRelation
----

`apply`...FIXME

NOTE: `apply` is used exclusively when `HashedRelation` is requested for a link:spark-sql-HashedRelation.adoc#apply[concrete HashedRelation].
