title: Standard Functions

# Standard Functions -- functions Object

`org.apache.spark.sql.functions` object defines built-in <<standard-functions, standard functions>> to work with (values produced by) <<spark-sql-Column.adoc#, columns>>.

You can access the standard functions using the following `import` statement in your Scala application:

[source, scala]
----
import org.apache.spark.sql.functions._
----

[[standard-functions]]
.(Subset of) Standard Functions in Spark SQL
[align="center",cols="1,1,2",width="100%",options="header"]
|===
|
|Name
|Description

.26+^.^| [[aggregate-functions]][[agg_funcs]] *Aggregate functions*

| <<spark-sql-aggregate-functions.adoc#approx_count_distinct, approx_count_distinct>>
a| [[approx_count_distinct]]

[source, scala]
----
approx_count_distinct(e: Column): Column
approx_count_distinct(columnName: String): Column
approx_count_distinct(e: Column, rsd: Double): Column
approx_count_distinct(columnName: String, rsd: Double): Column
----

| <<spark-sql-aggregate-functions.adoc#avg, avg>>
a| [[avg]]

[source, scala]
----
avg(e: Column): Column
avg(columnName: String): Column
----

| <<spark-sql-aggregate-functions.adoc#collect_list, collect_list>>
a| [[collect_list]]

[source, scala]
----
collect_list(e: Column): Column
collect_list(columnName: String): Column
----

| <<spark-sql-aggregate-functions.adoc#collect_set, collect_set>>
a| [[collect_set]]

[source, scala]
----
collect_set(e: Column): Column
collect_set(columnName: String): Column
----

| <<spark-sql-aggregate-functions.adoc#corr, corr>>
a| [[corr]]

[source, scala]
----
corr(column1: Column, column2: Column): Column
corr(columnName1: String, columnName2: String): Column
----

| <<spark-sql-aggregate-functions.adoc#count, count>>
a| [[count]]

[source, scala]
----
count(e: Column): Column
count(columnName: String): TypedColumn[Any, Long]
----

| <<spark-sql-aggregate-functions.adoc#countDistinct, countDistinct>>
a| [[countDistinct]]

[source, scala]
----
countDistinct(expr: Column, exprs: Column*): Column
countDistinct(columnName: String, columnNames: String*): Column
----

| <<spark-sql-aggregate-functions.adoc#covar_pop, covar_pop>>
a| [[covar_pop]]

[source, scala]
----
covar_pop(column1: Column, column2: Column): Column
covar_pop(columnName1: String, columnName2: String): Column
----

| <<spark-sql-aggregate-functions.adoc#covar_samp, covar_samp>>
a| [[covar_samp]]

[source, scala]
----
covar_samp(column1: Column, column2: Column): Column
covar_samp(columnName1: String, columnName2: String): Column
----

| <<spark-sql-aggregate-functions.adoc#first, first>>
a| [[first]]

[source, scala]
----
first(e: Column): Column
first(e: Column, ignoreNulls: Boolean): Column
first(columnName: String): Column
first(columnName: String, ignoreNulls: Boolean): Column
----

Returns the first value in a group. Returns the first non-null value when `ignoreNulls` flag on. If all values are null, then returns null.

| <<spark-sql-aggregate-functions.adoc#grouping, grouping>>
a| [[grouping]]

[source, scala]
----
grouping(e: Column): Column
grouping(columnName: String): Column
----

Indicates whether a given column is aggregated or not

| <<spark-sql-aggregate-functions.adoc#grouping_id, grouping_id>>
a| [[grouping_id]]

[source, scala]
----
grouping_id(cols: Column*): Column
grouping_id(colName: String, colNames: String*): Column
----

Computes the level of grouping

| <<spark-sql-aggregate-functions.adoc#kurtosis, kurtosis>>
a| [[kurtosis]]

[source, scala]
----
kurtosis(e: Column): Column
kurtosis(columnName: String): Column
----

| <<spark-sql-aggregate-functions.adoc#last, last>>
a| [[last]]

[source, scala]
----
last(e: Column, ignoreNulls: Boolean): Column
last(columnName: String, ignoreNulls: Boolean): Column
last(e: Column): Column
last(columnName: String): Column
----

| <<spark-sql-aggregate-functions.adoc#max, max>>
a| [[max]]

[source, scala]
----
max(e: Column): Column
max(columnName: String): Column
----

