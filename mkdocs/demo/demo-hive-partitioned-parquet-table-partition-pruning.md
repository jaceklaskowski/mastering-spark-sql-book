title: Hive Partitioned Parquet Table and Partition Pruning

# Demo: Hive Partitioned Parquet Table and Partition Pruning

The demo shows partition pruning optimization in Spark SQL for Hive partitioned tables in parquet format.

NOTE: The demo is a follow-up to link:demo-connecting-spark-sql-to-hive-metastore.adoc[Demo: Connecting Spark SQL to Hive Metastore (with Remote Metastore Server)]. Please finish it first before this demo.

The demo features the following:

* link:../hive/configuration-properties.adoc#spark.sql.hive.convertMetastoreParquet[spark.sql.hive.convertMetastoreParquet] configuration property is enabled (which is the default)

* Hadoop DFS for link:../spark-sql-StaticSQLConf.adoc#spark.sql.warehouse.dir[spark.sql.warehouse.dir]

## Create Hive Partitioned Table in Parquet Format

Create a Hive partitioned table in parquet format with some data.

```
CREATE TABLE hive_partitioned_table
(id BIGINT, name STRING)
COMMENT 'Demo: Hive Partitioned Parquet Table and Partition Pruning'
PARTITIONED BY (city STRING COMMENT 'City')
STORED AS PARQUET;

INSERT INTO hive_partitioned_table
PARTITION (city="Warsaw")
VALUES (0, 'Jacek');

INSERT INTO hive_partitioned_table
PARTITION (city="Paris")
VALUES (1, 'Agata');
```

## Accessing Hive Table in Spark Shell

Make sure that the table is accessible in Spark SQL.

```
assert(spark.conf.get("spark.sql.warehouse.dir").startsWith("hdfs"))

val tableName = "hive_partitioned_table"
assert(spark.table(tableName).collect.length == 2 /* rows */)

// Use the default value of spark.sql.hive.convertMetastoreParquet
assert(spark.conf.get("spark.sql.hive.convertMetastoreParquet").toBoolean)
```

Query the available partitions.

```
val parts = spark
  .sharedState
  .externalCatalog
  .listPartitions("default", tableName)
scala> parts.foreach(println)
CatalogPartition(
	Partition Values: [city=Warsaw]
	Location: hdfs://localhost:9000/user/hive/warehouse/hive_partitioned_table/city=Warsaw
	Serde Library: org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe
	InputFormat: org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat
	OutputFormat: org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat
	Storage Properties: [serialization.format=1]
	Partition Parameters: {rawDataSize=2, numFiles=1, transient_lastDdlTime=1585478385, totalSize=345, COLUMN_STATS_ACCURATE={"BASIC_STATS":"true"}, numRows=1}
	Created Time: Sun Mar 29 12:39:45 CEST 2020
	Last Access: Thu Jan 01 01:00:00 CET 1970
	Partition Statistics: 345 bytes, 1 rows)
CatalogPartition(
	Partition Values: [city=Paris]
	Location: hdfs://localhost:9000/user/hive/warehouse/hive_partitioned_table/city=Paris
	Serde Library: org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe
	InputFormat: org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat
	OutputFormat: org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat
	Storage Properties: [serialization.format=1]
	Partition Parameters: {rawDataSize=2, numFiles=1, transient_lastDdlTime=1585478387, totalSize=345, COLUMN_STATS_ACCURATE={"BASIC_STATS":"true"}, numRows=1}
	Created Time: Sun Mar 29 12:39:47 CEST 2020
	Last Access: Thu Jan 01 01:00:00 CET 1970
	Partition Statistics: 345 bytes, 1 rows)
```

Create another Hive table using Spark.

```
Seq("Warsaw").toDF("name").write.saveAsTable("cities")
```

Check out the table in Hive using `beeline`.

```
0: jdbc:hive2://localhost:10000> desc formatted cities;
+-------------------------------+----------------------------------------------------+----------------------------------------------------+
|           col_name            |                     data_type                      |                      comment                       |
+-------------------------------+----------------------------------------------------+----------------------------------------------------+
| # col_name                    | data_type                                          | comment                                            |
|                               | NULL                                               | NULL                                               |
| name                          | string                                             |                                                    |
|                               | NULL                                               | NULL                                               |
| # Detailed Table Information  | NULL                                               | NULL                                               |
| Database:                     | default                                            | NULL                                               |
| Owner:                        | jacek                                              | NULL                                               |
| CreateTime:                   | Sun Mar 29 14:39:40 CEST 2020                      | NULL                                               |
| LastAccessTime:               | UNKNOWN                                            | NULL                                               |
| Retention:                    | 0                                                  | NULL                                               |
| Location:                     | hdfs://localhost:9000/user/hive/warehouse/cities   | NULL                                               |
| Table Type:                   | MANAGED_TABLE                                      | NULL                                               |
| Table Parameters:             | NULL                                               | NULL                                               |
|                               | numFiles                                           | 1                                                  |
|                               | spark.sql.create.version                           | 2.4.5                                              |
|                               | spark.sql.sources.provider                         | parquet                                            |
|                               | spark.sql.sources.schema.numParts                  | 1                                                  |
|                               | spark.sql.sources.schema.part.0                    | {\"type\":\"struct\",\"fields\":[{\"name\":\"name\",\"type\":\"string\",\"nullable\":true,\"metadata\":{}}]} |
|                               | totalSize                                          | 425                                                |
|                               | transient_lastDdlTime                              | 1585485580                                         |
|                               | NULL                                               | NULL                                               |
| # Storage Information         | NULL                                               | NULL                                               |
| SerDe Library:                | org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe | NULL                                               |
| InputFormat:                  | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat | NULL                                               |
| OutputFormat:                 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat | NULL                                               |
| Compressed:                   | No                                                 | NULL                                               |
| Num Buckets:                  | -1                                                 | NULL                                               |
| Bucket Columns:               | []                                                 | NULL                                               |
| Sort Columns:                 | []                                                 | NULL                                               |
| Storage Desc Params:          | NULL                                               | NULL                                               |
|                               | path                                               | hdfs://localhost:9000/user/hive/warehouse/cities   |
|                               | serialization.format                               | 1                                                  |
+-------------------------------+----------------------------------------------------+----------------------------------------------------+
32 rows selected (0.075 seconds)
```

