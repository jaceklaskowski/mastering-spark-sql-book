title: NamedExpression

# NamedExpression -- Catalyst Expressions with Name, ID and Qualifier

`NamedExpression` is a <<contract, contract>> of link:spark-sql-Expression.adoc[Catalyst expressions] that have a <<name, name>>, <<exprId, exprId>>, and optional <<qualifier, qualifier>>.

[[contract]]
[source, scala]
----
package org.apache.spark.sql.catalyst.expressions

trait NamedExpression extends Expression {
  // only required methods that have no implementation
  def exprId: ExprId
  def name: String
  def newInstance(): NamedExpression
  def qualifier: Option[String]
  def toAttribute: Attribute
}
----

.NamedExpression Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| `exprId`
| [[exprId]] Used when...FIXME

| `name`
| [[name]] Used when...FIXME

| `qualifier`
| [[qualifier]] Used when...FIXME

| `toAttribute`
| [[toAttribute]]
|===

=== [[newExprId]] Creating ExprId -- `newExprId` Object Method

[source, scala]
----
newExprId: ExprId
----

`newExprId`...FIXME

NOTE: `newExprId` is used when...FIXME
