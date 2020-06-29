# IsolatedClientLoader Utility

:hive-version: 2.3.6
:hadoop-version: 2.10.0
:url-hive-javadoc: https://hive.apache.org/javadocs/r{hive-version}/api
:url-hadoop-javadoc: https://hadoop.apache.org/docs/r{hadoop-version}/api

`IsolatedClientLoader` is <<creating-instance, created>> for `HiveUtils` utility for link:HiveUtils.adoc#newClientForExecution[newClientForExecution] and link:HiveUtils.adoc#newClientForMetadata[newClientForMetadata].

[[logging]]
[TIP]
====
Enable `ALL` logging level for `org.apache.spark.sql.hive.client.IsolatedClientLoader` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.hive.client.IsolatedClientLoader=ALL
```

Refer to link:../spark-logging.adoc[Logging].
====

=== [[creating-instance]] Creating IsolatedClientLoader Instance

`IsolatedClientLoader` takes the following to be created:

* [[version]] `HiveVersion`
* [[sparkConf]] `SparkConf`
* [[hadoopConf]] Hadoop {url-hadoop-javadoc}/org/apache/hadoop/conf/Configuration.html[Configuration]
* [[execJars]] Execution JARs (default: empty)
* [[config]] Configuration (default: empty)
* [[isolationOn]] `isolationOn` flag (default: `true`)
* [[sharesHadoopClasses]] `sharesHadoopClasses` flag (default: `true`)
* [[rootClassLoader]] Root `ClassLoader` (default: `ClassLoader.getSystemClassLoader.getParent.getParent`)
* [[baseClassLoader]] Base class `ClassLoader` (default: `Thread.currentThread().getContextClassLoader`)
* [[sharedPrefixes]] Shared prefixes (default: empty)
* [[barrierPrefixes]] Barrier prefixes (default: empty)

`IsolatedClientLoader` initializes the <<internal-properties, internal properties>>.

=== [[hiveVersion]] Hive Metastore Version -- `hiveVersion` Utility

[source, scala]
----
hiveVersion(
  version: String): HiveVersion
----

`hiveVersion` creates a `HiveVersion` based on the input `version`.

Acceptable versions and synonyms:

* `12`, `0.12`, `0.12.0`
* `13`, `0.13`, `0.13.0`, `0.13.1`
* `14`, `0.14`, `0.14.0`
* `1.0`, `1.0.0`
* `1.1`, `1.1.0`
* `1.2`, `1.2.0`, `1.2.1`, `1.2.2`
* `2.0`, `2.0.0`, `2.0.1`
* `2.1`, `2.1.0`, `2.1.1`
* `2.2`, `2.2.0`
* `2.3`, `2.3.0`, `2.3.1`, `2.3.2`, `2.3.3`

[NOTE]
====
`hiveVersion` is used when:

* `HiveUtils` utility is used to link:HiveUtils.adoc#newClientForExecution[newClientForExecution] and link:HiveUtils.adoc#newClientForMetadata[newClientForMetadata]

* `IsolatedClientLoader` utility is used for an <<forVersion, IsolatedClientLoader for a given Hive metastore version>>
====

=== [[forVersion]] Creating IsolatedClientLoader for Given Hive Metastore Version -- `forVersion` Utility

[source, scala]
----
forVersion(
  hiveMetastoreVersion: String,
  hadoopVersion: String,
  sparkConf: SparkConf,
  hadoopConf: Configuration,
  config: Map[String, String] = Map.empty,
  ivyPath: Option[String] = None,
  sharedPrefixes: Seq[String] = Seq.empty,
  barrierPrefixes: Seq[String] = Seq.empty,
  sharesHadoopClasses: Boolean = true): IsolatedClientLoader
----

`forVersion`...FIXME

NOTE: `forVersion` is used when `HiveUtils` utility is used to link:HiveUtils.adoc#newClientForMetadata[newClientForMetadata].

=== [[createClient]] Creating HiveClient -- `createClient` Method

[source, scala]
----
createClient(): HiveClient
----

`createClient`...FIXME

[NOTE]
====
`createClient` is used when:

* `HiveUtils` utility is used to link:HiveUtils.adoc#newClientForExecution[newClientForExecution] and link:HiveUtils.adoc#newClientForMetadata[newClientForMetadata]

* `HiveClientImpl` is requested for a link:HiveClientImpl.adoc#newSession[new HiveClientImpl]
====