| <<spark-sql-aggregate-functions.adoc#mean, mean>>
a| [[mean]]

[source, scala]
----
mean(e: Column): Column
mean(columnName: String): Column
----

| <<spark-sql-aggregate-functions.adoc#min, min>>
a| [[min]]

[source, scala]
----
min(e: Column): Column
min(columnName: String): Column
----

| <<spark-sql-aggregate-functions.adoc#skewness, skewness>>
a| [[skewness]]

[source, scala]
----
skewness(e: Column): Column
skewness(columnName: String): Column
----

| <<spark-sql-aggregate-functions.adoc#stddev, stddev>>
a| [[stddev]]

[source, scala]
----
stddev(e: Column): Column
stddev(columnName: String): Column
----

| <<spark-sql-aggregate-functions.adoc#stddev_pop, stddev_pop>>
a| [[stddev_pop]]

[source, scala]
----
stddev_pop(e: Column): Column
stddev_pop(columnName: String): Column
----

| <<spark-sql-aggregate-functions.adoc#stddev_samp, stddev_samp>>
a| [[stddev_samp]]

[source, scala]
----
stddev_samp(e: Column): Column
stddev_samp(columnName: String): Column
----

| <<spark-sql-aggregate-functions.adoc#sum, sum>>
a| [[sum]]

[source, scala]
----
sum(e: Column): Column
sum(columnName: String): Column
----

| <<spark-sql-aggregate-functions.adoc#sumDistinct, sumDistinct>>
a| [[sumDistinct]]

[source, scala]
----
sumDistinct(e: Column): Column
sumDistinct(columnName: String): Column
----

| <<spark-sql-aggregate-functions.adoc#variance, variance>>
a| [[variance]]

[source, scala]
----
variance(e: Column): Column
variance(columnName: String): Column
----

| <<spark-sql-aggregate-functions.adoc#var_pop, var_pop>>
a| [[var_pop]]

[source, scala]
----
var_pop(e: Column): Column
var_pop(columnName: String): Column
----

| <<spark-sql-aggregate-functions.adoc#var_samp, var_samp>>
a| [[var_samp]]

[source, scala]
----
var_samp(e: Column): Column
var_samp(columnName: String): Column
----

.31+^.^| [[collection_funcs]] *Collection functions*

| <<spark-sql-functions-collection.adoc#array_contains, array_contains>>
a| [[array_contains]]

[source, scala]
----
array_contains(column: Column, value: Any): Column
----

| <<spark-sql-functions-collection.adoc#array_distinct, array_distinct>>
a| [[array_distinct]]

[source, scala]
----
array_distinct(e: Column): Column
----

(*New in 2.4.0*)

| <<spark-sql-functions-collection.adoc#array_except, array_except>>
a| [[array_except]]

[source, scala]
----
array_except(e: Column): Column
----

(*New in 2.4.0*)

| <<spark-sql-functions-collection.adoc#array_intersect, array_intersect>>
a| [[array_intersect]]

[source, scala]
----
array_intersect(col1: Column, col2: Column): Column
----

(*New in 2.4.0*)

| <<spark-sql-functions-collection.adoc#array_join, array_join>>
a| [[array_join]]

[source, scala]
----
array_join(column: Column, delimiter: String): Column
array_join(column: Column, delimiter: String, nullReplacement: String): Column
----

(*New in 2.4.0*)

| <<spark-sql-functions-collection.adoc#array_max, array_max>>
a| [[array_max]]

[source, scala]
----
array_max(e: Column): Column
----

(*New in 2.4.0*)

| <<spark-sql-functions-collection.adoc#array_min, array_min>>
a| [[array_min]]

[source, scala]
----
array_min(e: Column): Column
----

(*New in 2.4.0*)

| <<spark-sql-functions-collection.adoc#array_position, array_position>>
a| [[array_position]]

[source, scala]
----
array_position(column: Column, value: Any): Column
----

(*New in 2.4.0*)

| <<spark-sql-functions-collection.adoc#array_remove, array_remove>>
a| [[array_remove]]

[source, scala]
----
array_remove(column: Column, element: Any): Column
----

(*New in 2.4.0*)

| <<spark-sql-functions-collection.adoc#array_repeat, array_repeat>>
a| [[array_repeat]]

[source, scala]
----
array_repeat(e: Column, count: Int): Column
array_repeat(left: Column, right: Column): Column
----

