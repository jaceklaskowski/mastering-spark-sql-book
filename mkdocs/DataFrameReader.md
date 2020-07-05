# DataFrameReader -- Loading Data From External Data Sources

`DataFrameReader` is a <<methods, fluent API>> to describe the <<source, input data source>> that will be used to <<load, "load" data from an external data source>> (e.g. <<creating-dataframes-from-files, files>>, <<creating-dataframes-from-tables, tables>>, <<jdbc, JDBC>> or <<loading-dataset-of-string, Dataset[String]>>).

`DataFrameReader` is <<creating-instance, created>> (available) exclusively using <<spark-sql-SparkSession.adoc#read, SparkSession.read>>.

[source, scala]
----
import org.apache.spark.sql.SparkSession
assert(spark.isInstanceOf[SparkSession])

import org.apache.spark.sql.DataFrameReader
val reader = spark.read
assert(reader.isInstanceOf[DataFrameReader])
----

[[methods]]
.DataFrameReader API
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| <<csv, csv>>
a|

[source, scala]
----
csv(csvDataset: Dataset[String]): DataFrame
csv(path: String): DataFrame
csv(paths: String*): DataFrame
----

| <<format, format>>
a|

[source, scala]
----
format(source: String): DataFrameReader
----

| <<jdbc, jdbc>>
a|

[source, scala]
----
jdbc(
  url: String,
  table: String,
  predicates: Array[String],
  connectionProperties: Properties): DataFrame
jdbc(
  url: String,
  table: String,
  properties: Properties): DataFrame
jdbc(
  url: String,
  table: String,
  columnName: String,
  lowerBound: Long,
  upperBound: Long,
  numPartitions: Int,
  connectionProperties: Properties): DataFrame
----

| <<json, json>>
a|

[source, scala]
----
json(jsonDataset: Dataset[String]): DataFrame
json(path: String): DataFrame
json(paths: String*): DataFrame
----

| <<load, load>>
a|

[source, scala]
----
load(): DataFrame
load(path: String): DataFrame
load(paths: String*): DataFrame
----

| <<option, option>>
a|

[source, scala]
----
option(key: String, value: Boolean): DataFrameReader
option(key: String, value: Double): DataFrameReader
option(key: String, value: Long): DataFrameReader
option(key: String, value: String): DataFrameReader
----

| <<options, options>>
a|

[source, scala]
----
options(options: scala.collection.Map[String, String]): DataFrameReader
options(options: java.util.Map[String, String]): DataFrameReader
----

| <<orc, orc>>
a|

[source, scala]
----
orc(path: String): DataFrame
orc(paths: String*): DataFrame
----

| <<parquet, parquet>>
a|

[source, scala]
----
parquet(path: String): DataFrame
parquet(paths: String*): DataFrame
----

| <<schema, schema>>
a|

[source, scala]
----
schema(schemaString: String): DataFrameReader
schema(schema: StructType): DataFrameReader
----

| <<table, table>>
a|

[source, scala]
----
table(tableName: String): DataFrame
----

| <<text, text>>
a|

[source, scala]
----
text(path: String): DataFrame
text(paths: String*): DataFrame
----

| <<textFile, textFile>>
a|

[source, scala]
----
textFile(path: String): Dataset[String]
textFile(paths: String*): Dataset[String]
----

|===

`DataFrameReader` supports many <<creating-dataframes-from-files, file formats>> natively and offers the <<format, interface to define custom formats>>.

NOTE: `DataFrameReader` assumes <<parquet, parquet>> data source file format by default that you can change using link:spark-sql-properties.adoc#spark.sql.sources.default[spark.sql.sources.default] configuration property.

After you have described the *loading pipeline* (i.e. the "Extract" part of ETL in Spark SQL), you eventually "trigger" the loading using format-agnostic <<load, load>> or format-specific (e.g. <<json, json>>, <<csv, csv>>, <<jdbc, jdbc>>) operators.

[source, scala]
----
import org.apache.spark.sql.SparkSession
val spark: SparkSession = ...

import org.apache.spark.sql.DataFrame

// Using format-agnostic load operator
val csvs: DataFrame = spark
  .read
  .format("csv")
  .option("header", true)
  .option("inferSchema", true)
  .load("*.csv")

// Using format-specific load operator
val jsons: DataFrame = spark
  .read
  .json("metrics/*.json")
----

NOTE: All <<methods, methods>> of `DataFrameReader` merely describe a process of loading a data and do not trigger a Spark job (until an action is called).

---

`DataFrameReader` can read text files using <<textFile, textFile>> methods that return typed `Datasets`.

