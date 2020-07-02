title: CreateViewCommand

# CreateViewCommand Logical Command

`CreateViewCommand` is a <<spark-sql-LogicalPlan-RunnableCommand.adoc#, logical command>> for <<run, creating or replacing a view or a table>>.

`CreateViewCommand` is <<creating-instance, created>> to represent the following:

* <<spark-sql-SparkSqlAstBuilder.adoc#visitCreateView, CREATE VIEW AS>> SQL statements

* `Dataset` operators: <<spark-sql-dataset-operators.adoc#createTempView, Dataset.createTempView>>, <<spark-sql-dataset-operators.adoc#createOrReplaceTempView, Dataset.createOrReplaceTempView>>, <<spark-sql-dataset-operators.adoc#createGlobalTempView, Dataset.createGlobalTempView>> and <<spark-sql-dataset-operators.adoc#createOrReplaceGlobalTempView, Dataset.createOrReplaceGlobalTempView>>

CAUTION: FIXME What's the difference between `CreateTempViewUsing`?

`CreateViewCommand` works with different <<viewType, view types>>.

[[viewType]]
.CreateViewCommand Behaviour Per View Type
[options="header",cols="1m,2",width="100%"]
|===
| View Type
| Description / Side Effect

| LocalTempView
| [[LocalTempView]] A session-scoped *local temporary view* that is available until the session, that has created it, is stopped.

When executed, `CreateViewCommand` requests the link:spark-sql-SessionCatalog.adoc#createTempView[current `SessionCatalog` to create a temporary view].

| GlobalTempView
| [[GlobalTempView]] A cross-session *global temporary view* that is available until the Spark application stops.

When executed, `CreateViewCommand` requests the link:spark-sql-SessionCatalog.adoc#createGlobalTempView[current `SessionCatalog` to create a global view].

| PersistedView
| [[PersistedView]] A cross-session *persisted view* that is available until dropped.

When executed, `CreateViewCommand` checks if the table exists. If it does and replace is enabled `CreateViewCommand` requests the link:spark-sql-SessionCatalog.adoc#alterTable[current `SessionCatalog` to alter a table]. Otherwise, when the table does not exist, `CreateViewCommand` requests the link:spark-sql-SessionCatalog.adoc#createTable[current `SessionCatalog` to create it].
|===

[source, scala]
----
/* CREATE [OR REPLACE] [[GLOBAL] TEMPORARY]
VIEW [IF NOT EXISTS] tableIdentifier
[identifierCommentList] [COMMENT STRING]
[PARTITIONED ON identifierList]
[TBLPROPERTIES tablePropertyList] AS query */

// Demo table for "AS query" part
spark.range(10).write.mode("overwrite").saveAsTable("t1")

// The "AS" query
val asQuery = "SELECT * FROM t1"

// The following queries should all work fine
val q1 = "CREATE VIEW v1 AS " + asQuery
sql(q1)

val q2 = "CREATE OR REPLACE VIEW v1 AS " + asQuery
sql(q2)

val q3 = "CREATE OR REPLACE TEMPORARY VIEW v1 " + asQuery
sql(q3)

val q4 = "CREATE OR REPLACE GLOBAL TEMPORARY VIEW v1 " + asQuery
sql(q4)

val q5 = "CREATE VIEW IF NOT EXISTS v1 AS " + asQuery
sql(q5)

// The following queries should all fail
// the number of user-specified columns does not match the schema of the AS query
val qf1 = "CREATE VIEW v1 (c1 COMMENT 'comment', c2) AS " + asQuery
scala> sql(qf1)
org.apache.spark.sql.AnalysisException: The number of columns produced by the SELECT clause (num: `1`) does not match the number of column names specified by CREATE VIEW (num: `2`).;
  at org.apache.spark.sql.execution.command.CreateViewCommand.run(views.scala:134)
  at org.apache.spark.sql.execution.command.ExecutedCommandExec.sideEffectResult$lzycompute(commands.scala:70)
  at org.apache.spark.sql.execution.command.ExecutedCommandExec.sideEffectResult(commands.scala:68)
  at org.apache.spark.sql.execution.command.ExecutedCommandExec.executeCollect(commands.scala:79)
  at org.apache.spark.sql.Dataset$$anonfun$6.apply(Dataset.scala:190)
  at org.apache.spark.sql.Dataset$$anonfun$6.apply(Dataset.scala:190)
  at org.apache.spark.sql.Dataset$$anonfun$52.apply(Dataset.scala:3254)
  at org.apache.spark.sql.execution.SQLExecution$.withNewExecutionId(SQLExecution.scala:77)
  at org.apache.spark.sql.Dataset.withAction(Dataset.scala:3253)
  at org.apache.spark.sql.Dataset.<init>(Dataset.scala:190)
  at org.apache.spark.sql.Dataset$.ofRows(Dataset.scala:75)
  at org.apache.spark.sql.SparkSession.sql(SparkSession.scala:641)
  ... 49 elided

