# InMemoryCatalog

`InMemoryCatalog` is...FIXME

=== [[listPartitionsByFilter]] `listPartitionsByFilter` Method

[source, scala]
----
listPartitionsByFilter(
  db: String,
  table: String,
  predicates: Seq[Expression],
  defaultTimeZoneId: String): Seq[CatalogTablePartition]
----

NOTE: `listPartitionsByFilter` is part of link:spark-sql-ExternalCatalog.adoc#listPartitionsByFilter[ExternalCatalog Contract] to...FIXME.

`listPartitionsByFilter`...FIXME
