title: UnresolvedCatalogRelation

# UnresolvedCatalogRelation Leaf Logical Operator -- Placeholder of Catalog Tables

`UnresolvedCatalogRelation` is a link:spark-sql-LogicalPlan-LeafNode.adoc[leaf logical operator] that acts as a placeholder in a logical query plan for link:spark-sql-Analyzer-FindDataSourceTable.adoc[FindDataSourceTable] logical evaluation rule to resolve it to a concrete relation logical operator (i.e. link:spark-sql-LogicalPlan-LogicalRelation.adoc[LogicalRelations] for data source tables or link:hive/HiveTableRelation.adoc[HiveTableRelations] for hive tables).

`UnresolvedCatalogRelation` is <<creating-instance, created>> when `SessionCatalog` is requested to link:spark-sql-SessionCatalog.adoc#lookupRelation[find a relation] (for link:spark-sql-LogicalPlan-DescribeTableCommand.adoc[DescribeTableCommand] logical command or link:spark-sql-Analyzer-ResolveRelations.adoc[ResolveRelations] logical evaluation rule).

[source, scala]
----
// non-temporary global or local view
// database defined
// externalCatalog.getTable returns non-VIEW table
// Make the example reproducible
val tableName = "t1"
spark.sharedState.externalCatalog.dropTable(
  db = "default",
  table = tableName,
  ignoreIfNotExists = true,
  purge = true)
spark.range(10).write.saveAsTable(tableName)

scala> :type spark.sessionState.catalog
org.apache.spark.sql.catalyst.catalog.SessionCatalog

import org.apache.spark.sql.catalyst.TableIdentifier
val plan = spark.sessionState.catalog.lookupRelation(TableIdentifier(tableName))
scala> println(plan.numberedTreeString)
00 'SubqueryAlias t1
01 +- 'UnresolvedCatalogRelation `default`.`t1`, org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe
----

When <<creating-instance, created>>, `UnresolvedCatalogRelation` asserts that the database is specified.

[[resolved]]
`UnresolvedCatalogRelation` can never be <<spark-sql-LogicalPlan.adoc#resolved, resolved>> and is converted to a <<spark-sql-LogicalPlan-LogicalRelation.adoc#, LogicalRelation>> for a data source table or a link:hive/HiveTableRelation.adoc[HiveTableRelation] for a hive table at analysis phase.

[[output]]
`UnresolvedCatalogRelation` uses an empty <<spark-sql-catalyst-QueryPlan.adoc#output, output schema>>.

[[creating-instance]]
[[tableMeta]]
`UnresolvedCatalogRelation` takes a single <<spark-sql-CatalogTable.adoc#, CatalogTable>> when created.
