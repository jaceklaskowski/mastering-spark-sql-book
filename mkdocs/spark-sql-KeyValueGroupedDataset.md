title: KeyValueGroupedDataset

# KeyValueGroupedDataset -- Typed Grouping

`KeyValueGroupedDataset` is an experimental interface to <<operators, calculate aggregates over groups of objects>> in a typed link:spark-sql-Dataset.adoc[Dataset].

NOTE: link:spark-sql-RelationalGroupedDataset.adoc[RelationalGroupedDataset] is used for untyped ``Row``-based aggregates.

`KeyValueGroupedDataset` is created using link:spark-sql-basic-aggregation.adoc#groupByKey[Dataset.groupByKey] operator.

[source, scala]
----
val dataset: Dataset[Token] = ...
scala> val tokensByName = dataset.groupByKey(_.name)
tokensByName: org.apache.spark.sql.KeyValueGroupedDataset[String,Token] = org.apache.spark.sql.KeyValueGroupedDataset@1e3aad46
----

[[operators]]
.KeyValueGroupedDataset's Aggregate Operators (KeyValueGroupedDataset API)
[cols="1,3",options="header",width="100%"]
|===
| Operator
| Description

| `agg`
| [[agg]]

| `cogroup`
|

| `count`
|

| `flatMapGroups`
|

| `flatMapGroupsWithState`
|

| `keys`
|

| `keyAs`
|

| `mapGroups`
|

| `mapGroupsWithState`
|

| `mapValues`
|

| `reduceGroups`
|
|===

`KeyValueGroupedDataset` holds `keys` that were used for the object.

[source, scala]
----
scala> tokensByName.keys.show
+-----+
|value|
+-----+
|  aaa|
|  bbb|
+-----+
----

=== [[aggUntyped]] `aggUntyped` Internal Method

[source, scala]
----
aggUntyped(columns: TypedColumn[_, _]*): Dataset[_]
----

`aggUntyped`...FIXME

NOTE: `aggUntyped` is used exclusively when <<agg, KeyValueGroupedDataset.agg>> typed operator is used.

=== [[logicalPlan]] `logicalPlan` Internal Method

[source, scala]
----
logicalPlan: AnalysisBarrier
----

`logicalPlan`...FIXME

NOTE: `logicalPlan` is used when...FIXME
