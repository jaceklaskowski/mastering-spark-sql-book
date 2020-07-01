title: BaseSessionStateBuilder

# BaseSessionStateBuilder -- Generic Builder of SessionState

`BaseSessionStateBuilder` is the <<contract, abstraction>> of <<implementations, builders>> that can <<newBuilder, produce a new BaseSessionStateBuilder>> to <<createClone, create a SessionState>>.

NOTE: `BaseSessionStateBuilder` and link:spark-sql-StaticSQLConf.adoc#spark.sql.catalogImplementation[spark.sql.catalogImplementation] configuration property allow for Hive and non-Hive Spark deployments.

[[contract]]
.BaseSessionStateBuilder Contract (Abstract Methods Only)
[cols="30m,70",options="header",width="100%"]
|===
| Method
| Description

| newBuilder
a| [[newBuilder]]

[source, scala]
----
newBuilder: (SparkSession, Option[SessionState]) => BaseSessionStateBuilder
----

Produces a new `BaseSessionStateBuilder` for given link:spark-sql-SparkSession.adoc[SparkSession] and optional link:spark-sql-SessionState.adoc[SessionState]

Used when `BaseSessionStateBuilder` is requested to <<createClone, create a SessionState>>

|===

`BaseSessionStateBuilder` is <<creating-instance, created>> when `SparkSession` is requested for a link:spark-sql-SparkSession.adoc#instantiateSessionState[SessionState].

[source, scala]
----
scala> :type spark
org.apache.spark.sql.SparkSession

scala> :type spark.sessionState
org.apache.spark.sql.internal.SessionState
----

`BaseSessionStateBuilder` holds <<properties, properties>> that (together with <<newBuilder, newBuilder>>) are used to create a link:spark-sql-SessionState.adoc[SessionState].

[[properties]]
.BaseSessionStateBuilder's Properties
[cols="30m,70",options="header",width="100%"]
|===
| Name
| Description

| analyzer
| [[analyzer]] <<spark-sql-Analyzer.adoc#, Logical analyzer>>

| catalog
a| [[catalog]] <<spark-sql-SessionCatalog.adoc#, SessionCatalog>>

Used to create <<analyzer, Analyzer>>, <<optimizer, Optimizer>> and a <<build, SessionState>> itself

NOTE: link:hive/HiveSessionStateBuilder.adoc[HiveSessionStateBuilder] manages its own Hive-aware link:hive/HiveSessionStateBuilder.adoc#catalog[HiveSessionCatalog].

| conf
| [[conf]] link:spark-sql-SQLConf.adoc[SQLConf]

| customOperatorOptimizationRules
| [[customOperatorOptimizationRules]] Custom operator optimization rules to add to the <<spark-sql-Optimizer.adoc#extendedOperatorOptimizationRules, base Operator Optimization batch>>.

When requested for the custom rules, `customOperatorOptimizationRules` simply requests the <<extensions, SparkSessionExtensions>> to <<spark-sql-SparkSessionExtensions.adoc#buildOptimizerRules, buildOptimizerRules>>.

| experimentalMethods
| [[experimentalMethods]] link:spark-sql-ExperimentalMethods.adoc[ExperimentalMethods]

| extensions
| [[extensions]] link:spark-sql-SparkSessionExtensions.adoc[SparkSessionExtensions]

| functionRegistry
| [[functionRegistry]] link:spark-sql-FunctionRegistry.adoc[FunctionRegistry]

| listenerManager
| [[listenerManager]] link:spark-sql-ExecutionListenerManager.adoc[ExecutionListenerManager]

| optimizer
| [[optimizer]] <<spark-sql-SparkOptimizer.adoc#, SparkOptimizer>> (that is downcast to the base <<spark-sql-Optimizer.adoc#, Optimizer>>) that is <<spark-sql-SparkOptimizer.adoc#creating-instance, created>> with the <<catalog, SessionCatalog>> and the <<experimentalMethods, ExperimentalMethods>>.

Note that the `SparkOptimizer` adds the <<customOperatorOptimizationRules, customOperatorOptimizationRules>> to the <<spark-sql-Optimizer.adoc#extendedOperatorOptimizationRules, operator optimization rules>>.

