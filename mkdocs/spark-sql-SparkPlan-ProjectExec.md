title: ProjectExec

# ProjectExec Unary Physical Operator

`ProjectExec` is a link:spark-sql-SparkPlan.adoc#UnaryExecNode[unary physical operator] (i.e. with one <<child, child>> physical operator) that...FIXME

`ProjectExec` supports link:spark-sql-CodegenSupport.adoc[Java code generation] (aka _codegen_).

`ProjectExec` is <<creating-instance, created>> when:

* link:spark-sql-SparkStrategy-InMemoryScans.adoc[InMemoryScans] and link:hive/HiveTableScans.adoc[HiveTableScans] execution planning strategies are executed (and request `SparkPlanner` to link:spark-sql-SparkPlanner.adoc#pruneFilterProject[pruneFilterProject])

* `BasicOperators` execution planning strategy is requested to link:spark-sql-SparkStrategy-BasicOperators.adoc#Project[resolve a Project logical operator]

* `DataSourceStrategy` execution planning strategy is requested to link:spark-sql-SparkStrategy-DataSourceStrategy.adoc#pruneFilterProjectRaw[creates a RowDataSourceScanExec]

* `FileSourceStrategy` execution planning strategy is requested to link:spark-sql-SparkStrategy-FileSourceStrategy.adoc#apply[plan a LogicalRelation with a HadoopFsRelation]

* `ExtractPythonUDFs` physical optimization is requested to link:spark-sql-ExtractPythonUDFs.adoc#apply[optimize a physical query plan] (and link:spark-sql-ExtractPythonUDFs.adoc#extract[extracts Python UDFs])

[NOTE]
====
The following is the order of applying the above execution planning strategies to logical query plans when `SparkPlanner` or link:hive/HiveSessionStateBuilder.adoc#planner[Hive-specific SparkPlanner] are requested to link:spark-sql-catalyst-QueryPlanner.adoc#plan[plan a logical query plan into one or more physical query plans]:

. link:hive/HiveTableScans.adoc[HiveTableScans]
. link:spark-sql-SparkStrategy-FileSourceStrategy.adoc[FileSourceStrategy]
. link:spark-sql-SparkStrategy-DataSourceStrategy.adoc[DataSourceStrategy]
. link:spark-sql-SparkStrategy-InMemoryScans.adoc[InMemoryScans]
. link:spark-sql-SparkStrategy-BasicOperators.adoc[BasicOperators]
====

=== [[doExecute]] Executing Physical Operator (Generating RDD[InternalRow]) -- `doExecute` Method

[source, scala]
----
doExecute(): RDD[InternalRow]
----

NOTE: `doExecute` is part of <<spark-sql-SparkPlan.adoc#doExecute, SparkPlan Contract>> to generate the runtime representation of a structured query as a distributed computation over <<spark-sql-InternalRow.adoc#, internal binary rows>> on Apache Spark (i.e. `RDD[InternalRow]`).

`doExecute` requests the input <<child, child physical plan>> to link:spark-sql-SparkPlan.adoc#execute[produce an RDD of internal rows] and applies a <<doExecute-mapPartitionsWithIndexInternal, calculation over indexed partitions>> (using `RDD.mapPartitionsWithIndexInternal`).

.RDD.mapPartitionsWithIndexInternal
[source, scala]
----
mapPartitionsWithIndexInternal[U](
  f: (Int, Iterator[T]) => Iterator[U],
  preservesPartitioning: Boolean = false)
----

==== [[doExecute-mapPartitionsWithIndexInternal]] Inside `doExecute` (`RDD.mapPartitionsWithIndexInternal`)

Inside the function (that is part of `RDD.mapPartitionsWithIndexInternal`), `doExecute` creates an link:spark-sql-UnsafeProjection.adoc#create[UnsafeProjection] with the following:

. <<projectList, Named expressions>>

. link:spark-sql-catalyst-QueryPlan.adoc#output[Output] of the <<child, child>> physical operator as the input schema

. link:spark-sql-SparkPlan.adoc#subexpressionEliminationEnabled[subexpressionEliminationEnabled] flag

`doExecute` requests the `UnsafeProjection` to link:spark-sql-Projection.adoc#initialize[initialize] and maps over the internal rows (of a partition) using the projection.

=== [[creating-instance]] Creating ProjectExec Instance

`ProjectExec` takes the following when created:

* [[projectList]] link:spark-sql-Expression-NamedExpression.adoc[NamedExpressions] for the projection
* [[child]] Child link:spark-sql-SparkPlan.adoc[physical operator]

=== [[doConsume]] Generating Java Source Code for Consume Path in Whole-Stage Code Generation -- `doConsume` Method

[source, scala]
----
doConsume(ctx: CodegenContext, input: Seq[ExprCode], row: ExprCode): String
----

NOTE: `doConsume` is part of <<spark-sql-CodegenSupport.adoc#doConsume, CodegenSupport Contract>> to generate the Java source code for <<spark-sql-whole-stage-codegen.adoc#consume-path, consume path>> in Whole-Stage Code Generation.

`doConsume`...FIXME
