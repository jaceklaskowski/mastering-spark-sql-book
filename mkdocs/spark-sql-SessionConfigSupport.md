title: SessionConfigSupport

# SessionConfigSupport -- Data Sources with Session-Scoped Configuration Options

`SessionConfigSupport` is the <<contract, contract>> of <<implementations, DataSourceV2 data sources>> in <<spark-sql-data-source-api-v2.adoc#, Data Source API V2>> that use <<keyPrefix, custom key prefix for configuration options>> (i.e. options with *spark.datasource* prefix for the keys in <<spark-sql-SQLConf.adoc#, SQLConf>>).

With `SessionConfigSupport`, a data source can be configured by additional (session-scoped) configuration options that are specified in <<spark-sql-SparkSession.adoc#, SparkSession>> that extend user-defined options.

[[contract]]
[[keyPrefix]]
[source, java]
----
String keyPrefix()
----

`keyPrefix` is used exclusively when `DataSourceV2Utils` object is requested to <<spark-sql-DataSourceV2Utils.adoc#extractSessionConfigs, extract session configuration options>> (i.e. options with *spark.datasource* prefix for the keys) for <<spark-sql-DataSourceV2.adoc#, DataSourceV2>> data sources with <<spark-sql-SessionConfigSupport.adoc#, SessionConfigSupport>>.

`keyPrefix` must not be `null` or an `IllegalArgumentException` is thrown:

```
The data source config key prefix can't be null.
```
