title: RowDataSourceScanExec

# RowDataSourceScanExec Leaf Physical Operator

`RowDataSourceScanExec` is a link:spark-sql-SparkPlan-DataSourceScanExec.adoc[DataSourceScanExec] (and so indirectly a link:spark-sql-SparkPlan.adoc#LeafExecNode[leaf physical operator]) for scanning data from a <<relation, BaseRelation>>.

`RowDataSourceScanExec` is <<creating-instance, created>> to represent a link:spark-sql-LogicalPlan-LogicalRelation.adoc[LogicalRelation] with the following scan types when `DataSourceStrategy` execution planning strategy is link:spark-sql-SparkStrategy-DataSourceStrategy.adoc#apply[executed]:

* `CatalystScan`, `PrunedFilteredScan`, `PrunedScan` (indirectly when `DataSourceStrategy` is requested to link:spark-sql-SparkStrategy-DataSourceStrategy.adoc#pruneFilterProjectRaw[pruneFilterProjectRaw])

* `TableScan`

`RowDataSourceScanExec` marks the <<filters, filters>> that are included in the <<handledFilters, handledFilters>> with `*` (star) in the <<metadata, metadata>> that is used for a simple text representation.

[source, scala]
----
// DEMO RowDataSourceScanExec with a simple text representation with stars
----

=== [[doProduce]] Generating Java Source Code for Produce Path in Whole-Stage Code Generation -- `doProduce` Method

[source, scala]
----
doProduce(ctx: CodegenContext): String
----

NOTE: `doProduce` is part of <<spark-sql-CodegenSupport.adoc#doProduce, CodegenSupport Contract>> to generate the Java source code for <<spark-sql-whole-stage-codegen.adoc#produce-path, produce path>> in Whole-Stage Code Generation.

`doProduce`...FIXME

=== [[creating-instance]] Creating RowDataSourceScanExec Instance

`RowDataSourceScanExec` takes the following when created:

* [[fullOutput]] Output schema link:spark-sql-Expression-Attribute.adoc[attributes]
* [[requiredColumnsIndex]] Indices of required columns
* [[filters]] link:spark-sql-Filter.adoc[Filter predicates]
* [[handledFilters]] Handled link:spark-sql-Filter.adoc[filter predicates]
* [[rdd]] RDD of link:spark-sql-InternalRow.adoc[internal binary rows]
* [[relation]] link:spark-sql-BaseRelation.adoc[BaseRelation]
* [[tableIdentifier]] `TableIdentifier`

NOTE: The input <<filters, filter predicates>> and <<handledFilters, handled filters predicates>> are used exclusively for the <<metadata, metadata>> property that is part of link:spark-sql-SparkPlan-DataSourceScanExec.adoc#metadata[DataSourceScanExec Contract] to describe a scan for a link:spark-sql-SparkPlan-DataSourceScanExec.adoc#simpleString[simple text representation (in a query plan tree)].

=== [[metadata]] `metadata` Property

[source, scala]
----
metadata: Map[String, String]
----

NOTE: `metadata` is part of link:spark-sql-SparkPlan-DataSourceScanExec.adoc#metadata[DataSourceScanExec Contract] to describe a scan for a link:spark-sql-SparkPlan-DataSourceScanExec.adoc#simpleString[simple text representation (in a query plan tree)].

`metadata` marks the <<filters, filter predicates>> that are included in the <<handledFilters, handled filters predicates>> with `*` (star).

NOTE: Filter predicates with `*` (star) are to denote filters that are pushed down to a relation (aka _data source_).

In the end, `metadata` creates the following mapping:

. *ReadSchema* with the <<output, output>> converted to link:spark-sql-StructType.adoc#catalogString[catalog representation]

. *PushedFilters* with the marked and unmarked <<filters, filter predicates>>
