title: ObjectProducerExec

# ObjectProducerExec -- Physical Operators With Single Object Output

`ObjectProducerExec` is the <<contract, extension>> of the <<spark-sql-SparkPlan.adoc#, SparkPlan contract>> for <<implementations, physical operators>> that produce a single <<outputObjAttr, object>>.

[[contract]]
.ObjectProducerExec Contract (Abstract Methods Only)
[cols="30m,70",options="header",width="100%"]
|===
| Method
| Description

| outputObjAttr
a| [[outputObjAttr]]

[source, scala]
----
outputObjAttr: Attribute
----

Used when...FIXME

|===

[[implementations]]
.ObjectProducerExecs
[cols="30,70",options="header",width="100%"]
|===
| ObjectProducerExec
| Description

| CoGroupExec
| [[CoGroupExec]]

| <<spark-sql-SparkPlan-DeserializeToObjectExec.adoc#, DeserializeToObjectExec>>
| [[DeserializeToObjectExec]]

| <<spark-sql-SparkPlan-ExternalRDDScanExec.adoc#, ExternalRDDScanExec>>
| [[ExternalRDDScanExec]]

| FlatMapGroupsInRExec
| [[FlatMapGroupsInRExec]]

| FlatMapGroupsWithStateExec
| [[FlatMapGroupsWithStateExec]]

| <<spark-sql-SparkPlan-MapElementsExec.adoc#, MapElementsExec>>
| [[MapElementsExec]]

| MapGroupsExec
| [[MapGroupsExec]]

| MapPartitionsExec
| [[MapPartitionsExec]]

|===
