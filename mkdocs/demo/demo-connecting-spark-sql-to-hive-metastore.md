title: Connecting Spark SQL to Hive Metastore

# Demo: Connecting Spark SQL to Hive Metastore (with Remote Metastore Server)

:spark-version: 2.4.5
:hive-version: 2.3.6
:hadoop-version: 2.10.0
:url-hive-javadoc: https://hive.apache.org/javadocs/r{hive-version}/api
:url-hadoop-docs: https://hadoop.apache.org/docs/r{hadoop-version}
:url-hadoop-javadoc: {url-hadoop-docs}/api

The demo shows how to run Apache Spark {spark-version} with Apache Hive {hive-version} (on Apache Hadoop {hadoop-version}).

IMPORTANT: Support for Hadoop 3.x is expected in the upcoming https://issues.apache.org/jira/browse/SPARK-23710[Spark 3.0.0].

You'll be using a separate Remote Metastore Server to access table metadata via the Thrift protocol. It is in the discretion of the Remote Metastore Server to connect to the underlying JDBC-accessible relational database (e.g. PostgreSQL).

TIP: Read up https://docs.databricks.com/data/metastores/external-hive-metastore.html[External Apache Hive metastore] in the official documentation of Databricks platform that describes the topic in more details from the perspective of Apache Spark developers.

## Install Java 8

As per Hadoop's https://cwiki.apache.org/confluence/display/HADOOP/Hadoop+Java+Versions[Hadoop Java Versions]:

> Apache Hadoop from 2.7.x to 2.x support Java 7 and 8

As per Spark's https://spark.apache.org/docs/latest/#downloading[Downloading]:

> Spark runs on Java 8

Make sure you have Java 8 installed.

```
$ java -version
openjdk version "1.8.0_242"
OpenJDK Runtime Environment (AdoptOpenJDK)(build 1.8.0_242-b08)
OpenJDK 64-Bit Server VM (AdoptOpenJDK)(build 25.242-b08, mixed mode)
```

## Build Apache Spark for Apache Hadoop

Build Apache Spark with support for Apache Hadoop {hadoop-version}.

```
$ cd $SPARK_HOME
$ ./build/mvn \
    -Dhadoop.version=2.10.0 \
    -Pyarn,hive,hive-thriftserver \
    -Pscala-2.12 \
    -Pkubernetes \
    -DskipTests \
    clean install
```

```
$ ./bin/spark-shell --version
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.4.5
      /_/

Using Scala version 2.12.10, OpenJDK 64-Bit Server VM, 1.8.0_242
Branch HEAD
Compiled by user centos on 2020-02-02T21:10:50Z
Revision cee4ecbb16917fa85f02c635925e2687400aa56b
Url https://gitbox.apache.org/repos/asf/spark.git
Type --help for more information.
```

Assert the versions work in `spark-shell` before proceeding.

```
$ ./bin/spark-shell
scala> assert(spark.version == "2.4.5")

scala> assert(org.apache.hadoop.util.VersionInfo.getVersion == "2.10.0")

scala> assert(org.apache.hadoop.hive.shims.ShimLoader.getMajorVersion == "0.23")
```

## Set Up Single-Node Hadoop Cluster

_Hive uses Hadoop._

Download and install https://hadoop.apache.org/release/{hadoop-version}.html[Hadoop {hadoop-version}] (or more recent stable release of Apache Hadoop 2 line if available).

```
export HADOOP_HOME=/Users/jacek/dev/apps/hadoop
```

Follow the official documentation in {url-hadoop-docs}/hadoop-project-dist/hadoop-common/SingleCluster.html[Hadoop: Setting up a Single Node Cluster] to set up a single-node Hadoop installation.

```
$ $HADOOP_HOME/bin/hadoop version
Hadoop 2.10.0
Subversion ssh://git.corp.linkedin.com:29418/hadoop/hadoop.git -r e2f1f118e465e787d8567dfa6e2f3b72a0eb9194
Compiled by jhung on 2019-10-22T19:10Z
Compiled with protoc 2.5.0
From source with checksum 7b2d8877c5ce8c9a2cca5c7e81aa4026
This command was run using /Users/jacek/dev/apps/hadoop-2.10.0/share/hadoop/common/hadoop-common-2.10.0.jar
```

