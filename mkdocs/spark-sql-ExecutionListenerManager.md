title: ExecutionListenerManager

# ExecutionListenerManager -- Management Interface of QueryExecutionListeners

`ExecutionListenerManager` is the <<methods, management interface>> for `QueryExecutionListeners` that listen for execution metrics:

* Name of the action (that triggered a query execution)

* link:spark-sql-QueryExecution.adoc[QueryExecution]

* Execution time of this query (in nanoseconds)

`ExecutionListenerManager` is available as link:spark-sql-SparkSession.adoc#listenerManager[listenerManager] property of `SparkSession` (and link:spark-sql-SessionState.adoc#listenerManager[listenerManager] property of `SessionState`).

[source, scala]
----
scala> :type spark.listenerManager
org.apache.spark.sql.util.ExecutionListenerManager

scala> :type spark.sessionState.listenerManager
org.apache.spark.sql.util.ExecutionListenerManager
----

[[conf]]
[[creating-instance]]
`ExecutionListenerManager` takes a single `SparkConf` when created

While <<creating-instance, created>>, `ExecutionListenerManager` reads link:spark-sql-StaticSQLConf.adoc#spark.sql.queryExecutionListeners[spark.sql.queryExecutionListeners] configuration property with `QueryExecutionListeners` and <<register, registers>> them.

[[spark.sql.queryExecutionListeners]]
`ExecutionListenerManager` uses link:spark-sql-StaticSQLConf.adoc#spark.sql.queryExecutionListeners[spark.sql.queryExecutionListeners] configuration property as the list of `QueryExecutionListeners` that should be automatically added to newly created sessions (and registers them while <<creating-instance, being created>>).

[[methods]]
.ExecutionListenerManager's Public Methods
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| <<register, register>>
a|

[source, scala]
----
register(listener: QueryExecutionListener): Unit
----

| <<unregister, unregister>>
a|

[source, scala]
----
unregister(listener: QueryExecutionListener): Unit
----

| <<clear, clear>>
a|

[source, scala]
----
clear(): Unit
----
|===

`ExecutionListenerManager` is <<creating-instance, created>> exclusively when `BaseSessionStateBuilder` is requested for link:spark-sql-BaseSessionStateBuilder.adoc#listenerManager[ExecutionListenerManager] (while `SessionState` is link:spark-sql-BaseSessionStateBuilder.adoc#build[built]).

[[listeners]]
`ExecutionListenerManager` uses `listeners` internal registry for registered <<spark-sql-QueryExecutionListener.adoc#, QueryExecutionListeners>>.

=== [[onSuccess]] `onSuccess` Internal Method

[source, scala]
----
onSuccess(funcName: String, qe: QueryExecution, duration: Long): Unit
----

`onSuccess`...FIXME

[NOTE]
====
`onSuccess` is used when:

* `DataFrameWriter` is requested to link:spark-sql-DataFrameWriter.adoc#runCommand[run a logical command] (after it has finished with no exceptions)

* `Dataset` is requested to link:spark-sql-Dataset.adoc#withAction[withAction]
====

=== [[onFailure]] `onFailure` Internal Method

[source, scala]
----
onFailure(funcName: String, qe: QueryExecution, exception: Exception): Unit
----

`onFailure`...FIXME

[NOTE]
====
`onFailure` is used when:

* `DataFrameWriter` is requested to link:spark-sql-DataFrameWriter.adoc#runCommand[run a logical command] (after it has reported an exception)

* `Dataset` is requested to link:spark-sql-Dataset.adoc#withAction[withAction]
====

=== [[withErrorHandling]] `withErrorHandling` Internal Method

[source, scala]
----
withErrorHandling(f: QueryExecutionListener => Unit): Unit
----

`withErrorHandling`...FIXME

NOTE: `withErrorHandling` is used when `ExecutionListenerManager` is requested to <<onSuccess, onSuccess>> and <<onFailure, onFailure>>.

=== [[register]] Registering QueryExecutionListener -- `register` Method

[source, scala]
----
register(listener: QueryExecutionListener): Unit
----

Internally, `register` simply registers (adds) the input <<spark-sql-QueryExecutionListener.adoc#, QueryExecutionListener>> to the <<listeners, listeners>> internal registry.
