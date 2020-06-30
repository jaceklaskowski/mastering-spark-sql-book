title: DataFrameStatFunctions

# DataFrameStatFunctions -- Working With Statistic Functions

`DataFrameStatFunctions` is used to work with <<methods, statistic functions>> in a structured query (a <<spark-sql-DataFrame.adoc#, DataFrame>>).

[[methods]]
.DataFrameStatFunctions API
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| <<approxQuantile, approxQuantile>>
a|

[source, scala]
----
approxQuantile(
  cols: Array[String],
  probabilities: Array[Double],
  relativeError: Double): Array[Array[Double]]
approxQuantile(
  col: String,
  probabilities: Array[Double],
  relativeError: Double): Array[Double]
----

| <<bloomFilter, bloomFilter>>
a|

[source, scala]
----
bloomFilter(col: Column, expectedNumItems: Long, fpp: Double): BloomFilter
bloomFilter(col: Column, expectedNumItems: Long, numBits: Long): BloomFilter
bloomFilter(colName: String, expectedNumItems: Long, fpp: Double): BloomFilter
bloomFilter(colName: String, expectedNumItems: Long, numBits: Long): BloomFilter
----

| <<corr, corr>>
a|

[source, scala]
----
corr(col1: String, col2: String): Double
corr(col1: String, col2: String, method: String): Double
----

| <<countMinSketch, countMinSketch>>
a|

[source, scala]
----
countMinSketch(col: Column, eps: Double, confidence: Double, seed: Int): CountMinSketch
countMinSketch(col: Column, depth: Int, width: Int, seed: Int): CountMinSketch
countMinSketch(colName: String, eps: Double, confidence: Double, seed: Int): CountMinSketch
countMinSketch(colName: String, depth: Int, width: Int, seed: Int): CountMinSketch
----

| <<cov, cov>>
a|

[source, scala]
----
cov(col1: String, col2: String): Double
----

| <<crosstab, crosstab>>
a|

[source, scala]
----
crosstab(col1: String, col2: String): DataFrame
----

| <<freqItems, freqItems>>
a|

[source, scala]
----
freqItems(cols: Array[String]): DataFrame
freqItems(cols: Array[String], support: Double): DataFrame
freqItems(cols: Seq[String]): DataFrame
freqItems(cols: Seq[String], support: Double): DataFrame
----

| <<sampleBy, sampleBy>>
a|

[source, scala]
----
sampleBy[T](col: String, fractions: Map[T, Double], seed: Long): DataFrame
----
|===

[[creating-instance]]
`DataFrameStatFunctions` is available using <<spark-sql-Dataset-untyped-transformations.adoc#stat, stat>> untyped transformation.

[source, scala]
----
val q: DataFrame = ...
q.stat
----

=== [[approxQuantile]] `approxQuantile` Method

[source, scala]
----
approxQuantile(
  cols: Array[String],
  probabilities: Array[Double],
  relativeError: Double): Array[Array[Double]]
approxQuantile(
  col: String,
  probabilities: Array[Double],
  relativeError: Double): Array[Double]
----

`approxQuantile`...FIXME

=== [[bloomFilter]] `bloomFilter` Method

[source, scala]
----
bloomFilter(col: Column, expectedNumItems: Long, fpp: Double): BloomFilter
bloomFilter(col: Column, expectedNumItems: Long, numBits: Long): BloomFilter
bloomFilter(colName: String, expectedNumItems: Long, fpp: Double): BloomFilter
bloomFilter(colName: String, expectedNumItems: Long, numBits: Long): BloomFilter
----

`bloomFilter`...FIXME

=== [[buildBloomFilter]] `buildBloomFilter` Internal Method

[source, scala]
----
buildBloomFilter(col: Column, zero: BloomFilter): BloomFilter
----

`buildBloomFilter`...FIXME

NOTE: `convertToDouble` is used when...FIXME

=== [[corr]] `corr` Method

[source, scala]
----
corr(col1: String, col2: String): Double
corr(col1: String, col2: String, method: String): Double
----

`corr`...FIXME

=== [[countMinSketch]] `countMinSketch` Method

[source, scala]
----
countMinSketch(col: Column, eps: Double, confidence: Double, seed: Int): CountMinSketch
countMinSketch(col: Column, depth: Int, width: Int, seed: Int): CountMinSketch
countMinSketch(colName: String, eps: Double, confidence: Double, seed: Int): CountMinSketch
countMinSketch(colName: String, depth: Int, width: Int, seed: Int): CountMinSketch
// PRIVATE API
countMinSketch(col: Column, zero: CountMinSketch): CountMinSketch
----

`countMinSketch`...FIXME

=== [[cov]] `cov` Method

[source, scala]
----
cov(col1: String, col2: String): Double
----

`cov`...FIXME

=== [[crosstab]] `crosstab` Method

[source, scala]
----
crosstab(col1: String, col2: String): DataFrame
----

`crosstab`...FIXME

=== [[freqItems]] `freqItems` Method

[source, scala]
----
freqItems(cols: Array[String]): DataFrame
freqItems(cols: Array[String], support: Double): DataFrame
freqItems(cols: Seq[String]): DataFrame
freqItems(cols: Seq[String], support: Double): DataFrame
----

`freqItems`...FIXME

=== [[sampleBy]] `sampleBy` Method

[source, scala]
----
sampleBy[T](col: String, fractions: Map[T, Double], seed: Long): DataFrame
----

`sampleBy`...FIXME
