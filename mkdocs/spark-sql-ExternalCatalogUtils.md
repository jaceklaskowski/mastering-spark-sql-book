# ExternalCatalogUtils

`ExternalCatalogUtils` is...FIXME

=== [[prunePartitionsByFilter]] `prunePartitionsByFilter` Method

[source, scala]
----
prunePartitionsByFilter(
  catalogTable: CatalogTable,
  inputPartitions: Seq[CatalogTablePartition],
  predicates: Seq[Expression],
  defaultTimeZoneId: String): Seq[CatalogTablePartition]
----

`prunePartitionsByFilter`...FIXME

NOTE: `prunePartitionsByFilter` is used when link:spark-sql-InMemoryCatalog.adoc#listPartitionsByFilter[InMemoryCatalog] and link:hive/HiveExternalCatalog.adoc#listPartitionsByFilter[HiveExternalCatalog] are requested to list partitions by a filter.