This demo assumes running {url-hadoop-docs}/hadoop-project-dist/hadoop-common/SingleCluster.html#Pseudo-Distributed_Operation[a single-node in a pseudo-distributed mode where each Hadoop daemon runs in a separate Java process].

[TIP]
====
Use `hadoop.tmp.dir` configuration property as the base for temporary directories.

[source, xml]
----
<property>
  <name>hadoop.tmp.dir</name>
  <value>/tmp/my-hadoop-tmp-dir/hdfs/tmp</value>
  <description>The base for temporary directories.</description>
</property>
----

Use `./bin/hdfs getconf -confKey hadoop.tmp.dir` to check out the value

```
$ ./bin/hdfs getconf -confKey hadoop.tmp.dir
/tmp/my-hadoop-tmp-dir/hdfs/tmp
```
====

## fs.defaultFS Configuration Property (core-site.xml)

Edit `etc/hadoop/core-site.xml` and define `fs.defaultFS` and `hadoop.proxyuser.` properties.

[source, xml]
----
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
      <name>hadoop.proxyuser.[username].groups</name>
      <value>*</value>
    </property>
    <property>
      <name>hadoop.proxyuser.[username].hosts</name>
      <value>*</value>
    </property>
</configuration>
----

IMPORTANT: Replace `[username]` above with the local user (e.g. `jacek`) that will be used in `beeline`. Consult https://stackoverflow.com/q/43180305/1305344[this question] on StackOverflow.

## dfs.replication Configuration Property (hdfs-site.xml)

Edit `etc/hadoop/hdfs-site.xml` and define `dfs.replication` property as follows:

[source, xml]
----
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
----

## Passphrase-less SSH (macOS)

Turn *Remote Login* on in Mac OS X's Sharing preferences that allow remote users to connect to a Mac using the OpenSSH protocols.

```
$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa_hadoop
$ cat ~/.ssh/id_rsa_hadoop.pub >> ~/.ssh/authorized_keys
$ chmod 0600 ~/.ssh/authorized_keys
```

## Other Steps

You may want to set up `JAVA_HOME` in `etc/hadoop/hadoop-env.sh` as told in the file:

[quote]
----
# The only required environment variable is JAVA_HOME.  All others are
# optional.  When running a distributed configuration it is best to
# set JAVA_HOME in this file, so that it is correctly defined on
# remote nodes.
----

```
$ $HADOOP_HOME/bin/hdfs namenode -format
...
INFO common.Storage: Storage directory /tmp/hadoop-jacek/dfs/name has been successfully formatted.
...
```

[NOTE]
====
Use `./bin/hdfs namenode` to start a NameNode that will tell you that the local filesystem is not ready.

