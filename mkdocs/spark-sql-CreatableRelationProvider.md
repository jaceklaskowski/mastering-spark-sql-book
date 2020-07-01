title: CreatableRelationProvider

# CreatableRelationProvider -- Data Sources That Write Rows Differently Per Save Mode

`CreatableRelationProvider` is the <<contract, abstraction>> of <<implementations, data source providers>> that can <<createRelation, write the rows of a structured query (a DataFrame) differently per save mode>>.

[[contract]]
.CreatableRelationProvider Contract
[cols="30m,70",options="header",width="100%"]
|===
| Method
| Description

| createRelation
a| [[createRelation]]

[source, scala]
----
createRelation(
  sqlContext: SQLContext,
  mode: SaveMode,
  parameters: Map[String, String],
  data: DataFrame): BaseRelation
----

Creates a <<spark-sql-BaseRelation.adoc#, BaseRelation>> that represents the rows of a structured query (a DataFrame) saved to an external data source (per <<spark-sql-DataFrameWriter.adoc#SaveMode, SaveMode>>)

The save mode specifies what should happen when the target relation (destination) already exists.

Used when:

* `DataSource` is requested to <<spark-sql-DataSource.adoc#writeAndRead, write data to a data source per save mode followed by reading the rows back>> (when <<spark-sql-LogicalPlan-CreateDataSourceTableAsSelectCommand.adoc#, CreateDataSourceTableAsSelectCommand>> logical command is executed)

* <<spark-sql-LogicalPlan-SaveIntoDataSourceCommand.adoc#, SaveIntoDataSourceCommand>> logical command is executed

|===

[[implementations]]
.CreatableRelationProviders
[cols="30,70",options="header",width="100%"]
|===
| CreatableRelationProvider
| Description

| <<spark-sql-ConsoleSinkProvider.adoc#, ConsoleSinkProvider>>
| [[ConsoleSinkProvider]] Data source provider for <<spark-sql-console.adoc#, Console data source>>

| <<spark-sql-JdbcRelationProvider.adoc#, JdbcRelationProvider>>
| [[JdbcRelationProvider]] Data source provider for <<spark-sql-jdbc.adoc#, JDBC data source>>

| <<spark-sql-KafkaSourceProvider.adoc#, KafkaSourceProvider>>
| [[KafkaSourceProvider]] Data source provider for <<spark-sql-kafka.adoc#, Kafka data source>>

|===
