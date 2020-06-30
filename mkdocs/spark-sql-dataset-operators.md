title: Dataset API

# Dataset API -- Dataset Operators

Dataset API is a <<methods, set of operators>> with typed and untyped transformations, and actions to work with a structured query (as a <<spark-sql-Dataset.adoc#, Dataset>>) as a whole.

[[methods]]
[[operators]]
.Dataset Operators (Transformations and Actions)
[cols="1,3",options="header",width="100%"]
|===
| Operator
| Description

| <<spark-sql-Dataset-untyped-transformations.adoc#agg, agg>>
a| [[agg]]

[source, scala]
----
agg(aggExpr: (String, String), aggExprs: (String, String)*): DataFrame
agg(expr: Column, exprs: Column*): DataFrame
agg(exprs: Map[String, String]): DataFrame
----

An untyped transformation

| <<spark-sql-Dataset-typed-transformations.adoc#alias, alias>>
a| [[alias]]

[source, scala]
----
alias(alias: String): Dataset[T]
alias(alias: Symbol): Dataset[T]
----

A typed transformation that is a mere synonym of <<as-alias, as>>.

| <<spark-sql-Dataset-untyped-transformations.adoc#apply, apply>>
a| [[apply]]

[source, scala]
----
apply(colName: String): Column
----

An untyped transformation to select a column based on the column name (i.e. maps a `Dataset` onto a `Column`)

| <<spark-sql-Dataset-typed-transformations.adoc#as-alias, as>>
a| [[as-alias]]

[source, scala]
----
as(alias: String): Dataset[T]
as(alias: Symbol): Dataset[T]
----

A typed transformation

| <<spark-sql-Dataset-typed-transformations.adoc#as-type, as>>
a| [[as-type]]

[source, scala]
----
as[U : Encoder]: Dataset[U]
----

A typed transformation to enforce a type, i.e. marking the records in the `Dataset` as of a given data type (_data type conversion_). `as` simply changes the view of the data that is passed into typed operations (e.g. <<map, map>>) and does not eagerly project away any columns that are not present in the specified class.

| <<spark-sql-Dataset-basic-actions.adoc#cache, cache>>
a| [[cache]]

[source, scala]
----
cache(): this.type
----

A basic action that is a mere synonym of <<persist, persist>>.

| <<spark-sql-Dataset-basic-actions.adoc#checkpoint, checkpoint>>
a| [[checkpoint]]

[source, scala]
----
checkpoint(): Dataset[T]
checkpoint(eager: Boolean): Dataset[T]
----

A basic action to checkpoint the `Dataset` in a reliable way (using a reliable HDFS-compliant file system, e.g. Hadoop HDFS or Amazon S3)

| <<spark-sql-Dataset-typed-transformations.adoc#coalesce, coalesce>>
a| [[coalesce]]

[source, scala]
----
coalesce(numPartitions: Int): Dataset[T]
----

A typed transformation to repartition a Dataset

| <<spark-sql-Dataset-untyped-transformations.adoc#col, col>>
a| [[col]]

[source, scala]
----
col(colName: String): Column
----

An untyped transformation to create a column (reference) based on the column name

| <<spark-sql-Dataset-actions.adoc#collect, collect>>
a| [[collect]]

[source, scala]
----
collect(): Array[T]
----

An action

| <<spark-sql-Dataset-untyped-transformations.adoc#colRegex, colRegex>>
a| [[colRegex]]

[source, scala]
----
colRegex(colName: String): Column
----

An untyped transformation to create a column (reference) based on the column name specified as a regex

| <<spark-sql-Dataset-basic-actions.adoc#columns, columns>>
a| [[columns]]

[source, scala]
----
columns: Array[String]
----

A basic action

| <<spark-sql-Dataset-actions.adoc#count, count>>
a| [[count]]

[source, scala]
----
count(): Long
----

An action to count the number of rows

| <<spark-sql-Dataset-basic-actions.adoc#createGlobalTempView, createGlobalTempView>>
a| [[createGlobalTempView]]

[source, scala]
----
createGlobalTempView(viewName: String): Unit
----

A basic action

| <<spark-sql-Dataset-basic-actions.adoc#createOrReplaceGlobalTempView, createOrReplaceGlobalTempView>>
a| [[createOrReplaceGlobalTempView]]

[source, scala]
----
createOrReplaceGlobalTempView(viewName: String): Unit
----

A basic action

| <<spark-sql-Dataset-basic-actions.adoc#createOrReplaceTempView, createOrReplaceTempView>>
a| [[createOrReplaceTempView]]

[source, scala]
----
createOrReplaceTempView(viewName: String): Unit
----

A basic action

| <<spark-sql-Dataset-basic-actions.adoc#createTempView, createTempView>>
a| [[createTempView]]

[source, scala]
----
createTempView(viewName: String): Unit
----

A basic action

| <<spark-sql-Dataset-untyped-transformations.adoc#crossJoin, crossJoin>>
a| [[crossJoin]]

[source, scala]
----
crossJoin(right: Dataset[_]): DataFrame
----

An untyped transformation

| <<spark-sql-Dataset-untyped-transformations.adoc#cube, cube>>
a| [[cube]]

[source, scala]
----
cube(cols: Column*): RelationalGroupedDataset
cube(col1: String, cols: String*): RelationalGroupedDataset
----

An untyped transformation

| <<spark-sql-Dataset-actions.adoc#describe, describe>>
a| [[describe]]

[source, scala]
----
describe(cols: String*): DataFrame
----

An action

| <<spark-sql-Dataset-typed-transformations.adoc#distinct, distinct>>
a| [[distinct]]

[source, scala]
----
distinct(): Dataset[T]
----

A typed transformation that is a mere synonym of <<dropDuplicates, dropDuplicates>> (with all the columns of the `Dataset`)

| <<spark-sql-Dataset-untyped-transformations.adoc#drop, drop>>
a| [[drop]]

[source, scala]
----
drop(colName: String): DataFrame
drop(colNames: String*): DataFrame
drop(col: Column): DataFrame
----

An untyped transformation

| <<spark-sql-Dataset-typed-transformations.adoc#dropDuplicates, dropDuplicates>>
a| [[dropDuplicates]]

[source, scala]
----
dropDuplicates(): Dataset[T]
dropDuplicates(colNames: Array[String]): Dataset[T]
dropDuplicates(colNames: Seq[String]): Dataset[T]
dropDuplicates(col1: String, cols: String*): Dataset[T]
----

A typed transformation

| <<spark-sql-Dataset-basic-actions.adoc#dtypes, dtypes>>
a| [[dtypes]]

[source, scala]
----
dtypes: Array[(String, String)]
----

A basic action

| <<spark-sql-Dataset-typed-transformations.adoc#except, except>>
a| [[except]]

[source, scala]
----
except(
  other: Dataset[T]): Dataset[T]
----

A typed transformation

| <<spark-sql-Dataset-typed-transformations.adoc#exceptAll, exceptAll>>
a| [[exceptAll]]

[source, scala]
----
exceptAll(
  other: Dataset[T]): Dataset[T]
----

(*New in 2.4.0*) A typed transformation

| <<spark-sql-Dataset-basic-actions.adoc#explain, explain>>
a| [[explain]]

[source, scala]
----
explain(): Unit
explain(extended: Boolean): Unit
----

A basic action to display the logical and physical plans of the `Dataset`, i.e. displays the logical and physical plans (with optional cost and codegen summaries) to the standard output

| <<spark-sql-Dataset-typed-transformations.adoc#filter, filter>>
a| [[filter]]

[source, scala]
----
filter(condition: Column): Dataset[T]
filter(conditionExpr: String): Dataset[T]
filter(func: T => Boolean): Dataset[T]
----

A typed transformation

| <<spark-sql-Dataset-actions.adoc#first, first>>
a| [[first]]

[source, scala]
----
first(): T
----

An action that is a mere synonym of <<head, head>>

| <<spark-sql-Dataset-typed-transformations.adoc#flatMap, flatMap>>
a| [[flatMap]]

[source, scala]
----
flatMap[U : Encoder](func: T => TraversableOnce[U]): Dataset[U]
----

A typed transformation

| <<spark-sql-Dataset-actions.adoc#foreach, foreach>>
a| [[foreach]]

[source, scala]
----
foreach(f: T => Unit): Unit
----

An action

| <<spark-sql-Dataset-actions.adoc#foreachPartition, foreachPartition>>
a| [[foreachPartition]]

[source, scala]
----
foreachPartition(f: Iterator[T] => Unit): Unit
----

An action

| <<spark-sql-Dataset-untyped-transformations.adoc#groupBy, groupBy>>
a| [[groupBy]]

[source, scala]
----
groupBy(cols: Column*): RelationalGroupedDataset
groupBy(col1: String, cols: String*): RelationalGroupedDataset
----

An untyped transformation

| <<spark-sql-Dataset-typed-transformations.adoc#groupByKey, groupByKey>>
a| [[groupByKey]]

[source, scala]
----
groupByKey[K: Encoder](func: T => K): KeyValueGroupedDataset[K, T]
----

A typed transformation

| <<spark-sql-Dataset-actions.adoc#head, head>>
a| [[head]]

[source, scala]
----
head(): T // <1>
head(n: Int): Array[T]
----
<1> Uses `1` for `n`

An action

| <<spark-sql-Dataset-basic-actions.adoc#hint, hint>>
a| [[hint]]

[source, scala]
----
hint(name: String, parameters: Any*): Dataset[T]
----

A basic action to specify a hint (and optional parameters)

| <<spark-sql-Dataset-basic-actions.adoc#inputFiles, inputFiles>>
a| [[inputFiles]]

[source, scala]
----
inputFiles: Array[String]
----

A basic action

| <<spark-sql-Dataset-typed-transformations.adoc#intersect, intersect>>
a| [[intersect]]

[source, scala]
----
intersect(other: Dataset[T]): Dataset[T]
----

A typed transformation

| <<spark-sql-Dataset-typed-transformations.adoc#intersectAll, intersectAll>>
a| [[intersectAll]]

[source, scala]
----
intersectAll(other: Dataset[T]): Dataset[T]
----

(*New in 2.4.0*) A typed transformation

| <<spark-sql-Dataset-basic-actions.adoc#isEmpty, isEmpty>>
a| [[isEmpty]]

[source, scala]
----
isEmpty: Boolean
----

(*New in 2.4.0*) A basic action

| <<spark-sql-Dataset-basic-actions.adoc#isLocal, isLocal>>
a| [[isLocal]]

[source, scala]
----
isLocal: Boolean
----

A basic action

| isStreaming
a| [[isStreaming]]

[source, scala]
----
isStreaming: Boolean
----

| <<spark-sql-Dataset-untyped-transformations.adoc#join, join>>
a| [[join]]

[source, scala]
----
join(right: Dataset[_]): DataFrame
join(right: Dataset[_], usingColumn: String): DataFrame
join(right: Dataset[_], usingColumns: Seq[String]): DataFrame
join(right: Dataset[_], usingColumns: Seq[String], joinType: String): DataFrame
join(right: Dataset[_], joinExprs: Column): DataFrame
join(right: Dataset[_], joinExprs: Column, joinType: String): DataFrame
----

An untyped transformation

| <<spark-sql-Dataset-typed-transformations.adoc#joinWith, joinWith>>
a| [[joinWith]]

[source, scala]
----
joinWith[U](other: Dataset[U], condition: Column): Dataset[(T, U)]
joinWith[U](other: Dataset[U], condition: Column, joinType: String): Dataset[(T, U)]
----

A typed transformation

| <<spark-sql-Dataset-typed-transformations.adoc#limit, limit>>
a| [[limit]]

[source, scala]
----
limit(n: Int): Dataset[T]
----

A typed transformation

| <<spark-sql-Dataset-basic-actions.adoc#localCheckpoint, localCheckpoint>>
a| [[localCheckpoint]]

[source, scala]
----
localCheckpoint(): Dataset[T]
localCheckpoint(eager: Boolean): Dataset[T]
----

A basic action to checkpoint the `Dataset` locally on executors (and therefore unreliably)

| <<spark-sql-Dataset-typed-transformations.adoc#map, map>>
a| [[map]]

[source, scala]
----
map[U: Encoder](func: T => U): Dataset[U]
----

A typed transformation

| <<spark-sql-Dataset-typed-transformations.adoc#mapPartitions, mapPartitions>>
a| [[mapPartitions]]

[source, scala]
----
mapPartitions[U : Encoder](func: Iterator[T] => Iterator[U]): Dataset[U]
----

A typed transformation

| <<spark-sql-Dataset-untyped-transformations.adoc#na, na>>
a| [[na]]

[source, scala]
----
na: DataFrameNaFunctions
----

An untyped transformation

| <<spark-sql-Dataset-typed-transformations.adoc#orderBy, orderBy>>
a| [[orderBy]]

[source, scala]
----
orderBy(sortExprs: Column*): Dataset[T]
orderBy(sortCol: String, sortCols: String*): Dataset[T]
----

A typed transformation

| <<spark-sql-Dataset-basic-actions.adoc#persist, persist>>
a| [[persist]]

[source, scala]
----
persist(): this.type
persist(newLevel: StorageLevel): this.type
----

A basic action to persist the `Dataset`

NOTE: Although its category `persist` is not an action in the common sense that means executing _anything_ in a Spark cluster (i.e. execution on the driver or on executors). It acts only as a marker to perform Dataset persistence once an action is really executed.

| <<spark-sql-Dataset-basic-actions.adoc#printSchema, printSchema>>
a| [[printSchema]]

[source, scala]
----
printSchema(): Unit
----

A basic action

| <<spark-sql-Dataset-typed-transformations.adoc#randomSplit, randomSplit>>
a| [[randomSplit]]

[source, scala]
----
randomSplit(weights: Array[Double]): Array[Dataset[T]]
randomSplit(weights: Array[Double], seed: Long): Array[Dataset[T]]
----

A typed transformation to split a `Dataset` randomly into two `Datasets`

| <<spark-sql-Dataset-basic-actions.adoc#rdd, rdd>>
a| [[rdd]]

[source, scala]
----
rdd: RDD[T]
----

A basic action

| <<spark-sql-Dataset-actions.adoc#reduce, reduce>>
a| [[reduce]]

[source, scala]
----
reduce(func: (T, T) => T): T
----

An action to reduce the records of the `Dataset` using the specified binary function.

| <<spark-sql-Dataset-typed-transformations.adoc#repartition, repartition>>
a| [[repartition]]

[source, scala]
----
repartition(partitionExprs: Column*): Dataset[T]
repartition(numPartitions: Int): Dataset[T]
repartition(numPartitions: Int, partitionExprs: Column*): Dataset[T]
----

A typed transformation to repartition a Dataset

| <<spark-sql-Dataset-typed-transformations.adoc#repartitionByRange, repartitionByRange>>
a| [[repartitionByRange]]

[source, scala]
----
repartitionByRange(partitionExprs: Column*): Dataset[T]
repartitionByRange(numPartitions: Int, partitionExprs: Column*): Dataset[T]
----

A typed transformation

| <<spark-sql-Dataset-untyped-transformations.adoc#rollup, rollup>>
a| [[rollup]]

[source, scala]
----
rollup(cols: Column*): RelationalGroupedDataset
rollup(col1: String, cols: String*): RelationalGroupedDataset
----

An untyped transformation

| <<spark-sql-Dataset-typed-transformations.adoc#sample, sample>>
a| [[sample]]

[source, scala]
----
sample(withReplacement: Boolean, fraction: Double): Dataset[T]
sample(withReplacement: Boolean, fraction: Double, seed: Long): Dataset[T]
sample(fraction: Double): Dataset[T]
sample(fraction: Double, seed: Long): Dataset[T]
----

A typed transformation

| <<spark-sql-Dataset-basic-actions.adoc#schema, schema>>
a| [[schema]]

[source, scala]
----
schema: StructType
----

A basic action

| <<spark-sql-Dataset-untyped-transformations.adoc#select, select>>
a| [[select]]

[source, scala]
----
// Untyped transformations
select(cols: Column*): DataFrame
select(col: String, cols: String*): DataFrame

// Typed transformations
select[U1](c1: TypedColumn[T, U1]): Dataset[U1]
select[U1, U2](c1: TypedColumn[T, U1], c2: TypedColumn[T, U2]): Dataset[(U1, U2)]
select[U1, U2, U3](
  c1: TypedColumn[T, U1],
  c2: TypedColumn[T, U2],
  c3: TypedColumn[T, U3]): Dataset[(U1, U2, U3)]
select[U1, U2, U3, U4](
  c1: TypedColumn[T, U1],
  c2: TypedColumn[T, U2],
  c3: TypedColumn[T, U3],
  c4: TypedColumn[T, U4]): Dataset[(U1, U2, U3, U4)]
select[U1, U2, U3, U4, U5](
  c1: TypedColumn[T, U1],
  c2: TypedColumn[T, U2],
  c3: TypedColumn[T, U3],
  c4: TypedColumn[T, U4],
  c5: TypedColumn[T, U5]): Dataset[(U1, U2, U3, U4, U5)]
----

An (untyped and typed) transformation

| <<spark-sql-Dataset-untyped-transformations.adoc#selectExpr, selectExpr>>
a| [[selectExpr]]

[source, scala]
----
selectExpr(exprs: String*): DataFrame
----

An untyped transformation

| <<spark-sql-Dataset-actions.adoc#show, show>>
a| [[show]]

[source, scala]
----
show(): Unit
show(truncate: Boolean): Unit
show(numRows: Int): Unit
show(numRows: Int, truncate: Boolean): Unit
show(numRows: Int, truncate: Int): Unit
show(numRows: Int, truncate: Int, vertical: Boolean): Unit
----

An action

| <<spark-sql-Dataset-typed-transformations.adoc#sort, sort>>
a| [[sort]]

[source, scala]
----
sort(sortExprs: Column*): Dataset[T]
sort(sortCol: String, sortCols: String*): Dataset[T]
----

A typed transformation to sort elements globally (across partitions). Use <<sortWithinPartitions, sortWithinPartitions>> transformation for partition-local sort

| <<spark-sql-Dataset-typed-transformations.adoc#sortWithinPartitions, sortWithinPartitions>>
a| [[sortWithinPartitions]]

[source, scala]
----
sortWithinPartitions(sortExprs: Column*): Dataset[T]
sortWithinPartitions(sortCol: String, sortCols: String*): Dataset[T]
----

A typed transformation to sort elements within partitions (aka _local sort_). Use <<sort, sort>> transformation for global sort (across partitions)

| <<spark-sql-Dataset-untyped-transformations.adoc#stat, stat>>
a| [[stat]]

[source, scala]
----
stat: DataFrameStatFunctions
----

An untyped transformation

| <<spark-sql-Dataset-basic-actions.adoc#storageLevel, storageLevel>>
a| [[storageLevel]]

[source, scala]
----
storageLevel: StorageLevel
----

A basic action

| <<spark-sql-Dataset-actions.adoc#summary, summary>>
a| [[summary]]

[source, scala]
----
summary(statistics: String*): DataFrame
----

An action to calculate statistics (e.g. `count`, `mean`, `stddev`, `min`, `max` and `25%`, `50%`, `75%` percentiles)

| <<spark-sql-Dataset-actions.adoc#take, take>>
a| [[take]]

[source, scala]
----
take(n: Int): Array[T]
----

An action to take the first records of a Dataset

| <<spark-sql-Dataset-basic-actions.adoc#toDF, toDF>>
a| [[toDF]]

[source, scala]
----
toDF(): DataFrame
toDF(colNames: String*): DataFrame
----

A basic action to convert a Dataset to a DataFrame

| <<spark-sql-Dataset-typed-transformations.adoc#toJSON, toJSON>>
a| [[toJSON]]

[source, scala]
----
toJSON: Dataset[String]
----

A typed transformation

| <<spark-sql-Dataset-actions.adoc#toLocalIterator, toLocalIterator>>
a| [[toLocalIterator]]

[source, scala]
----
toLocalIterator(): java.util.Iterator[T]
----

An action that returns an iterator with all rows in the `Dataset`. The iterator will consume as much memory as the largest partition in the `Dataset`.

| <<spark-sql-Dataset-typed-transformations.adoc#transform, transform>>
a| [[transform]]

[source, scala]
----
transform[U](t: Dataset[T] => Dataset[U]): Dataset[U]
----

A typed transformation for chaining custom transformations

| <<spark-sql-Dataset-typed-transformations.adoc#union, union>>
a| [[union]]

[source, scala]
----
union(other: Dataset[T]): Dataset[T]
----

A typed transformation

| <<spark-sql-Dataset-typed-transformations.adoc#unionByName, unionByName>>
a| [[unionByName]]

[source, scala]
----
unionByName(other: Dataset[T]): Dataset[T]
----

A typed transformation

| <<spark-sql-Dataset-basic-actions.adoc#unpersist, unpersist>>
a| [[unpersist]]

[source, scala]
----
unpersist(): this.type // <1>
unpersist(blocking: Boolean): this.type
----
<1> Uses `unpersist` with `blocking` disabled (`false`)

A basic action to unpersist the `Dataset`

| <<spark-sql-Dataset-typed-transformations.adoc#where, where>>
a| [[where]]

[source, scala]
----
where(condition: Column): Dataset[T]
where(conditionExpr: String): Dataset[T]
----

A typed transformation

| <<spark-sql-Dataset-untyped-transformations.adoc#withColumn, withColumn>>
a| [[withColumn]]

[source, scala]
----
withColumn(colName: String, col: Column): DataFrame
----

An untyped transformation

| <<spark-sql-Dataset-untyped-transformations.adoc#withColumnRenamed, withColumnRenamed>>
a| [[withColumnRenamed]]

[source, scala]
----
withColumnRenamed(existingName: String, newName: String): DataFrame
----

An untyped transformation

| <<spark-sql-Dataset-basic-actions.adoc#write, write>>
a| [[write]]

[source, scala]
----
write: DataFrameWriter[T]
----

A basic action that returns a <<spark-sql-DataFrameWriter.adoc#, DataFrameWriter>> for saving the content of the (non-streaming) `Dataset` out to an external storage
|===
