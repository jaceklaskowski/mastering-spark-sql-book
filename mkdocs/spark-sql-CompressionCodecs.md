# CompressionCodecs

`CompressionCodecs` is a utility object...FIXME

[[getCodecClassName]]
[[shortCompressionCodecNames]]
.Known Compression Codecs
[cols="1,3",options="header",width="100%"]
|===
| Alias
| Fully-Qualified Class Name

| `none`
|

| `uncompressed`
|

| `bzip2`
| `org.apache.hadoop.io.compress.BZip2Codec`

| `deflate`
| `org.apache.hadoop.io.compress.DeflateCodec`

| `gzip`
| `org.apache.hadoop.io.compress.GzipCodec`

| `lz4`
| `org.apache.hadoop.io.compress.Lz4Codec`

| `snappy`
| `org.apache.hadoop.io.compress.SnappyCodec`
|===

=== [[setCodecConfiguration]] `setCodecConfiguration` Method

[source, scala]
----
setCodecConfiguration(conf: Configuration, codec: String): Unit
----

`setCodecConfiguration` sets compression-related configurations to the Hadoop `Configuration` per the input `codec`.

NOTE: The input `codec` should be a fully-qualified class name, i.e. `org.apache.hadoop.io.compress.SnappyCodec`.

If the input `codec` is defined (i.e. not `null`), `setCodecConfiguration` sets the following <<setCodecConfiguration-codec, configuration properties>>.

[[setCodecConfiguration-codec]]
.Compression-Related Hadoop Configuration Properties (codec defined)
[cols="1,3",options="header",width="100%"]
|===
| Name
| Value

| `mapreduce.output.fileoutputformat.compress`
| `true`

| `mapreduce.output.fileoutputformat.compress.type`
| `BLOCK`

| `mapreduce.output.fileoutputformat.compress.codec`
| The input `codec` name

| `mapreduce.map.output.compress`
| `true`

| `mapreduce.map.output.compress.codec`
| The input `codec` name
|===

If the input `codec` is not defined (i.e. `null`), `setCodecConfiguration` sets the following <<setCodecConfiguration-codec-undefined, configuration properties>>.

[[setCodecConfiguration-codec-undefined]]
.Compression-Related Hadoop Configuration Properties (codec not defined)
[cols="1,3",options="header",width="100%"]
|===
| Name
| Value

| `mapreduce.output.fileoutputformat.compress`
| `false`

| `mapreduce.map.output.compress`
| `false`
|===

NOTE: `setCodecConfiguration` is used when link:spark-sql-CSVFileFormat.adoc#prepareWrite[CSVFileFormat], link:spark-sql-JsonFileFormat.adoc#prepareWrite[JsonFileFormat] and link:spark-sql-TextFileFormat.adoc#prepareWrite[TextFileFormat] are requested to `prepareWrite`.
