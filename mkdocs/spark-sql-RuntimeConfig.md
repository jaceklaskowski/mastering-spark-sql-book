title: RuntimeConfig

# RuntimeConfig -- Management Interface of Runtime Configuration

`RuntimeConfig` is the <<methods, management interface>> of the <<sqlConf, runtime configuration>>.

[[methods]]
.RuntimeConfig API
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| <<get, get>>
a|

[source, scala]
----
get(key: String): String
get(key: String, default: String): String
----

| <<getAll, getAll>>
a|

[source, scala]
----
getAll: Map[String, String]
----

| <<getOption, getOption>>
a|

[source, scala]
----
getOption(key: String): Option[String]
----

| `isModifiable`
a| [[isModifiable]]

[source, scala]
----
isModifiable(key: String): Boolean
----

(*New in 2.4.0*)

| <<set, set>>
a|

[source, scala]
----
set(key: String, value: Boolean): Unit
set(key: String, value: Long): Unit
set(key: String, value: String): Unit
----

| <<unset, unset>>
a|

[source, scala]
----
unset(key: String): Unit
----
|===

`RuntimeConfig` is available using the <<spark-sql-SparkSession.adoc#conf, conf>> attribute of a `SparkSession`.

[source, scala]
----
scala> :type spark
org.apache.spark.sql.SparkSession

scala> :type spark.conf
org.apache.spark.sql.RuntimeConfig
----

.RuntimeConfig, SparkSession and SQLConf
image::images/spark-sql-RuntimeConfig.png[align="center"]

`RuntimeConfig` is <<creating-instance, created>> exclusively when `SparkSession` is requested for <<spark-sql-SparkSession.adoc#conf, one>>.

[[sqlConf]]
[[creating-instance]]
`RuntimeConfig` takes a <<spark-sql-SQLConf.adoc#, SQLConf>> when created.

=== [[get]] `get` Method

[source, scala]
----
get(key: String): String
get(key: String, default: String): String
----

`get`...FIXME

NOTE: `get` is used when...FIXME

=== [[getAll]] `getAll` Method

[source, scala]
----
getAll: Map[String, String]
----

`getAll`...FIXME

NOTE: `getAll` is used when...FIXME

=== [[getOption]] `getOption` Method

[source, scala]
----
getOption(key: String): Option[String]
----

`getOption`...FIXME

NOTE: `getOption` is used when...FIXME

=== [[set]] `set` Method

[source, scala]
----
set(key: String, value: Boolean): Unit
set(key: String, value: Long): Unit
set(key: String, value: String): Unit
----

`set`...FIXME

NOTE: `set` is used when...FIXME

=== [[unset]] `unset` Method

[source, scala]
----
unset(key: String): Unit
----

`unset`...FIXME

NOTE: `unset` is used when...FIXME
