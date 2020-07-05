# ExternalRDD Leaf Logical Operator

`ExternalRDD` is a link:spark-sql-LogicalPlan-LeafNode.adoc[leaf logical operator] that is a logical representation of (the data from) an RDD in a logical query plan.

`ExternalRDD` is <<creating-instance, created>> when:

* `SparkSession` is requested to create a link:spark-sql-SparkSession.adoc#createDataFrame[DataFrame from RDD of product types] (e.g. Scala case classes, tuples) or link:spark-sql-SparkSession.adoc#createDataset[Dataset from RDD of a given type]

* `ExternalRDD` is requested to <<newInstance, create a new instance>>

[source, scala]
----
val pairsRDD = sc.parallelize((0, "zero") :: (1, "one") :: (2, "two") :: Nil)

// A tuple of Int and String is a product type
scala> :type pairsRDD
org.apache.spark.rdd.RDD[(Int, String)]

val pairsDF = spark.createDataFrame(pairsRDD)

// ExternalRDD represents the pairsRDD in the query plan
val logicalPlan = pairsDF.queryExecution.logical
scala> println(logicalPlan.numberedTreeString)
00 SerializeFromObject [assertnotnull(assertnotnull(input[0, scala.Tuple2, true]))._1 AS _1#10, staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, assertnotnull(assertnotnull(input[0, scala.Tuple2, true]))._2, true, false) AS _2#11]
01 +- ExternalRDD [obj#9]
----

`ExternalRDD` is a <<newInstance, MultiInstanceRelation>> and a `ObjectProducer`.

NOTE: `ExternalRDD` is resolved to link:spark-sql-SparkPlan-ExternalRDDScanExec.adoc[ExternalRDDScanExec] when `BasicOperators` execution planning strategy is link:spark-sql-SparkStrategy-BasicOperators.adoc#ExternalRDD[executed].

=== [[newInstance]] `newInstance` Method

[source, scala]
----
newInstance(): LogicalRDD.this.type
----

NOTE: `newInstance` is part of link:spark-sql-MultiInstanceRelation.adoc#newInstance[MultiInstanceRelation Contract] to...FIXME.

`newInstance`...FIXME

=== [[computeStats]] Computing Statistics -- `computeStats` Method

[source, scala]
----
computeStats(): Statistics
----

NOTE: `computeStats` is part of link:spark-sql-LogicalPlan-LeafNode.adoc#computeStats[LeafNode Contract] to compute statistics for link:spark-sql-cost-based-optimization.adoc[cost-based optimizer].

`computeStats`...FIXME

=== [[creating-instance]] Creating ExternalRDD Instance

`ExternalRDD` takes the following when created:

* [[outputObjAttr]] Output schema link:spark-sql-Expression-Attribute.adoc[attribute]
* [[rdd]] `RDD` of `T`
* [[session]] link:spark-sql-SparkSession.adoc[SparkSession]

=== [[apply]] Creating ExternalRDD -- `apply` Factory Method

[source, scala]
----
apply[T: Encoder](rdd: RDD[T], session: SparkSession): LogicalPlan
----

`apply`...FIXME

NOTE: `apply` is used when `SparkSession` is requested to create a link:spark-sql-SparkSession.adoc#createDataFrame[DataFrame from RDD of product types] (e.g. Scala case classes, tuples) or link:spark-sql-SparkSession.adoc#createDataset[Dataset from RDD of a given type].
