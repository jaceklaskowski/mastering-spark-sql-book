title: ReadSupport

# ReadSupport -- "Readable" Data Sources

`ReadSupport` is the <<contract, abstraction>> of <<implementations, "readable" data sources>> in the <<spark-sql-data-source-api-v2.adoc#, Data Source API V2>> that can <<createReader, create a DataSourceReader>> for reading data (_data scan_).

[[contract]]
[[createReader]]
`ReadSupport` defines a single `createReader` method that creates a <<spark-sql-DataSourceReader.adoc#, DataSourceReader>>.

[source, java]
----
DataSourceReader createReader(DataSourceOptions options)
DataSourceReader createReader(StructType schema, DataSourceOptions options)
----

`createReader` is used when `DataSourceV2Relation` leaf logical operator is <<spark-sql-LogicalPlan-DataSourceV2Relation.adoc#create, created>> (when `DataFrameReader` is requested to <<spark-sql-DataFrameReader.adoc#load, "load" data (as a DataFrame)>> from a data source with <<spark-sql-ReadSupport.adoc#, ReadSupport>>).

[source, scala]
----
// FIXME: Demo
// spark.read.format(...) that is DataSourceV2 and ReadSupport
// DataFrameReader.load() creates a DataFrame with a DataSourceV2Relation operator
----

Internally, `ReadSupport` is accessed implicitly when `DataSourceV2Relation` logical operator is requested to <<spark-sql-LogicalPlan-DataSourceV2Relation.adoc#newReader, create a DataSourceReader>>.

[[implementations]]
NOTE: There are no production implementations of the <<contract, ReadSupport Contract>> in Spark SQL yet.