[source, scala]
----
import org.apache.spark.sql.SparkSession
val spark: SparkSession = ...

import org.apache.spark.sql.Dataset
val lines: Dataset[String] = spark
  .read
  .textFile("README.md")
----

NOTE: Loading datasets using <<textFile, textFile>> methods allows for additional preprocessing before final processing of the string values as <<json, json>> or <<csv, csv>> lines.

---

[[loading-dataset-of-string]]
`DataFrameReader` can load datasets from `Dataset[String]` (with lines being complete "files") using format-specific <<csv, csv>> and <<json, json>> operators.

[source, scala]
----
val csvLine = "0,Warsaw,Poland"

import org.apache.spark.sql.Dataset
val cities: Dataset[String] = Seq(csvLine).toDS
scala> cities.show
+---------------+
|          value|
+---------------+
|0,Warsaw,Poland|
+---------------+

// Define schema explicitly (as below)
// or
// option("header", true) + option("inferSchema", true)
import org.apache.spark.sql.types.StructType
val schema = new StructType()
  .add($"id".long.copy(nullable = false))
  .add($"city".string)
  .add($"country".string)
scala> schema.printTreeString
root
 |-- id: long (nullable = false)
 |-- city: string (nullable = true)
 |-- country: string (nullable = true)

import org.apache.spark.sql.DataFrame
val citiesDF: DataFrame = spark
  .read
  .schema(schema)
  .csv(cities)
scala> citiesDF.show
+---+------+-------+
| id|  city|country|
+---+------+-------+
|  0|Warsaw| Poland|
+---+------+-------+
----

=== [[format]] Specifying Format Of Input Data Source -- `format` method

[source, scala]
----
format(source: String): DataFrameReader
----

You use `format` to configure `DataFrameReader` to use appropriate `source` format.

Supported data formats:

* `json`
* `csv` (since **2.0.0**)
* `parquet` (see link:spark-parquet.adoc[Parquet])
* `orc`
* `text`
* <<jdbc, jdbc>>
* `libsvm` -- only when used in `format("libsvm")`

NOTE: Spark SQL allows for link:spark-sql-datasource-custom-formats.adoc[developing custom data source formats].

=== [[schema]] Specifying Schema -- `schema` method

[source, scala]
----
schema(schema: StructType): DataFrameReader
----

`schema` allows for specifying the `schema` of a data source (that the `DataFrameReader` is about to read a dataset from).

[source, scala]
----
import org.apache.spark.sql.types.StructType
val schema = new StructType()
  .add($"id".long.copy(nullable = false))
  .add($"city".string)
  .add($"country".string)
scala> schema.printTreeString
root
 |-- id: long (nullable = false)
 |-- city: string (nullable = true)
 |-- country: string (nullable = true)

import org.apache.spark.sql.DataFrameReader
val r: DataFrameReader = spark.read.schema(schema)
----

NOTE: Some formats can infer schema from datasets (e.g. <<csv, csv>> or <<json, json>>) using <<option, inferSchema>> option.

TIP: Read up on link:spark-sql-schema.adoc[Schema].

=== [[option]][[options]] Specifying Load Options -- `option` and `options` Methods

[source, scala]
----
option(key: String, value: String): DataFrameReader
option(key: String, value: Boolean): DataFrameReader
option(key: String, value: Long): DataFrameReader
option(key: String, value: Double): DataFrameReader
----

You can also use `options` method to describe different options in a single `Map`.

[source, scala]
----
options(options: scala.collection.Map[String, String]): DataFrameReader
----

=== [[creating-dataframes-from-files]] Loading Datasets from Files (into DataFrames) Using Format-Specific Load Operators

`DataFrameReader` supports the following file formats:

* <<json, JSON>>
* <<csv, CSV>>
* <<parquet, parquet>>
* <<orc, ORC>>
* <<text, text>>

==== [[json]] `json` method

[source, scala]
----
json(path: String): DataFrame
json(paths: String*): DataFrame
json(jsonDataset: Dataset[String]): DataFrame
json(jsonRDD: RDD[String]): DataFrame
----

New in **2.0.0**: `prefersDecimal`

==== [[csv]] `csv` method

[source, scala]
----
csv(path: String): DataFrame
csv(paths: String*): DataFrame
csv(csvDataset: Dataset[String]): DataFrame
----

==== [[parquet]] `parquet` method

[source, scala]
----
parquet(path: String): DataFrame
parquet(paths: String*): DataFrame
----

The supported options:

* <<compression, compression>> (default: `snappy`)

