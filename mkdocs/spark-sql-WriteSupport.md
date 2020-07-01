title: WriteSupport

# WriteSupport -- "Writable" Data Sources

`WriteSupport` is the <<contract, abstraction>> of <<implementations, "writable" data sources>> in the <<spark-sql-data-source-api-v2.adoc#, Data Source API V2>> that can <<createWriter, create a DataSourceWriter>> for writing data out.

[[contract]]
[[createWriter]]
`WriteSupport` defines a single `createWriter` method that creates an optional <<spark-sql-DataSourceWriter.adoc#, DataSourceWriter>> per <<spark-sql-DataFrameWriter.adoc#SaveMode, SaveMode>> (and can create no `DataSourceWriter` when not needed per mode)

[source, java]
----
Optional<DataSourceWriter> createWriter(
  String writeUUID,
  StructType schema,
  SaveMode mode,
  DataSourceOptions options)
----

`createWriter` is used when:

* `DataFrameWriter` is requested to <<spark-sql-DataFrameWriter.adoc#save, save a DataFrame to a data source>> (for <<spark-sql-DataSourceV2.adoc#, DataSourceV2>> data sources with <<WriteSupport, WriteSupport>>)

* `DataSourceV2Relation` leaf logical operator is requested to <<spark-sql-LogicalPlan-DataSourceV2Relation.adoc#newWriter, create a DataSourceWriter>> (indirectly via <<spark-sql-LogicalPlan-DataSourceV2Relation.adoc#createWriter, createWriter>> implicit method)

[source, scala]
----
// FIXME: Demo
// df.write.format(...) that is DataSourceV2 and WriteSupport
----

[[implementations]]
NOTE: There are no production implementations of the <<contract, WriteSupport Contract>> in Spark SQL yet.
