# CSVFileFormat

[[shortName]]
`CSVFileFormat` is a link:spark-sql-TextBasedFileFormat.adoc[TextBasedFileFormat] for *csv* format (i.e. link:spark-sql-DataSourceRegister.adoc#shortName[registers itself to handle files in csv format] and converts them to Spark SQL rows).

[source, scala]
----
spark.read.format("csv").load("csv-datasets")

// or the same as above using a shortcut
spark.read.csv("csv-datasets")
----

`CSVFileFormat` uses <<CSVOptions, CSV options>> (that in turn are used to configure the underlying CSV parser from https://github.com/uniVocity/univocity-parsers[uniVocity-parsers] project).

[[options]]
[[CSVOptions]]
.CSVFileFormat's Options
[cols="1,1,3",options="header",width="100%"]
|===
| Option
| Default Value
| Description

| [[charset]] `charset`
| `UTF-8`
|

Alias of <<encoding, encoding>>

| [[charToEscapeQuoteEscaping]] `charToEscapeQuoteEscaping`
| `\\`
| One character to...FIXME

| [[codec]] `codec`
|
a| Compression codec that can be either one of the link:spark-sql-CompressionCodecs.adoc#shortCompressionCodecNames[known aliases] or a fully-qualified class name.

Alias of <<compression, compression>>

| [[columnNameOfCorruptRecord]] `columnNameOfCorruptRecord`
|
|

| [[comment]] `comment`
| `\u0000`
|

| [[compression]] `compression`
|
a| Compression codec that can be either one of the link:spark-sql-CompressionCodecs.adoc#shortCompressionCodecNames[known aliases] or a fully-qualified class name.

Alias of <<codec, codec>>

| [[dateFormat]] `dateFormat`
| `yyyy-MM-dd`
| Uses `en_US` locale

| [[delimiter]] `delimiter`
| `,` (comma)
|

Alias of <<sep, sep>>

| [[encoding]] `encoding`
| `UTF-8`
|

Alias of <<charset, charset>>

| [[escape]] `escape`
| `\\`
|

| [[escapeQuotes]] `escapeQuotes`
| `true`
|

| [[header_]] `header`
|
|

| [[ignoreLeadingWhiteSpace]] `ignoreLeadingWhiteSpace`
a|
* `false` (for reading)
* `true` (for writing)
|

| [[ignoreTrailingWhiteSpace]] `ignoreTrailingWhiteSpace`
a|
* `false` (for reading)
* `true` (for writing)
|

| [[inferSchema]] `inferSchema`
|
|

| [[maxCharsPerColumn]] `maxCharsPerColumn`
| `-1`
|

| [[maxColumns]] `maxColumns`
| `20480`
|

| [[mode]] `mode`
| `PERMISSIVE`
a|

Possible values:

* `DROPMALFORMED`
* `PERMISSIVE` (default)
* `FAILFAST`

| [[multiLine]] `multiLine`
| `false`
|

| [[nanValue]] `nanValue`
| `NaN`
|

| [[negativeInf]] `negativeInf`
| `-Inf`
|

| [[nullValue]] `nullValue`
| (empty string)
|

| [[positiveInf]] `positiveInf`
| `Inf`
|

| [[sep]] `sep`
| `,` (comma)
|

Alias of <<delimiter, delimiter>>

| [[timestampFormat]] `timestampFormat`
| `yyyy-MM-dd'T'HH:mm:ss.SSSXXX`
| Uses <<timeZone, timeZone>> and `en_US` locale

| [[timeZone]] `timeZone`
| link:spark-sql-properties.adoc#spark.sql.session.timeZone[spark.sql.session.timeZone]
|

| [[quote]] `quote`
| `\"`
|

| [[quoteAll]] `quoteAll`
| `false`
|
|===

=== [[prepareWrite]] Preparing Write Job -- `prepareWrite` Method

[source, scala]
----
prepareWrite(
  sparkSession: SparkSession,
  job: Job,
  options: Map[String, String],
  dataSchema: StructType): OutputWriterFactory
----

NOTE: `prepareWrite` is part of the <<spark-sql-FileFormat.adoc#prepareWrite, FileFormat Contract>> to prepare a write job.

`prepareWrite`...FIXME

=== [[buildReader]] Building Partitioned Data Reader -- `buildReader` Method

[source, scala]
----
buildReader(
  sparkSession: SparkSession,
  dataSchema: StructType,
  partitionSchema: StructType,
  requiredSchema: StructType,
  filters: Seq[Filter],
  options: Map[String, String],
  hadoopConf: Configuration): (PartitionedFile) => Iterator[InternalRow]
----

NOTE: `buildReader` is part of the <<spark-sql-FileFormat.adoc#buildReader, FileFormat Contract>> to build a <<spark-sql-PartitionedFile.adoc#, PartitionedFile>> reader.

`buildReader`...FIXME
