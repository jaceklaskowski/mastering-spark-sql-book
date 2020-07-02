title: DataWritingCommand

# DataWritingCommand -- Logical Commands That Write Query Data

`DataWritingCommand` is an <<contract, extension>> of the <<spark-sql-LogicalPlan-Command.adoc#, Command contract>> for <<implementations, logical commands>> that write the result of executing <<query, query>> (_query data_) to a relation when <<run, executed>>.

`DataWritingCommand` is resolved to a <<spark-sql-SparkPlan-DataWritingCommandExec.adoc#, DataWritingCommandExec>> physical operator when <<spark-sql-SparkStrategy-BasicOperators.adoc#, BasicOperators>> execution planning strategy is executed (i.e. plan a <<spark-sql-LogicalPlan.adoc#, logical plan>> to a <<spark-sql-SparkPlan.adoc#, physical plan>>).

[[contract]]
.DataWritingCommand Contract
[cols="1m,2",options="header",width="100%"]
|===
| Property
| Description

| outputColumnNames
a| [[outputColumnNames]]

[source, scala]
----
outputColumnNames: Seq[String]
----

The output column names of the <<query, analyzed input query plan>>

Used when `DataWritingCommand` is requested for the <<outputColumns, outputColumns>>

| query
a| [[query]]

[source, scala]
----
query: LogicalPlan
----

The analyzed <<spark-sql-LogicalPlan.adoc#, logical query plan>> representing the data to write (i.e. whose result will be inserted into a relation)

Used when `DataWritingCommand` is requested for the <<children, child nodes>> and <<outputColumns, outputColumns>>.

| run
a| [[run]]

[source, scala]
----
run(
  sparkSession: SparkSession,
  child: SparkPlan): Seq[Row]
----

Executes the command to write query data (the result of executing link:spark-sql-SparkPlan.adoc[structured query])

Used when:

* `DataWritingCommandExec` physical operator is requested for the <<spark-sql-SparkPlan-DataWritingCommandExec.adoc#sideEffectResult, sideEffectResult>>

* `DataSource` is requested to <<spark-sql-DataSource.adoc#writeAndRead, write data to a data source per save mode followed by reading rows back>> (when <<spark-sql-LogicalPlan-CreateDataSourceTableAsSelectCommand.adoc#run, CreateDataSourceTableAsSelectCommand>> logical command is executed)
|===

[[children]]
When requested for the <<spark-sql-LogicalPlan-Command.adoc#children, child nodes>>, `DataWritingCommand` simply returns the <<query, logical query plan>>.

`DataWritingCommand` defines custom <<metrics, performance metrics>>.

[[metrics]]
.DataWritingCommand's Performance Metrics
[cols="1m,2,2",options="header",width="100%"]
|===
| Key
| Name (in web UI)
| Description

| numFiles
| number of written files
| [[numFiles]]

| numOutputBytes
| bytes of written output
| [[numOutputBytes]]

| numOutputRows
| number of output rows
| [[numOutputRows]]

| numParts
| number of dynamic part
| [[numParts]]
|===

The <<metrics, performance metrics>> are used when:

* `DataWritingCommand` is requested for the <<basicWriteJobStatsTracker, BasicWriteJobStatsTracker>>

* `DataWritingCommandExec` physical operator is requested for the <<spark-sql-SparkPlan-DataWritingCommandExec.adoc#metrics, metrics>>

[[extensions]]
.DataWritingCommands (Direct Implementations and Extensions Only)
[cols="1,2",options="header",width="100%"]
|===
| DataWritingCommand
| Description

| link:spark-sql-LogicalPlan-CreateDataSourceTableAsSelectCommand.adoc[CreateDataSourceTableAsSelectCommand]
| [[CreateDataSourceTableAsSelectCommand]]

| link:hive/CreateHiveTableAsSelectCommand.adoc[CreateHiveTableAsSelectCommand]
| [[CreateHiveTableAsSelectCommand]]

| link:spark-sql-LogicalPlan-InsertIntoHadoopFsRelationCommand.adoc[InsertIntoHadoopFsRelationCommand]
| [[InsertIntoHadoopFsRelationCommand]]

| link:hive/SaveAsHiveFile.adoc[SaveAsHiveFile]
| [[SaveAsHiveFile]] Commands that write query result as Hive files (i.e. link:hive/InsertIntoHiveDirCommand.adoc[InsertIntoHiveDirCommand] and link:hive/InsertIntoHiveTable.adoc[InsertIntoHiveTable])

|===

=== [[basicWriteJobStatsTracker]] `basicWriteJobStatsTracker` Method

[source, scala]
----
basicWriteJobStatsTracker(hadoopConf: Configuration): BasicWriteJobStatsTracker
----

`basicWriteJobStatsTracker` simply creates and returns a new <<spark-sql-BasicWriteJobStatsTracker.adoc#, BasicWriteJobStatsTracker>> (with the given Hadoop `Configuration` and the <<metrics, metrics>>).

[NOTE]
====
`basicWriteJobStatsTracker` is used when:

* `SaveAsHiveFile` is requested to <<hive/SaveAsHiveFile.adoc#saveAsHiveFile, saveAsHiveFile>> (when link:hive/InsertIntoHiveDirCommand.adoc[InsertIntoHiveDirCommand] and link:hive/InsertIntoHiveTable.adoc[InsertIntoHiveTable] logical commands are executed)

* <<spark-sql-LogicalPlan-InsertIntoHadoopFsRelationCommand.adoc#, InsertIntoHadoopFsRelationCommand>> logical command is executed
====

=== [[outputColumns]] Output Columns -- `outputColumns` Method

[source, scala]
----
outputColumns: Seq[Attribute]
----

`outputColumns`...FIXME

[NOTE]
====
`outputColumns` is used when:

* link:hive/CreateHiveTableAsSelectCommand.adoc[CreateHiveTableAsSelectCommand], link:hive/InsertIntoHiveDirCommand.adoc[InsertIntoHiveDirCommand] and <<spark-sql-LogicalPlan-InsertIntoHadoopFsRelationCommand.adoc#, InsertIntoHadoopFsRelationCommand>> logical commands are executed

* `SaveAsHiveFile` is requested to <<hive/SaveAsHiveFile.adoc#saveAsHiveFile, saveAsHiveFile>>
====
