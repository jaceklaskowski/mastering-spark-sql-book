title: TableScan

# TableScan -- Relations with Column Pruning

`TableScan` is the <<contract, contract>> of <<implementations, BaseRelations>> with support for <<buildScan, column pruning>>, i.e. can eliminate unneeded columns before producing an RDD containing all of its tuples as `Row` objects.

[[contract]]
[source, scala]
----
package org.apache.spark.sql.sources

trait PrunedScan {
  def buildScan(): RDD[Row]
}
----

.TableScan Contract
[cols="1m,2",options="header",width="100%"]
|===
| Property
| Description

| buildScan
| [[buildScan]] Building distributed data scan with column pruning

In other words, `buildScan` creates a `RDD[Row]` to represent a distributed data scan (i.e. scanning over data in a relation).

Used exclusively when `DataSourceStrategy` execution planning strategy is requested to link:spark-sql-SparkStrategy-DataSourceStrategy.adoc#TableScan[plan a LogicalRelation with a TableScan].
|===

[[implementations]]
NOTE: <<spark-sql-KafkaRelation.adoc#, KafkaRelation>> is the one and only known implementation of the <<contract, TableScan Contract>> in Spark SQL.
