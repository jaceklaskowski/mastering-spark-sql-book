# DataWriterFactory

`DataWriterFactory` is a <<contract, contract>>...FIXME

[[contract]]
[source, java]
----
package org.apache.spark.sql.sources.v2.writer;

public interface DataWriterFactory<T> extends Serializable {
  DataWriter<T> createDataWriter(int partitionId, int attemptNumber);
}
----

[NOTE]
====
`DataWriterFactory` is an `Evolving` contract that is evolving towards becoming a stable API, but is not a stable API yet and can change from one feature release to another release.

In other words, using the contract is as treading on thin ice.
====

.DataWriterFactory Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| [[createDataWriter]] `createDataWriter`
a| Gives the link:spark-sql-DataWriter.adoc[DataWriter] for a partition ID and attempt number

Used when:

* `InternalRowDataWriterFactory` is requested to link:spark-sql-InternalRowDataWriterFactory.adoc#createDataWriter[createDataWriter]

* `DataWritingSparkTask` is requested to link:spark-sql-DataWritingSparkTask.adoc#run[run] and link:spark-sql-DataWritingSparkTask.adoc#runContinuous[runContinuous]
|===