// CREATE VIEW ... PARTITIONED ON is not allowed
val qf2 = "CREATE VIEW v1 PARTITIONED ON (c1, c2) AS " + asQuery
scala> sql(qf2)
org.apache.spark.sql.catalyst.parser.ParseException:
Operation not allowed: CREATE VIEW ... PARTITIONED ON(line 1, pos 0)

// Use the same name of t1 for a new view
val qf3 = "CREATE VIEW t1 AS " + asQuery
scala> sql(qf3)
org.apache.spark.sql.AnalysisException: `t1` is not a view;
  at org.apache.spark.sql.execution.command.CreateViewCommand.run(views.scala:156)
  at org.apache.spark.sql.execution.command.ExecutedCommandExec.sideEffectResult$lzycompute(commands.scala:70)
  at org.apache.spark.sql.execution.command.ExecutedCommandExec.sideEffectResult(commands.scala:68)
  at org.apache.spark.sql.execution.command.ExecutedCommandExec.executeCollect(commands.scala:79)
  at org.apache.spark.sql.Dataset$$anonfun$6.apply(Dataset.scala:190)
  at org.apache.spark.sql.Dataset$$anonfun$6.apply(Dataset.scala:190)
  at org.apache.spark.sql.Dataset$$anonfun$52.apply(Dataset.scala:3254)
  at org.apache.spark.sql.execution.SQLExecution$.withNewExecutionId(SQLExecution.scala:77)
  at org.apache.spark.sql.Dataset.withAction(Dataset.scala:3253)
  at org.apache.spark.sql.Dataset.<init>(Dataset.scala:190)
  at org.apache.spark.sql.Dataset$.ofRows(Dataset.scala:75)
  at org.apache.spark.sql.SparkSession.sql(SparkSession.scala:641)
  ... 49 elided

// View already exists
val qf4 = "CREATE VIEW v1 AS " + asQuery
scala> sql(qf4)
org.apache.spark.sql.AnalysisException: View `v1` already exists. If you want to update the view definition, please use ALTER VIEW AS or CREATE OR REPLACE VIEW AS;
  at org.apache.spark.sql.execution.command.CreateViewCommand.run(views.scala:169)
  at org.apache.spark.sql.execution.command.ExecutedCommandExec.sideEffectResult$lzycompute(commands.scala:70)
  at org.apache.spark.sql.execution.command.ExecutedCommandExec.sideEffectResult(commands.scala:68)
  at org.apache.spark.sql.execution.command.ExecutedCommandExec.executeCollect(commands.scala:79)
  at org.apache.spark.sql.Dataset$$anonfun$6.apply(Dataset.scala:190)
  at org.apache.spark.sql.Dataset$$anonfun$6.apply(Dataset.scala:190)
  at org.apache.spark.sql.Dataset$$anonfun$52.apply(Dataset.scala:3254)
  at org.apache.spark.sql.execution.SQLExecution$.withNewExecutionId(SQLExecution.scala:77)
  at org.apache.spark.sql.Dataset.withAction(Dataset.scala:3253)
  at org.apache.spark.sql.Dataset.<init>(Dataset.scala:190)
  at org.apache.spark.sql.Dataset$.ofRows(Dataset.scala:75)
  at org.apache.spark.sql.SparkSession.sql(SparkSession.scala:641)
  ... 49 elided
----

[[innerChildren]]
`CreateViewCommand` returns the <<child, child logical query plan>> when requested for the <<spark-sql-catalyst-TreeNode.adoc#innerChildren, inner nodes>> (that should be shown as an inner nested tree of this node).

[source, scala]
----
val sqlText = "CREATE VIEW v1 AS " + asQuery
val plan = spark.sessionState.sqlParser.parsePlan(sqlText)
scala> println(plan.numberedTreeString)
00 CreateViewCommand `v1`, SELECT * FROM t1, false, false, PersistedView
01    +- 'Project [*]
02       +- 'UnresolvedRelation `t1`
----

=== [[prepareTable]] Creating CatalogTable -- `prepareTable` Internal Method

[source, scala]
----
prepareTable(session: SparkSession, analyzedPlan: LogicalPlan): CatalogTable
----

`prepareTable`...FIXME

NOTE: `prepareTable` is used exclusively when `CreateViewCommand` logical command is <<run, executed>>.

=== [[run]] Executing Logical Command -- `run` Method

[source, scala]
----
run(sparkSession: SparkSession): Seq[Row]
----

NOTE: `run` is part of <<spark-sql-LogicalPlan-RunnableCommand.adoc#run, RunnableCommand Contract>> to execute (run) a logical command.