`optimizer` is used when `BaseSessionStateBuilder` is requested to <<build, create a SessionState>> (for the <<spark-sql-SessionState.adoc#optimizerBuilder, optimizerBuilder>> function to create an <<spark-sql-Optimizer.adoc#, Optimizer>> when requested for the <<spark-sql-SessionState.adoc#optimizer, Optimizer>>).

| planner
| [[planner]] link:spark-sql-SparkPlanner.adoc[SparkPlanner]

| resourceLoader
| [[resourceLoader]] `SessionResourceLoader`

| sqlParser
| [[sqlParser]] link:spark-sql-ParserInterface.adoc[ParserInterface]

| streamingQueryManager
| [[streamingQueryManager]] Spark Structured Streaming's `StreamingQueryManager`

| udfRegistration
| [[udfRegistration]] link:spark-sql-UDFRegistration.adoc[UDFRegistration]

|===

[[implementations]]
.BaseSessionStateBuilders
[cols="30,70",options="header",width="100%"]
|===
| BaseSessionStateBuilder
| Description

| link:spark-sql-SessionStateBuilder.adoc[SessionStateBuilder]
| [[SessionStateBuilder]]

| link:hive/HiveSessionStateBuilder.adoc[HiveSessionStateBuilder]
| [[HiveSessionStateBuilder]]

|===

[[NewBuilder]]
[NOTE]
====
`BaseSessionStateBuilder` defines a type alias https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/internal/BaseSessionStateBuilder.scala#L57[NewBuilder] for a function to create a `BaseSessionStateBuilder`.

[source, scala]
----
type NewBuilder = (SparkSession, Option[SessionState]) => BaseSessionStateBuilder
----
====

NOTE: `BaseSessionStateBuilder` is an experimental and unstable API.

=== [[creating-instance]] Creating BaseSessionStateBuilder Instance

`BaseSessionStateBuilder` takes the following to be created:

* [[session]] link:spark-sql-SparkSession.adoc[SparkSession]
* [[parentState]] Optional link:spark-sql-SessionState.adoc[SessionState]

=== [[createClone]] Creating Clone of SessionState (Lazily) -- `createClone` Method

[source, scala]
----
createClone: (SparkSession, SessionState) => SessionState
----

`createClone` creates a link:spark-sql-SessionState.adoc[SessionState] (lazily as a function) using <<newBuilder, newBuilder>> followed by <<build, build>>.

NOTE: `createClone` is used when `BaseSessionStateBuilder` is requested for a <<build, SessionState>>.

=== [[build]] Creating SessionState -- `build` Method

[source, scala]
----
build(): SessionState
----

`build` creates a link:spark-sql-SessionState.adoc[SessionState] with the following:

* link:spark-sql-SparkSession.adoc#sharedState[SharedState] of the <<session, SparkSession>>
* <<conf, SQLConf>>
* <<experimentalMethods, ExperimentalMethods>>
* <<functionRegistry, FunctionRegistry>>
* <<udfRegistration, UDFRegistration>>
* <<catalog, SessionCatalog>>
* <<sqlParser, ParserInterface>>
* <<analyzer, Analyzer>>
* <<optimizer, Optimizer>>
* <<planner, SparkPlanner>>
* <<streamingQueryManager, StreamingQueryManager>>
* <<listenerManager, ExecutionListenerManager>>
* <<resourceLoader, SessionResourceLoader>>
* <<createQueryExecution, createQueryExecution>>
* <<createClone, createClone>>

[NOTE]
====
`build` is used when:

* `SparkSession` is requested for a link:spark-sql-SparkSession.adoc#sessionState[SessionState] (that in turn link:spark-sql-SparkSession.adoc#instantiateSessionState[builds one using a class name] based on link:spark-sql-StaticSQLConf.adoc#spark.sql.catalogImplementation[spark.sql.catalogImplementation] configuration property)

* `BaseSessionStateBuilder` is requested to <<createClone, create a clone>> of a `SessionState`
====

=== [[createQueryExecution]] Getting Function to Create QueryExecution For LogicalPlan -- `createQueryExecution` Method

[source, scala]
----
createQueryExecution: LogicalPlan => QueryExecution
----

`createQueryExecution` simply returns a function that takes a <<spark-sql-LogicalPlan.adoc#, LogicalPlan>> and creates a <<spark-sql-QueryExecution.adoc#creating-instance, QueryExecution>> with the <<session, SparkSession>> and the logical plan.

NOTE: `createQueryExecution` is used exclusively when `BaseSessionStateBuilder` is requested to <<build, create a SessionState instance>>.
