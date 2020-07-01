title: Options

# AvroOptions -- Avro Data Source Options

`AvroOptions` represents the <<options, options>> of the <<spark-sql-avro.adoc#, Avro data source>>.

[[options]]
.Options for Avro Data Source
[cols="1m,1,2",options="header",width="100%"]
|===
| Option / Key
| Default Value
| Description

| avroSchema
| (undefined)
| [[avroSchema]] Avro schema in JSON format

| compression
| (undefined)
a| [[compression]] Specifies the compression codec to use when writing Avro data to disk

NOTE: If the option is not defined explicitly, Avro data source uses <<spark-sql-properties.adoc#spark.sql.avro.compression.codec, spark.sql.avro.compression.codec>> configuration property.

| ignoreExtension
| `false`
a| [[ignoreExtension]] Controls whether Avro data source should read all Avro files regardless of their extension (`true`) or not (`false`)

By default, Avro data source reads only files with `.avro` file extension.

NOTE: If the option is not defined explicitly, Avro data source uses `avro.mapred.ignore.inputs.without.extension` Hadoop runtime property.

| recordName
| `topLevelRecord`
| [[recordName]] Top-level record name when writing Avro data to disk

Consult https://avro.apache.org/docs/1.8.2/spec.html#schema_record[Apache Avro™ 1.8.2 Specification]

| recordNamespace
| (empty)
| [[recordNamespace]] Record namespace when writing Avro data to disk

Consult https://avro.apache.org/docs/1.8.2/spec.html#schema_record[Apache Avro™ 1.8.2 Specification]
|===

NOTE: The <<options, options>> are case-insensitive.

`AvroOptions` is <<creating-instance, created>> when `AvroFileFormat` is requested to <<spark-sql-AvroFileFormat.adoc#inferSchema, inferSchema>>, <<spark-sql-AvroFileFormat.adoc#prepareWrite, prepareWrite>> and <<spark-sql-AvroFileFormat.adoc#buildReader, buildReader>>.

=== [[creating-instance]] Creating AvroOptions Instance

`AvroOptions` takes the following when created:

* [[parameters]] Case-insensitive configuration parameters (i.e. `Map[String, String]`)
* [[conf]] Hadoop https://hadoop.apache.org/docs/r3.1.1/api/org/apache/hadoop/conf/Configuration.html[Configuration]
