title: CatalogTablePartition

# CatalogTablePartition -- Partition Specification of Table

`CatalogTablePartition` is the *partition specification* of a table, i.e. the metadata of the partitions of a table.

`CatalogTablePartition` is <<creating-instance, created>> when:

* `HiveClientImpl` is requested to link:hive/HiveClientImpl.adoc#fromHivePartition[retrieve a table partition metadata]

* `AlterTableAddPartitionCommand` and `AlterTableRecoverPartitionsCommand` logical commands are executed

`CatalogTablePartition` can hold the <<stats, table statistics>> that...FIXME

[[simpleString]]
The *readable text representation* of a `CatalogTablePartition` (aka `simpleString`) is...FIXME

NOTE: `simpleString` is used exclusively when `ShowTablesCommand` is <<spark-sql-LogicalPlan-ShowTablesCommand.adoc#run, executed>> (with a partition specification).

[[toString]]
`CatalogTablePartition` uses the following *text representation* (i.e. `toString`)...FIXME

=== [[creating-instance]] Creating CatalogTablePartition Instance

`CatalogTablePartition` takes the following when created:

* [[spec]] Partition specification
* [[storage]] <<spark-sql-CatalogStorageFormat.adoc#, CatalogStorageFormat>>
* [[parameters]] Parameters (default: an empty collection)
* [[stats]] <<spark-sql-CatalogStatistics.adoc#, Table statistics>> (default: `None`)

=== [[toLinkedHashMap]] Converting Partition Specification to LinkedHashMap -- `toLinkedHashMap` Method

[source, scala]
----
toLinkedHashMap: mutable.LinkedHashMap[String, String]
----

`toLinkedHashMap` converts the partition specification to a collection of pairs (`LinkedHashMap[String, String]`) with the following fields and their values:

* *Partition Values* with the <<spec, spec>>
* <<spark-sql-CatalogStorageFormat.adoc#toLinkedHashMap, Storage specification>> (of the given <<storage, CatalogStorageFormat>>)
* *Partition Parameters* with the <<parameters, parameters>> (if not empty)
* *Partition Statistics* with the <<stats, CatalogStatistics>> (if available)

[NOTE]
====
`toLinkedHashMap` is used when:

* `DescribeTableCommand` logical command is <<spark-sql-LogicalPlan-DescribeTableCommand.adoc#run, executed>> (with the link:spark-sql-LogicalPlan-DescribeTableCommand.adoc#isExtended[isExtended] flag on and a non-empty link:spark-sql-LogicalPlan-DescribeTableCommand.adoc#partitionSpec[partitionSpec]).

* `CatalogTablePartition` is requested for either a <<simpleString, simple>> or a <<toString, catalog>> text representation
====

=== [[location]] `location` Method

[source, scala]
----
location: URI
----

`location` simply returns the <<spark-sql-CatalogStorageFormat.adoc#locationUri, location URI>> of the <<storage, CatalogStorageFormat>> or throws an `AnalysisException`:

```
Partition [[specString]] did not specify locationUri
```

NOTE: `location` is used when...FIXME
