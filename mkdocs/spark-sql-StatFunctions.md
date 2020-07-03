# StatFunctions Helper Object

`StatFunctions` is a Scala object that defines the <<methods, methods>> that are used for...FIXME

[[methods]]
.StatFunctions API
[cols="1,3",options="header",width="100%"]
|===
| Method
| Description

| <<calculateCov, calculateCov>>
a|
[source, scala]
----
calculateCov(df: DataFrame, cols: Seq[String]): Double
----

| <<crossTabulate, crossTabulate>>
a|
[source, scala]
----
crossTabulate(df: DataFrame, col1: String, col2: String): DataFrame
----

| <<multipleApproxQuantiles, multipleApproxQuantiles>>
a|
[source, scala]
----
multipleApproxQuantiles(
  df: DataFrame,
  cols: Seq[String],
  probabilities: Seq[Double],
  relativeError: Double): Seq[Seq[Double]]
----

| <<pearsonCorrelation, pearsonCorrelation>>
a|
[source, scala]
----
pearsonCorrelation(df: DataFrame, cols: Seq[String]): Double
----

| <<summary, summary>>
a|
[source, scala]
----
summary(ds: Dataset[_], statistics: Seq[String]): DataFrame
----
|===

=== [[calculateCov]] `calculateCov` Method

[source, scala]
----
calculateCov(df: DataFrame, cols: Seq[String]): Double
----

`calculateCov`...FIXME

NOTE: `calculateCov` is used when...FIXME

=== [[crossTabulate]] `crossTabulate` Method

[source, scala]
----
crossTabulate(
  df: DataFrame,
  col1: String,
  col2: String): DataFrame
----

`crossTabulate`...FIXME

NOTE: `crossTabulate` is used when...FIXME

=== [[multipleApproxQuantiles]] `multipleApproxQuantiles` Method

[source, scala]
----
multipleApproxQuantiles(
  df: DataFrame,
  cols: Seq[String],
  probabilities: Seq[Double],
  relativeError: Double): Seq[Seq[Double]]
----

`multipleApproxQuantiles`...FIXME

NOTE: `multipleApproxQuantiles` is used when...FIXME

=== [[pearsonCorrelation]] `pearsonCorrelation` Method

[source, scala]
----
pearsonCorrelation(df: DataFrame, cols: Seq[String]): Double
----

`pearsonCorrelation`...FIXME

NOTE: `pearsonCorrelation` is used when...FIXME

=== [[summary]] Generate Summary Statistics of Dataset (as DataFrame) -- `summary` Method

[source, scala]
----
summary(
  ds: Dataset[_],
  statistics: Seq[String]): DataFrame
----

`summary`...FIXME

NOTE: `summary` is used exclusively when <<spark-sql-dataset-operators.adoc#summary, Dataset.summary>> action is used.
