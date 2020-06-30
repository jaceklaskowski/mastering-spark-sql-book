title: implicits Object

# implicits Object -- Implicits Conversions

`implicits` object gives <<methods, implicit conversions>> for converting Scala objects (incl. RDDs) into a `Dataset`, `DataFrame`, `Columns` or supporting such conversions (through <<Encoders, Encoders>>).

[[methods]]
.implicits API
[cols="1,2",options="header",width="100%"]
|===
| Name
| Description

| `localSeqToDatasetHolder`
a| [[localSeqToDatasetHolder]] Creates a <<DatasetHolder, DatasetHolder>> with the input `Seq[T]` converted to a `Dataset[T]` (using <<spark-sql-SparkSession.adoc#createDataset, SparkSession.createDataset>>).

[source, scala]
----
implicit def localSeqToDatasetHolder[T : Encoder](s: Seq[T]): DatasetHolder[T]
----

| Encoders
| [[Encoders]] <<spark-sql-Encoders.adoc#, Encoders>> for primitive and object types in Scala and Java (aka _boxed types_)

| `StringToColumn`
a| [[StringToColumn]] Converts `$"name"` into a <<spark-sql-Column.adoc#, Column>>

[source, scala]
----
implicit class StringToColumn(val sc: StringContext)
----

| `rddToDatasetHolder`
a| [[rddToDatasetHolder]]

[source, scala]
----
implicit def rddToDatasetHolder[T : Encoder](rdd: RDD[T]): DatasetHolder[T]
----

| `symbolToColumn`
a| [[symbolToColumn]]

[source, scala]
----
implicit def symbolToColumn(s: Symbol): ColumnName
----
|===

`implicits` object is defined inside <<spark-sql-SparkSession.adoc#implicits, SparkSession>> and hence requires that you build a <<spark-sql-SparkSession.adoc#builder, SparkSession>> instance first before importing `implicits` conversions.

[source, scala]
----
import org.apache.spark.sql.SparkSession
val spark: SparkSession = ...
import spark.implicits._

scala> val ds = Seq("I am a shiny Dataset!").toDS
ds: org.apache.spark.sql.Dataset[String] = [value: string]

scala> val df = Seq("I am an old grumpy DataFrame!").toDF
df: org.apache.spark.sql.DataFrame = [value: string]

scala> val df = Seq("I am an old grumpy DataFrame with text column!").toDF("text")
df: org.apache.spark.sql.DataFrame = [text: string]

val rdd = sc.parallelize(Seq("hello, I'm a very low-level RDD"))
scala> val ds = rdd.toDS
ds: org.apache.spark.sql.Dataset[String] = [value: string]
----

[TIP]
====
In Scala REPL-based environments, e.g. `spark-shell`, use `:imports` to know what imports are in scope.
====

[source, scala]
----
scala> :help imports

show import history, identifying sources of names

scala> :imports
 1) import org.apache.spark.SparkContext._ (69 terms, 1 are implicit)
 2) import spark.implicits._       (1 types, 67 terms, 37 are implicit)
 3) import spark.sql               (1 terms)
 4) import org.apache.spark.sql.functions._ (354 terms)
----

`implicits` object extends `SQLImplicits` abstract class.

=== [[DatasetHolder]][[toDS]][[toDF]] `DatasetHolder` Scala Case Class

[[ds]]
[[creating-instance]]
`DatasetHolder` is a Scala case class that, when created, takes a `Dataset[T]`.

`DatasetHolder` is <<creating-instance, created>> (implicitly) when <<rddToDatasetHolder, rddToDatasetHolder>> and <<localSeqToDatasetHolder, localSeqToDatasetHolder>> implicit conversions are used.

`DatasetHolder` has `toDS` and `toDF` methods that simply return the <<ds, Dataset[T]>> (it was created with) or a `DataFrame` (using <<spark-sql-dataset-operators.adoc#toDF, Dataset.toDF>> operator), respectively.

[source, scala]
----
toDS(): Dataset[T]
toDF(): DataFrame
toDF(colNames: String*): DataFrame
----
