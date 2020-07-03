# RDDConversions Helper Object

`RDDConversions` is a Scala object that is used to <<productToRowRdd, productToRowRdd>> and <<rowToRowRdd, rowToRowRdd>> methods.

=== [[productToRowRdd]] `productToRowRdd` Method

[source, scala]
----
productToRowRdd[A <: Product](data: RDD[A], outputTypes: Seq[DataType]): RDD[InternalRow]
----

`productToRowRdd`...FIXME

NOTE: `productToRowRdd` is used when...FIXME

=== [[rowToRowRdd]] Converting Scala Objects In Rows to Values Of Catalyst Types -- `rowToRowRdd` Method

[source, scala]
----
rowToRowRdd(data: RDD[Row], outputTypes: Seq[DataType]): RDD[InternalRow]
----

`rowToRowRdd` maps over partitions of the input `RDD[Row]` (using `RDD.mapPartitions` operator) that creates a `MapPartitionsRDD` with a "map" function.

TIP: Use `RDD.toDebugString` to see the additional `MapPartitionsRDD` in an RDD lineage.

The "map" function takes a Scala `Iterator` of link:spark-sql-Row.adoc[Row] objects and does the following:

. Creates a `GenericInternalRow` (of the size that is the number of columns per the input `Seq[DataType]`)

. link:spark-sql-CatalystTypeConverters.adoc#createToCatalystConverter[Creates a converter function] for every `DataType` in `Seq[DataType]`

. For every link:spark-sql-Row.adoc[Row] object in the partition (iterator), applies the converter function per position and adds the result value to the `GenericInternalRow`

. In the end, returns a `GenericInternalRow` for every row

NOTE: `rowToRowRdd` is used exclusively when `DataSourceStrategy` execution planning strategy is link:spark-sql-SparkStrategy-DataSourceStrategy.adoc#apply[executed] (and requested to link:spark-sql-SparkStrategy-DataSourceStrategy.adoc#toCatalystRDD[toCatalystRDD]).
