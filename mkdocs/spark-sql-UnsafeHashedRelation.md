# UnsafeHashedRelation

`UnsafeHashedRelation` is...FIXME

=== [[get]] `get` Method

[source, scala]
----
get(key: InternalRow): Iterator[InternalRow]
----

NOTE: `get` is part of link:spark-sql-HashedRelation.adoc#get[HashedRelation Contract] to give the internal rows for the given key or `null`.

`get`...FIXME

=== [[getValue]] Getting Value Row for Given Key -- `getValue` Method

[source, scala]
----
getValue(key: InternalRow): InternalRow
----

NOTE: `getValue` is part of link:spark-sql-HashedRelation.adoc#getValue[HashedRelation Contract] to give the value internal row for a given key.

`getValue`...FIXME

=== [[apply]] Creating UnsafeHashedRelation Instance -- `apply` Factory Method

[source, scala]
----
apply(
  input: Iterator[InternalRow],
  key: Seq[Expression],
  sizeEstimate: Int,
  taskMemoryManager: TaskMemoryManager): HashedRelation
----

`apply`...FIXME

NOTE: `apply` is used when...FIXME
