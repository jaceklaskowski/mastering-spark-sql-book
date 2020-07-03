# HiveTableRelation Leaf Logical Operator -- Representing Hive Tables in Logical Plan

`HiveTableRelation` is a link:../spark-sql-LogicalPlan-LeafNode.adoc[leaf logical operator] that represents a Hive table in a link:../spark-sql-LogicalPlan.adoc[logical query plan].

`HiveTableRelation` is <<creating-instance, created>> when `FindDataSourceTable` logical evaluation rule is requested to link:../spark-sql-Analyzer-FindDataSourceTable.adoc#apply[resolve UnresolvedCatalogRelations in a logical plan] (for link:../spark-sql-Analyzer-FindDataSourceTable.adoc#readHiveTable[Hive tables]).

NOTE: `HiveTableRelation` can be link:RelationConversions.adoc#convert[converted to a HadoopFsRelation] based on link:configuration-properties.adoc#spark.sql.hive.convertMetastoreParquet[spark.sql.hive.convertMetastoreParquet] and link:configuration-properties.adoc#spark.sql.hive.convertMetastoreOrc[spark.sql.hive.convertMetastoreOrc] properties (and "disappears" from a logical plan when enabled).

`HiveTableRelation` is <<isPartitioned, partitioned>> when it has at least one <<partitionCols, partition column>>.

[[MultiInstanceRelation]]
`HiveTableRelation` is a link:../spark-sql-MultiInstanceRelation.adoc[MultiInstanceRelation].

`HiveTableRelation` is converted (_resolved_) to as follows:

* link:HiveTableScanExec.adoc[HiveTableScanExec] physical operator in link:HiveTableScans.adoc[HiveTableScans] execution planning strategy

* link:InsertIntoHiveTable.adoc[InsertIntoHiveTable] command in link:HiveAnalysis.adoc[HiveAnalysis] logical resolution rule

[source, scala]
----
val tableName = "h1"

// Make the example reproducible
val db = spark.catalog.currentDatabase
import spark.sharedState.{externalCatalog => extCatalog}
extCatalog.dropTable(
  db, table = tableName, ignoreIfNotExists = true, purge = true)

// sql("CREATE TABLE h1 (id LONG) USING hive")
import org.apache.spark.sql.types.StructType
spark.catalog.createTable(
  tableName,
  source = "hive",
  schema = new StructType().add($"id".long),
  options = Map.empty[String, String])

val h1meta = extCatalog.getTable(db, tableName)
scala> println(h1meta.provider.get)
hive

// Looks like we've got the testing space ready for the experiment
val h1 = spark.table(tableName)

import org.apache.spark.sql.catalyst.dsl.plans._
val plan = table(tableName).insertInto("t2", overwrite = true)
scala> println(plan.numberedTreeString)
00 'InsertIntoTable 'UnresolvedRelation `t2`, true, false
01 +- 'UnresolvedRelation `h1`

// ResolveRelations logical rule first to resolve UnresolvedRelations
import spark.sessionState.analyzer.ResolveRelations
val rrPlan = ResolveRelations(plan)
scala> println(rrPlan.numberedTreeString)
00 'InsertIntoTable 'UnresolvedRelation `t2`, true, false
01 +- 'SubqueryAlias h1
02    +- 'UnresolvedCatalogRelation `default`.`h1`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe

// FindDataSourceTable logical rule next to resolve UnresolvedCatalogRelations
import org.apache.spark.sql.execution.datasources.FindDataSourceTable
val findTablesRule = new FindDataSourceTable(spark)
val planWithTables = findTablesRule(rrPlan)

// At long last...
// Note HiveTableRelation in the logical plan
scala> println(planWithTables.numberedTreeString)
00 'InsertIntoTable 'UnresolvedRelation `t2`, true, false
01 +- SubqueryAlias h1
02    +- HiveTableRelation `default`.`h1`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, [id#13L]
----

The link:../spark-sql-CatalogTable.adoc[metadata] of a `HiveTableRelation` (in a catalog) has to meet the requirements:

* The link:../spark-sql-CatalogTable.adoc#identifier[database] is defined
* The link:../spark-sql-CatalogTable.adoc#partitionSchema[partition schema] is of the same type as <<partitionCols, partitionCols>>
* The link:../spark-sql-CatalogTable.adoc#dataSchema[data schema] is of the same type as <<dataCols, dataCols>>

[[output]]
`HiveTableRelation` has the output attributes made up of <<dataCols, data>> followed by <<partitionCols, partition>> columns.

=== [[computeStats]] Computing Statistics -- `computeStats` Method

[source, scala]
----
computeStats(): Statistics
----

NOTE: `computeStats` is part of link:../spark-sql-LogicalPlan-LeafNode.adoc#computeStats[LeafNode Contract] to compute statistics for link:../spark-sql-cost-based-optimization.adoc[cost-based optimizer].

`computeStats` takes the link:../spark-sql-CatalogTable.adoc#stats[table statistics] from the <<tableMeta, table metadata>> if defined and link:../spark-sql-CatalogStatistics.adoc#toPlanStats[converts them to Spark statistics] (with <<output, output columns>>).

If the table statistics are not available, `computeStats` reports an `IllegalStateException`.

```
table stats must be specified.
```

=== [[creating-instance]] Creating HiveTableRelation Instance

`HiveTableRelation` takes the following when created:

* [[tableMeta]] link:../spark-sql-CatalogTable.adoc[Table metadata]
* [[dataCols]] Columns (as a collection of `AttributeReferences`)
* [[partitionCols]] <<partition-columns, Partition columns>> (as a collection of `AttributeReferences`)

=== [[partition-columns]] Partition Columns

When created, `HiveTableRelation` is given the <<partitionCols, partition columns>>.

link:../spark-sql-Analyzer-FindDataSourceTable.adoc[FindDataSourceTable] logical evaluation rule creates a `HiveTableRelation` based on a link:../spark-sql-CatalogTable.adoc[table specification] (from a catalog).

The <<partitionCols, partition columns>> are exactly link:../spark-sql-CatalogTable.adoc#partitionSchema[partitions] of the link:../spark-sql-CatalogTable.adoc[table specification].

=== [[isPartitioned]] `isPartitioned` Method

[source, scala]
----
isPartitioned: Boolean
----

`isPartitioned` is `true` when there is at least one <<partitionCols, partition column>>.

[NOTE]
====
`isPartitioned` is used when:

* `HiveMetastoreCatalog` is requested to link:HiveMetastoreCatalog.adoc#convertToLogicalRelation[convert a HiveTableRelation to a LogicalRelation over a HadoopFsRelation]

* link:RelationConversions.adoc[RelationConversions] logical posthoc evaluation rule is executed (on a link:RelationConversions.adoc#apply-InsertIntoTable[InsertIntoTable])

* `HiveTableScanExec` physical operator is link:HiveTableScanExec.adoc#doExecute[executed]
====