```
$ ./bin/hdfs namenode
18/01/09 15:43:11 INFO namenode.NameNode: STARTUP_MSG:
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = japila.local/192.168.1.2
STARTUP_MSG:   args = []
STARTUP_MSG:   version = 2.7.5
...
18/01/09 15:43:11 INFO namenode.NameNode: fs.defaultFS is hdfs://localhost:9000
18/01/09 15:43:11 INFO namenode.NameNode: Clients are to use localhost:9000 to access this namenode/service.
...
18/01/09 15:43:12 INFO hdfs.DFSUtil: Starting Web-server for hdfs at: http://0.0.0.0:50070
...
18/01/09 15:43:13 WARN common.Storage: Storage directory /private/tmp/hadoop-jacek/dfs/name does not exist
18/01/09 15:43:13 WARN namenode.FSNamesystem: Encountered exception loading fsimage
org.apache.hadoop.hdfs.server.common.InconsistentFSStateException: Directory /private/tmp/hadoop-jacek/dfs/name is in an inconsistent state: storage directory does not exist or is not accessible.
	at org.apache.hadoop.hdfs.server.namenode.FSImage.recoverStorageDirs(FSImage.java:382)
	at org.apache.hadoop.hdfs.server.namenode.FSImage.recoverTransitionRead(FSImage.java:233)
	at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.loadFSImage(FSNamesystem.java:984)
	at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.loadFromDisk(FSNamesystem.java:686)
	at org.apache.hadoop.hdfs.server.namenode.NameNode.loadNamesystem(NameNode.java:586)
	at org.apache.hadoop.hdfs.server.namenode.NameNode.initialize(NameNode.java:646)
	at org.apache.hadoop.hdfs.server.namenode.NameNode.<init>(NameNode.java:820)
	at org.apache.hadoop.hdfs.server.namenode.NameNode.<init>(NameNode.java:804)
	at org.apache.hadoop.hdfs.server.namenode.NameNode.createNameNode(NameNode.java:1516)
	at org.apache.hadoop.hdfs.server.namenode.NameNode.main(NameNode.java:1582)
...
18/01/09 15:43:13 ERROR namenode.NameNode: Failed to start namenode.
org.apache.hadoop.hdfs.server.common.InconsistentFSStateException: Directory /private/tmp/hadoop-jacek/dfs/name is in an inconsistent state: storage directory does not exist or is not accessible.
	at org.apache.hadoop.hdfs.server.namenode.FSImage.recoverStorageDirs(FSImage.java:382)
	at org.apache.hadoop.hdfs.server.namenode.FSImage.recoverTransitionRead(FSImage.java:233)
	at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.loadFSImage(FSNamesystem.java:984)
	at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.loadFromDisk(FSNamesystem.java:686)
	at org.apache.hadoop.hdfs.server.namenode.NameNode.loadNamesystem(NameNode.java:586)
	at org.apache.hadoop.hdfs.server.namenode.NameNode.initialize(NameNode.java:646)
	at org.apache.hadoop.hdfs.server.namenode.NameNode.<init>(NameNode.java:820)
	at org.apache.hadoop.hdfs.server.namenode.NameNode.<init>(NameNode.java:804)
	at org.apache.hadoop.hdfs.server.namenode.NameNode.createNameNode(NameNode.java:1516)
	at org.apache.hadoop.hdfs.server.namenode.NameNode.main(NameNode.java:1582)
```
====

Start Hadoop DFS using `start-dfs.sh` (and `tail -f logs/hadoop-\*-datanode-*.log`)

```
$ $HADOOP_HOME/sbin/start-dfs.sh
Starting namenodes on [localhost]
localhost: starting namenode, logging to /Users/jacek/dev/apps/hadoop-2.10.0/logs/hadoop-jacek-namenode-japila-new.local.out
localhost: starting datanode, logging to /Users/jacek/dev/apps/hadoop-2.10.0/logs/hadoop-jacek-datanode-japila-new.local.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /Users/jacek/dev/apps/hadoop-2.10.0/logs/hadoop-jacek-secondarynamenode-japila-new.local.out
```

List Hadoop's JVM processes using `jps -lm`.

```
$ jps -lm
50773 org.apache.hadoop.hdfs.server.datanode.DataNode
50870 org.apache.hadoop.hdfs.server.namenode.SecondaryNameNode
50695 org.apache.hadoop.hdfs.server.namenode.NameNode
```

NOTE: FIXME Are the steps in {url-hadoop-docs}/hadoop-project-dist/hadoop-common/SingleCluster.html#YARN_on_a_Single_Node[YARN on a Single Node] required for Hive?

## Running Hive

NOTE: Following the steps in https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-RunningHive[Running Hive].

```
$HADOOP_HOME/bin/hadoop fs -mkdir /tmp
$HADOOP_HOME/bin/hadoop fs -chmod g+w /tmp
```

```
$HADOOP_HOME/bin/hadoop fs -mkdir -p /user/hive/warehouse
$HADOOP_HOME/bin/hadoop fs -chmod g+w /user/hive/warehouse
```

Download and install http://hive.apache.org/downloads.html[Hive {hive-version}] (or more recent stable release of Apache Hive 2 line if available).

```
export HIVE_HOME=/Users/jacek/dev/apps/hive
```

## Install PostgreSQL

