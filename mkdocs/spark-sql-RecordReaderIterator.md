title: RecordReaderIterator

# RecordReaderIterator -- Scala Iterator over Hadoop RecordReader's Values

[[creating-instance]]
[[rowReader]]
`RecordReaderIterator` is a Scala https://www.scala-lang.org/api/2.12.x/scala/collection/Iterator.html[scala.collection.Iterator] over the values of a Hadoop https://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/mapreduce/RecordReader.html[RecordReader].

`RecordReaderIterator` is <<creating-instance, created>> when:

* New <<spark-sql-OrcFileFormat.adoc#buildReaderWithPartitionValues, OrcFileFormat>> and <<spark-sql-ParquetFileFormat.adoc#buildReaderWithPartitionValues, ParquetFileFormat>> are requested to build a data reader

* <<spark-sql-spark-HadoopFileLinesReader.adoc#iterator, HadoopFileLinesReader>> and `HadoopFileWholeTextReader` are requested for an value iterator

* Legacy `OrcFileFormat` is requested to <<spark-sql-OrcFileFormat.adoc#buildReader, build a data reader>>

[[close]]
When requested to close, `RecordReaderIterator` simply requests the underlying <<rowReader, RecordReader>> to close.

[[hasNext]]
When requested to check whether or not there more internal rows, `RecordReaderIterator` simply requests the underlying <<rowReader, RecordReader>> for `nextKeyValue`.

[[next]]
When requested for the next internal row, `RecordReaderIterator` simply requests the underlying <<rowReader, RecordReader>> for `getCurrentValue`.
