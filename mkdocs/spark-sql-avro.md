# Avro Data Source

Spark SQL supports structured queries over <<spark-sql-AvroFileFormat.adoc#, Avro files>> as well as in <<functions, columns>> (in a `DataFrame`).

[NOTE]
====
https://avro.apache.org/[Apache Avro] is a data serialization format and provides the following features:

* Language-independent (with language bindings for popular programming languages, e.g. Java, Python)
* Rich data structures
* A compact, fast, binary data format (encoding)
* A container file for sequences of Avro data (aka _Avro data files_)
* Remote procedure call (RPC)
* Optional code generation (optimization) to read or write data files, and implement RPC protocols
====

Avro data source is provided by the `spark-avro` external module. You should include it as a dependency in your Spark application (e.g. `spark-submit --packages` or in `build.sbt`).

```
org.apache.spark:spark-avro_2.12:2.4.0
```

The following shows how to include the `spark-avro` module in a `spark-shell` session.

```
$ ./bin/spark-shell --packages org.apache.spark:spark-avro_2.12:2.4.0
```

[[functions]]
.Functions for Avro
[cols="1,2",options="header",width="100%"]
|===
| Name
| Description

| <<from_avro-internals, from_avro>>
a| [[from_avro]]

[source, scala]
----
from_avro(data: Column, jsonFormatSchema: String): Column
----

Parses an Avro-encoded binary column and converts to a Catalyst value per JSON-encoded Avro schema

| <<to_avro-internals, to_avro>>
a| [[to_avro]]

[source, scala]
----
to_avro(data: Column): Column
----

Converts a column to an Avro-encoded binary column
|===

After the module is loaded, you should import the `org.apache.spark.sql.avro` package to have the <<from_avro, from_avro>> and <<to_avro, to_avro>> functions available.

[source, scala]
----
import org.apache.spark.sql.avro._
----

=== [[to_avro-internals]] Converting Column to Avro-Encoded Binary Column -- `to_avro` Method

[source, scala]
----
to_avro(data: Column): Column
----

`to_avro` creates a <<spark-sql-Column.adoc#, Column>> with the <<spark-sql-Expression-CatalystDataToAvro.adoc#, CatalystDataToAvro>> unary expression (with the <<spark-sql-Column.adoc#expr, Catalyst expression>> of the given `data` column).

[source, scala]
----
import org.apache.spark.sql.avro._
val q = spark.range(1).withColumn("to_avro_id", to_avro('id))
scala> q.show
+---+----------+
| id|to_avro_id|
+---+----------+
|  0|      [00]|
+---+----------+

val logicalPlan = q.queryExecution.logical
scala> println(logicalPlan.numberedTreeString)
00 'Project [id#33L, catalystdatatoavro('id) AS to_avro_id#35]
01 +- Range (0, 1, step=1, splits=Some(8))

import org.apache.spark.sql.avro.CatalystDataToAvro
// Let's use QueryExecution.analyzed instead
// https://issues.apache.org/jira/browse/SPARK-26063
val analyzedPlan = q.queryExecution.analyzed
val toAvroExpr = analyzedPlan.expressions.drop(1).head.children.head.asInstanceOf[CatalystDataToAvro]
scala> println(toAvroExpr.sql)
to_avro(`id`, bigint)
----

=== [[from_avro-internals]] Converting Avro-Encoded Column to Catalyst Value -- `from_avro` Method

[source, scala]
----
from_avro(data: Column, jsonFormatSchema: String): Column
----

`from_avro` creates a <<spark-sql-Column.adoc#, Column>> with the <<spark-sql-Expression-AvroDataToCatalyst.adoc#, AvroDataToCatalyst>> unary expression (with the <<spark-sql-Column.adoc#expr, Catalyst expression>> of the given `data` column and the `jsonFormatSchema` JSON-encoded schema).

[source, scala]
----
import org.apache.spark.sql.avro._
val data = spark.range(1).withColumn("to_avro_id", to_avro('id))

// Use from_avro to decode to_avro-encoded id column
val jsonFormatSchema = s"""
  |{
  |  "type": "long",
  |  "name": "id"
  |}
""".stripMargin
val q = data.select(from_avro('to_avro_id, jsonFormatSchema) as "id_from_avro")
scala> q.show
+------------+
|id_from_avro|
+------------+
|           0|
+------------+

val logicalPlan = q.queryExecution.logical
scala> println(logicalPlan.numberedTreeString)
00 'Project [avrodatatocatalyst('to_avro_id,
01 {
02   "type": "long",
03   "name": "id"
04 }
05 ) AS id_from_avro#77]
06 +- Project [id#66L, catalystdatatoavro(id#66L) AS to_avro_id#68]
07    +- Range (0, 1, step=1, splits=Some(8))

import org.apache.spark.sql.avro.AvroDataToCatalyst
// Let's use QueryExecution.analyzed instead
// https://issues.apache.org/jira/browse/SPARK-26063
val analyzedPlan = q.queryExecution.analyzed
val fromAvroExpr = analyzedPlan.expressions.head.children.head.asInstanceOf[AvroDataToCatalyst]
scala> println(fromAvroExpr.sql)
from_avro(`to_avro_id`, bigint)
----
