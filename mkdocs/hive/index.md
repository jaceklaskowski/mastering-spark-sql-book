# Hive Data Source

Spark SQL supports Apache Hive using *Hive data source*. Spark SQL allows executing structured queries on Hive tables, a persistent Hive metastore, support for Hive serdes and user-defined functions.

TIP: Consult link:../demo/demo-connecting-spark-sql-to-hive-metastore.adoc[Demo: Connecting Spark SQL to Hive Metastore (with Remote Metastore Server)] to learn in a more practical approach.

In order to use Hive-related features in a Spark SQL application a link:../spark-sql-SparkSession.adoc[SparkSession] has to be created with link:../spark-sql-SparkSession-Builder.adoc#enableHiveSupport[Builder.enableHiveSupport].

Hive Data Source uses custom link:configuration-properties.adoc[Spark SQL configuration properties] (in addition to <<hive-configuration-properties, Hive's>>).

Hive Data Source uses link:HiveTableRelation.adoc[HiveTableRelation] to represent Hive tables. `HiveTableRelation` can be link:RelationConversions.adoc#convert[converted to a HadoopFsRelation] based on link:configuration-properties.adoc#spark.sql.hive.convertMetastoreParquet[spark.sql.hive.convertMetastoreParquet] and link:configuration-properties.adoc#spark.sql.hive.convertMetastoreOrc[spark.sql.hive.convertMetastoreOrc] properties (and "disappears" from a logical plan when enabled).

Hive Data Source uses link:HiveSessionStateBuilder.adoc[HiveSessionStateBuilder] (to build a Hive-specific link:../spark-sql-SparkSession.adoc#sessionState[SessionState]) and link:HiveExternalCatalog.adoc[HiveExternalCatalog].

Hive Data Source uses link:HiveClientImpl.adoc[HiveClientImpl] for meta data/DDL operations (using calls to a Hive metastore).

## Hive Configuration Properties

Hive uses <<properties, configuration properties>> that can be defined in <<hive-configuration-files, configuration files>> and as System properties (that have a higher precedence).

In Spark applications, you could use `--driver-java-options` or `--conf spark.hadoop.[property]` to define Hive configuration properties.

```shell
$SPARK_HOME/bin/spark-shell \
  --jars \
    $HIVE_HOME/lib/hive-metastore-2.3.6.jar,\
    $HIVE_HOME/lib/hive-exec-2.3.6.jar,\
    $HIVE_HOME/lib/hive-common-2.3.6.jar,\
    $HIVE_HOME/lib/hive-serde-2.3.6.jar,\
    $HIVE_HOME/lib/guava-14.0.1.jar \
  --conf spark.sql.hive.metastore.version=2.3 \
  --conf spark.sql.hive.metastore.jars=$HIVE_HOME"/lib/*" \
  --conf spark.sql.warehouse.dir=hdfs://localhost:9000/user/hive/warehouse \
  --driver-java-options="-Dhive.exec.scratchdir=/tmp/hive-scratchdir" \
  --conf spark.hadoop.hive.exec.scratchdir=/tmp/hive-scratchdir-conf
```

[[properties]]
.HiveConf's Configuration Properties
[cols="1a",options="header",width="100%"]
|===
| Configuration Property

| [[hive.enforce.bucketing]] *hive.enforce.bucketing*

| [[hive.enforce.sorting]] *hive.enforce.sorting*

| [[hive.exec.compress.output]] *hive.exec.compress.output*

Default: `false`

| [[hive.exec.dynamic.partition]] *hive.exec.dynamic.partition*

| [[hive.exec.dynamic.partition.mode]] *hive.exec.dynamic.partition.mode*

| [[hive.exec.log4j.file]][[HIVE_EXEC_LOG4J_FILE]] *hive.exec.log4j.file*

Hive log4j configuration file for execution mode(sub command).

If the property is not set, then logging will be initialized using `hive-exec-log4j2.properties` found on the classpath.

If the property is set, the value must be a valid URI (`java.net.URI`, e.g. `file:///tmp/my-logging.xml`).

| [[hive.exec.scratchdir]][[SCRATCHDIR]] *hive.exec.scratchdir*

HDFS root scratch dir for Hive jobs which gets created with write all (733) permission. For each connecting user, an HDFS scratch dir: `hive.exec.scratchdir`/<username> is created, with <<hive.scratch.dir.permission, hive.scratch.dir.permission>>.

Default: `/tmp/hive`

| [[hive.exec.stagingdir]][[STAGINGDIR]] *hive.exec.stagingdir*

Temporary staging directory that will be created inside table locations in order to support HDFS encryption.

Default: `.hive-staging`

This replaces <<hive.exec.scratchdir, hive.exec.scratchdir>> for query results with the exception of read-only tables. In all cases <<hive.exec.scratchdir, hive.exec.scratchdir>> is still used for other temporary files, such as job plans.

| [[hive.log4j.file]][[HIVE_LOG4J_FILE]] *hive.log4j.file*

Hive log4j configuration file.

If the property is not set, then logging will be initialized using `hive-log4j2.properties` found on the classpath.

If the property is set, the value must be a valid URI (`java.net.URI`, e.g. `file:///tmp/my-logging.xml`).

| [[hive.metastore.connect.retries]][[METASTORETHRIFTCONNECTIONRETRIES]] *hive.metastore.connect.retries*

Number of retries while opening a connection to metastore

Default: `3`

| [[hive.metastore.port]][[METASTORE_SERVER_PORT]] *hive.metastore.port*

Port of the Hive metastore

Default: `9083`

| [[hive.metastore.uris]][[METASTOREURIS]] *hive.metastore.uris*

Comma-separated list of the thrift URIs of remote metastores that is used by metastore clients to connect (incl. Spark SQL applications)

| [[hive.scratch.dir.permission]][[SCRATCHDIRPERMISSION]] *hive.scratch.dir.permission*

The permission for the user specific scratch directories that get created.

Default: `700`

| [[javax.jdo.option.ConnectionDriverName]][[METASTORE_CONNECTION_DRIVER]] *javax.jdo.option.ConnectionDriverName*

Driver class name for a JDBC metastore

Default: `org.apache.derby.jdbc.EmbeddedDriver`

| [[javax.jdo.option.ConnectionPassword]][[METASTOREPWD]] *javax.jdo.option.ConnectionPassword*

Password to use against metastore database

Default: `mine`

| [[javax.jdo.option.ConnectionURL]][[METASTORECONNECTURLKEY]] *javax.jdo.option.ConnectionURL*

JDBC connect string for a JDBC metastore. To use SSL to encrypt/authenticate the connection, provide database-specific SSL flag in the connection URL (`jdbc:postgresql://myhost/db?ssl=true` for postgres database).

Default: `jdbc:derby:;databaseName=metastore_db;create=true`

| [[javax.jdo.option.ConnectionUserName]][[METASTORE_CONNECTION_USER_NAME]] *javax.jdo.option.ConnectionUserName*

Username to use against metastore database

Default: `APP`

|===

## Hive Configuration Files

`HiveConf` loads `hive-default.xml` when on the classpath.

`HiveConf` loads and prints out the location of `hive-site.xml` configuration file (when on the classpath, in `$HIVE_CONF_DIR` or `$HIVE_HOME/conf` directories, or in the directory with the jar file with `HiveConf` class).

Enable ALL logging level in `conf/log4j.properties`:

```
log4j.logger.org.apache.hadoop.hive=ALL
```

Execute the following `spark.sharedState.externalCatalog.getTable("default", "t1")` to have the following INFO message in the logs:

```
Found configuration file [url]
```

IMPORTANT: Spark SQL loads `hive-site.xml` found in `$SPARK_HOME/conf` while Hive in `$SPARK_HOME`. Make sure there are no two configuration files that could lead to hard to diagnose issues at runtime.