New in *2.0.0*: `snappy` is the default Parquet codec. See https://github.com/apache/spark/commit/2f0b882e5c8787b09bedcc8208e6dcc5662dbbab[[SPARK-14482\][SQL\] Change default Parquet codec from gzip to snappy].

[[compression]] The compressions supported:

* `none` or `uncompressed`
* `snappy` - the default codec in Spark *2.0.0*.
* `gzip` - the default codec in Spark before *2.0.0*
* `lzo`

[source, scala]
----
val tokens = Seq("hello", "henry", "and", "harry")
  .zipWithIndex
  .map(_.swap)
  .toDF("id", "token")

val parquetWriter = tokens.write
parquetWriter.option("compression", "none").save("hello-none")

// The exception is mostly for my learning purposes
// so I know where and how to find the trace to the compressions
// Sorry...
scala> parquetWriter.option("compression", "unsupported").save("hello-unsupported")
java.lang.IllegalArgumentException: Codec [unsupported] is not available. Available codecs are uncompressed, gzip, lzo, snappy, none.
  at org.apache.spark.sql.execution.datasources.parquet.ParquetOptions.<init>(ParquetOptions.scala:43)
  at org.apache.spark.sql.execution.datasources.parquet.DefaultSource.prepareWrite(ParquetRelation.scala:77)
  at org.apache.spark.sql.execution.datasources.InsertIntoHadoopFsRelation$$anonfun$run$1$$anonfun$4.apply(InsertIntoHadoopFsRelation.scala:122)
  at org.apache.spark.sql.execution.datasources.InsertIntoHadoopFsRelation$$anonfun$run$1$$anonfun$4.apply(InsertIntoHadoopFsRelation.scala:122)
  at org.apache.spark.sql.execution.datasources.BaseWriterContainer.driverSideSetup(WriterContainer.scala:103)
  at org.apache.spark.sql.execution.datasources.InsertIntoHadoopFsRelation$$anonfun$run$1.apply$mcV$sp(InsertIntoHadoopFsRelation.scala:141)
  at org.apache.spark.sql.execution.datasources.InsertIntoHadoopFsRelation$$anonfun$run$1.apply(InsertIntoHadoopFsRelation.scala:116)
  at org.apache.spark.sql.execution.datasources.InsertIntoHadoopFsRelation$$anonfun$run$1.apply(InsertIntoHadoopFsRelation.scala:116)
  at org.apache.spark.sql.execution.SQLExecution$.withNewExecutionId(SQLExecution.scala:53)
  at org.apache.spark.sql.execution.datasources.InsertIntoHadoopFsRelation.run(InsertIntoHadoopFsRelation.scala:116)
  at org.apache.spark.sql.execution.command.ExecutedCommand.sideEffectResult$lzycompute(commands.scala:61)
  at org.apache.spark.sql.execution.command.ExecutedCommand.sideEffectResult(commands.scala:59)
  at org.apache.spark.sql.execution.command.ExecutedCommand.doExecute(commands.scala:73)
  at org.apache.spark.sql.execution.SparkPlan$$anonfun$execute$1.apply(SparkPlan.scala:118)
  at org.apache.spark.sql.execution.SparkPlan$$anonfun$execute$1.apply(SparkPlan.scala:118)
  at org.apache.spark.sql.execution.SparkPlan$$anonfun$executeQuery$1.apply(SparkPlan.scala:137)
  at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:151)
  at org.apache.spark.sql.execution.SparkPlan.executeQuery(SparkPlan.scala:134)
  at org.apache.spark.sql.execution.SparkPlan.execute(SparkPlan.scala:117)
  at org.apache.spark.sql.execution.QueryExecution.toRdd$lzycompute(QueryExecution.scala:65)
  at org.apache.spark.sql.execution.QueryExecution.toRdd(QueryExecution.scala:65)
  at org.apache.spark.sql.execution.datasources.DataSource.write(DataSource.scala:390)
  at org.apache.spark.sql.DataFrameWriter.save(DataFrameWriter.scala:247)
  at org.apache.spark.sql.DataFrameWriter.save(DataFrameWriter.scala:230)
  ... 48 elided
----

==== [[orc]] `orc` method

[source, scala]
----
orc(path: String): DataFrame
orc(paths: String*): DataFrame
----

*Optimized Row Columnar (ORC)* file format is a highly efficient columnar format to store Hive data with more than 1,000 columns and improve performance. ORC format was introduced in Hive version 0.11 to use and retain the type information from the table definition.

TIP: Read https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC[ORC Files] document to learn about the ORC file format.

==== [[text]] `text` method

