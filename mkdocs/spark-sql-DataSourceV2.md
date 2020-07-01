title: DataSourceV2

# DataSourceV2 -- Data Sources in Data Source API V2

`DataSourceV2` is the fundamental abstraction of the <<implementations, data sources>> in the <<spark-sql-data-source-api-v2.adoc#, Data Source API V2>>.

[[contract]]
`DataSourceV2` defines no methods or values and simply acts as a *marker interface*.

[source, java]
----
package org.apache.spark.sql.sources.v2;

public interface DataSourceV2 {}
----

[NOTE]
====
Implementations should at least use <<spark-sql-ReadSupport.adoc#, ReadSupport>> or <<spark-sql-WriteSupport.adoc#, WriteSupport>> interfaces for readable or writable data sources, respectively.

Otherwise, an `AnalysisException` is thrown:

```
org.apache.spark.sql.AnalysisException: dawid is not a valid Spark SQL Data Source.;
  at org.apache.spark.sql.execution.datasources.DataSource.resolveRelation(DataSource.scala:386)
  at org.apache.spark.sql.DataFrameReader.loadV1Source(DataFrameReader.scala:223)
  at org.apache.spark.sql.DataFrameReader.load(DataFrameReader.scala:208)
  at org.apache.spark.sql.DataFrameReader.load(DataFrameReader.scala:167)
  ... 49 elided
```
====

[NOTE]
====
`DataSourceV2` is an `Evolving` contract that is evolving towards becoming a stable API, but is not a stable API yet and can change from one feature release to another release.

In other words, using the contract is as treading on thin ice.
====

[[implementations]]
.DataSourceV2s
[cols="1m,3",options="header",width="100%"]
|===
| DataSourceV2
| Description

| ConsoleSinkProvider
| [[ConsoleSinkProvider]] Used in Spark Structured Streaming

| ContinuousReadSupport
| [[ContinuousReadSupport]] Used in Spark Structured Streaming

| MemorySinkV2
| [[MemorySinkV2]] Used in Spark Structured Streaming

| MicroBatchReadSupport
| [[MicroBatchReadSupport]] Used in Spark Structured Streaming

| RateSourceProvider
| [[RateSourceProvider]] Used in Spark Structured Streaming

| RateSourceProviderV2
| [[RateSourceProviderV2]] Used in Spark Structured Streaming

| ReadSupport
| [[ReadSupport]]

| ReadSupportWithSchema
| [[ReadSupportWithSchema]]

| <<spark-sql-SessionConfigSupport.adoc#, SessionConfigSupport>>
| [[SessionConfigSupport]]

| StreamWriteSupport
| [[StreamWriteSupport]]

| WriteSupport
| [[WriteSupport]]

|===