`run` requests the input `SparkSession` for the <<spark-sql-SparkSession.adoc#sessionState, SessionState>> that is in turn requested to <<spark-sql-SessionState.adoc#executePlan, "execute">> the <<child, child logical plan>> (which simply creates a <<spark-sql-QueryExecution.adoc#creating-instance, QueryExecution>>).

[NOTE]
====
`run` uses a <<spark-sql-LogicalPlan.adoc#logical-plan-to-be-analyzed-idiom, common idiom>> in Spark SQL to make sure that a logical plan can be analyzed, i.e.

[source, scala]
----
val qe = sparkSession.sessionState.executePlan(child)
qe.assertAnalyzed()
val analyzedPlan = qe.analyzed
----
====

`run` <<verifyTemporaryObjectsNotExists, verifyTemporaryObjectsNotExists>>.

`run` requests the input `SparkSession` for the <<spark-sql-SparkSession.adoc#sessionState, SessionState>> that is in turn requested for the <<spark-sql-SessionState.adoc#catalog, SessionCatalog>>.

`run` then branches off per the <<viewType, ViewType>>:

* For <<LocalTempView, local temporary views>>, `run` <<aliasPlan, alias>> the analyzed plan and requests the `SessionCatalog` to <<spark-sql-SessionCatalog.adoc#createTempView, create or replace a local temporary view>>

* For <<GlobalTempView, global temporary views>>, `run` also <<aliasPlan, alias>> the analyzed plan and requests the `SessionCatalog` to <<spark-sql-SessionCatalog.adoc#createGlobalTempView, create or replace a global temporary view>>

* For <<PersistedView, persisted views>>, `run` asks the `SessionCatalog` whether the <<spark-sql-SessionCatalog.adoc#tableExists, table exists or not>> (given <<name, TableIdentifier>>).

** If the <<name, table>> exists and the <<allowExisting, allowExisting>> flag is on, `run` simply does nothing (and exits)

** If the <<name, table>> exists and the <<replace, replace>> flag is on, `run` requests the `SessionCatalog` for the <<spark-sql-SessionCatalog.adoc#getTableMetadata, table metadata>> and replaces the table, i.e. `run` requests the `SessionCatalog` to <<spark-sql-SessionCatalog.adoc#dropTable, drop the table>> followed by <<spark-sql-SessionCatalog.adoc#createTable, re-creating it>> (with a <<prepareTable, new CatalogTable>>)

** If however the <<name, table>> does not exist, `run` simply requests the `SessionCatalog` to <<spark-sql-SessionCatalog.adoc#createTable, create it>> (with a <<prepareTable, new CatalogTable>>)

`run` throws an `AnalysisException` for <<PersistedView, persisted views>> when they already exist, the <<allowExisting, allowExisting>> flag is off and the table type is not a view.

```
[name] is not a view
```

`run` throws an `AnalysisException` for <<PersistedView, persisted views>> when they already exist and the <<allowExisting, allowExisting>> and <<replace, replace>> flags are off.

```
View [name] already exists. If you want to update the view definition, please use ALTER VIEW AS or CREATE OR REPLACE VIEW AS
```

`run` throws an `AnalysisException` if the <<userSpecifiedColumns, userSpecifiedColumns>> are defined and their numbers is different from the number of <<spark-sql-catalyst-QueryPlan.adoc#output, output schema attributes>> of the analyzed logical plan.

```
The number of columns produced by the SELECT clause (num: `[output.length]`) does not match the number of column names specified by CREATE VIEW (num: `[userSpecifiedColumns.length]`).
```

=== [[creating-instance]] Creating CreateViewCommand Instance

`CreateViewCommand` takes the following when created:

* [[name]] `TableIdentifier`
* [[userSpecifiedColumns]] User-defined columns (as `Seq[(String, Option[String])]`)
* [[comment]] Optional comment
* [[properties]] Properties (as `Map[String, String]`)
* [[originalText]] Optional DDL statement
* [[child]] Child <<spark-sql-LogicalPlan.adoc#, logical plan>>
* [[allowExisting]] `allowExisting` flag
* [[replace]] `replace` flag
* <<viewType, ViewType>>

=== [[verifyTemporaryObjectsNotExists]] `verifyTemporaryObjectsNotExists` Internal Method

[source, scala]
----
verifyTemporaryObjectsNotExists(sparkSession: SparkSession): Unit
----

`verifyTemporaryObjectsNotExists`...FIXME

NOTE: `verifyTemporaryObjectsNotExists` is used exclusively when `CreateViewCommand` logical command is <<run, executed>>.

=== [[aliasPlan]] `aliasPlan` Internal Method

[source, scala]
----
aliasPlan(session: SparkSession, analyzedPlan: LogicalPlan): LogicalPlan
----

`aliasPlan`...FIXME

NOTE: `aliasPlan` is used when `CreateViewCommand` logical command is <<run, executed>> (and <<prepareTable, prepareTable>>).
