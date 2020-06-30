# ColumnarBatch

`ColumnarBatch` allows to work with multiple <<columns, ColumnVectors>> as a row-wise table.

[source, scala]
----
import org.apache.spark.sql.types._
val schema = new StructType()
  .add("intCol", IntegerType)
  .add("doubleCol", DoubleType)
  .add("intCol2", IntegerType)
  .add("string", BinaryType)

val capacity = 4 * 1024 // 4k
import org.apache.spark.memory.MemoryMode
import org.apache.spark.sql.execution.vectorized.OnHeapColumnVector
val columns = schema.fields.map { field =>
  new OnHeapColumnVector(capacity, field.dataType)
}

import org.apache.spark.sql.vectorized.ColumnarBatch
val batch = new ColumnarBatch(columns.toArray)

// Add a row [1, 1.1, NULL]
columns(0).putInt(0, 1)
columns(1).putDouble(0, 1.1)
columns(2).putNull(0)
columns(3).putByteArray(0, "Hello".getBytes(java.nio.charset.StandardCharsets.UTF_8))
batch.setNumRows(1)

assert(batch.getRow(0).numFields == 4)
----

`ColumnarBatch` is <<creating-instance, created>> when:

* `InMemoryTableScanExec` physical operator is requested to link:spark-sql-SparkPlan-InMemoryTableScanExec.adoc#createAndDecompressColumn[createAndDecompressColumn]

* `VectorizedParquetRecordReader` is requested to link:spark-sql-VectorizedParquetRecordReader.adoc#initBatch[initBatch]

* `OrcColumnarBatchReader` is requested to `initBatch`

* `ColumnVectorUtils` is requested to `toBatch`

* `ArrowPythonRunner` is requested for a `Iterator[ColumnarBatch]` (i.e. `newReaderIterator`)

* `ArrowConverters` is requested for a `ArrowRowIterator` (i.e. `fromPayloadIterator`)

[[creating-instance]]
[[columns]]
`ColumnarBatch` takes an array of <<spark-sql-ColumnVector.adoc#, ColumnVectors>> to be created. `ColumnarBatch` immediately initializes the internal <<row, MutableColumnarRow>>.

[[numCols]]
The number of columns in a `ColumnarBatch` is the number of <<columns, ColumnVectors>> (this batch was created with).

[NOTE]
====
`ColumnarBatch` is an `Evolving` contract that is evolving towards becoming a stable API, but is not a stable API yet and can change from one feature release to another release.

In other words, using the contract is as treading on thin ice.
====

[[internal-registries]]
.ColumnarBatch's Internal Properties (e.g. Registries, Counters and Flags)
[cols="1m,3",options="header",width="100%"]
|===
| Name
| Description

| numRows
| [[numRows]] Number of rows

| row
| [[row]] `MutableColumnarRow` over the <<columns, ColumnVectors>>
|===

=== [[rowIterator]] Iterator Over InternalRows (in Batch) -- `rowIterator` Method

[source, java]
----
Iterator<InternalRow> rowIterator()
----

`rowIterator`...FIXME

[NOTE]
====
`rowIterator` is used when:

* `ArrowConverters` is requested to `fromBatchIterator`

* `AggregateInPandasExec`, `WindowInPandasExec`, and `FlatMapGroupsInPandasExec` physical operators are requested to execute (`doExecute`)

* `ArrowEvalPythonExec` physical operator is requested to `evaluate`
====

=== [[setNumRows]] Specifying Number of Rows (in Batch) -- `setNumRows` Method

[source, java]
----
void setNumRows(int numRows)
----

In essence, `setNumRows` resets the batch and makes it available for reuse.

Internally, `setNumRows` simply sets the <<numRows, numRows>> to the given `numRows`.

[NOTE]
====
`setNumRows` is used when:

* `OrcColumnarBatchReader` is requested to `nextBatch`

* `VectorizedParquetRecordReader` is requested to <<spark-sql-VectorizedParquetRecordReader.adoc#nextBatch, nextBatch>> (when `VectorizedParquetRecordReader` is requested to <<spark-sql-VectorizedParquetRecordReader.adoc#nextKeyValue, nextKeyValue>>)

* `ColumnVectorUtils` is requested to `toBatch` (for testing only)

* `ArrowConverters` is requested to `fromBatchIterator`

* `InMemoryTableScanExec` physical operator is requested to <<spark-sql-SparkPlan-InMemoryTableScanExec.adoc#createAndDecompressColumn, createAndDecompressColumn>>

* `ArrowPythonRunner` is requested for a `ReaderIterator` (`newReaderIterator`)
====