(*New in 2.4.0*)

| <<spark-sql-functions-collection.adoc#array_sort, array_sort>>
a| [[array_sort]]

[source, scala]
----
array_sort(e: Column): Column
----

(*New in 2.4.0*)

| <<spark-sql-functions-collection.adoc#array_union, array_union>>
a| [[array_union]]

[source, scala]
----
array_union(col1: Column, col2: Column): Column
----

(*New in 2.4.0*)

| <<spark-sql-functions-collection.adoc#arrays_zip, arrays_zip>>
a| [[arrays_zip]]

[source, scala]
----
arrays_zip(e: Column*): Column
----

(*New in 2.4.0*)

| <<spark-sql-functions-collection.adoc#arrays_overlap, arrays_overlap>>
a| [[arrays_overlap]]

[source, scala]
----
arrays_overlap(a1: Column, a2: Column): Column
----

(*New in 2.4.0*)

| <<spark-sql-functions-collection.adoc#element_at, element_at>>
a| [[element_at]]

[source, scala]
----
element_at(column: Column, value: Any): Column
----

(*New in 2.4.0*)

| link:spark-sql-functions-collection.adoc#explode[explode]
a| [[explode]]

[source, scala]
----
explode(e: Column): Column
----

| link:spark-sql-functions-collection.adoc#explode_outer[explode_outer]
a| [[explode_outer]]

[source, scala]
----
explode_outer(e: Column): Column
----

Creates a new row for each element in the given array or map column. If the array/map is `null` or empty then `null` is produced.

| <<spark-sql-functions-collection.adoc#flatten, flatten>>
a| [[flatten]]

[source, scala]
----
flatten(e: Column): Column
----

(*New in 2.4.0*)

| <<spark-sql-functions-collection.adoc#from_json, from_json>>
a| [[from_json]]

[source, scala]
----
from_json(e: Column, schema: Column): Column // <1>
from_json(e: Column, schema: DataType): Column
from_json(e: Column, schema: DataType, options: Map[String, String]): Column
from_json(e: Column, schema: String, options: Map[String, String]): Column
from_json(e: Column, schema: StructType): Column
from_json(e: Column, schema: StructType, options: Map[String, String]): Column
----
<1> *New in 2.4.0*

Parses a column with a JSON string into a link:spark-sql-StructType.adoc[StructType] or link:spark-sql-DataType.adoc#ArrayType[ArrayType] of `StructType` elements with the specified schema.

| <<spark-sql-functions-collection.adoc#map_concat, map_concat>>
a| [[map_concat]]

[source, scala]
----
map_concat(cols: Column*): Column
----

(*New in 2.4.0*)

| <<spark-sql-functions-collection.adoc#map_from_entries, map_from_entries>>
a| [[map_from_entries]]

[source, scala]
----
map_from_entries(e: Column): Column
----

(*New in 2.4.0*)

| <<spark-sql-functions-collection.adoc#map_keys, map_keys>>
a| [[map_keys]]

[source, scala]
----
map_keys(e: Column): Column
----

| <<spark-sql-functions-collection.adoc#map_values, map_values>>
a| [[map_values]]

[source, scala]
----
map_values(e: Column): Column
----

| <<spark-sql-functions-collection.adoc#posexplode, posexplode>>
a| [[posexplode]]

[source, scala]
----
posexplode(e: Column): Column
----

| <<spark-sql-functions-collection.adoc#posexplode_outer, posexplode_outer>>
a| [[posexplode_outer]]

[source, scala]
----
posexplode_outer(e: Column): Column
----

| <<spark-sql-functions-collection.adoc#reverse, reverse>>
a| [[reverse]]

[source, scala]
----
reverse(e: Column): Column
----

Returns a reversed string or an array with reverse order of elements

NOTE: Support for reversing arrays is *new in 2.4.0*.

| <<spark-sql-functions-collection.adoc#schema_of_json, schema_of_json>>
a| [[schema_of_json]]

[source, scala]
----
schema_of_json(json: Column): Column
schema_of_json(json: String): Column
----

(*New in 2.4.0*)

| <<spark-sql-functions-collection.adoc#sequence, sequence>>
a| [[sequence]]

[source, scala]
----
sequence(start: Column, stop: Column): Column
sequence(start: Column, stop: Column, step: Column): Column
----

