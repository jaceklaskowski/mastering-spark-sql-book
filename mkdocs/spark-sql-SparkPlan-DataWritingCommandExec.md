title: DataWritingCommandExec

# DataWritingCommandExec Physical Operator

`DataWritingCommandExec` is a <<spark-sql-SparkPlan.adoc#, physical operator>> that is the execution environment for a <<cmd, DataWritingCommand>> logical command at <<doExecute, execution time>>.

`DataWritingCommandExec` is <<creating-instance, created>> exclusively when <<spark-sql-SparkStrategy-BasicOperators.adoc#, BasicOperators>> execution planning strategy is requested to plan a <<spark-sql-LogicalPlan-DataWritingCommand.adoc#, DataWritingCommand>> logical command.

[[metrics]]
When requested for <<spark-sql-SparkPlan.adoc#metrics, performance metrics>>, `DataWritingCommandExec` simply requests the <<cmd, DataWritingCommand>> for <<spark-sql-LogicalPlan-DataWritingCommand.adoc#metrics, them>>.

[[internal-registries]]
.DataWritingCommandExec's Internal Properties (e.g. Registries, Counters and Flags)
[cols="1m,2",options="header",width="100%"]
|===
| Name
| Description

| sideEffectResult
| [[sideEffectResult]] Collection of <<spark-sql-InternalRow.adoc#, InternalRows>> (`Seq[InternalRow]`) that is the result of executing the <<cmd, DataWritingCommand>> (with the <<child, SparkPlan>>)

Used when `DataWritingCommandExec` is requested to <<executeCollect, executeCollect>>, <<executeToIterator, executeToIterator>>, <<executeTake, executeTake>> and <<doExecute, doExecute>>
|===

=== [[creating-instance]] Creating DataWritingCommandExec Instance

`DataWritingCommandExec` takes the following when created:

* [[cmd]] <<spark-sql-LogicalPlan-DataWritingCommand.adoc#, DataWritingCommand>>
* [[child]] Child <<spark-sql-SparkPlan.adoc#, physical plan>>

=== [[executeCollect]] Executing Physical Operator and Collecting Results -- `executeCollect` Method

[source, scala]
----
executeCollect(): Array[InternalRow]
----

NOTE: `executeCollect` is part of the <<spark-sql-SparkPlan.adoc#executeCollect, SparkPlan Contract>> to execute the physical operator and collect results.

`executeCollect`...FIXME

=== [[executeToIterator]] `executeToIterator` Method

[source, scala]
----
executeToIterator: Iterator[InternalRow]
----

NOTE: `executeToIterator` is part of the <<spark-sql-SparkPlan.adoc#executeToIterator, SparkPlan Contract>> to...FIXME.

`executeToIterator`...FIXME

=== [[executeTake]] Taking First N UnsafeRows -- `executeTake` Method

[source, scala]
----
executeTake(limit: Int): Array[InternalRow]
----

NOTE: `executeTake` is part of the <<spark-sql-SparkPlan.adoc#executeTake, SparkPlan Contract>> to take the first n `UnsafeRows`.

`executeTake`...FIXME

=== [[doExecute]] Executing Physical Operator (Generating RDD[InternalRow]) -- `doExecute` Method

[source, scala]
----
doExecute(): RDD[InternalRow]
----

NOTE: `doExecute` is part of the <<spark-sql-SparkPlan.adoc#doExecute, SparkPlan Contract>> to generate the runtime representation of a structured query as a distributed computation over <<spark-sql-InternalRow.adoc#, internal binary rows>> on Apache Spark (i.e. `RDD[InternalRow]`).

`doExecute` simply requests the <<spark-sql-SparkPlan.adoc#sqlContext, SQLContext>> for the <<spark-sql-SQLContext.adoc#sparkContext, SparkContext>> that is then requested to distribute (`parallelize`) the <<sideEffectResult, sideEffectResult>> (over 1 partition).
