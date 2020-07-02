title: ScalarSubquery

# ScalarSubquery (SubqueryExpression) Expression

`ScalarSubquery` is a link:spark-sql-Expression-SubqueryExpression.adoc[SubqueryExpression] that returns a single row and a single column only.

`ScalarSubquery` represents a structured query that can be used as a "column".

IMPORTANT: Spark SQL uses the name of `ScalarSubquery` twice to represent a `SubqueryExpression` (this page) and  an link:spark-sql-Expression-ExecSubqueryExpression-ScalarSubquery.adoc[ExecSubqueryExpression]. You've been warned.

`ScalarSubquery` is <<creating-instance, created>> exclusively when `AstBuilder` is requested to link:spark-sql-AstBuilder.adoc#visitSubqueryExpression[parse a subquery expression].

[source, scala]
----
// FIXME DEMO

// Borrowed from ExpressionParserSuite.scala
// ScalarSubquery(table("tbl").select('max.function('val))) > 'current)
val sql = "(select max(val) from tbl) > current"

// 'a === ScalarSubquery(table("s").select('b))
val sql = "a = (select b from s)"

// Borrowed from PlanParserSuite.scala
// table("t").select(ScalarSubquery(table("s").select('max.function('b))).as("ss"))
val sql = "select (select max(b) from s) ss from t"

// table("t").where('a === ScalarSubquery(table("s").select('b))).select(star())
val sql = "select * from t where a = (select b from s)"

// table("t").groupBy('g)('g).where('a > ScalarSubquery(table("s").select('b)))
val sql = "select g from t group by g having a > (select b from s)"
----

=== [[creating-instance]] Creating ScalarSubquery Instance

`ScalarSubquery` takes the following when created:

* [[plan]] Subquery link:spark-sql-LogicalPlan.adoc[logical plan]
* [[children]] Child link:spark-sql-Expression.adoc[expressions] (default: no children)
* [[exprId]] Expression ID (as `ExprId` and defaults to a link:spark-sql-Expression-NamedExpression.adoc#newExprId[new ExprId])
