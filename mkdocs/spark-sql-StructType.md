title: StructType

# StructType -- Data Type for Schema Definition

[[fields]]
[[creating-instance]]
`StructType` is a built-in <<spark-sql-DataType.adoc#, data type>> that is a collection of <<spark-sql-StructField.adoc#, StructFields>>.

`StructType` is used to define a schema or its part.

You can compare two `StructType` instances to see whether they are equal.

[source, scala]
----
import org.apache.spark.sql.types.StructType

val schemaUntyped = new StructType()
  .add("a", "int")
  .add("b", "string")

import org.apache.spark.sql.types.{IntegerType, StringType}
val schemaTyped = new StructType()
  .add("a", IntegerType)
  .add("b", StringType)

scala> schemaUntyped == schemaTyped
res0: Boolean = true
----

`StructType` <<sql, presents itself>> as `<struct>` or `STRUCT` in query plans or SQL.

[NOTE]
====
`StructType` is a `Seq[StructField]` and therefore all things `Seq` apply equally here.

[source, scala]
----
scala> schemaTyped.foreach(println)
StructField(a,IntegerType,true)
StructField(b,StringType,true)
----

Read the official documentation of Scala's http://www.scala-lang.org/api/current/scala/collection/Seq.html[scala.collection.Seq].
====

As of Spark 2.4.0, `StructType` can be converted to DDL format using <<toDDL, toDDL>> method.

.Example: Using StructType.toDDL
[source, scala]
----
// Generating a schema from a case class
// Because we're all properly lazy
case class Person(id: Long, name: String)
import org.apache.spark.sql.Encoders
val schema = Encoders.product[Person].schema
scala> println(schema.toDDL)
`id` BIGINT,`name` STRING
----

=== [[fromAttributes]] `fromAttributes` Method

[source, scala]
----
fromAttributes(attributes: Seq[Attribute]): StructType
----

`fromAttributes`...FIXME

NOTE: `fromAttributes` is used when...FIXME

=== [[toAttributes]] `toAttributes` Method

[source, scala]
----
toAttributes: Seq[AttributeReference]
----

`toAttributes`...FIXME

NOTE: `toAttributes` is used when...FIXME

=== [[add]] Adding Fields to Schema -- `add` Method

You can add a new `StructField` to your `StructType`. There are different variants of `add` method that all make for a new `StructType` with the field added.

[source, scala]
----
add(field: StructField): StructType
add(name: String, dataType: DataType): StructType
add(name: String, dataType: DataType, nullable: Boolean): StructType
add(
  name: String,
  dataType: DataType,
  nullable: Boolean,
  metadata: Metadata): StructType
add(
  name: String,
  dataType: DataType,
  nullable: Boolean,
  comment: String): StructType
add(name: String, dataType: String): StructType
add(name: String, dataType: String, nullable: Boolean): StructType
add(
  name: String,
  dataType: String,
  nullable: Boolean,
  metadata: Metadata): StructType
add(
  name: String,
  dataType: String,
  nullable: Boolean,
  comment: String): StructType
----

=== [[sql]][[catalogString]][[simpleString]] DataType Name Conversions

[source, scala]
----
simpleString: String
catalogString: String
sql: String
----

`StructType` as a custom `DataType` is used in query plans or SQL. It can present itself using `simpleString`, `catalogString` or `sql` (see link:spark-sql-DataType.adoc#contract[DataType Contract]).

[source, scala]
----
scala> schemaTyped.simpleString
res0: String = struct<a:int,b:string>

scala> schemaTyped.catalogString
res1: String = struct<a:int,b:string>

scala> schemaTyped.sql
res2: String = STRUCT<`a`: INT, `b`: STRING>
----

=== [[apply]] Accessing StructField -- `apply` Method

[source, scala]
----
apply(name: String): StructField
----

`StructType` defines its own `apply` method that gives you an easy access to a `StructField` by name.

[source, scala]
----
scala> schemaTyped.printTreeString
root
 |-- a: integer (nullable = true)
 |-- b: string (nullable = true)

scala> schemaTyped("a")
res4: org.apache.spark.sql.types.StructField = StructField(a,IntegerType,true)
----

=== [[apply-seq]] Creating StructType from Existing StructType -- `apply` Method

[source, scala]
----
apply(names: Set[String]): StructType
----

This variant of `apply` lets you create a `StructType` out of an existing `StructType` with the `names` only.

[source, scala]
----
scala> schemaTyped(names = Set("a"))
res0: org.apache.spark.sql.types.StructType = StructType(StructField(a,IntegerType,true))
----

It will throw an `IllegalArgumentException` exception when a field could not be found.

[source, scala]
----
scala> schemaTyped(names = Set("a", "c"))
java.lang.IllegalArgumentException: Field c does not exist.
  at org.apache.spark.sql.types.StructType.apply(StructType.scala:275)
  ... 48 elided
----

=== [[printTreeString]] Displaying Schema As Tree -- `printTreeString` Method

[source, scala]
----
printTreeString(): Unit
----

`printTreeString` prints out the schema to standard output.

[source, scala]
----
scala> schemaTyped.printTreeString
root
 |-- a: integer (nullable = true)
 |-- b: string (nullable = true)
----

Internally, it uses `treeString` method to build the tree and then `println` it.

=== [[fromDDL]] Creating StructType For DDL-Formatted Text -- `fromDDL` Object Method

[source, scala]
----
fromDDL(ddl: String): StructType
----

`fromDDL`...FIXME

NOTE: `fromDDL` is used when...FIXME

=== [[toDDL]] Converting to DDL Format -- `toDDL` Method

[source, scala]
----
toDDL: String
----

`toDDL` converts all the <<fields, fields>> to DDL format and concatenates them using the comma (`,`).
