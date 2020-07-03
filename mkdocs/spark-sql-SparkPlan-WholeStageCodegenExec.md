title: WholeStageCodegenExec

# WholeStageCodegenExec Unary Physical Operator for Java Code Generation

`WholeStageCodegenExec` is a link:spark-sql-SparkPlan.adoc#UnaryExecNode[unary physical operator] that is one of the two physical operators that lay the foundation for the link:spark-sql-whole-stage-codegen.adoc[Whole-Stage Java Code Generation] for a *Codegened Execution Pipeline* of a structured query.

NOTE: link:spark-sql-SparkPlan-InputAdapter.adoc[InputAdapter] is the other physical operator for Codegened Execution Pipeline of a structured query.

`WholeStageCodegenExec` itself supports the link:spark-sql-CodegenSupport.adoc[Java code generation] and so when <<doExecute, executed>> triggers code generation for the entire child physical plan subtree of a structured query.

[source, scala]
----
val q = spark.range(10).where('id === 4)
scala> q.queryExecution.debug.codegen
Found 1 WholeStageCodegen subtrees.
== Subtree 1 / 1 ==
*(1) Filter (id#3L = 4)
+- *(1) Range (0, 10, step=1, splits=8)

Generated code:
/* 001 */ public Object generate(Object[] references) {
/* 002 */   return new GeneratedIteratorForCodegenStage1(references);
/* 003 */ }
/* 004 */
/* 005 */ final class GeneratedIteratorForCodegenStage1 extends org.apache.spark.sql.execution.BufferedRowIterator {
...
----

[TIP]
====
Consider using link:spark-sql-debugging-query-execution.adoc[Debugging Query Execution facility] to deep dive into the whole-stage code generation.
====

[source, scala]
----
val q = spark.range(10).where('id === 4)
import org.apache.spark.sql.execution.debug._
scala> q.debugCodegen()
Found 1 WholeStageCodegen subtrees.
== Subtree 1 / 1 ==
*(1) Filter (id#0L = 4)
+- *(1) Range (0, 10, step=1, splits=8)

Generated code:
/* 001 */ public Object generate(Object[] references) {
/* 002 */   return new GeneratedIteratorForCodegenStage1(references);
/* 003 */ }
/* 004 */
/* 005 */ final class GeneratedIteratorForCodegenStage1 extends org.apache.spark.sql.execution.BufferedRowIterator {
...
----

[TIP]
====
Use the following to enable comments in generated code.

[source, scala]
----
org.apache.spark.SparkEnv.get.conf.set("spark.sql.codegen.comments", "true")
----
====

[source, scala]
----
val q = spark.range(10).where('id === 4)
import org.apache.spark.sql.execution.debug._
scala> q.debugCodegen()
Found 1 WholeStageCodegen subtrees.
== Subtree 1 / 1 ==
*(1) Filter (id#6L = 4)
+- *(1) Range (0, 10, step=1, splits=8)

Generated code:
/* 001 */ public Object generate(Object[] references) {
/* 002 */   return new GeneratedIteratorForCodegenStage1(references);
/* 003 */ }
/* 004 */
/* 005 */ /**
 * Codegend pipeline for stage (id=1)
 * *(1) Filter (id#6L = 4)
 * +- *(1) Range (0, 10, step=1, splits=8)
 */
/* 006 */ final class GeneratedIteratorForCodegenStage1 extends org.apache.spark.sql.execution.BufferedRowIterator {
...
----

`WholeStageCodegenExec` is <<creating-instance, created>> when:

* `CollapseCodegenStages` physical query optimization is link:spark-sql-CollapseCodegenStages.adoc#apply[executed] (with link:spark-sql-whole-stage-codegen.adoc#spark.sql.codegen.wholeStage[spark.sql.codegen.wholeStage] configuration property enabled)

* `FileSourceScanExec` leaf physical operator is <<spark-sql-SparkPlan-FileSourceScanExec.adoc#doExecute, executed>> (with the <<spark-sql-SparkPlan-FileSourceScanExec.adoc#supportsBatch, supportsBatch>> flag enabled)

* `InMemoryTableScanExec` leaf physical operator is <<spark-sql-SparkPlan-InMemoryTableScanExec.adoc#doExecute, executed>> (with the <<spark-sql-SparkPlan-InMemoryTableScanExec.adoc#supportsBatch, supportsBatch>> flag enabled)

* `DataSourceV2ScanExec` leaf physical operator is <<spark-sql-SparkPlan-DataSourceV2ScanExec.adoc#doExecute, executed>> (with the <<spark-sql-SparkPlan-DataSourceV2ScanExec.adoc#supportsBatch, supportsBatch>> flag enabled)

NOTE: link:spark-sql-whole-stage-codegen.adoc#spark.sql.codegen.wholeStage[spark.sql.codegen.wholeStage] property is enabled by default.

[[creating-instance]]
[[child]]
[[codegenStageId]]
`WholeStageCodegenExec` takes a single `child` link:spark-sql-SparkPlan.adoc[physical operator] (a physical subquery tree) and *codegen stage ID* when created.

NOTE: `WholeStageCodegenExec` <<doCodeGen, requires>> that the single <<child, child>> physical operator <<spark-sql-CodegenSupport.adoc#, supports Java code generation>>.

[source, scala]
----
// RangeExec physical operator does support codegen
import org.apache.spark.sql.execution.RangeExec
import org.apache.spark.sql.catalyst.plans.logical.Range
val rangeExec = RangeExec(Range(start = 0, end = 1, step = 1, numSlices = 1))

import org.apache.spark.sql.execution.WholeStageCodegenExec
val rdd = WholeStageCodegenExec(rangeExec)(codegenStageId = 0).execute()
----

[[generateTreeString]]
`WholeStageCodegenExec` marks the <<child, child>> physical operator with `*` (star) prefix and <<codegenStageId, per-query codegen stage ID>> (in round brackets) in the link:spark-sql-catalyst-TreeNode.adoc#generateTreeString[text representation of a physical plan tree].

[source, scala]
----
scala> println(plan.numberedTreeString)
00 *(1) Project [id#117L]
01 +- *(1) BroadcastHashJoin [id#117L], [cast(id#115 as bigint)], Inner, BuildRight
02    :- *(1) Range (0, 1, step=1, splits=8)
03    +- BroadcastExchange HashedRelationBroadcastMode(List(cast(input[0, int, false] as bigint)))
04       +- Generate explode(ids#112), false, [id#115]
05          +- LocalTableScan [ids#112]
----

NOTE: As `WholeStageCodegenExec` is created as a result of link:spark-sql-CollapseCodegenStages.adoc[CollapseCodegenStages] physical query optimization rule, it is only executed in link:spark-sql-QueryExecution.adoc#executedPlan[executedPlan] phase of a query execution (that you can only notice by the `*` star prefix in a plan output).

[source, scala]
----
val q = spark.range(9)

// we need executedPlan with WholeStageCodegenExec physical operator "injected"
val plan = q.queryExecution.executedPlan

// Note the star prefix of Range that marks WholeStageCodegenExec
// As a matter of fact, there are two physical operators in play here
// i.e. WholeStageCodegenExec with Range as the child
scala> println(plan.numberedTreeString)
00 *Range (0, 9, step=1, splits=8)

// Let's unwrap Range physical operator
// and access the parent WholeStageCodegenExec
import org.apache.spark.sql.execution.WholeStageCodegenExec
val wsce = plan.asInstanceOf[WholeStageCodegenExec]

// Trigger code generation of the entire query plan tree
val (ctx, code) = wsce.doCodeGen

// CodeFormatter can pretty-print the code
import org.apache.spark.sql.catalyst.expressions.codegen.CodeFormatter
scala> println(CodeFormatter.format(code))
/* 001 */ public Object generate(Object[] references) {
/* 002 */   return new GeneratedIterator(references);
/* 003 */ }
/* 004 */
/* 005 */ /**
 * Codegend pipeline for
 * Range (0, 9, step=1, splits=8)
 */
/* 006 */ final class GeneratedIterator extends org.apache.spark.sql.execution.BufferedRowIterator {
...
----

When <<doExecute, executed>>, `WholeStageCodegenExec` gives <<pipelineTime, pipelineTime>> performance metric.

[[metrics]]
.WholeStageCodegenExec's Performance Metrics
[cols="1,2,2",options="header",width="100%"]
|===
| Key
| Name (in web UI)
| Description

| [[pipelineTime]] `pipelineTime`
| (empty)
| Time of how long the whole-stage codegend pipeline has been running (i.e. the elapsed time since the underlying link:spark-sql-BufferedRowIterator.adoc[BufferedRowIterator] had been created and the internal rows were all consumed).
|===

.WholeStageCodegenExec in web UI (Details for Query)
image::images/spark-sql-WholeStageCodegenExec-webui.png[align="center"]

TIP: Use link:spark-sql-Dataset.adoc#explain[explain] operator to know the physical plan of a query and find out whether or not `WholeStageCodegen` is in use.

[source, scala]
----
val q = spark.range(10).where('id === 4)
// Note the stars in the output that are for codegened operators
scala> q.explain
== Physical Plan ==
*Filter (id#0L = 4)
+- *Range (0, 10, step=1, splits=8)
----

NOTE: link:spark-sql-SparkPlan.adoc[Physical plans] that support code generation extend link:spark-sql-CodegenSupport.adoc[CodegenSupport].

[[logging]]
[TIP]
====
Enable `DEBUG` logging level for `org.apache.spark.sql.execution.WholeStageCodegenExec` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.execution.WholeStageCodegenExec=DEBUG
```

Refer to link:spark-logging.adoc[Logging].
====

=== [[doExecute]] Executing Physical Operator (Generating RDD[InternalRow]) -- `doExecute` Method

[source, scala]
----
doExecute(): RDD[InternalRow]
----

NOTE: `doExecute` is part of <<spark-sql-SparkPlan.adoc#doExecute, SparkPlan Contract>> to generate the runtime representation of a structured query as a distributed computation over <<spark-sql-InternalRow.adoc#, internal binary rows>> on Apache Spark (i.e. `RDD[InternalRow]`).

`doExecute` <<doCodeGen, generates the Java source code for the child physical plan subtree>> first and uses `CodeGenerator` to link:spark-sql-CodeGenerator.adoc#compile[compile it] right afterwards.

If compilation goes well, `doExecute` branches off per the number of link:spark-sql-CodegenSupport.adoc#inputRDDs[input RDDs].

NOTE: `doExecute` only supports up to two link:spark-sql-CodegenSupport.adoc#inputRDDs[input RDDs].

CAUTION: FIXME Finish the "success" path



If the size of the generated codes is greater than <<spark-sql-properties.adoc#spark.sql.codegen.hugeMethodLimit, spark.sql.codegen.hugeMethodLimit>> (which defaults to `65535`), `doExecute` prints out the following INFO message:

```
Found too long generated codes and JIT optimization might not work: the bytecode size ([maxCodeSize]) is above the limit [spark.sql.codegen.hugeMethodLimit], and the whole-stage codegen was disabled for this plan (id=[codegenStageId]). To avoid this, you can raise the limit `spark.sql.codegen.hugeMethodLimit`:
[treeString]
```

In the end, `doExecute` requests the <<child, child>> physical operator to <<spark-sql-SparkPlan.adoc#execute, execute>> (that triggers physical query planning and generates an `RDD[InternalRow]`) and returns it.

NOTE: `doExecute` skips requesting the <<child, child>> physical operator to <<spark-sql-SparkPlan.adoc#execute, execute>> for <<spark-sql-SparkPlan-FileSourceScanExec.adoc#, FileSourceScanExec>> leaf physical operator with <<spark-sql-SparkPlan-FileSourceScanExec.adoc#supportsBatch, supportsBatch>> flag enabled (as `FileSourceScanExec` operator uses `WholeStageCodegenExec` operator when <<spark-sql-SparkPlan-FileSourceScanExec.adoc#doExecute, FileSourceScanExec>>).

If compilation fails and link:spark-sql-properties.adoc#spark.sql.codegen.fallback[spark.sql.codegen.fallback] configuration property is enabled, `doExecute` prints out the following WARN message to the logs, requests the <<child, child>> physical operator to link:spark-sql-SparkPlan.adoc#execute[execute] and returns it.

```
Whole-stage codegen disabled for plan (id=[codegenStageId]):
 [treeString]
```

=== [[doCodeGen]] Generating Java Source Code for Child Physical Plan Subtree -- `doCodeGen` Method

[source, scala]
----
doCodeGen(): (CodegenContext, CodeAndComment)
----

`doCodeGen` creates a new <<spark-sql-CodegenContext.adoc#creating-instance, CodegenContext>> and requests the single <<child, child>> physical operator to <<spark-sql-CodegenSupport.adoc#produce, generate a Java source code for produce code path>> (with the new `CodegenContext` and the `WholeStageCodegenExec` physical operator itself).

`doCodeGen` <<spark-sql-CodegenContext.adoc#addNewFunction, adds the new function>> under the name of `processNext`.

`doCodeGen` <<generatedClassName, generates the class name>>.

`doCodeGen` generates the final Java source code of the following format:

[source, scala]
----
public Object generate(Object[] references) {
  return new [className](references);
}

/**
 * Codegend pipeline for stage (id=[codegenStageId])
 * [treeString]
 */
final class [className] extends BufferedRowIterator {

  private Object[] references;
  private scala.collection.Iterator[] inputs;
  // ctx.declareMutableStates()

  public [className](Object[] references) {
    this.references = references;
  }

  public void init(int index, scala.collection.Iterator[] inputs) {
    partitionIndex = index;
    this.inputs = inputs;
    // ctx.initMutableStates()
    // ctx.initPartition()
  }

  // ctx.emitExtraCode()

  // ctx.declareAddedFunctions()
}
----

NOTE: `doCodeGen` requires that the single <<child, child>> physical operator <<spark-sql-CodegenSupport.adoc#, supports Java code generation>>.

`doCodeGen` cleans up the generated code (using `CodeFormatter` to `stripExtraNewLines`, `stripOverlappingComments`).

`doCodeGen` prints out the following DEBUG message to the logs:

```
DEBUG WholeStageCodegenExec:
[cleanedSource]
```

In the end, `doCodeGen` returns the <<spark-sql-CodegenContext.adoc#, CodegenContext>> and the Java source code (as a `CodeAndComment`).

[NOTE]
====
`doCodeGen` is used when:

* `WholeStageCodegenExec` is <<doExecute, executed>>

* Debugging Query Execution is requested to <<spark-sql-debugging-query-execution.adoc#debugCodegen, display a Java source code generated for a structured query in Whole-Stage Code Generation>>
====

=== [[doConsume]] Generating Java Source Code for Consume Path in Whole-Stage Code Generation -- `doConsume` Method

[source, scala]
----
doConsume(ctx: CodegenContext, input: Seq[ExprCode], row: ExprCode): String
----

NOTE: `doConsume` is part of <<spark-sql-CodegenSupport.adoc#doConsume, CodegenSupport Contract>> to generate the Java source code for <<spark-sql-whole-stage-codegen.adoc#consume-path, consume path>> in Whole-Stage Code Generation.

`doConsume` generates a Java source code that:

. Takes (from the input `row`) the code to evaluate a Catalyst expression on an input `InternalRow`
. Takes (from the input `row`) the term for a value of the result of the evaluation
  a. Adds `.copy()` to the term if <<needCopyResult, needCopyResult>> is turned on
. Wraps the term inside `append()` code block

[source, scala]
----
import org.apache.spark.sql.catalyst.expressions.codegen.CodegenContext
val ctx = new CodegenContext()

import org.apache.spark.sql.catalyst.expressions.codegen.ExprCode
val exprCode = ExprCode(code = "my_code", isNull = "false", value = "my_value")

// wsce defined above, i.e at the top of the page
val consumeCode = wsce.doConsume(ctx, input = Seq(), row = exprCode)
scala> println(consumeCode)
my_code
append(my_value);
----

=== [[generatedClassName]] Generating Class Name -- `generatedClassName` Method

[source, scala]
----
generatedClassName(): String
----

`generatedClassName` gives a class name per link:spark-sql-properties.adoc#spark.sql.codegen.useIdInClassName[spark.sql.codegen.useIdInClassName] configuration property:

* `GeneratedIteratorForCodegenStage` with the <<codegenStageId, codegen stage ID>> when enabled (`true`)

* `GeneratedIterator` when disabled (`false`)

NOTE: `generatedClassName` is used exclusively when `WholeStageCodegenExec` unary physical operator is requested to <<doCodeGen, generate the Java source code for the child physical plan subtree>>.

=== [[isTooManyFields]] `isTooManyFields` Object Method

[source, scala]
----
isTooManyFields(conf: SQLConf, dataType: DataType): Boolean
----

`isTooManyFields`...FIXME

NOTE: `isTooManyFields` is used when...FIXME