You'll set up a remote metastore database (as https://cwiki.apache.org/confluence/display/Hive/AdminManual+Metastore+Administration#AdminManualMetastoreAdministration-RemoteMetastoreDatabase[This configuration of metastore database is recommended for any real use.]) and you'll be using https://www.enterprisedb.com/downloads/postgres-postgresql-downloads[PostgreSQL 12.2].

```
$ pg_ctl -D /usr/local/var/postgres start
server started
```

Download the most current version of https://jdbc.postgresql.org/download.html#current[PostgreSQL JDBC Driver], e.g. PostgreSQL JDBC 4.2 Driver, 42.2.11. Save the jar file (`postgresql-42.2.11.jar`) in `$HIVE_HOME/lib`.

## Setting Up Remote Metastore Database

Create a database and a user in PostgreSQL for Hive.

```
createdb hive_demo
```

```
createuser APP
```

Create `conf/hive-site.xml` (based on `conf/hive-default.xml.template`) with the following properties:

[source, xml]
----
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:postgresql://localhost:5432/hive_demo</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>org.postgresql.Driver</value>
  </property>
  <property>
    <name>hive.metastore.warehouse.dir</name>
    <value>hdfs://localhost:9000/user/hive/warehouse</value>
  </property>
</configuration>
----

Use the https://cwiki.apache.org/confluence/display/Hive/Hive+Schema+Tool[Hive Schema Tool] to create the metastore tables.

```
$ $HIVE_HOME/bin/schematool -dbType postgres -initSchema
Metastore connection URL:	 jdbc:postgresql://localhost:5432/hive_demo
Metastore Connection Driver :	 org.postgresql.Driver
Metastore connection User:	 APP
Starting metastore schema initialization to 2.3.0
Initialization script hive-schema-2.3.0.postgres.sql
Initialization script completed
schemaTool completed
```

```
$ $HIVE_HOME/bin/schematool -dbType postgres -info
Metastore connection URL:	 jdbc:postgresql://localhost:5432/hive_demo
Metastore Connection Driver :	 org.postgresql.Driver
Metastore connection User:	 APP
Hive distribution version:	 2.3.0
Metastore schema version:	 2.3.0
schemaTool completed
```

As per the https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-RunningHiveServer2andBeeline.1[official documentation of Hive]:

> HiveCLI is now deprecated in favor of Beeline

Run HiveServer2.

```
$HIVE_HOME/bin/hiveserver2
```

Run Beeline (the HiveServer2 CLI).

```
$ $HIVE_HOME/bin/beeline -u jdbc:hive2://localhost:10000
...
Connecting to jdbc:hive2://localhost:10000
Connected to: Apache Hive (version 2.3.6)
Driver: Hive JDBC (version 2.3.6)
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 2.3.6 by Apache Hive
0: jdbc:hive2://localhost:10000>
```

## Start Hive Metastore Server