(*New in 2.4.0*)

| <<spark-sql-functions-collection.adoc#shuffle, shuffle>>
a| [[shuffle]]

[source, scala]
----
shuffle(e: Column): Column
----

(*New in 2.4.0*)

| <<spark-sql-functions-collection.adoc#size, size>>
a| [[size]]

[source, scala]
----
size(e: Column): Column
----

Returns the size of the given array or map. Returns -1 if `null`.

| <<spark-sql-functions-collection.adoc#slice, slice>>
a| [[slice]]

[source, scala]
----
slice(x: Column, start: Int, length: Int): Column
----

(*New in 2.4.0*)

.9+^.^| [[datetime_funcs]] *Date and time functions*
| <<spark-sql-functions-datetime.adoc#current_date, current_date>>
a| [[current_date]]

[source, scala]
----
current_date(): Column
----

| <<spark-sql-functions-datetime.adoc#current_timestamp, current_timestamp>>
a| [[current_timestamp]]

[source, scala]
----
current_timestamp(): Column
----

| <<spark-sql-functions-datetime.adoc#from_utc_timestamp, from_utc_timestamp>>
a| [[from_utc_timestamp]]

[source, scala]
----
from_utc_timestamp(ts: Column, tz: String): Column
from_utc_timestamp(ts: Column, tz: Column): Column  // <1>
----
<1> *New in 2.4.0*

| <<spark-sql-functions-datetime.adoc#months_between, months_between>>
a| [[months_between]]

[source, scala]
----
months_between(end: Column, start: Column): Column
months_between(end: Column, start: Column, roundOff: Boolean): Column // <1>
----
<1> *New in 2.4.0*

| <<spark-sql-functions-datetime.adoc#to_date, to_date>>
a| [[to_date]]

[source, scala]
----
to_date(e: Column): Column
to_date(e: Column, fmt: String): Column
----

| <<spark-sql-functions-datetime.adoc#to_timestamp, to_timestamp>>
a| [[to_timestamp]]

[source, scala]
----
to_timestamp(s: Column): Column
to_timestamp(s: Column, fmt: String): Column
----

| <<spark-sql-functions-datetime.adoc#to_utc_timestamp, to_utc_timestamp>>
a| [[to_utc_timestamp]]

[source, scala]
----
to_utc_timestamp(ts: Column, tz: String): Column
to_utc_timestamp(ts: Column, tz: Column): Column // <1>
----
<1> *New in 2.4.0*

| <<spark-sql-functions-datetime.adoc#unix_timestamp, unix_timestamp>>
a| [[unix_timestamp]] Converts current or specified time to Unix timestamp (in seconds)

[source, scala]
----
unix_timestamp(): Column
unix_timestamp(s: Column): Column
unix_timestamp(s: Column, p: String): Column
----

| <<spark-sql-functions-datetime.adoc#window, window>>
a| [[window]] Generates tumbling time windows

[source, scala]
----
window(
  timeColumn: Column,
  windowDuration: String): Column
window(
  timeColumn: Column,
  windowDuration: String,
  slideDuration: String): Column
window(
  timeColumn: Column,
  windowDuration: String,
  slideDuration: String,
  startTime: String): Column
----

1+^.^| *Math functions*
| <<bin, bin>>
| Converts the value of a long column to binary format

.11+^.^| *Regular functions* (Non-aggregate functions)

| [[array]] link:spark-sql-functions-regular-functions.adoc#array[array]
|

| [[broadcast]] link:spark-sql-functions-regular-functions.adoc#broadcast[broadcast]
|

| [[coalesce]] link:spark-sql-functions-regular-functions.adoc#coalesce[coalesce]
| Gives the first non-``null`` value among the given columns or `null`

| [[col]][[column]] link:spark-sql-functions-regular-functions.adoc#col[col] and link:spark-sql-functions-regular-functions.adoc#column[column]
| Creating link:spark-sql-Column.adoc[Columns]

| xref:spark-sql-functions-regular-functions.adoc#expr[expr]
| [[expr]]

| [[lit]] link:spark-sql-functions-regular-functions.adoc#lit[lit]
|

| [[map]] link:spark-sql-functions-regular-functions.adoc#map[map]
|

| <<spark-sql-functions-regular-functions.adoc#monotonically_increasing_id, monotonically_increasing_id>>
| [[monotonically_increasing_id]] Returns monotonically increasing 64-bit integers that are guaranteed to be monotonically increasing and unique, but not consecutive.

