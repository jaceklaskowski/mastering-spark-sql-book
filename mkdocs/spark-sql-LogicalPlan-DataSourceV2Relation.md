title: DataSourceV2Relation

# DataSourceV2Relation Leaf Logical Operator

`DataSourceV2Relation` is a <<spark-sql-LogicalPlan-LeafNode.adoc#, leaf logical operator>> that represents a data scan (_data reading_) or data writing in the <<spark-sql-data-source-api-v2.adoc#, Data Source API V2>>.

`DataSourceV2Relation` is <<creating-instance, created>> (indirectly via <<create, create>> helper method) exclusively when `DataFrameReader` is requested to <<spark-sql-DataFrameReader.adoc#load, "load" data (as a DataFrame)>> (from a data source with <<spark-sql-ReadSupport.adoc#, ReadSupport>>).

[[creating-instance]]
`DataSourceV2Relation` takes the following to be created:

* [[source]] <<spark-sql-DataSourceV2.adoc#, DataSourceV2>>
* [[output]] Output <<spark-sql-Expression-AttributeReference.adoc#, attributes>> (`Seq[AttributeReference]`)
* [[options]] Options (`Map[String, String]`)
* [[tableIdent]] Optional `TableIdentifier` (default: undefined, i.e. `None`)
* [[userSpecifiedSchema]] User-defined <<spark-sql-StructType.adoc#, schema>> (default: undefined, i.e. `None`)

When used to represent a data scan (_data reading_), `DataSourceV2Relation` is planned (_translated_) to a <<spark-sql-SparkPlan-ProjectExec.adoc#, ProjectExec>> with a <<spark-sql-SparkPlan-DataSourceV2ScanExec.adoc#, DataSourceV2ScanExec>> physical operator (possibly under the <<spark-sql-SparkPlan-FilterExec.adoc#, FilterExec>> operator) when <<spark-sql-SparkStrategy-DataSourceV2Strategy.adoc#, DataSourceV2Strategy>> execution planning strategy is requested to <<spark-sql-SparkStrategy-DataSourceV2Strategy.adoc#apply-DataSourceV2Relation, plan a logical plan>>.

