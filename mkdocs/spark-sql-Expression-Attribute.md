title: Attribute

# Attribute -- Base of Leaf Named Expressions

`Attribute` is the <<contract, base>> of <<extensions, leaf named expressions>>.

NOTE: link:spark-sql-catalyst-QueryPlan.adoc#output[QueryPlan uses Attributes] to build the link:spark-sql-StructType.adoc[schema] of the query (it represents).

[[contract]]
[source, scala]
----
package org.apache.spark.sql.catalyst.expressions

abstract class Attribute extends ... {
  // only required properties (vals and methods) that have no implementation
  // the others follow
  def withMetadata(newMetadata: Metadata): Attribute
  def withName(newName: String): Attribute
  def withNullability(newNullability: Boolean): Attribute
  def withQualifier(newQualifier: Option[String]): Attribute
  def newInstance(): Attribute
}
----

.Attribute Contract
[cols="1m,2",options="header",width="100%"]
|===
| Property
| Description

| withMetadata
| [[withMetadata]]

| withName
| [[withName]]

| withNullability
| [[withNullability]]

| withQualifier
| [[withQualifier]]

| newInstance
| [[newInstance]]
|===

[[references]]
When requested for <<spark-sql-Expression.adoc#references, references>>, `Attribute` gives the reference to itself only.

[[toAttribute]]
As a <<spark-sql-Expression-NamedExpression.adoc#, NamedExpression>>, `Attribute` gives the reference to itself only when requested for <<spark-sql-Expression-NamedExpression.adoc#toAttribute, toAttribute>>.

[[extensions]]
.Attributes (Direct Implementations)
[cols="1,2",options="header",width="100%"]
|===
| Attribute
| Description

| <<spark-sql-Expression-AttributeReference.adoc#, AttributeReference>>
| [[AttributeReference]]

| <<spark-sql-Expression-PrettyAttribute.adoc#, PrettyAttribute>>
| [[PrettyAttribute]]

| <<spark-sql-Expression-UnresolvedAttribute.adoc#, UnresolvedAttribute>>
| [[UnresolvedAttribute]]
|===

As an optimization, `Attribute` is marked as to not tolerate `nulls`, and when given a `null` input produces a `null` output.