| [[struct]] link:spark-sql-functions-regular-functions.adoc#struct[struct]
|

| [[typedLit]] link:spark-sql-functions-regular-functions.adoc#typedLit[typedLit]
|

| [[when]] link:spark-sql-functions-regular-functions.adoc#when[when]
|

.2+^.^| *String functions*
| <<split, split>>
|

| <<upper, upper>>
|

1.2+^.^| *UDF functions*
| <<udf, udf>>
| Creating UDFs

| <<callUDF, callUDF>>
| Executing an UDF by name with variable-length list of columns

.11+^.^| [[window-functions]] *Window functions*

| [[cume_dist]] <<spark-sql-functions-windows.adoc#cume_dist, cume_dist>>
a|

[source, scala]
----
cume_dist(): Column
----

Computes the cumulative distribution of records across window partitions

| [[currentRow]] <<spark-sql-functions-windows.adoc#currentRow, currentRow>>
a|

[source, scala]
----
currentRow(): Column
----

| [[dense_rank]] <<spark-sql-functions-windows.adoc#dense_rank, dense_rank>>
a|

[source, scala]
----
dense_rank(): Column
----

Computes the rank of records per window partition

| [[lag]] <<spark-sql-functions-windows.adoc#lag, lag>>
a|

[source, scala]
----
lag(e: Column, offset: Int): Column
lag(columnName: String, offset: Int): Column
lag(columnName: String, offset: Int, defaultValue: Any): Column
----

| [[lead]] <<spark-sql-functions-windows.adoc#lead, lead>>
a|

[source, scala]
----
lead(columnName: String, offset: Int): Column
lead(e: Column, offset: Int): Column
lead(columnName: String, offset: Int, defaultValue: Any): Column
lead(e: Column, offset: Int, defaultValue: Any): Column
----

| [[ntile]] <<spark-sql-functions-windows.adoc#ntile, ntile>>
a|

[source, scala]
----
ntile(n: Int): Column
----

Computes the ntile group

| [[percent_rank]] <<spark-sql-functions-windows.adoc#percent_rank, percent_rank>>
a|

[source, scala]
----
percent_rank(): Column
----

Computes the rank of records per window partition

| [[rank]] <<spark-sql-functions-windows.adoc#rank, rank>>
a|

[source, scala]
----
rank(): Column
----

Computes the rank of records per window partition

| [[row_number]] <<spark-sql-functions-windows.adoc#row_number, row_number>>
a|

[source, scala]
----
row_number(): Column
----

Computes the sequential numbering per window partition

| [[unboundedFollowing]] <<spark-sql-functions-windows.adoc#unboundedFollowing, unboundedFollowing>>
a|

[source, scala]
----
unboundedFollowing(): Column
----

| [[unboundedPreceding]] <<spark-sql-functions-windows.adoc#unboundedPreceding, unboundedPreceding>>
a|

[source, scala]
----
unboundedPreceding(): Column
----
|===

TIP: The page gives only a brief ovierview of the many functions available in `functions` object and so you should read the http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.functions$[official documentation of the `functions` object].

=== [[callUDF]] Executing UDF by Name and Variable-Length Column List -- `callUDF` Function

[source, scala]
----
callUDF(udfName: String, cols: Column*): Column
----

`callUDF` executes an UDF by `udfName` and variable-length list of columns.

=== [[udf]] Defining UDFs -- `udf` Function

[source, scala]
----
udf(f: FunctionN[...]): UserDefinedFunction
----

The `udf` family of functions allows you to create link:spark-sql-udfs.adoc[user-defined functions (UDFs)] based on a user-defined function in Scala. It accepts `f` function of 0 to 10 arguments and the input and output types are automatically inferred (given the types of the respective input and output types of the function `f`).

[source, scala]
----
import org.apache.spark.sql.functions._
val _length: String => Int = _.length
val _lengthUDF = udf(_length)

// define a dataframe
val df = sc.parallelize(0 to 3).toDF("num")

// apply the user-defined function to "num" column
scala> df.withColumn("len", _lengthUDF($"num")).show
+---+---+
|num|len|
+---+---+
|  0|  1|
|  1|  1|
|  2|  1|
|  3|  1|
+---+---+
----

