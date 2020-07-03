# ObjectConsumerExec -- Unary Physical Operators with Child Physical Operator with One-Attribute Output Schema

`ObjectConsumerExec` is the <<contract, contract>> of <<implementations, unary physical operators>> with the child physical operator using a one-attribute <<spark-sql-catalyst-QueryPlan.adoc#output, output schema>>.

[[contract]]
[source, scala]
----
package org.apache.spark.sql.execution

trait ObjectConsumerExec extends UnaryExecNode {
  // No properties (vals and methods) that have no implementation
}
----

[[references]]
`ObjectConsumerExec` requests the child physical operator for the <<spark-sql-catalyst-QueryPlan.adoc#outputSet, output schema attribute set>> when requested for the <<spark-sql-catalyst-QueryPlan.adoc#references, references>>.

[[implementations]]
.ObjectConsumerExecs
[cols="1,2",options="header",width="100%"]
|===
| ObjectConsumerExec
| Description

| `AppendColumnsWithObjectExec`
| [[AppendColumnsWithObjectExec]]

| <<spark-sql-SparkPlan-MapElementsExec.adoc#, MapElementsExec>>
| [[MapElementsExec]]

| `MapPartitionsExec`
| [[MapPartitionsExec]]

| <<spark-sql-SparkPlan-SerializeFromObjectExec.adoc#, SerializeFromObjectExec>>
| [[SerializeFromObjectExec]]
|===

=== [[inputObjectType]] `inputObjectType` Method

[source, scala]
----
inputObjectType: DataType
----

`inputObjectType` simply returns the <<spark-sql-Expression.adoc#dataType, data type>> of the single <<spark-sql-catalyst-QueryPlan.adoc#output, output attribute>> of the child physical operator.

NOTE: `inputObjectType` is used when...FIXME
