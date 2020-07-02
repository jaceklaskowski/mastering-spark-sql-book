title: PlanExpression

# PlanExpression -- Expressions with Query Plans

`PlanExpression` is the <<contract, contract>> for link:spark-sql-Expression.adoc[Catalyst expressions] that contain a link:spark-sql-catalyst-QueryPlan.adoc[QueryPlan].

[[contract]]
[source, scala]
----
package org.apache.spark.sql.catalyst.expressions

abstract class PlanExpression[T <: QueryPlan[_]] extends Expression {
  // only required methods that have no implementation
  // the others follow
  def exprId: ExprId
  def plan: T
  def withNewPlan(plan: T): PlanExpression[T]
}
----

.PlanExpression Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| `exprId`
| [[exprId]] Used when...FIXME

| `plan`
| [[plan]] Used when...FIXME

| `withNewPlan`
| [[withNewPlan]] Used when...FIXME
|===

[[implementations]]
.PlanExpressions
[cols="1,2",options="header",width="100%"]
|===
| PlanExpression
| Description

| [[ExecSubqueryExpression]] link:spark-sql-Expression-ExecSubqueryExpression.adoc[ExecSubqueryExpression]
|

| [[SubqueryExpression]] link:spark-sql-Expression-SubqueryExpression.adoc[SubqueryExpression]
|
|===
