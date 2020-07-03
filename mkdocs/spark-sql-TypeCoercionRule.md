# TypeCoercionRule -- Type Coercion Rules

`TypeCoercionRule` is the <<contract, contract>> of <<implementations, logical rules>> to <<coerceTypes, coerce>> and <<propagateTypes, propagate>> types in <<spark-sql-LogicalPlan.adoc#, logical plans>>.

[[contract]]
[source, scala]
----
package org.apache.spark.sql.catalyst.analysis

trait TypeCoercionRule extends Rule[LogicalPlan] {
  // only required methods that have no implementation
  // the others follow
  def coerceTypes(plan: LogicalPlan): LogicalPlan
}
----

.(Subset of) TypeCoercionRule Contract
[cols="1m,2",options="header",width="100%"]
|===
| Method
| Description

| coerceTypes
| [[coerceTypes]] Coerce types in a <<spark-sql-LogicalPlan.adoc#, logical plan>>

Used exclusively when `TypeCoercionRule` is <<apply, executed>>
|===

`TypeCoercionRule` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

[[implementations]]
.TypeCoercionRules
[cols="1,2",options="header",width="100%"]
|===
| TypeCoercionRule
| Description

| CaseWhenCoercion
| [[CaseWhenCoercion]]

| ConcatCoercion
| [[ConcatCoercion]]

| DecimalPrecision
| [[DecimalPrecision]]

| Division
| [[Division]]

| EltCoercion
| [[EltCoercion]]

| FunctionArgumentConversion
| [[FunctionArgumentConversion]]

| IfCoercion
| [[IfCoercion]]

| ImplicitTypeCasts
| [[ImplicitTypeCasts]]

| <<spark-sql-Analyzer-TypeCoercionRule-InConversion.adoc#, InConversion>>
| [[InConversion]]

| PromoteStrings
| [[PromoteStrings]]

| StackCoercion
| [[StackCoercion]]

| <<spark-sql-Analyzer-TypeCoercionRule-WindowFrameCoercion.adoc#, WindowFrameCoercion>>
| [[WindowFrameCoercion]]
|===

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply` <<coerceTypes, coerceTypes>> in the input <<spark-sql-LogicalPlan.adoc#, LogicalPlan>> and returns the following:

* The input <<spark-sql-LogicalPlan.adoc#, LogicalPlan>> itself if <<spark-sql-catalyst-TreeNode.adoc#fastEquals, matches>> the coerced plan

* The plan after <<propagateTypes, propagateTypes>> on the coerced plan

=== [[propagateTypes]] `propagateTypes` Internal Method

[source, scala]
----
propagateTypes(plan: LogicalPlan): LogicalPlan
----

`propagateTypes`...FIXME

NOTE: `propagateTypes` is used exclusively when `TypeCoercionRule` is <<apply, executed>>.
