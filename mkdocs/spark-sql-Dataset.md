title: Dataset

# Dataset -- Structured Query with Data Encoder

*Dataset* is a strongly-typed data structure in Spark SQL that represents a structured query.

NOTE: A structured query can be written using SQL or <<spark-sql-dataset-operators.adoc#, Dataset API>>.

The following figure shows the relationship between different entities of Spark SQL that all together give the `Dataset` data structure.

.Dataset's Internals
image::images/spark-sql-Dataset.png[align="center"]

It is therefore fair to say that `Dataset` consists of the following three elements:

. <<spark-sql-QueryExecution.adoc#, QueryExecution>> (with the parsed unanalyzed <<spark-sql-LogicalPlan.adoc#, LogicalPlan>> of a structured query)

. <<spark-sql-Encoder.adoc#, Encoder>> (of the type of the records for fast serialization and deserialization to and from <<spark-sql-InternalRow.adoc#, InternalRow>>)

. <<spark-sql-SparkSession.adoc#, SparkSession>>

When <<creating-instance, created>>, `Dataset` takes such a 3-element tuple with a `SparkSession`, a `QueryExecution` and an `Encoder`.

`Dataset` is <<creating-instance, created>> when:

* <<apply, Dataset.apply>> (for a <<spark-sql-LogicalPlan.adoc#, LogicalPlan>> and a <<spark-sql-SparkSession.adoc#, SparkSession>> with the <<spark-sql-Encoder.adoc#, Encoder>> in a Scala implicit scope)

* <<ofRows, Dataset.ofRows>> (for a <<spark-sql-LogicalPlan.adoc#, LogicalPlan>> and a <<spark-sql-SparkSession.adoc#, SparkSession>>)

* <<spark-sql-Dataset-untyped-transformations.adoc#toDF, Dataset.toDF>> untyped transformation is used

* <<spark-sql-Dataset-typed-transformations.adoc#select, Dataset.select>>, <<spark-sql-Dataset-typed-transformations.adoc#randomSplit, Dataset.randomSplit>> and <<spark-sql-Dataset-typed-transformations.adoc#mapPartitions, Dataset.mapPartitions>> typed transformations are used

* <<spark-sql-KeyValueGroupedDataset.adoc#agg, KeyValueGroupedDataset.agg>> operator is used (that requests `KeyValueGroupedDataset` to <<spark-sql-KeyValueGroupedDataset.adoc#aggUntyped, aggUntyped>>)

* <<spark-sql-SparkSession.adoc#emptyDataset, SparkSession.emptyDataset>> and <<spark-sql-SparkSession.adoc#range, SparkSession.range>> operators are used

* `CatalogImpl` is requested to
<<spark-sql-CatalogImpl.adoc#makeDataset, makeDataset>> (when requested to <<spark-sql-CatalogImpl.adoc#listDatabases, list databases>>, <<spark-sql-CatalogImpl.adoc#listTables, tables>>, <<spark-sql-CatalogImpl.adoc#listFunctions, functions>> and <<spark-sql-CatalogImpl.adoc#listColumns, columns>>)

* Spark Structured Streaming's `MicroBatchExecution` is requested to `runBatch`

Datasets are _lazy_ and structured query operators and expressions are only triggered when an action is invoked.

[source, scala]
----
import org.apache.spark.sql.SparkSession
val spark: SparkSession = ...

scala> val dataset = spark.range(5)
dataset: org.apache.spark.sql.Dataset[Long] = [id: bigint]

// Variant 1: filter operator accepts a Scala function
dataset.filter(n => n % 2 == 0).count

// Variant 2: filter operator accepts a Column-based SQL expression
dataset.filter('value % 2 === 0).count

// Variant 3: filter operator accepts a SQL query
dataset.filter("value % 2 = 0").count
----

The <<spark-sql-dataset-operators.adoc#, Dataset API>> offers declarative and type-safe operators that makes for an improved experience for data processing (comparing to link:spark-sql-DataFrame.adoc[DataFrames] that were a set of index- or column name-based link:spark-sql-Row.adoc[Rows]).

[NOTE]
====
`Dataset` was first introduced in Apache Spark *1.6.0* as an experimental feature, and has since turned itself into a fully supported API.

As of Spark *2.0.0*, link:spark-sql-DataFrame.adoc[DataFrame] - the flagship data abstraction of previous versions of Spark SQL - is currently a _mere_ type alias for `Dataset[Row]`:

[source, scala]
----
type DataFrame = Dataset[Row]
----

See https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/package.scala#L45[package object sql].
====

`Dataset` offers convenience of RDDs with the performance optimizations of DataFrames and the strong static type-safety of Scala. The last feature of bringing the strong type-safety to link:spark-sql-DataFrame.adoc[DataFrame] makes Dataset so appealing. All the features together give you a more functional programming interface to work with structured data.

[source, scala]
----
scala> spark.range(1).filter('id === 0).explain(true)
== Parsed Logical Plan ==
'Filter ('id = 0)
+- Range (0, 1, splits=8)

== Analyzed Logical Plan ==
id: bigint
Filter (id#51L = cast(0 as bigint))
+- Range (0, 1, splits=8)

== Optimized Logical Plan ==
Filter (id#51L = 0)
+- Range (0, 1, splits=8)

== Physical Plan ==
*Filter (id#51L = 0)
+- *Range (0, 1, splits=8)

scala> spark.range(1).filter(_ == 0).explain(true)
== Parsed Logical Plan ==
'TypedFilter <function1>, class java.lang.Long, [StructField(value,LongType,true)], unresolveddeserializer(newInstance(class java.lang.Long))
+- Range (0, 1, splits=8)

== Analyzed Logical Plan ==
id: bigint
TypedFilter <function1>, class java.lang.Long, [StructField(value,LongType,true)], newInstance(class java.lang.Long)
+- Range (0, 1, splits=8)

== Optimized Logical Plan ==
TypedFilter <function1>, class java.lang.Long, [StructField(value,LongType,true)], newInstance(class java.lang.Long)
+- Range (0, 1, splits=8)

== Physical Plan ==
*Filter <function1>.apply
+- *Range (0, 1, splits=8)
----

It is only with Datasets to have syntax and analysis checks at compile time (that was not possible using link:spark-sql-DataFrame.adoc[DataFrame], regular SQL queries or even RDDs).

Using `Dataset` objects turns `DataFrames` of link:spark-sql-Row.adoc[Row] instances into a `DataFrames` of case classes with proper names and types (following their equivalents in the case classes). Instead of using indices to access respective fields in a DataFrame and cast it to a type, all this is automatically handled by Datasets and checked by the Scala compiler.

If however a link:spark-sql-LogicalPlan.adoc[LogicalPlan] is used to <<creating-instance, create a `Dataset`>>, the logical plan is first link:spark-sql-SessionState.adoc#executePlan[executed] (using the current link:spark-sql-SessionState.adoc#executePlan[SessionState] in the `SparkSession`) that yields the link:spark-sql-QueryExecution.adoc[QueryExecution] plan.

A `Dataset` is <<Queryable, Queryable>> and `Serializable`, i.e. can be saved to a persistent storage.

NOTE: link:spark-sql-SparkSession.adoc[SparkSession] and link:spark-sql-QueryExecution.adoc[QueryExecution] are transient attributes of a `Dataset` and therefore do not participate in Dataset serialization. The only _firmly-tied_ feature of a `Dataset` is the link:spark-sql-Encoder.adoc[Encoder].

You can request the <<spark-sql-dataset-operators.adoc#toDF, "untyped" view>> of a Dataset or access the link:spark-sql-dataset-operators.adoc#rdd[RDD] that is generated after executing the query. It is supposed to give you a more pleasant experience while transitioning from the legacy RDD-based or DataFrame-based APIs you may have used in the earlier versions of Spark SQL or encourage migrating from Spark Core's RDD API to Spark SQL's Dataset API.

The default storage level for `Datasets` is link:spark-rdd-caching.adoc[MEMORY_AND_DISK] because recomputing the in-memory columnar representation of the underlying table is expensive. You can however link:spark-sql-caching-and-persistence.adoc#persist[persist a `Dataset`].

NOTE: Spark 2.0 has introduced a new query model called link:spark-structured-streaming.adoc[Structured Streaming] for continuous incremental execution of structured queries. That made possible to consider Datasets a static and bounded as well as streaming and unbounded data sets with a single unified API for different execution models.

A `Dataset` is link:spark-sql-dataset-operators.adoc#isLocal[local] if it was created from local collections using link:spark-sql-SparkSession.adoc#emptyDataset[SparkSession.emptyDataset] or link:spark-sql-SparkSession.adoc#createDataset[SparkSession.createDataset] methods and their derivatives like <<toDF,toDF>>. If so, the queries on the Dataset can be optimized and run locally, i.e. without using Spark executors.

NOTE: `Dataset` makes sure that the underlying `QueryExecution` is link:spark-sql-QueryExecution.adoc#analyzed[analyzed] and link:spark-sql-Analyzer-CheckAnalysis.adoc#checkAnalysis[checked].

[[properties]]
[[attributes]]
.Dataset's Properties
[cols="1,2",options="header",width="100%",separator="!"]
!===
! Name
! Description

! [[boundEnc]] `boundEnc`
! link:spark-sql-ExpressionEncoder.adoc[ExpressionEncoder]

Used when...FIXME

! [[deserializer]] `deserializer`
a! Deserializer link:spark-sql-Expression.adoc[expression] to convert internal rows to objects of type `T`

Created lazily by requesting the <<exprEnc, ExpressionEncoder>> to link:spark-sql-ExpressionEncoder.adoc#resolveAndBind[resolveAndBind]

Used when:

* `Dataset` is <<apply, created>> (for a logical plan in a given `SparkSession`)

* link:spark-sql-dataset-operators.adoc#spark-sql-dataset-operators.adoc[Dataset.toLocalIterator] operator is used (to create a Java `Iterator` of objects of type `T`)

* `Dataset` is requested to <<collectFromPlan, collect all rows from a spark plan>>

! [[exprEnc]] `exprEnc`
! Implicit link:spark-sql-ExpressionEncoder.adoc[ExpressionEncoder]

Used when...FIXME

! `logicalPlan`
a! [[logicalPlan]] Analyzed <<spark-sql-LogicalPlan.adoc#, logical plan>> with all <<spark-sql-LogicalPlan-Command.adoc#, logical commands>> executed and turned into a <<spark-sql-LogicalPlan-LocalRelation.adoc#creating-instance, LocalRelation>>.

[source, scala]
----
logicalPlan: LogicalPlan
----

When initialized, `logicalPlan` requests the <<queryExecution, QueryExecution>> for <<spark-sql-QueryExecution.adoc#analyzed, analyzed logical plan>>. If the plan is a <<spark-sql-LogicalPlan-Command.adoc#, logical command>> or a union thereof, `logicalPlan` <<withAction, executes the QueryExecution>> (using <<spark-sql-SparkPlan.adoc#executeCollect, executeCollect>>).

! `planWithBarrier`
a! [[planWithBarrier]]

[source, scala]
----
planWithBarrier: AnalysisBarrier
----

! [[rdd]] `rdd`
a! (lazily-created) link:spark-rdd.adoc[RDD] of JVM objects of type `T` (as converted from rows in `Dataset` in the link:spark-sql-InternalRow.adoc[internal binary row format]).

[source, scala]
----
rdd: RDD[T]
----

NOTE: `rdd` gives `RDD` with the extra execution step to convert rows from their internal binary row format to JVM objects that will impact the JVM memory as the objects are inside JVM (while were outside before). You should not use `rdd` directly.

Internally, `rdd` first link:spark-sql-CatalystSerde.adoc#deserialize[creates a new logical plan that deserializes] the Dataset's <<logicalPlan, logical plan>>.

[source, scala]
----
val dataset = spark.range(5).withColumn("group", 'id % 2)
scala> dataset.rdd.toDebugString
res1: String =
(8) MapPartitionsRDD[8] at rdd at <console>:26 [] // <-- extra deserialization step
 |  MapPartitionsRDD[7] at rdd at <console>:26 []
 |  MapPartitionsRDD[6] at rdd at <console>:26 []
 |  MapPartitionsRDD[5] at rdd at <console>:26 []
 |  ParallelCollectionRDD[4] at rdd at <console>:26 []

// Compare with a more memory-optimized alternative
// Avoids copies and has no schema
scala> dataset.queryExecution.toRdd.toDebugString
res2: String =
(8) MapPartitionsRDD[11] at toRdd at <console>:26 []
 |  MapPartitionsRDD[10] at toRdd at <console>:26 []
 |  ParallelCollectionRDD[9] at toRdd at <console>:26 []
----

`rdd` then requests `SessionState` to link:spark-sql-SessionState.adoc#executePlan[execute the logical plan] to get the corresponding link:spark-sql-QueryExecution.adoc#toRdd[RDD of binary rows].

NOTE: `rdd` uses <<sparkSession, SparkSession>> to link:spark-sql-SparkSession.adoc#sessionState[access `SessionState`].

`rdd` then requests the Dataset's <<exprEnc, ExpressionEncoder>> for the link:spark-sql-Expression.adoc#dataType[data type] of the rows (using link:spark-sql-ExpressionEncoder.adoc#deserializer[deserializer] expression) and link:spark-rdd-transformations.adoc#mapPartitions[maps over them (per partition)] to create records of the expected type `T`.

NOTE: `rdd` is at the "boundary" between the internal binary row format and the JVM type of the dataset. Avoid the extra deserialization step to lower JVM memory requirements of your Spark application.

! [[sqlContext]] `sqlContext`
! Lazily-created link:spark-sql-SQLContext.adoc[SQLContext]

Used when...FIXME
!===

=== [[inputFiles]] Getting Input Files of Relations (in Structured Query) -- `inputFiles` Method

[source, scala]
----
inputFiles: Array[String]
----

`inputFiles` requests <<queryExecution, QueryExecution>> for link:spark-sql-QueryExecution.adoc#optimizedPlan[optimized logical plan] and collects the following logical operators:

* link:spark-sql-LogicalPlan-LogicalRelation.adoc[LogicalRelation] with link:spark-sql-FileRelation.adoc[FileRelation] (as the link:spark-sql-LogicalPlan-LogicalRelation.adoc#relation[BaseRelation])

* link:spark-sql-FileRelation.adoc[FileRelation]

* link:hive/HiveTableRelation.adoc[HiveTableRelation]

`inputFiles` then requests the logical operators for their underlying files:

* link:spark-sql-FileRelation.adoc#inputFiles[inputFiles] of the `FileRelations`

* link:spark-sql-CatalogStorageFormat.adoc#locationUri[locationUri] of the `HiveTableRelation`

=== [[resolve]] `resolve` Internal Method

[source, scala]
----
resolve(colName: String): NamedExpression
----

CAUTION: FIXME

=== [[creating-instance]] Creating Dataset Instance

`Dataset` takes the following when created:

* [[sparkSession]] link:spark-sql-SparkSession.adoc[SparkSession]
* [[queryExecution]] link:spark-sql-QueryExecution.adoc[QueryExecution]
* [[encoder]] link:spark-sql-Encoder.adoc[Encoder] for the type `T` of the records

NOTE: You can also create a `Dataset` using link:spark-sql-LogicalPlan.adoc[LogicalPlan] that is immediately link:spark-sql-SessionState.adoc#executePlan[executed using `SessionState`].

Internally, `Dataset` requests <<queryExecution, QueryExecution>> to link:spark-sql-QueryExecution.adoc#assertAnalyzed[analyze itself].

`Dataset` initializes the <<internal-registries, internal registries and counters>>.

=== [[isLocal]] Is Dataset Local? -- `isLocal` Method

[source, scala]
----
isLocal: Boolean
----

`isLocal` flag is enabled (i.e. `true`) when operators like `collect` or `take` could be run locally, i.e. without using executors.

Internally, `isLocal` checks whether the logical query plan of a `Dataset` is link:spark-sql-LogicalPlan-LocalRelation.adoc[LocalRelation].

=== [[isStreaming]] Is Dataset Streaming? -- `isStreaming` method

[source, scala]
----
isStreaming: Boolean
----

`isStreaming` is enabled (i.e. `true`) when the logical plan link:spark-sql-LogicalPlan.adoc#isStreaming[is streaming].

Internally, `isStreaming` takes the Dataset's link:spark-sql-LogicalPlan.adoc[logical plan] and gives link:spark-sql-LogicalPlan.adoc#isStreaming[whether the plan is streaming or not].

=== [[Queryable]] Queryable

CAUTION: FIXME

=== [[withNewRDDExecutionId]] `withNewRDDExecutionId` Internal Method

[source, scala]
----
withNewRDDExecutionId[U](body: => U): U
----

`withNewRDDExecutionId` executes the input `body` action under <<spark-sql-SQLExecution.adoc#withNewExecutionId, new execution id>>.

CAUTION: FIXME What's the difference between `withNewRDDExecutionId` and <<withNewExecutionId, withNewExecutionId>>?

NOTE: `withNewRDDExecutionId` is used when <<spark-sql-dataset-operators.adoc#foreach, Dataset.foreach>> and <<spark-sql-dataset-operators.adoc#foreachPartition, Dataset.foreachPartition>> actions are used.

=== [[ofRows]] Creating DataFrame (For Logical Query Plan and SparkSession) -- `ofRows` Internal Factory Method

[source, scala]
----
ofRows(sparkSession: SparkSession, logicalPlan: LogicalPlan): DataFrame
----

NOTE: `ofRows` is part of `Dataset` Scala object that is marked as a `private[sql]` and so can only be accessed from code in `org.apache.spark.sql` package.

`ofRows` returns link:spark-sql-DataFrame.adoc[DataFrame] (which is the type alias for `Dataset[Row]`). `ofRows` uses link:spark-sql-RowEncoder.adoc[RowEncoder] to convert the schema (based on the input `logicalPlan` logical plan).

Internally, `ofRows` link:spark-sql-SessionState.adoc#executePlan[prepares the input `logicalPlan` for execution] and creates a `Dataset[Row]` with the current link:spark-sql-SparkSession.adoc[SparkSession], the link:spark-sql-QueryExecution.adoc[QueryExecution] and link:spark-sql-RowEncoder.adoc[RowEncoder].

[NOTE]
====
`ofRows` is used when:

* `DataFrameReader` is requested to <<spark-sql-DataFrameReader.adoc#load, load data from a data source>>

* `Dataset` is requested to execute <<checkpoint, checkpoint>>, `mapPartitionsInR`, <<withPlan, untyped transformations>> and <<withSetOperator, set-based typed transformations>>

* `RelationalGroupedDataset` is requested to <<spark-sql-RelationalGroupedDataset.adoc#toDF, create a DataFrame from aggregate expressions>>, `flatMapGroupsInR` and `flatMapGroupsInPandas`

* `SparkSession` is requested to <<spark-sql-SparkSession.adoc#baseRelationToDataFrame, create a DataFrame from a BaseRelation>>, <<spark-sql-SparkSession.adoc#createDataFrame, createDataFrame>>, <<spark-sql-SparkSession.adoc#internalCreateDataFrame, internalCreateDataFrame>>, <<spark-sql-SparkSession.adoc#sql, sql>> and <<spark-sql-SparkSession.adoc#table, table>>

* `CacheTableCommand`, <<spark-sql-LogicalPlan-CreateTempViewUsing.adoc#run, CreateTempViewUsing>>, <<spark-sql-LogicalPlan-InsertIntoDataSourceCommand.adoc#run, InsertIntoDataSourceCommand>> and `SaveIntoDataSourceCommand` logical commands are executed (run)

* `DataSource` is requested to <<spark-sql-DataSource.adoc#writeAndRead, writeAndRead>> (for a <<spark-sql-CreatableRelationProvider.adoc#, CreatableRelationProvider>>)

* `FrequentItems` is requested to `singlePassFreqItems`

* `StatFunctions` is requested to `crossTabulate` and `summary`

* Spark Structured Streaming's `DataStreamReader` is requested to `load`

* Spark Structured Streaming's `DataStreamWriter` is requested to `start`

* Spark Structured Streaming's `FileStreamSource` is requested to `getBatch`

* Spark Structured Streaming's `MemoryStream` is requested to `toDF`
====

=== [[withNewExecutionId]] Tracking Multi-Job Structured Query Execution (PySpark) -- `withNewExecutionId` Internal Method

[source, scala]
----
withNewExecutionId[U](body: => U): U
----

`withNewExecutionId` executes the input `body` action under <<spark-sql-SQLExecution.adoc#withNewExecutionId, new execution id>>.

NOTE: `withNewExecutionId` sets a unique execution id so that all Spark jobs belong to the `Dataset` action execution.

[NOTE]
====
`withNewExecutionId` is used exclusively when `Dataset` is executing Python-based actions (i.e. `collectToPython`, `collectAsArrowToPython` and `toPythonIterator`) that are not of much interest in this gitbook.

Feel free to contact me at jacek@japila.pl if you think I should re-consider my decision.
====

=== [[withAction]] Executing Action Under New Execution ID -- `withAction` Internal Method

[source, scala]
----
withAction[U](name: String, qe: QueryExecution)(action: SparkPlan => U)
----

`withAction` requests `QueryExecution` for the link:spark-sql-QueryExecution.adoc#executedPlan[optimized physical query plan] and link:spark-sql-SparkPlan.adoc[resets the metrics] of every physical operator (in the physical plan).

`withAction` requests `SQLExecution` to <<spark-sql-SQLExecution.adoc#withNewExecutionId, execute>> the input `action` with the executable physical plan (tracked under a new execution id).

In the end, `withAction` notifies `ExecutionListenerManager` that the `name` action has finished link:spark-sql-ExecutionListenerManager.adoc#onSuccess[successfully] or link:spark-sql-ExecutionListenerManager.adoc#onFailure[with an exception].

NOTE: `withAction` uses <<sparkSession, SparkSession>> to access link:spark-sql-SparkSession.adoc#listenerManager[ExecutionListenerManager].

[NOTE]
====
`withAction` is used when `Dataset` is requested for the following:

* <<logicalPlan, Computing the logical plan>> (and executing a link:spark-sql-LogicalPlan-Command.adoc[logical command] or their `Union`)

* Dataset operators: <<spark-sql-dataset-operators.adoc#collect, collect>>, <<spark-sql-dataset-operators.adoc#count, count>>, <<spark-sql-dataset-operators.adoc#head, head>> and <<spark-sql-dataset-operators.adoc#toLocalIterator, toLocalIterator>>
====

=== [[apply]] Creating Dataset Instance (For LogicalPlan and SparkSession) -- `apply` Internal Factory Method

[source, scala]
----
apply[T: Encoder](sparkSession: SparkSession, logicalPlan: LogicalPlan): Dataset[T]
----

NOTE: `apply` is part of `Dataset` Scala object that is marked as a `private[sql]` and so can only be accessed from code in `org.apache.spark.sql` package.

`apply`...FIXME

[NOTE]
====
`apply` is used when:

* `Dataset` is requested to execute <<withTypedPlan, typed transformations>> and <<withSetOperator, set-based typed transformations>>

* Spark Structured Streaming's `MemoryStream` is requested to `toDS`
====

=== [[collectFromPlan]] Collecting All Rows From Spark Plan -- `collectFromPlan` Internal Method

[source, scala]
----
collectFromPlan(plan: SparkPlan): Array[T]
----

`collectFromPlan`...FIXME

NOTE: `collectFromPlan` is used for link:spark-sql-dataset-operators.adoc#head[Dataset.head], link:spark-sql-dataset-operators.adoc#collect[Dataset.collect] and link:spark-sql-dataset-operators.adoc#collectAsList[Dataset.collectAsList] operators.

=== [[selectUntyped]] `selectUntyped` Internal Method

[source, scala]
----
selectUntyped(columns: TypedColumn[_, _]*): Dataset[_]
----

`selectUntyped`...FIXME

NOTE: `selectUntyped` is used exclusively when <<spark-sql-Dataset-typed-transformations.adoc#select, Dataset.select>> typed transformation is used.

=== [[withTypedPlan]] Helper Method for Typed Transformations -- `withTypedPlan` Internal Method

[source, scala]
----
withTypedPlan[U: Encoder](logicalPlan: LogicalPlan): Dataset[U]
----

`withTypedPlan`...FIXME

NOTE: `withTypedPlan` is annotated with Scala's https://www.scala-lang.org/api/current/scala/inline.html[@inline] annotation that requests the Scala compiler to try especially hard to inline it.

NOTE: `withTypedPlan` is used in the `Dataset` <<spark-sql-Dataset-typed-transformations.adoc#, typed transformations>>, i.e. <<spark-sql-Dataset-typed-transformations.adoc#withWatermark, withWatermark>>, <<spark-sql-Dataset-typed-transformations.adoc#joinWith, joinWith>>, <<spark-sql-Dataset-typed-transformations.adoc#hint, hint>>, <<spark-sql-Dataset-typed-transformations.adoc#as, as>>, <<spark-sql-Dataset-typed-transformations.adoc#filter, filter>>, <<spark-sql-Dataset-typed-transformations.adoc#limit, limit>>, <<spark-sql-Dataset-typed-transformations.adoc#sample, sample>>, <<spark-sql-Dataset-typed-transformations.adoc#dropDuplicates, dropDuplicates>>, <<spark-sql-Dataset-typed-transformations.adoc#filter, filter>>, <<spark-sql-Dataset-typed-transformations.adoc#map, map>>, <<spark-sql-Dataset-typed-transformations.adoc#repartition, repartition>>, <<spark-sql-Dataset-typed-transformations.adoc#repartitionByRange, repartitionByRange>>, <<spark-sql-Dataset-typed-transformations.adoc#coalesce, coalesce>> and <<spark-sql-Dataset-typed-transformations.adoc#sort, sort>> with <<spark-sql-Dataset-typed-transformations.adoc#sortWithinPartitions, sortWithinPartitions>> (through the <<sortInternal, sortInternal>> internal method).

=== [[withSetOperator]] Helper Method for Set-Based Typed Transformations -- `withSetOperator` Internal Method

[source, scala]
----
withSetOperator[U: Encoder](
  logicalPlan: LogicalPlan): Dataset[U]
----

`withSetOperator`...FIXME

NOTE: `withSetOperator` is annotated with Scala's https://www.scala-lang.org/api/current/scala/inline.html[@inline] annotation that requests the Scala compiler to try especially hard to inline it.

NOTE: `withSetOperator` is used for the link:spark-sql-Dataset-typed-transformations.adoc[Dataset's typed transformations] (i.e. link:spark-sql-Dataset-typed-transformations.adoc#union[union], link:spark-sql-Dataset-typed-transformations.adoc#unionByName[unionByName], link:spark-sql-Dataset-typed-transformations.adoc#intersect[intersect], link:spark-sql-Dataset-typed-transformations.adoc#intersectAll[intersectAll], link:spark-sql-Dataset-typed-transformations.adoc#except[except] and link:spark-sql-Dataset-typed-transformations.adoc#exceptAll[exceptAll]).

=== [[sortInternal]] `sortInternal` Internal Method

[source, scala]
----
sortInternal(global: Boolean, sortExprs: Seq[Column]): Dataset[T]
----

`sortInternal` <<withTypedPlan, creates a Dataset>> with <<spark-sql-LogicalPlan-Sort.adoc#, Sort>> unary logical operator (and the <<logicalPlan, logicalPlan>> as the <<spark-sql-LogicalPlan-Sort.adoc#child, child logical plan>>).

[source, scala]
----
val nums = Seq((0, "zero"), (1, "one")).toDF("id", "name")
// Creates a Sort logical operator:
// - descending sort direction for id column (specified explicitly)
// - name column is wrapped with ascending sort direction
val numsSorted = nums.sort('id.desc, 'name)
val logicalPlan = numsSorted.queryExecution.logical
scala> println(logicalPlan.numberedTreeString)
00 'Sort ['id DESC NULLS LAST, 'name ASC NULLS FIRST], true
01 +- Project [_1#11 AS id#14, _2#12 AS name#15]
02    +- LocalRelation [_1#11, _2#12]
----

Internally, `sortInternal` firstly builds ordering expressions for the given `sortExprs` columns, i.e. takes the `sortExprs` columns and makes sure that they are <<spark-sql-Expression-SortOrder.adoc#, SortOrder>> expressions already (and leaves them untouched) or wraps them into <<spark-sql-Expression-SortOrder.adoc#, SortOrder>> expressions with <<spark-sql-Expression-SortOrder.adoc#Ascending, Ascending>> sort direction.

In the end, `sortInternal` <<withTypedPlan, creates a Dataset>> with <<spark-sql-LogicalPlan-Sort.adoc#, Sort>> unary logical operator (with the ordering expressions, the given `global` flag, and the <<logicalPlan, logicalPlan>> as the <<spark-sql-LogicalPlan-Sort.adoc#child, child logical plan>>).

NOTE: `sortInternal` is used for the <<spark-sql-dataset-operators.adoc#sort, sort>> and <<spark-sql-dataset-operators.adoc#sortWithinPartitions, sortWithinPartitions>> typed transformations in the Dataset API (with the only change of the `global` flag being enabled and disabled, respectively).

=== [[withPlan]] Helper Method for Untyped Transformations and Basic Actions -- `withPlan` Internal Method

[source, scala]
----
withPlan(logicalPlan: LogicalPlan): DataFrame
----

`withPlan` simply uses <<ofRows, ofRows>> internal factory method to create a `DataFrame` for the input <<spark-sql-LogicalPlan.adoc#, LogicalPlan>> and the current <<sparkSession, SparkSession>>.

NOTE: `withPlan` is annotated with Scala's https://www.scala-lang.org/api/current/scala/inline.html[@inline] annotation that requests the Scala compiler to try especially hard to inline it.

NOTE: `withPlan` is used in the `Dataset` <<spark-sql-Dataset-untyped-transformations.adoc#, untyped transformations>> (i.e. <<spark-sql-Dataset-untyped-transformations.adoc#join, join>>, <<spark-sql-Dataset-untyped-transformations.adoc#crossJoin, crossJoin>> and <<spark-sql-Dataset-untyped-transformations.adoc#select, select>>) and <<spark-sql-Dataset-basic-actions.adoc#, basic actions>> (i.e. <<spark-sql-Dataset-basic-actions.adoc#createTempView, createTempView>>, <<spark-sql-Dataset-basic-actions.adoc#createOrReplaceTempView, createOrReplaceTempView>>, <<spark-sql-Dataset-basic-actions.adoc#createGlobalTempView, createGlobalTempView>> and <<spark-sql-Dataset-basic-actions.adoc#createOrReplaceGlobalTempView, createOrReplaceGlobalTempView>>).

=== [[i-want-more]] Further Reading and Watching

* (video) https://youtu.be/i7l3JQRx7Qw[Structuring Spark: DataFrames, Datasets, and Streaming]
