title: JDBCRelation

# JDBCRelation -- Relation with Inserting or Overwriting Data, Column Pruning and Filter Pushdown

`JDBCRelation` is a <<BaseRelation, BaseRelation>> that supports <<InsertableRelation, inserting or overwriting data>> and <<PrunedFilteredScan, column pruning with filter pushdown>>.

[[BaseRelation]]
As a <<spark-sql-BaseRelation.adoc#,BaseRelation>>, `JDBCRelation` defines the <<schema, schema of tuples (data)>> and the <<sqlContext, SQLContext>>.

[[InsertableRelation]]
As a <<spark-sql-InsertableRelation.adoc#,InsertableRelation>>, `JDBCRelation` supports <<insert, inserting or overwriting data>>.

[[PrunedFilteredScan]]
As a <<spark-sql-PrunedFilteredScan.adoc#,PrunedFilteredScan>>, `JDBCRelation` supports <<buildScan, building distributed data scan with column pruning and filter pushdown>>.

`JDBCRelation` is <<creating-instance, created>> when:

* `DataFrameReader` is requested to link:spark-sql-DataFrameReader.adoc#jdbc[load data from an external table using JDBC data source]

* `JdbcRelationProvider` is requested to link:spark-sql-JdbcRelationProvider.adoc#createRelation-RelationProvider[create a BaseRelation for reading data from a JDBC table]

[[toString]]
When requested for a human-friendly text representation, `JDBCRelation` requests the <<jdbcOptions, JDBCOptions>> for the name of the table and the <<parts, number of partitions>> (if defined).

```
JDBCRelation([table]) [numPartitions=[number]]
```

.JDBCRelation in web UI (Details for Query)
image::images/spark-sql-JDBCRelation-webui-query-details.png[align="center"]

```
scala> df.explain
== Physical Plan ==
*Scan JDBCRelation(projects) [numPartitions=1] [id#0,name#1,website#2] ReadSchema: struct<id:int,name:string,website:string>
```

[[sqlContext]]
`JDBCRelation` uses the <<sparkSession, SparkSession>> to return a link:spark-sql-SparkSession.adoc#sqlContext[SQLContext].

[[needConversion]]
`JDBCRelation` turns the <<spark-sql-BaseRelation.adoc#needConversion, needConversion>> flag off (to announce that <<buildScan, buildScan>> returns an `RDD[InternalRow]` already and `DataSourceStrategy` execution planning strategy does not have to do the <<spark-sql-SparkStrategy-DataSourceStrategy.adoc#PrunedFilteredScan, RDD conversion>>).

=== [[creating-instance]] Creating JDBCRelation Instance

`JDBCRelation` takes the following when created:

* [[parts]] Array of Spark Core's `Partitions`
* [[jdbcOptions]] link:spark-sql-JDBCOptions.adoc[JDBCOptions]
* [[sparkSession]] link:spark-sql-SparkSession.adoc[SparkSession]

=== [[unhandledFilters]] Finding Unhandled Filter Predicates -- `unhandledFilters` Method

[source, scala]
----
unhandledFilters(filters: Array[Filter]): Array[Filter]
----

NOTE: `unhandledFilters` is part of <<spark-sql-BaseRelation.adoc#unhandledFilters, BaseRelation Contract>> to find unhandled <<spark-sql-Filter.adoc#, Filter predicates>>.

`unhandledFilters` returns the <<spark-sql-Filter.adoc#, Filter predicates>> in the input `filters` that could not be <<spark-sql-JDBCRDD.adoc#compileFilter, converted to a SQL expression>> (and are therefore unhandled by the JDBC data source natively).

=== [[schema]] Schema of Tuples (Data) -- `schema` Property

[source, scala]
----
schema: StructType
----

NOTE: `schema` is part of link:spark-sql-BaseRelation.adoc#schema[BaseRelation Contract] to return the link:spark-sql-StructType.adoc[schema] of the tuples in a relation.

`schema` uses `JDBCRDD` to link:spark-sql-JDBCRDD.adoc#resolveTable[resolveTable] given the <<jdbcOptions, JDBCOptions>> (that simply returns the link:spark-sql-StructType.adoc[Catalyst schema] of the table, also known as the default table schema).

If link:spark-sql-JDBCOptions.adoc#customSchema[customSchema] JDBC option was defined, `schema` uses `JdbcUtils` to link:spark-sql-JdbcUtils.adoc#getCustomSchema[replace the data types in the default table schema].

=== [[insert]] Inserting or Overwriting Data to JDBC Table -- `insert` Method

[source, scala]
----
insert(data: DataFrame, overwrite: Boolean): Unit
----

NOTE: `insert` is part of <<spark-sql-InsertableRelation.adoc#insert, InsertableRelation Contract>> that inserts or overwrites data in a <<spark-sql-BaseRelation.adoc#, relation>>.

`insert` simply requests the input `DataFrame` for a <<spark-sql-dataset-operators.adoc#write, DataFrameWriter>> that in turn is requested to <<spark-sql-DataFrameWriter.adoc#jdbc, save the data to a table using the JDBC data source>> (itself!) with the <<spark-sql-JDBCOptions.adoc#url, url>>, <<spark-sql-JDBCOptions.adoc#table, table>> and <<spark-sql-JDBCOptions.adoc#asProperties, all options>>.

`insert` also requests the `DataFrameWriter` to <<spark-sql-DataFrameWriter.adoc#mode, set the save mode>> as <<spark-sql-DataFrameWriter.adoc#Overwrite, Overwrite>> or <<spark-sql-DataFrameWriter.adoc#Append, Append>> per the input `overwrite` flag.

NOTE: `insert` uses a "trick" to reuse a code that is responsible for <<spark-sql-JdbcRelationProvider.adoc#createRelation-CreatableRelationProvider, saving data to a JDBC table>>.

=== [[buildScan]] Building Distributed Data Scan with Column Pruning and Filter Pushdown -- `buildScan` Method

[source, scala]
----
buildScan(requiredColumns: Array[String], filters: Array[Filter]): RDD[Row]
----

NOTE: `buildScan` is part of <<spark-sql-PrunedFilteredScan.adoc#buildScan, PrunedFilteredScan Contract>> to build a distributed data scan (as a `RDD[Row]`) with support for column pruning and filter pushdown.

`buildScan` uses the `JDBCRDD` object to <<spark-sql-JDBCRDD.adoc#scanTable, create a RDD[Row] for a distributed data scan>>.
