# JDBC Data Source

Spark SQL supports loading data from tables using JDBC.

.JDBC
****
The *JDBC* API is the Java™ SE standard for database-independent connectivity between the Java™ programming language and a wide range of databases: SQL or NoSQL databases and tabular data sources like spreadsheets or flat files.

Read more on the JDBC API in http://www.oracle.com/technetwork/java/overview-141217.html[JDBC Overview] and in the official Java SE 8 documentation in https://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/[Java JDBC API].
****

As a Spark developer, you use <<spark-sql-DataFrameReader.adoc#jdbc, DataFrameReader.jdbc>> to load data from an external table using JDBC.

[source, scala]
----
val table = spark.read.jdbc(url, table, properties)

// Alternatively
val table = spark.read.format("jdbc").options(...).load(...)
----

These one-liners create a <<spark-sql-DataFrame.adoc#, DataFrame>> that represents the distributed process of loading data from a database and a table (with additional properties).
