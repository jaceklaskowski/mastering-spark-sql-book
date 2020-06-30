title: StructField

# StructField -- Single Field in StructType

[[creating-instance]]
`StructField` describes a single field in a <<spark-sql-StructType.adoc#, StructType>> with the following:

* [[name]] Name
* [[dataType]] <<spark-sql-DataType.adoc#, DataType>>
* [[nullable]] `nullable` flag (enabled by default)
* [[metadata]] `Metadata` (empty by default)

A comment is part of metadata under `comment` key and is used to build a Hive column or when describing a table.

[source, scala]
----
scala> schemaTyped("a").getComment
res0: Option[String] = None

scala> schemaTyped("a").withComment("this is a comment").getComment
res1: Option[String] = Some(this is a comment)
----

As of Spark 2.4.0, `StructField` can be converted to DDL format using <<toDDL, toDDL>> method.

.Example: Using StructField.toDDL
[source, scala]
----
import org.apache.spark.sql.types.MetadataBuilder
val metadata = new MetadataBuilder()
  .putString("comment", "this is a comment")
  .build
import org.apache.spark.sql.types.{LongType, StructField}
val f = new StructField(name = "id", dataType = LongType, nullable = false, metadata)
scala> println(f.toDDL)
`id` BIGINT COMMENT 'this is a comment'
----

=== [[toDDL]] Converting to DDL Format -- `toDDL` Method

[source, scala]
----
toDDL: String
----

`toDDL` gives a text in the format:

```
[quoted name] [dataType][optional comment]
```

[NOTE]
====
`toDDL` is used when:

* `StructType` is requested to <<spark-sql-StructType.adoc#toDDL, convert itself to DDL format>>

* <<spark-sql-LogicalPlan-ShowCreateTableCommand.adoc#, ShowCreateTableCommand>> logical command is executed (and <<spark-sql-LogicalPlan-ShowCreateTableCommand.adoc#showHiveTableHeader, showHiveTableHeader>>, <<spark-sql-LogicalPlan-ShowCreateTableCommand.adoc#showHiveTableNonDataColumns, showHiveTableNonDataColumns>>, <<spark-sql-LogicalPlan-ShowCreateTableCommand.adoc#showDataSourceTableDataColumns, showDataSourceTableDataColumns>>)
====