Start the Hive Metastore Server (as described in https://cwiki.apache.org/confluence/display/Hive/AdminManual+Metastore+Administration#AdminManualMetastoreAdministration-RemoteMetastoreServer[Remote Metastore Server]).
```
$HIVE_HOME/bin/hive --service metastore
...
Starting Hive Metastore Server
```

That is the server Spark SQL applications are going to connect to for metadata of Hive tables.

## Connecting Apache Spark to Apache Hive

Create `$SPARK_HOME/conf/hive-site.xml` and define `hive.metastore.uris` configuration property (that is the thrift URL of the Hive Metastore Server).

[source, xml]
----
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>hive.metastore.uris</name>
    <value>thrift://localhost:9083</value>
  </property>
</configuration>
----

Optionally, you may want to add the following to `conf/log4j.properties` for a more low-level logging:

```
log4j.logger.org.apache.spark.sql.hive.HiveUtils$=ALL
log4j.logger.org.apache.spark.sql.internal.SharedState=ALL
log4j.logger.org.apache.spark.sql.hive.client.HiveClientImpl=ALL
```

Start `spark-shell`.

```
$SPARK_HOME/bin/spark-shell \
  --jars \
    $HIVE_HOME/lib/hive-metastore-2.3.6.jar,\
    $HIVE_HOME/lib/hive-exec-2.3.6.jar,\
    $HIVE_HOME/lib/hive-common-2.3.6.jar,\
    $HIVE_HOME/lib/hive-serde-2.3.6.jar,\
    $HIVE_HOME/lib/guava-14.0.1.jar \
  --conf spark.sql.hive.metastore.version=2.3 \
  --conf spark.sql.hive.metastore.jars=$HIVE_HOME"/lib/*" \
  --conf spark.sql.warehouse.dir=hdfs://localhost:9000/user/hive/warehouse
```

You should see the following welcome message:

```
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.4.5
      /_/

Using Scala version 2.12.10 (OpenJDK 64-Bit Server VM, Java 1.8.0_242)
Type in expressions to have them evaluated.
Type :help for more information.

scala>
```

With the `scala>` prompt you made sure that the `spark.sql.hive.metastore.version` and the JAR files are all correct (as the check happens while the `SparkSession` is created). _Congratulations!_

You may also want to check out the `spark.sql.catalogImplementation` internal property that should be `hive`. With the extra logging turned on, you should also see the configuration file loaded (`hive-site.xml`) and the warehouse location.

```
scala> spark.conf.get("spark.sql.catalogImplementation")
INFO SharedState: loading hive config file: file:/Users/jacek/dev/oss/spark/conf/hive-site.xml
INFO SharedState: Setting hive.metastore.warehouse.dir ('null') to the value of spark.sql.warehouse.dir ('hdfs://localhost:9000/user/hive/warehouse').
INFO SharedState: Warehouse path is 'hdfs://localhost:9000/user/hive/warehouse'.
res0: String = hive
```

The most critical step is to check out the remote connection with the Hive Metastore Server (via the thrift protocol). Execute the following command to list all tables known to Spark SQL (incl. Hive tables if there were any, but there are none by default).

```
scala> spark.catalog.listTables.show
+----+--------+-----------+---------+-----------+
|name|database|description|tableType|isTemporary|
+----+--------+-----------+---------+-----------+
+----+--------+-----------+---------+-----------+
```

There is one database in Hive by default.

```
0: jdbc:hive2://localhost:10000> show databases;
+----------------+
| database_name  |
+----------------+
| default        |
+----------------+
1 row selected (0.067 seconds)
```

List the tables in the `default` database. There should be some Hive tables listed.

```
scala> spark.sharedState.externalCatalog.listTables("default")
```

Create a partitioned table in Hive (based on the https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-CreateTableCreate/Drop/TruncateTable[official documentation of Hive]). Execute the following DDL in beeline.

```
CREATE TABLE demo_sales
(id BIGINT, qty BIGINT, name STRING)
COMMENT 'Demo: Connecting Spark SQL to Hive Metastore'
PARTITIONED BY (rx_mth_cd STRING COMMENT 'Prescription Date YYYYMM aggregated')
STORED AS PARQUET;
```

[TIP]
====
You can also create a Hive table from `spark-shell` using `CREATE TABLE ... USING hive` queries.

```
CREATE TABLE hive_table_name (id LONG) USING hive
```
====

In case of permission denied errors as the one below:

```
MetaException(message:Got exception: org.apache.hadoop.security.AccessControlException Permission denied: user=anonymous, access=WRITE, inode="/user/hive/warehouse":jacek:supergroup:drwxrwxr-x
```

you may want to simply change the permissions of the warehouse directory to allow anybody to write:

```
$ $HADOOP_HOME/bin/hadoop fs -chmod 777 /user/hive/warehouse
$ $HADOOP_HOME/bin/hadoop fs -ls /user/hive
Found 1 items
drwxrwxrwx   - jacek supergroup          0 2020-03-21 11:15 /user/hive/warehouse
```

Check out the table directory on HDFS.

```
$ $HADOOP_HOME/bin/hadoop fs -ls /user/hive/warehouse
Found 1 items
drwxrwxrwx   - anonymous supergroup          0 2020-03-22 16:07 /user/hive/warehouse/demo_sales
```

Insert some data.

```
# (id BIGINT, qty BIGINT, name STRING)
# PARTITIONED BY (rx_mth_cd STRING COMMENT 'Prescription Date YYYYMM aggregated')
INSERT INTO demo_sales PARTITION (rx_mth_cd="202002") VALUES (2, 2000, 'two');
```

Query the records in the table.

```
0: jdbc:hive2://localhost:10000> SELECT * FROM demo_sales;
+----------------+-----------------+------------------+-----------------------+
| demo_sales.id  | demo_sales.qty  | demo_sales.name  | demo_sales.rx_mth_cd  |
+----------------+-----------------+------------------+-----------------------+
| 2              | 2000            | two              | 202002                |
+----------------+-----------------+------------------+-----------------------+
1 row selected (0.112 seconds)
```

Display the partitions (there should really be one).

```
0: jdbc:hive2://localhost:10000> SHOW PARTITIONS demo_sales;
+-------------------+
|     partition     |
+-------------------+
| rx_mth_cd=202002  |
+-------------------+
1 row selected (0.084 seconds)
```

Check out the table directory on HDFS.

```
$ $HADOOP_HOME/bin/hadoop fs -ls -R /user/hive/warehouse/demo_sales
drwxrwxrwx   - anonymous supergroup          0 2020-03-22 16:10 /user/hive/warehouse/demo_sales/rx_mth_cd=202002
-rwxrwxrwx   1 anonymous supergroup        454 2020-03-22 16:10 /user/hive/warehouse/demo_sales/rx_mth_cd=202002/000000_0
```

Time for some Spark.

Query the tables in the `default` database. There should be at least the one you've just created.

```
scala> spark.sharedState.externalCatalog.listTables("default")
res6: Seq[String] = Buffer(demo_sales)
```

Query the rows in the table.

```
scala> spark.table("demo_sales").show
+---+----+----+---------+
| id| qty|name|rx_mth_cd|
+---+----+----+---------+
|  2|2000| two|   202002|
+---+----+----+---------+
```

Display the metadata of the table from the Spark catalog (`DESCRIBE EXTENDED` SQL command).

```
scala> sql("DESCRIBE EXTENDED demo_sales").show(Integer.MAX_VALUE, truncate = false)
+----------------------------+--------------------------------------------------------------+-----------------------------------+
|col_name                    |data_type                                                     |comment                            |
+----------------------------+--------------------------------------------------------------+-----------------------------------+
|id                          |bigint                                                        |null                               |
|qty                         |bigint                                                        |null                               |
|name                        |string                                                        |null                               |
|rx_mth_cd                   |string                                                        |Prescription Date YYYYMM aggregated|
|# Partition Information     |                                                              |                                   |
|# col_name                  |data_type                                                     |comment                            |
|rx_mth_cd                   |string                                                        |Prescription Date YYYYMM aggregated|
|                            |                                                              |                                   |
|# Detailed Table Information|                                                              |                                   |
|Database                    |default                                                       |                                   |
|Table                       |demo_sales                                                    |                                   |
|Owner                       |anonymous                                                     |                                   |
|Created Time                |Sun Mar 22 16:09:18 CET 2020                                  |                                   |
|Last Access                 |Thu Jan 01 01:00:00 CET 1970                                  |                                   |
|Created By                  |Spark 2.2 or prior                                            |                                   |
|Type                        |MANAGED                                                       |                                   |
|Provider                    |hive                                                          |                                   |
|Comment                     |Demo: Connecting Spark SQL to Hive Metastore                  |                                   |
|Table Properties            |[transient_lastDdlTime=1584889905]                            |                                   |
|Location                    |hdfs://localhost:9000/user/hive/warehouse/demo_sales          |                                   |
|Serde Library               |org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe   |                                   |
|InputFormat                 |org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |                                   |
|OutputFormat                |org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat|                                   |
|Storage Properties          |[serialization.format=1]                                      |                                   |
|Partition Provider          |Catalog                                                       |                                   |
+----------------------------+--------------------------------------------------------------+-----------------------------------+
```

It all worked fine. _Congratulations!_
