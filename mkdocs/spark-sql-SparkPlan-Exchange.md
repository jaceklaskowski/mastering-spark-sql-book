# Exchange -- Unary Physical Operators for Data Exchange

`Exchange` is the base of link:spark-sql-SparkPlan.adoc#UnaryExecNode[unary physical operators] that exchange data among multiple threads or processes.

[[output]]
When requested for the link:spark-sql-catalyst-QueryPlan.adoc#output[output schema], `Exchange` simply uses the child physical operator's output schema.

[[implementations]]
.Exchanges
[cols="1,2",options="header",width="100%"]
|===
| Exchange
| Description

| [[BroadcastExchangeExec]] link:spark-sql-SparkPlan-BroadcastExchangeExec.adoc[BroadcastExchangeExec]
|

| [[ShuffleExchangeExec]] link:spark-sql-SparkPlan-ShuffleExchangeExec.adoc[ShuffleExchangeExec]
|
|===