## Explore Partition Pruning

You'll be using link:../spark-sql-Expression-In.adoc[In] expression in structured queries to learn more on partition pruning.

TIP: Enable `INFO` logging level for link:../PrunedInMemoryFileIndex.adoc#logging[org.apache.spark.sql.execution.datasources.PrunedInMemoryFileIndex] logger.

Use a fixed list of cities to filter by (which should trigger partition pruning).

```
val q = sql(s"""SELECT * FROM $tableName WHERE city IN ('Warsaw')""")
scala> q.explain(extended = true)
== Parsed Logical Plan ==
'Project [*]
+- 'Filter 'city IN (Warsaw)
   +- 'UnresolvedRelation `hive_partitioned_table`

== Analyzed Logical Plan ==
id: bigint, name: string, city: string
Project [id#101L, name#102, city#103]
+- Filter city#103 IN (Warsaw)
   +- SubqueryAlias `default`.`hive_partitioned_table`
      +- Relation[id#101L,name#102,city#103] parquet

== Optimized Logical Plan ==
Project [id#101L, name#102, city#103]
+- Filter (isnotnull(city#103) && (city#103 = Warsaw))
   +- Relation[id#101L,name#102,city#103] parquet

== Physical Plan ==
*(1) FileScan parquet default.hive_partitioned_table[id#101L,name#102,city#103] Batched: true, Format: Parquet, Location: PrunedInMemoryFileIndex[hdfs://localhost:9000/user/hive/warehouse/hive_partitioned_table/city=War..., PartitionCount: 1, PartitionFilters: [isnotnull(city#103), (city#103 = Warsaw)], PushedFilters: [], ReadSchema: struct<id:bigint,name:string>
```

Note the `PartitionFilters` field of the leaf `FileScan` node in the physical plan. It uses an link:../PrunedInMemoryFileIndex.adoc[PrunedInMemoryFileIndex] (for the partition index). Let's explore it.

```
import org.apache.spark.sql.execution.FileSourceScanExec
val scan = q.queryExecution.executedPlan.collect { case op: FileSourceScanExec => op }.head

val index = scan.relation.location
scala> println(s"Time of partition metadata listing: ${index.metadataOpsTimeNs.get}ns")
Time of partition metadata listing: 41703540ns

// You may also want to review metadataTime metric in web UI
// Includes the above time and the time to list files

// You should see the following value (YMMV)
scan.execute.collect
scala> println(scan.metrics("metadataTime").value)
41
```

Use a subquery to filter by and note the `PartitionFilters` field of `FileScan` operator (which is not supported for partition pruning since the values to filter partitions by are not known until the execution time).

```
val q = sql(s"""SELECT * FROM $tableName WHERE city IN (SELECT * FROM cities)""")
scala> q.explain(extended = true)
== Parsed Logical Plan ==
'Project [*]
+- 'Filter 'city IN (list#104 [])
   :  +- 'Project [*]
   :     +- 'UnresolvedRelation `cities`
   +- 'UnresolvedRelation `hive_partitioned_table`

== Analyzed Logical Plan ==
id: bigint, name: string, city: string
Project [id#113L, name#114, city#115]
+- Filter city#115 IN (list#104 [])
   :  +- Project [name#108]
   :     +- SubqueryAlias `default`.`cities`
   :        +- Relation[name#108] parquet
   +- SubqueryAlias `default`.`hive_partitioned_table`
      +- Relation[id#113L,name#114,city#115] parquet

== Optimized Logical Plan ==
Join LeftSemi, (city#115 = name#108)
:- Relation[id#113L,name#114,city#115] parquet
+- Relation[name#108] parquet

== Physical Plan ==
*(2) BroadcastHashJoin [city#115], [name#108], LeftSemi, BuildRight
:- *(2) FileScan parquet default.hive_partitioned_table[id#113L,name#114,city#115] Batched: true, Format: Parquet, Location: CatalogFileIndex[hdfs://localhost:9000/user/hive/warehouse/hive_partitioned_table], PartitionCount: 2, PartitionFilters: [], PushedFilters: [], ReadSchema: struct<id:bigint,name:string>
+- BroadcastExchange HashedRelationBroadcastMode(List(input[0, string, true]))
   +- *(1) FileScan parquet default.cities[name#108] Batched: true, Format: Parquet, Location: InMemoryFileIndex[hdfs://localhost:9000/user/hive/warehouse/cities], PartitionFilters: [], PushedFilters: [], ReadSchema: struct<name:string>
```
