title: AlterViewAsCommand

# AlterViewAsCommand Logical Command

`AlterViewAsCommand` is a link:spark-sql-LogicalPlan-RunnableCommand.adoc[logical command] for `ALTER VIEW` SQL statement to alter a view.

`AlterViewAsCommand` works with a table identifier (as `TableIdentifier`), the original SQL text, and a link:spark-sql-LogicalPlan.adoc[LogicalPlan] for the SQL query.

NOTE: `AlterViewAsCommand` is described by `alterViewQuery` labeled alternative in `statement` expression in `SqlBase.g4` and parsed using link:spark-sql-SparkSqlParser.adoc[SparkSqlParser].

When <<run, executed>>, `AlterViewAsCommand` attempts to link:spark-sql-SessionCatalog.adoc#alterTempViewDefinition[alter a temporary view in the current `SessionCatalog`] first, and if that "fails", <<alterPermanentView, alters the permanent view>>.

=== [[run]] Executing Logical Command -- `run` Method

[source, scala]
----
run(session: SparkSession): Seq[Row]
----

NOTE: `run` is part of <<spark-sql-LogicalPlan-RunnableCommand.adoc#run, RunnableCommand Contract>> to execute (run) a logical command.

`run`...FIXME

=== [[alterPermanentView]] `alterPermanentView` Internal Method

[source, scala]
----
alterPermanentView(session: SparkSession, analyzedPlan: LogicalPlan): Unit
----

`alterPermanentView`...FIXME

NOTE: `alterPermanentView` is used when...FIXME
