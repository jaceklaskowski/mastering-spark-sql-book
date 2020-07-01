title: PrunedFilteredScan

# PrunedFilteredScan -- Relations with Column Pruning and Filter Pushdown

`PrunedFilteredScan` is the <<contract, contract>> of <<implementations, BaseRelations>> with support for <<buildScan, column pruning>> (i.e. eliminating unneeded columns) and <<buildScan, filter pushdown>> (i.e. filtering using selected predicates only).

[[contract]]
[source, scala]
----
package org.apache.spark.sql.sources

trait PrunedFilteredScan {
  def buildScan(requiredColumns: Array[String], filters: Array[Filter]): RDD[Row]
}
----

.PrunedFilteredScan Contract
[cols="1,2",options="header",width="100%"]
|===
| Property
| Description

| `buildScan`
| [[buildScan]] Building distributed data scan with column pruning and filter pushdown

In other words, `buildScan` creates a `RDD[Row]` to represent a distributed data scan (i.e. scanning over data in a relation)

Used exclusively when `DataSourceStrategy` execution planning strategy is requested to link:spark-sql-SparkStrategy-DataSourceStrategy.adoc#PrunedFilteredScan[plan a LogicalRelation with a PrunedFilteredScan].
|===

NOTE: `PrunedFilteredScan` is a "lighter" and stable version of the <<spark-sql-CatalystScan.adoc#, CatalystScan Contract>>.

[[implementations]]
NOTE: <<spark-sql-JDBCRelation.adoc#, JDBCRelation>> is the one and only known implementation of the <<contract, PrunedFilteredScan Contract>> in Spark SQL.

[[example]]
[source, scala]
----
// Use :paste to define MyBaseRelation case class
// BEGIN
import org.apache.spark.sql.sources.PrunedFilteredScan
import org.apache.spark.sql.sources.BaseRelation
import org.apache.spark.sql.types.{StructField, StructType, StringType}
import org.apache.spark.sql.SQLContext
import org.apache.spark.sql.sources.Filter
import org.apache.spark.rdd.RDD
import org.apache.spark.sql.Row
case class MyBaseRelation(sqlContext: SQLContext) extends BaseRelation with PrunedFilteredScan {
  override def schema: StructType = StructType(StructField("a", StringType) :: Nil)
  def buildScan(requiredColumns: Array[String], filters: Array[Filter]): RDD[Row] = {
    println(s">>> [buildScan] requiredColumns = ${requiredColumns.mkString(",")}")
    println(s">>> [buildScan] filters = ${filters.mkString(",")}")
    import sqlContext.implicits._
    (0 to 4).toDF.rdd
  }
}
// END
val scan = MyBaseRelation(spark.sqlContext)

import org.apache.spark.sql.catalyst.plans.logical.LogicalPlan
import org.apache.spark.sql.execution.datasources.LogicalRelation
val plan: LogicalPlan = LogicalRelation(scan)

scala> println(plan.numberedTreeString)
00 Relation[a#1] MyBaseRelation(org.apache.spark.sql.SQLContext@4a57ad67)

import org.apache.spark.sql.execution.datasources.DataSourceStrategy
val strategy = DataSourceStrategy(spark.sessionState.conf)

val sparkPlan = strategy(plan).head
// >>> [buildScan] requiredColumns = a
// >>> [buildScan] filters =
scala> println(sparkPlan.numberedTreeString)
00 Scan MyBaseRelation(org.apache.spark.sql.SQLContext@4a57ad67) [a#8] PushedFilters: [], ReadSchema: struct<a:string>
----
