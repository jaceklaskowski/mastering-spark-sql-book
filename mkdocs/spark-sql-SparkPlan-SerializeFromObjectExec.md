title: SerializeFromObjectExec

# SerializeFromObjectExec Unary Physical Operator

`SerializeFromObjectExec` is a <<spark-sql-SparkPlan.adoc#UnaryExecNode, unary physical operator>> (i.e. with one <<child, child>> physical operator) that supports <<spark-sql-CodegenSupport.adoc#, Java code generation>>.

`SerializeFromObjectExec` supports Java code generation with the <<doProduce, doProduce>>, <<doConsume, doConsume>> and <<inputRDDs, inputRDDs>> methods.

`SerializeFromObjectExec` is a <<spark-sql-ObjectConsumerExec.adoc#, ObjectConsumerExec>>.

`SerializeFromObjectExec` is <<creating-instance, created>> exclusively when <<spark-sql-SparkStrategy-BasicOperators.adoc#, BasicOperators>> execution planning strategy is requested to <<spark-sql-SparkStrategy-BasicOperators.adoc#apply, plan>> a `SerializeFromObject` logical operator.

[[inputRDDs]]
[[outputPartitioning]]
`SerializeFromObjectExec` uses the <<child, child>> physical operator when requested for the <<spark-sql-CodegenSupport.adoc#inputRDDs, input RDDs>> and the <<spark-sql-SparkPlan.adoc#outputPartitioning, outputPartitioning>>.

[[output]]
`SerializeFromObjectExec` uses the <<serializer, serializer>> for the <<spark-sql-catalyst-QueryPlan.adoc#output, output schema attributes>>.

=== [[creating-instance]] Creating SerializeFromObjectExec Instance

`SerializeFromObjectExec` takes the following when created:

* [[serializer]] Serializer (as `Seq[NamedExpression]`)
* [[child]] Child <<spark-sql-SparkPlan.adoc#, physical operator>> (that supports <<spark-sql-CodegenSupport.adoc#, Java code generation>>)

=== [[doConsume]] Generating Java Source Code for Consume Path in Whole-Stage Code Generation -- `doConsume` Method

[source, scala]
----
doConsume(ctx: CodegenContext, input: Seq[ExprCode], row: ExprCode): String
----

NOTE: `doConsume` is part of <<spark-sql-CodegenSupport.adoc#doConsume, CodegenSupport Contract>> to generate the Java source code for <<spark-sql-whole-stage-codegen.adoc#consume-path, consume path>> in Whole-Stage Code Generation.

`doConsume`...FIXME

=== [[doProduce]] Generating Java Source Code for Produce Path in Whole-Stage Code Generation -- `doProduce` Method

[source, scala]
----
doProduce(ctx: CodegenContext): String
----

NOTE: `doProduce` is part of <<spark-sql-CodegenSupport.adoc#doProduce, CodegenSupport Contract>> to generate the Java source code for <<spark-sql-whole-stage-codegen.adoc#produce-path, produce path>> in Whole-Stage Code Generation.

`doProduce`...FIXME

=== [[doExecute]] Executing Physical Operator (Generating RDD[InternalRow]) -- `doExecute` Method

[source, scala]
----
doExecute(): RDD[InternalRow]
----

NOTE: `doExecute` is part of <<spark-sql-SparkPlan.adoc#doExecute, SparkPlan Contract>> to generate the runtime representation of a structured query as a distributed computation over <<spark-sql-InternalRow.adoc#, internal binary rows>> on Apache Spark (i.e. `RDD[InternalRow]`).

`doExecute` requests the <<child, child>> physical operator to <<spark-sql-SparkPlan.adoc#execute, execute>> (that triggers physical query planning and generates an `RDD[InternalRow]`) and transforms it by executing the following function on internal rows per partition with index (using `RDD.mapPartitionsWithIndexInternal` that creates another RDD):

. Creates an <<spark-sql-UnsafeProjection.adoc#create, UnsafeProjection>> for the <<serializer, serializer>>

. Requests the `UnsafeProjection` to <<spark-sql-Projection.adoc#initialize, initialize>> (for the partition index)

. Executes the `UnsafeProjection` on all internal binary rows in the partition

NOTE: `doExecute` (by `RDD.mapPartitionsWithIndexInternal`) adds a new `MapPartitionsRDD` to the RDD lineage. Use `RDD.toDebugString` to see the additional `MapPartitionsRDD`.
