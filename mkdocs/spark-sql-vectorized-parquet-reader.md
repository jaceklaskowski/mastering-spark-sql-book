# Vectorized Parquet Decoding (Reader)

*Vectorized Parquet Decoding* (aka *Vectorized Parquet Reader*) allows for reading datasets in parquet format in batches, i.e. rows are decoded in batches. That aims at improving memory locality and cache utilization.

Quoting https://issues.apache.org/jira/browse/SPARK-12854[SPARK-12854 Vectorize Parquet reader]:

> The parquet encodings are largely designed to decode faster in batches, column by column. This can speed up the decoding considerably.

Vectorized Parquet Decoding is used exclusively when `ParquetFileFormat` is requested for a <<spark-sql-ParquetFileFormat.adoc#buildReaderWithPartitionValues, data reader>> when <<spark.sql.parquet.enableVectorizedReader, spark.sql.parquet.enableVectorizedReader>> property is enabled (`true`) and the read schema uses <<spark-sql-DataType.adoc#AtomicType, AtomicTypes>> data types only.

Vectorized Parquet Decoding uses <<spark-sql-VectorizedParquetRecordReader.adoc#, VectorizedParquetRecordReader>> for vectorized decoding.

=== [[spark.sql.parquet.enableVectorizedReader]] spark.sql.parquet.enableVectorizedReader Configuration Property

link:spark-sql-properties.adoc#spark.sql.parquet.enableVectorizedReader[spark.sql.parquet.enableVectorizedReader] configuration property is on by default.

[source, scala]
----
val isParquetVectorizedReaderEnabled = spark.conf.get("spark.sql.parquet.enableVectorizedReader").toBoolean
assert(isParquetVectorizedReaderEnabled, "spark.sql.parquet.enableVectorizedReader should be enabled by default")
----