`text` method loads a text file.

[source, scala]
----
text(path: String): DataFrame
text(paths: String*): DataFrame
----

===== [[text-example]] Example

[source, scala]
----
val lines: Dataset[String] = spark.read.text("README.md").as[String]

scala> lines.show
+--------------------+
|               value|
+--------------------+
|      # Apache Spark|
|                    |
|Spark is a fast a...|
|high-level APIs i...|
|supports general ...|
|rich set of highe...|
|MLlib for machine...|
|and Spark Streami...|
|                    |
|<http://spark.apa...|
|                    |
|                    |
|## Online Documen...|
|                    |
|You can find the ...|
|guide, on the [pr...|
|and [project wiki...|
|This README file ...|
|                    |
|   ## Building Spark|
+--------------------+
only showing top 20 rows
----

=== [[table]][[creating-dataframes-from-tables]] Loading Table to DataFrame -- `table` Method

[source, scala]
----
table(tableName: String): DataFrame
----

`table` loads the content of the `tableName` table into an untyped link:spark-sql-DataFrame.adoc[DataFrame].


[source, scala]
----
scala> spark.catalog.tableExists("t1")
res1: Boolean = true

// t1 exists in the catalog
// let's load it
val t1 = spark.read.table("t1")
----

NOTE: `table` simply passes the call to link:spark-sql-SparkSession.adoc#table[SparkSession.table] after making sure that a <<schema, user-defined schema>> has not been specified.

=== [[jdbc]] Loading Data From External Table using JDBC Data Source -- `jdbc` Method

[source, scala]
----
jdbc(url: String, table: String, properties: Properties): DataFrame
jdbc(
  url: String,
  table: String,
  predicates: Array[String],
  connectionProperties: Properties): DataFrame
jdbc(
  url: String,
  table: String,
  columnName: String,
  lowerBound: Long,
  upperBound: Long,
  numPartitions: Int,
  connectionProperties: Properties): DataFrame
----

`jdbc` loads data from an external table using the <<spark-sql-jdbc.adoc#, JDBC data source>>.

Internally, `jdbc` creates a link:spark-sql-JDBCOptions.adoc#creating-instance[JDBCOptions] from the input `url`, `table` and `extraOptions` with `connectionProperties`.

`jdbc` then creates one `JDBCPartition` per `predicates`.

In the end, `jdbc` requests the <<sparkSession, SparkSession>> to link:spark-sql-SparkSession.adoc#baseRelationToDataFrame[create a DataFrame] for a link:spark-sql-JDBCRelation.adoc[JDBCRelation] (with `JDBCPartitions` and `JDBCOptions` created earlier).

[NOTE]
====
`jdbc` does not support a custom <<schema, schema>> and throws an `AnalysisException` if defined:

```
User specified schema not supported with `[jdbc]`
```
====

NOTE: `jdbc` method uses `java.util.Properties` (and appears overly Java-centric). Use <<format, format("jdbc")>> instead.

TIP: Review the exercise link:exercises/spark-exercise-dataframe-jdbc-postgresql.adoc[Creating DataFrames from Tables using JDBC and PostgreSQL].

=== [[textFile]] Loading Datasets From Text Files -- `textFile` Method

[source, scala]
----
textFile(path: String): Dataset[String]
textFile(paths: String*): Dataset[String]
----

`textFile` loads one or many text files into a typed link:spark-sql-Dataset.adoc[Dataset[String\]].

[source, scala]
----
import org.apache.spark.sql.SparkSession
val spark: SparkSession = ...

import org.apache.spark.sql.Dataset
val lines: Dataset[String] = spark
  .read
  .textFile("README.md")
----

NOTE: `textFile` are similar to <<text, text>> family of methods in that they both read text files but `text` methods return untyped `DataFrame` while `textFile` return typed `Dataset[String]`.

Internally, `textFile` passes calls on to <<text, text>> method and link:spark-sql-Dataset.adoc#select[selects] the only `value` column before it applies `Encoders.STRING` link:spark-sql-Encoder.adoc[encoder].

=== [[creating-instance]] Creating DataFrameReader Instance

`DataFrameReader` takes the following to be created:

* [[sparkSession]] <<spark-sql-SparkSession.adoc#, SparkSession>>

`DataFrameReader` initializes the <<internal-properties, internal properties>>.

=== [[loadV1Source]] Loading Dataset (Data Source API V1) -- `loadV1Source` Internal Method

[source, scala]
----
loadV1Source(paths: String*): DataFrame
----

`loadV1Source` creates a link:spark-sql-DataSource.adoc#apply[DataSource] and requests it to link:spark-sql-DataSource.adoc#resolveRelation[resolve the underlying relation (as a BaseRelation)].

In the end, `loadV1Source` requests <<sparkSession, SparkSession>> to link:spark-sql-SparkSession.adoc#baseRelationToDataFrame[create a DataFrame from the BaseRelation].

NOTE: `loadV1Source` is used when `DataFrameReader` is requested to <<load, load>> (and the data source is neither of `DataSourceV2` type nor a link:spark-sql-DataSourceReader.adoc[DataSourceReader] could not be created).

=== [[load]] "Loading" Data As DataFrame -- `load` Method

[source, scala]
----
load(): DataFrame
load(path: String): DataFrame
load(paths: String*): DataFrame
----

`load` loads a dataset from a data source (with optional support for multiple `paths`) as an untyped link:spark-sql-DataFrame.adoc[DataFrame].

Internally, `load` link:spark-sql-DataSource.adoc#lookupDataSource[lookupDataSource] for the <<source, source>>. `load` then branches off per its type (i.e. whether it is of `DataSourceV2` marker type or not).

For a "Data Source V2" data source, `load`...FIXME

Otherwise, if the <<source, source>> is not a "Data Source V2" data source, `load` simply <<loadV1Source, loadV1Source>>.

`load` throws a `AnalysisException` when the <<source, source format>> is `hive`.

```
Hive data source can only be used with tables, you can not read files of Hive data source directly.
```

=== [[assertNoSpecifiedSchema]] `assertNoSpecifiedSchema` Internal Method

[source, scala]
----
assertNoSpecifiedSchema(operation: String): Unit
----

`assertNoSpecifiedSchema` throws a `AnalysisException` if the <<userSpecifiedSchema, userSpecifiedSchema>> is defined.

```
User specified schema not supported with `[operation]`
```

NOTE: `assertNoSpecifiedSchema` is used when `DataFrameReader` is requested to load data using <<jdbc, jdbc>>, <<table, table>> and <<textFile, textFile>>.

=== [[verifyColumnNameOfCorruptRecord]] `verifyColumnNameOfCorruptRecord` Internal Method

[source, scala]
----
verifyColumnNameOfCorruptRecord(
  schema: StructType,
  columnNameOfCorruptRecord: String): Unit
----

`verifyColumnNameOfCorruptRecord`...FIXME

NOTE: `verifyColumnNameOfCorruptRecord` is used when `DataFrameReader` is requested to load data using <<json, json>> and <<csv, csv>>.

=== [[source]] Input Data Source -- `source` Internal Property

[source, scala]
----
source: String
----

`source` is the name of the input data source (aka _format_ or _provider_) that will be used to <<load, "load" data (as a DataFrame)>>.

In other words, the <<methods, DataFrameReader fluent API>> is simply to describe the input data source.

The default data source is <<parquet, parquet>> per <<spark-sql-properties.adoc#spark.sql.sources.default, spark.sql.sources.default>> configuration property.

`source` is usually specified using <<format, format>> method.

[IMPORTANT]
====
`source` must not be `hive` or an `AnalysisException` is thrown:

```
Hive data source can only be used with tables, you can not read files of Hive data source directly.
```
====

Once defined explicitly (using <<format, format>> method) or implicitly (<<spark-sql-properties.adoc#spark.sql.sources.default, spark.sql.sources.default>> configuration property), `source` is resolved using <<spark-sql-DataSource.adoc#lookupDataSource, DataSource>> utility.

NOTE: `source` is used exclusively when `DataFrameReader` is requested to <<load, "load" data (as a DataFrame)>> (explicitly or using <<loadV1Source, loadV1Source>>).

=== [[internal-properties]] Internal Properties

[cols="30m,70",options="header",width="100%"]
|===
| Name
| Description

| extraOptions
a| [[extraOptions]]

Used when...FIXME

| userSpecifiedSchema
| [[userSpecifiedSchema]] Optional *used-specified schema* (default: `None`, i.e. undefined)

Set when `DataFrameReader` is requested to <<schema, set a schema>>, <<load, load a data from an external data source>>, <<loadV1Source, loadV1Source>> (when creating a link:spark-sql-DataSource.adoc#userSpecifiedSchema[DataSource]), and load a data using <<json, json>> and <<csv, csv>> file formats

Used when `DataFrameReader` is requested to <<assertNoSpecifiedSchema, assertNoSpecifiedSchema>> (while loading data using <<jdbc, jdbc>>, <<table, table>> and <<textFile, textFile>>)

|===
