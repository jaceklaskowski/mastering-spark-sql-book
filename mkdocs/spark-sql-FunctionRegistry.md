title: FunctionRegistry

# FunctionRegistry -- Contract for Function Registries (Catalogs)

`FunctionRegistry` is the <<contract, contract>> of <<implementations, function registries>> (_catalogs_) of native and user-defined functions.

[[contract]]
[source, scala]
----
package org.apache.spark.sql.catalyst.analysis

trait FunctionRegistry {
  // only required properties (vals and methods) that have no implementation
  // the others follow
  def clear(): Unit
  def dropFunction(name: FunctionIdentifier): Boolean
  def listFunction(): Seq[FunctionIdentifier]
  def lookupFunction(name: FunctionIdentifier): Option[ExpressionInfo]
  def lookupFunction(name: FunctionIdentifier, children: Seq[Expression]): Expression
  def lookupFunctionBuilder(name: FunctionIdentifier): Option[FunctionBuilder]
  def registerFunction(
    name: FunctionIdentifier,
    info: ExpressionInfo,
    builder: FunctionBuilder): Unit
}
----

.FunctionRegistry Contract
[cols="1m,2",options="header",width="100%"]
|===
| Property
| Description

| clear
| [[clear]] Used exclusively when `SessionCatalog` is requested to <<spark-sql-SessionCatalog.adoc#reset, reset>>

| dropFunction
| [[dropFunction]] Used when...FIXME

| listFunction
| [[listFunction]] Used when...FIXME

| lookupFunction
a| [[lookupFunction]]

Used when:

* `FunctionRegistry` is requested to <<functionExists, functionExists>>

* `SessionCatalog` is requested to <<spark-sql-SessionCatalog.adoc#lookupFunction, find a function by name>>, <<spark-sql-SessionCatalog.adoc#lookupFunctionInfo, lookupFunctionInfo>> or <<spark-sql-SessionCatalog.adoc#reset, reset>>

* `HiveSessionCatalog` is requested to link:hive/HiveSessionCatalog.adoc#lookupFunction0[lookupFunction0]

| lookupFunctionBuilder
| [[lookupFunctionBuilder]] Used when...FIXME

| registerFunction
a| [[registerFunction]]

Used when:

* `SessionCatalog` is requested to <<spark-sql-SessionCatalog.adoc#registerFunction, registerFunction>> or <<spark-sql-SessionCatalog.adoc#reset, reset>>

* `FunctionRegistry` is requested for a <<builtin, SimpleFunctionRegistry with the built-in functions registered>> or <<createOrReplaceTempFunction, createOrReplaceTempFunction>>

* `SimpleFunctionRegistry` is requested to `clone`
|===

[[implementations]]
NOTE: The one and only `FunctionRegistry` available in Spark SQL is <<SimpleFunctionRegistry, SimpleFunctionRegistry>>.

`FunctionRegistry` is available through link:spark-sql-SessionState.adoc#functionRegistry[functionRegistry] property of a `SessionState` (that is available as <<spark-sql-SparkSession.adoc#sessionState, sessionState>> property of a `SparkSession`).

[source, scala]
----
scala> :type spark
org.apache.spark.sql.SparkSession

scala> :type spark.sessionState.functionRegistry
org.apache.spark.sql.catalyst.analysis.FunctionRegistry
----

NOTE: You can register a new user-defined function using link:spark-sql-UDFRegistration.adoc[UDFRegistration].

[[attributes]]
.FunctionRegistry's Attributes
[width="100%",cols="1,2",options="header"]
|===
| Name
| Description

| [[builtin]] `builtin`
| <<SimpleFunctionRegistry, SimpleFunctionRegistry>> with the <<expressions, built-in functions>> registered.
|===

[[expressions]]
`FunctionRegistry` manages *function expression registry* of <<spark-sql-Expression.adoc#, Catalyst expressions>> and the corresponding built-in/native SQL functions (that can be used in SQL statements).

.(Subset of) FunctionRegistry's Catalyst Expression to SQL Function Mapping
[cols="1,1m",options="header",width="100%"]
|===
| Catalyst Expression
| SQL Function

| <<spark-sql-Expression-CumeDist.adoc#, CumeDist>>
| [[cume_dist]] cume_dist

| `IfNull`
| [[ifnull]] ifnull

| `Left`
| [[left]] left

| <<spark-sql-Expression-MonotonicallyIncreasingID.adoc#, MonotonicallyIncreasingID>>
| [[monotonically_increasing_id]] monotonically_increasing_id

| `NullIf`
| [[nullif]] nullif

| `Nvl`
| [[nvl]] nvl

| `Nvl2`
| [[nvl2]] nvl2

| <<spark-sql-Expression-ParseToDate.adoc#, ParseToDate>>
| [[to_date]] to_date

| <<spark-sql-Expression-ParseToTimestamp.adoc#, ParseToTimestamp>>
| [[to_timestamp]] to_timestamp

| `Right`
| [[right]] right

| <<spark-sql-CreateStruct.adoc#registryEntry, CreateNamedStruct>>
| [[struct]] struct
|===

=== [[expression]] `expression` Internal Method

[source, scala]
----
expression[T <: Expression](name: String)
  (implicit tag: ClassTag[T]): (String, (ExpressionInfo, FunctionBuilder))
----

`expression`...FIXME

NOTE: `expression` is used when...FIXME

=== [[SimpleFunctionRegistry]] SimpleFunctionRegistry

`SimpleFunctionRegistry` is the default <<FunctionRegistry, FunctionRegistry>> that is backed by a hash map (with optional case sensitivity).

=== [[createOrReplaceTempFunction]] `createOrReplaceTempFunction` Final Method

[source, scala]
----
createOrReplaceTempFunction(name: String, builder: FunctionBuilder): Unit
----

`createOrReplaceTempFunction`...FIXME

NOTE: `createOrReplaceTempFunction` is used exclusively when `UDFRegistration` is requested to register an <<spark-sql-UDFRegistration.adoc#register, user-defined function>>, <<spark-sql-UDFRegistration.adoc#register-UserDefinedAggregateFunction, user-defined aggregate function>>, <<spark-sql-UDFRegistration.adoc#register-UserDefinedFunction, user-defined function (as UserDefinedFunction)>> or `registerPython`.

=== [[functionExists]] `functionExists` Method

[source, scala]
----
functionExists(name: FunctionIdentifier): Boolean
----

`functionExists`...FIXME

NOTE: `functionExists` is used when...FIXME