When used to represent a data write (with <<spark-sql-LogicalPlan-AppendData.adoc#, AppendData>> logical operator), `DataSourceV2Relation` is planned (_translated_) to a <<spark-sql-SparkPlan-WriteToDataSourceV2Exec.adoc#, WriteToDataSourceV2Exec>> physical operator (with the <<newWriter, DataSourceWriter>>) when <<spark-sql-SparkStrategy-DataSourceV2Strategy.adoc#, DataSourceV2Strategy>> execution planning strategy is requested to <<spark-sql-SparkStrategy-DataSourceV2Strategy.adoc#apply-AppendData, plan a logical plan>>.

`DataSourceV2Relation` object defines a `SourceHelpers` implicit class that extends <<spark-sql-DataSourceV2.adoc#, DataSourceV2>> instances with the additional <<extension-methods, extension methods>>.

=== [[create]] Creating DataSourceV2Relation Instance -- `create` Factory Method

[source, scala]
----
create(
  source: DataSourceV2,
  options: Map[String, String],
  tableIdent: Option[TableIdentifier] = None,
  userSpecifiedSchema: Option[StructType] = None): DataSourceV2Relation
----

`create` requests the given <<spark-sql-DataSourceV2.adoc#, DataSourceV2>> to create a <<spark-sql-DataSourceReader.adoc#, DataSourceReader>> (with the given options and user-specified schema).

`create` finds the table in the given options unless the optional `tableIdent` is defined.

In the end, `create` <<creating-instance, creates a DataSourceV2Relation>>.

NOTE: `create` is used exclusively when `DataFrameReader` is requested to <<spark-sql-DataFrameReader.adoc#load, "load" data (as a DataFrame)>> (from a data source with <<spark-sql-ReadSupport.adoc#, ReadSupport>>).

=== [[computeStats]] Computing Statistics -- `computeStats` Method

[source, scala]
----
computeStats(): Statistics
----

NOTE: `computeStats` is part of the <<spark-sql-LogicalPlan-LeafNode.adoc#computeStats, LeafNode Contract>> to compute a <<spark-sql-Statistics.adoc#, Statistics>>.

`computeStats`...FIXME

=== [[newReader]] Creating DataSourceReader -- `newReader` Method

[source, scala]
----
newReader(): DataSourceReader
----

`newReader` simply requests (_delegates to_) the <<source, DataSourceV2>> to <<createReader, create a DataSourceReader>>.

NOTE: `DataSourceV2Relation` object defines the <<SourceHelpers, SourceHelpers>> implicit class to "extend" the marker <<spark-sql-DataSourceV2.adoc#, DataSourceV2>> type with the method to <<createReader, create a DataSourceReader>>.

[NOTE]
====
`newReader` is used when:

* `DataSourceV2Relation` is requested to <<computeStats, computeStats>>

* `DataSourceV2Strategy` execution planning strategy is requested to <<spark-sql-SparkStrategy-DataSourceV2Strategy.adoc#apply-DataSourceV2Relation, plan a DataSourceV2Relation logical operator>>
====

=== [[newWriter]] Creating DataSourceWriter -- `newWriter` Method

[source, scala]
----
newWriter(): DataSourceWriter
----

`newWriter` simply requests (_delegates to_) the <<source, DataSourceV2>> to <<createWriter, create a DataSourceWriter>>.

NOTE: `DataSourceV2Relation` object defines the <<SourceHelpers, SourceHelpers>> implicit class to "extend" the marker <<spark-sql-DataSourceV2.adoc#, DataSourceV2>> type with the method to <<createWriter, create a DataSourceWriter>>.

NOTE: `newWriter` is used exclusively when `DataSourceV2Strategy` execution planning strategy is requested to <<spark-sql-SparkStrategy-DataSourceV2Strategy.adoc#apply-AppendData, plan an AppendData logical operator>>.

=== [[SourceHelpers]] SourceHelpers Implicit Class

`DataSourceV2Relation` object defines a `SourceHelpers` implicit class that extends <<spark-sql-DataSourceV2.adoc#, DataSourceV2>> instances with the additional <<extension-methods, extension methods>>.

[[extension-methods]]
.SourceHelpers' Extension Methods
[cols="1m,3",options="header",width="100%"]
|===
| Method
| Description

| asReadSupport
a| [[asReadSupport]]

[source, scala]
----
asReadSupport: ReadSupport
----

Used exclusively for <<createReader, createReader>> implicit method

| asWriteSupport
a| [[asWriteSupport]]

[source, scala]
----
asWriteSupport: WriteSupport
----

Used when...FIXME

| name
a| [[name]]

[source, scala]
----
name: String
----

Used when...FIXME

| createReader
a| [[createReader]]

[source, scala]
----
createReader(
  options: Map[String, String],
  userSpecifiedSchema: Option[StructType]): DataSourceReader
----

Used when:

* `DataSourceV2Relation` logical operator is requested to <<newReader, create a DataSourceReader>>

* `DataSourceV2Relation` factory object is requested to <<create, create a DataSourceV2Relation>> (when `DataFrameReader` is requested to <<spark-sql-DataFrameReader.adoc#load, "load" data (as a DataFrame)>> from a data source with <<spark-sql-ReadSupport.adoc#, ReadSupport>>)

| createWriter
a| [[createWriter]]

[source, scala]
----
createWriter(
  options: Map[String, String],
  schema: StructType): DataSourceWriter
----

Creates a <<spark-sql-DataSourceWriter.adoc#, DataSourceWriter>>

Used when...FIXME

|===

TIP: Read up on https://docs.scala-lang.org/overviews/core/implicit-classes.html[implicit classes] in the https://docs.scala-lang.org/[official documentation of the Scala programming language].
