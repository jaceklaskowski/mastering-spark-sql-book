title: InSubquery

# InSubquery Expression

`InSubquery` is a link:spark-sql-Expression-ExecSubqueryExpression.adoc[ExecSubqueryExpression] that...FIXME

`InSubquery` is <<creating-instance, created>> when...FIXME

=== [[updateResult]] `updateResult` Method

[source, scala]
----
updateResult(): Unit
----

NOTE: `updateResult` is part of link:spark-sql-Expression-ExecSubqueryExpression.adoc#updateResult[ExecSubqueryExpression Contract] to...FIXME.

`updateResult`...FIXME

=== [[creating-instance]] Creating InSubquery Instance

`InSubquery` takes the following when created:

* [[child]] Child link:spark-sql-Expression.adoc[expression]
* [[plan]] link:spark-sql-SparkPlan-SubqueryExec.adoc[SubqueryExec] physical operator
* [[exprId]] Expression ID (as `ExprId`)
* [[result]] `result` array (default: `null`)
* [[updated]] `updated` flag (default: `false`)
