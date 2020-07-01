title: BucketSpec

# BucketSpec -- Bucketing Specification of Table

[[creating-instance]]
`BucketSpec` is the *bucketing specification* of a table, i.e. the metadata of the link:spark-sql-bucketing.adoc[bucketing] of a table.

`BucketSpec` includes the following:

* [[numBuckets]] Number of buckets
* [[bucketColumnNames]] Bucket column names - the names of the columns used for buckets (at least one)
* [[sortColumnNames]] Sort column names - the names of the columns used to sort data in buckets

The <<numBuckets, number of buckets>> has to be between `0` and `100000` exclusive (or an `AnalysisException` is thrown).

`BucketSpec` is <<creating-instance, created>> when:

. `DataFrameWriter` is requested to link:spark-sql-DataFrameWriter.adoc#saveAsTable[saveAsTable] (and does link:spark-sql-DataFrameWriter.adoc#getBucketSpec[getBucketSpec])

. `HiveExternalCatalog` is requested to link:hive/HiveExternalCatalog.adoc#getBucketSpecFromTableProperties[getBucketSpecFromTableProperties] and link:hive/HiveExternalCatalog.adoc#tableMetaToTableProps[tableMetaToTableProps]

. `HiveClientImpl` is requested to link:hive/HiveClientImpl.adoc#getTableOption[retrieve a table metadata]

. `SparkSqlAstBuilder` is requested to link:spark-sql-SparkSqlAstBuilder.adoc#visitBucketSpec[visitBucketSpec] (for `CREATE TABLE` SQL statement with `CLUSTERED BY` and `INTO n BUCKETS` with optional `SORTED BY` clauses)

[[toString]]
`BucketSpec` uses the following *text representation* (i.e. `toString`):

```
[numBuckets] buckets, bucket columns: [[bucketColumnNames]], sort columns: [[sortColumnNames]]
```

[source, scala]
----
import org.apache.spark.sql.catalyst.catalog.BucketSpec
val bucketSpec = BucketSpec(
  numBuckets = 8,
  bucketColumnNames = Seq("col1"),
  sortColumnNames = Seq("col2"))
scala> println(bucketSpec)
8 buckets, bucket columns: [col1], sort columns: [col2]
----

=== [[toLinkedHashMap]] Converting Bucketing Specification to LinkedHashMap -- `toLinkedHashMap` Method

[source, scala]
----
toLinkedHashMap: mutable.LinkedHashMap[String, String]
----

`toLinkedHashMap` converts the bucketing specification to a collection of pairs (`LinkedHashMap[String, String]`) with the following fields and their values:

* *Num Buckets* with the <<numBuckets, numBuckets>>
* *Bucket Columns* with the <<bucketColumnNames, bucketColumnNames>>
* *Sort Columns* with the <<sortColumnNames, sortColumnNames>>

`toLinkedHashMap` quotes the column names.

[source, scala]
----
scala> println(bucketSpec.toLinkedHashMap)
Map(Num Buckets -> 8, Bucket Columns -> [`col1`], Sort Columns -> [`col2`])
----

[NOTE]
====
`toLinkedHashMap` is used when:

* `CatalogTable` is requested for <<spark-sql-CatalogTable.adoc#toLinkedHashMap, toLinkedHashMap>>

* `DescribeTableCommand` logical command is <<spark-sql-LogicalPlan-DescribeTableCommand.adoc#run, executed>> with a non-empty <<partitionSpec, partitionSpec>> and the <<spark-sql-LogicalPlan-DescribeTableCommand.adoc#isExtended, isExtended>> flag on (that uses <<spark-sql-LogicalPlan-DescribeTableCommand.adoc#describeFormattedDetailedPartitionInfo, describeFormattedDetailedPartitionInfo>>).
====
