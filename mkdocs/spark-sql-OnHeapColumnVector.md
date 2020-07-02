# OnHeapColumnVector

`OnHeapColumnVector` is a concrete link:spark-sql-WritableColumnVector.adoc[WritableColumnVector] that...FIXME

`OnHeapColumnVector` is <<creating-instance, created>> when:

* `OnHeapColumnVector` is requested to <<allocateColumns, allocate column vectors>> and <<reserveNewColumn, reserveNewColumn>>

* `OrcColumnarBatchReader` is requested to `initBatch`

=== [[allocateColumns]] Allocating Column Vectors -- `allocateColumns` Static Method

[source, java]
----
OnHeapColumnVector[] allocateColumns(int capacity, StructType schema) // <1>
OnHeapColumnVector[] allocateColumns(int capacity, StructField[] fields)
----
<1> Simply converts `StructType` to `StructField[]` and calls the other `allocateColumns`

`allocateColumns` creates an array of `OnHeapColumnVector` for every field (to hold `capacity` number of elements of the link:spark-sql-DataType.adoc[data type] per field).

[NOTE]
====
`allocateColumns` is used when:

* `AggregateHashMap` is created

* `InMemoryTableScanExec` is requested to link:spark-sql-SparkPlan-InMemoryTableScanExec.adoc#createAndDecompressColumn[createAndDecompressColumn]

* `VectorizedParquetRecordReader` is requested to link:spark-sql-VectorizedParquetRecordReader.adoc#initBatch[initBatch] (with `ON_HEAP` memory mode)

* `OrcColumnarBatchReader` is requested to `initBatch` (with `ON_HEAP` memory mode)

* `ColumnVectorUtils` is requested to convert an iterator of rows into a single `ColumnBatch` (aka `toBatch`)
====

=== [[creating-instance]] Creating OnHeapColumnVector Instance

`OnHeapColumnVector` takes the following when created:

* [[capacity]] Number of elements to hold in a vector (aka `capacity`)
* [[type]] link:spark-sql-DataType.adoc[Data type] of the elements stored

When created, `OnHeapColumnVector` <<reserveInternal, reserveInternal>> (for the given <<capacity, capacity>>) and link:spark-sql-WritableColumnVector.adoc#reset[reset].

=== [[reserveInternal]] `reserveInternal` Method

[source, java]
----
void reserveInternal(int newCapacity)
----

NOTE: `reserveInternal` is part of link:spark-sql-WritableColumnVector.adoc#reserveInternal[WritableColumnVector Contract] to...FIXME.

`reserveInternal`...FIXME

=== [[reserveNewColumn]] `reserveNewColumn` Method

[source, java]
----
OnHeapColumnVector reserveNewColumn(int capacity, DataType type)
----

NOTE: `reserveNewColumn` is part of link:spark-sql-WritableColumnVector.adoc#reserveNewColumn[WritableColumnVector Contract] to...FIXME.

`reserveNewColumn`...FIXME
