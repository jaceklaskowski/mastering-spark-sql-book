# HadoopFileLinesReader

`HadoopFileLinesReader` is a Scala http://www.scala-lang.org/api/2.11.11/#scala.collection.Iterator[Iterator] of Apache Hadoop's https://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/io/Text.html[org.apache.hadoop.io.Text].

`HadoopFileLinesReader` is <<creating-instance, created>> to access datasets in the following data sources:

* `SimpleTextSource`
* `LibSVMFileFormat`
* `TextInputCSVDataSource`
* `TextInputJsonDataSource`
* link:spark-sql-TextFileFormat.adoc[TextFileFormat]

`HadoopFileLinesReader` uses the internal <<iterator, iterator>> that handles accessing files using Hadoop's FileSystem API.

=== [[creating-instance]] Creating HadoopFileLinesReader Instance

`HadoopFileLinesReader` takes the following when created:

* [[file]] link:spark-sql-PartitionedFile.adoc[PartitionedFile]
* [[conf]] Hadoop's `Configuration`

=== [[iterator]] `iterator` Internal Property

[source, scala]
----
iterator: RecordReaderIterator[Text]
----

When <<creating-instance, created>>, `HadoopFileLinesReader` creates an internal `iterator` that uses Hadoop's https://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/mapreduce/lib/input/FileSplit.html[org.apache.hadoop.mapreduce.lib.input.FileSplit] with Hadoop's https://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/fs/Path.html[org.apache.hadoop.fs.Path] and <<file, file>>.

`iterator` creates Hadoop's `TaskAttemptID`, `TaskAttemptContextImpl` and `LineRecordReader`.

`iterator` initializes `LineRecordReader` and passes it on to a <<spark-sql-RecordReaderIterator.adoc#, RecordReaderIterator>>.

NOTE: `iterator` is used for ``Iterator``-specific methods, i.e. `hasNext`, `next` and `close`.
