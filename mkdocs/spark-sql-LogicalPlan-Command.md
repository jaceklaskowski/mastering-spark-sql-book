title: Command

# Command -- Eagerly-Executed Logical Operator

`Command` is the *marker interface* for link:spark-sql-LogicalPlan.adoc[logical operators] that represent non-query commands that are executed early in the link:spark-sql-QueryExecution.adoc#query-plan-lifecycle[query plan lifecycle] (unlike logical plans in general).

NOTE: `Command` is executed when a `Dataset` is requested for the link:spark-sql-Dataset.adoc#logicalPlan[logical plan] (which is after the query has been link:spark-sql-QueryExecution.adoc#analyzed[analyzed]).

[[output]]
`Command` has no link:spark-sql-catalyst-QueryPlan.adoc#output[output schema] by default.

[[children]]
`Command` has no child logical operators (which makes it similar to link:spark-sql-LogicalPlan-LeafNode.adoc[leaf logical operators]).

[[implementations]]
.Commands (Direct Implementations)
[cols="30m,70",options="header",width="100%"]
|===
| Command
| Description

| xref:spark-sql-LogicalPlan-DataWritingCommand.adoc[DataWritingCommand]
| [[DataWritingCommand]]

| xref:spark-sql-LogicalPlan-RunnableCommand.adoc[RunnableCommand]
| [[RunnableCommand]]

|===
