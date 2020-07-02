# VectorizedColumnReader

`VectorizedColumnReader` is a vectorized column reader that <<spark-sql-VectorizedParquetRecordReader.adoc#columnReaders, VectorizedParquetRecordReader>> uses for <<spark-sql-vectorized-parquet-reader.adoc#, Vectorized Parquet Decoding>>.

`VectorizedColumnReader` is <<creating-instance, created>> exclusively when `VectorizedParquetRecordReader` is requested to <<spark-sql-VectorizedParquetRecordReader.adoc#checkEndOfRowGroup, checkEndOfRowGroup>> (when requested to <<nextBatch, read next rows into a columnar batch>>).

Once <<creating-instance, created>>, `VectorizedColumnReader` is requested to <<readBatch, read rows as a batch>> (when `VectorizedParquetRecordReader` is requested to <<spark-sql-VectorizedParquetRecordReader.adoc#nextBatch, read next rows into a columnar batch>>).

`VectorizedColumnReader` is given a <<spark-sql-WritableColumnVector.adoc#, WritableColumnVector>> to store rows  <<readBatch, read as a batch>>.

[[creating-instance]]
`VectorizedColumnReader` takes the following to be created:

* [[descriptor]] Parquet `ColumnDescriptor`
* [[originalType]] Parquet `OriginalType`
* [[pageReader]] Parquet `PageReader`
* [[convertTz]] `TimeZone` (for timezone conversion to apply to int96 timestamps. `null` for no conversion)

=== [[readBatch]] Reading Rows As Batch -- `readBatch` Method

[source, java]
----
void readBatch(
  int total,
  WritableColumnVector column) throws IOException
----

`readBatch`...FIXME

NOTE: `readBatch` is used exclusively when `VectorizedParquetRecordReader` is requested to <<spark-sql-VectorizedParquetRecordReader.adoc#nextBatch, read next rows into a columnar batch>>.