Since Spark 2.0.0, there is another variant of `udf` function:

[source, scala]
----
udf(f: AnyRef, dataType: DataType): UserDefinedFunction
----

`udf(f: AnyRef, dataType: DataType)` allows you to use a Scala closure for the function argument (as `f`) and explicitly declaring the output data type (as `dataType`).

[source, scala]
----
// given the dataframe above

import org.apache.spark.sql.types.IntegerType
val byTwo = udf((n: Int) => n * 2, IntegerType)

scala> df.withColumn("len", byTwo($"num")).show
+---+---+
|num|len|
+---+---+
|  0|  0|
|  1|  2|
|  2|  4|
|  3|  6|
+---+---+
----

=== [[split]] `split` Function

[source, scala]
----
split(str: Column, pattern: String): Column
----

`split` function splits `str` column using `pattern`. It returns a new `Column`.

NOTE: `split` UDF uses https://docs.oracle.com/javase/8/docs/api/java/lang/String.html#split-java.lang.String-int-[java.lang.String.split(String regex, int limit)] method.

[source, scala]
----
val df = Seq((0, "hello|world"), (1, "witaj|swiecie")).toDF("num", "input")
val withSplit = df.withColumn("split", split($"input", "[|]"))

scala> withSplit.show
+---+-------------+----------------+
|num|        input|           split|
+---+-------------+----------------+
|  0|  hello|world|  [hello, world]|
|  1|witaj|swiecie|[witaj, swiecie]|
+---+-------------+----------------+
----

NOTE: `.$|()[{^?*+\` are RegEx's meta characters and are considered special.

=== [[upper]] `upper` Function

[source, scala]
----
upper(e: Column): Column
----

`upper` function converts a string column into one with all letter upper. It returns a new `Column`.

NOTE: The following example uses two functions that accept a `Column` and return another to showcase how to chain them.

[source, scala]
----
val df = Seq((0,1,"hello"), (2,3,"world"), (2,4, "ala")).toDF("id", "val", "name")
val withUpperReversed = df.withColumn("upper", reverse(upper($"name")))

scala> withUpperReversed.show
+---+---+-----+-----+
| id|val| name|upper|
+---+---+-----+-----+
|  0|  1|hello|OLLEH|
|  2|  3|world|DLROW|
|  2|  4|  ala|  ALA|
+---+---+-----+-----+
----

=== [[bin]] Converting Long to Binary Format (in String Representation) -- `bin` Function

[source, scala]
----
bin(e: Column): Column
bin(columnName: String): Column // <1>
----
<1> Calls the first `bin` with `columnName` as a `Column`

`bin` converts the long value in a column to its binary format (i.e. as an unsigned integer in base 2) with no extra leading 0s.

[source, scala]
----
scala> spark.range(5).withColumn("binary", bin('id)).show
+---+------+
| id|binary|
+---+------+
|  0|     0|
|  1|     1|
|  2|    10|
|  3|    11|
|  4|   100|
+---+------+

val withBin = spark.range(5).withColumn("binary", bin('id))
scala> withBin.printSchema
root
 |-- id: long (nullable = false)
 |-- binary: string (nullable = false)
----

Internally, `bin` creates a link:spark-sql-Column.adoc[Column] with `Bin` unary expression.

[source, scala]
----
scala> withBin.queryExecution.logical
res2: org.apache.spark.sql.catalyst.plans.logical.LogicalPlan =
'Project [*, bin('id) AS binary#14]
+- Range (0, 5, step=1, splits=Some(8))
----

NOTE: `Bin` unary expression uses link:++https://docs.oracle.com/javase/8/docs/api/java/lang/Long.html#toBinaryString-long-++[java.lang.Long.toBinaryString] for the conversion.

[NOTE]
====
`Bin` expression supports link:spark-sql-Expression.adoc#doGenCode[code generation] (aka _CodeGen_).

```
val withBin = spark.range(5).withColumn("binary", bin('id))
scala> withBin.queryExecution.debug.codegen
Found 1 WholeStageCodegen subtrees.
== Subtree 1 / 1 ==
*Project [id#19L, bin(id#19L) AS binary#22]
+- *Range (0, 5, step=1, splits=Some(8))
...
/* 103 */           UTF8String project_value1 = null;
/* 104 */           project_value1 = UTF8String.fromString(java.lang.Long.toBinaryString(range_value));

```
====
