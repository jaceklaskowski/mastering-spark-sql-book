title: ColumnarBatchScan

# ColumnarBatchScan -- Physical Operators With Vectorized Reader

`ColumnarBatchScan` is an <<contract, extension>> of <<spark-sql-CodegenSupport.adoc#, CodegenSupport contract>> for <<implementations, physical operators>> that <<supportsBatch, support columnar batch scan>> (aka *vectorized reader*).

`ColumnarBatchScan` uses the <<supportsBatch, supportsBatch>> flag that is enabled (i.e. `true`) by default. It is expected that physical operators would override it to support vectorized decoding only when specific conditions are met (i.e. link:spark-sql-SparkPlan-FileSourceScanExec.adoc#supportsBatch[FileSourceScanExec], link:spark-sql-SparkPlan-InMemoryTableScanExec.adoc#supportsBatch[InMemoryTableScanExec] and link:spark-sql-SparkPlan-DataSourceV2ScanExec.adoc#supportsBatch[DataSourceV2ScanExec] physical operators).

[[needsUnsafeRowConversion]]
`ColumnarBatchScan` uses the `needsUnsafeRowConversion` flag to control the name of the variable for an input row while link:spark-sql-CodegenSupport.adoc#consume[generating the Java source code to consume generated columns or row from a physical operator] that is used while <<produceRows, generating the Java source code for producing rows>>. `needsUnsafeRowConversion` flag is enabled (i.e. `true`) by default that gives no name for the row term.

[[metrics]]
.ColumnarBatchScan's Performance Metrics
[cols="1m,2,2",options="header",width="100%"]
|===
| Key
| Name (in web UI)
| Description

| numOutputRows
| number of output rows
| [[numOutputRows]]

| scanTime
| scan time
| [[scanTime]]
|===

[[implementations]]
.ColumnarBatchScans
[cols="1,3",options="header",width="100%"]
|===
| ColumnarBatchScan
| Description

| <<spark-sql-SparkPlan-DataSourceV2ScanExec.adoc#, DataSourceV2ScanExec>>
a| [[DataSourceV2ScanExec]] <<spark-sql-SparkPlan-DataSourceV2ScanExec.adoc#supportsBatch, Supports vectorized decoding>> for <<spark-sql-DataSourceV2ScanExec.adoc#reader, SupportsScanColumnarBatch data readers>> that <<spark-sql-SupportsScanColumnarBatch.adoc#enableBatchRead, can read data in batch>> (default: `true`)

| <<spark-sql-SparkPlan-FileSourceScanExec.adoc#, FileSourceScanExec>>
| [[FileSourceScanExec]] <<spark-sql-SparkPlan-FileSourceScanExec.adoc#supportsBatch, Supports vectorized decoding>> for <<spark-sql-FileFormat.adoc#, FileFormats>> that <<spark-sql-FileFormat.adoc#supportBatch, support returning columnar batches>> (default: `false`)

| <<spark-sql-SparkPlan-InMemoryTableScanExec.adoc#, InMemoryTableScanExec>>
a| [[InMemoryTableScanExec]] <<spark-sql-SparkPlan-InMemoryTableScanExec.adoc#supportsBatch, Supports vectorized decoding>> when all of the following hold:

* <<spark-sql-properties.adoc#spark.sql.inMemoryColumnarStorage.enableVectorizedReader, spark.sql.inMemoryColumnarStorage.enableVectorizedReader>> property is enabled (default: `true`)

* Uses primitive data types only for the output schema

* Number of fields in the output schema is not more than <<spark-sql-properties.adoc#spark.sql.codegen.maxFields, spark.sql.codegen.maxFields>> property (default: `100`)
|===

=== [[genCodeColumnVector]] `genCodeColumnVector` Internal Method

[source, scala]
----
genCodeColumnVector(
  ctx: CodegenContext,
  columnVar: String,
  ordinal: String,
  dataType: DataType,
  nullable: Boolean): ExprCode
----

`genCodeColumnVector`...FIXME

NOTE: `genCodeColumnVector` is used exclusively when `ColumnarBatchScan` is requested to <<produceBatches, produceBatches>>.

=== [[produceBatches]] Generating Java Source Code to Produce Columnar Batches (for Vectorized Reading) -- `produceBatches` Internal Method

[source, scala]
----
produceBatches(ctx: CodegenContext, input: String): String
----

`produceBatches` gives the Java source code to produce batches...FIXME

[source, scala]
----
// Example to show produceBatches to generate a Java source code
// Uses InMemoryTableScanExec as a ColumnarBatchScan

// Create a DataFrame
val ids = spark.range(10)
// Cache it (and trigger the caching since it is lazy)
ids.cache.foreach(_ => ())

import org.apache.spark.sql.execution.columnar.InMemoryTableScanExec
// we need executedPlan with WholeStageCodegenExec physical operator
// this will make sure the code generation starts at the right place
val plan = ids.queryExecution.executedPlan
val scan = plan.collectFirst { case e: InMemoryTableScanExec => e }.get

assert(scan.supportsBatch, "supportsBatch flag should be on to trigger produceBatches")

import org.apache.spark.sql.catalyst.expressions.codegen.CodegenContext
val ctx = new CodegenContext

// produceBatches is private so we have to trigger it from "outside"
// It could be doProduce with supportsBatch flag on but it is protected
// (doProduce will also take care of the extra input `input` parameter)
// let's do this the only one right way
import org.apache.spark.sql.execution.CodegenSupport
val parent = plan.p(0).asInstanceOf[CodegenSupport]
val produceCode = scan.produce(ctx, parent)

scala> println(produceCode)



if (inmemorytablescan_mutableStateArray1[1] == null) {
  inmemorytablescan_nextBatch1();
}
while (inmemorytablescan_mutableStateArray1[1] != null) {
  int inmemorytablescan_numRows1 = inmemorytablescan_mutableStateArray1[1].numRows();
  int inmemorytablescan_localEnd1 = inmemorytablescan_numRows1 - inmemorytablescan_batchIdx1;
  for (int inmemorytablescan_localIdx1 = 0; inmemorytablescan_localIdx1 < inmemorytablescan_localEnd1; inmemorytablescan_localIdx1++) {
    int inmemorytablescan_rowIdx1 = inmemorytablescan_batchIdx1 + inmemorytablescan_localIdx1;
    long inmemorytablescan_value2 = inmemorytablescan_mutableStateArray2[1].getLong(inmemorytablescan_rowIdx1);
inmemorytablescan_mutableStateArray5[1].write(0, inmemorytablescan_value2);
append(inmemorytablescan_mutableStateArray3[1]);
    if (shouldStop()) { inmemorytablescan_batchIdx1 = inmemorytablescan_rowIdx1 + 1; return; }
  }
  inmemorytablescan_batchIdx1 = inmemorytablescan_numRows1;
  inmemorytablescan_mutableStateArray1[1] = null;
  inmemorytablescan_nextBatch1();
}
((org.apache.spark.sql.execution.metric.SQLMetric) references[3] /* scanTime */).add(inmemorytablescan_scanTime1 / (1000 * 1000));
inmemorytablescan_scanTime1 = 0;

// the code does not look good and begs for some polishing
// (You can only imagine how the Polish me looks when I say "polishing" :))

import org.apache.spark.sql.execution.WholeStageCodegenExec
val wsce = plan.asInstanceOf[WholeStageCodegenExec]

// Trigger code generation of the entire query plan tree
val (ctx, code) = wsce.doCodeGen

// CodeFormatter can pretty-print the code
import org.apache.spark.sql.catalyst.expressions.codegen.CodeFormatter
println(CodeFormatter.format(code))
----

NOTE: `produceBatches` is used exclusively when `ColumnarBatchScan` is requested to <<doProduce, generate the Java source code for produce path in whole-stage code generation>> (when <<supportsBatch, supportsBatch>> flag is on).

=== [[supportsBatch]] `supportsBatch` Method

[source, scala]
----
supportsBatch: Boolean = true
----

`supportsBatch` flag controls whether a link:spark-sql-FileFormat.adoc[FileFormat] supports link:spark-sql-vectorized-parquet-reader.adoc[vectorized decoding] or not. `supportsBatch` is enabled (i.e. `true`) by default.

[NOTE]
====
`supportsBatch` is used when:

* `ColumnarBatchScan` is requested to <<doProduce, generate the Java source code for produce path in whole-stage code generation>>

* `FileSourceScanExec` physical operator is requested for link:spark-sql-SparkPlan-FileSourceScanExec.adoc#metadata[metadata] (for *Batched* metadata) and to link:spark-sql-SparkPlan-FileSourceScanExec.adoc#doExecute[execute]

* `InMemoryTableScanExec` physical operator is requested for link:spark-sql-SparkPlan-InMemoryTableScanExec.adoc#supportCodegen[supportCodegen] flag, link:spark-sql-SparkPlan-InMemoryTableScanExec.adoc#inputRDD[input RDD] and to link:spark-sql-SparkPlan-InMemoryTableScanExec.adoc#doExecute[execute]

* `DataSourceV2ScanExec` physical operator is requested to link:spark-sql-SparkPlan-DataSourceV2ScanExec.adoc#doExecute[execute]
====

=== [[doProduce]] Generating Java Source Code for Produce Path in Whole-Stage Code Generation -- `doProduce` Method

[source, scala]
----
doProduce(ctx: CodegenContext): String
----

NOTE: `doProduce` is part of <<spark-sql-CodegenSupport.adoc#doProduce, CodegenSupport Contract>> to generate the Java source code for <<spark-sql-whole-stage-codegen.adoc#produce-path, produce path>> in Whole-Stage Code Generation.

`doProduce` firstly requests the input `CodegenContext` to link:spark-sql-CodegenContext.adoc#addMutableState[add a mutable state] for the first input RDD of a <<implementations, physical operator>>.

`doProduce` <<produceBatches, produceBatches>> when <<supportsBatch, supportsBatch>> is enabled or <<produceRows, produceRows>>.

NOTE: <<supportsBatch, supportsBatch>> is enabled by default unless overriden by a physical operator.

[source, scala]
----
// Example 1: ColumnarBatchScan with supportsBatch enabled
// Let's create a query with a InMemoryTableScanExec physical operator that supports batch decoding
// InMemoryTableScanExec is a ColumnarBatchScan
val q = spark.range(4).cache
val plan = q.queryExecution.executedPlan

import org.apache.spark.sql.execution.columnar.InMemoryTableScanExec
val inmemoryScan = plan.collectFirst { case exec: InMemoryTableScanExec => exec }.get

assert(inmemoryScan.supportsBatch)

import org.apache.spark.sql.catalyst.expressions.codegen.CodegenContext
val ctx = new CodegenContext
import org.apache.spark.sql.execution.CodegenSupport
val parent = plan.asInstanceOf[CodegenSupport]
val code = inmemoryScan.produce(ctx, parent)
scala> println(code)



if (inmemorytablescan_mutableStateArray1[1] == null) {
  inmemorytablescan_nextBatch1();
}
while (inmemorytablescan_mutableStateArray1[1] != null) {
  int inmemorytablescan_numRows1 = inmemorytablescan_mutableStateArray1[1].numRows();
  int inmemorytablescan_localEnd1 = inmemorytablescan_numRows1 - inmemorytablescan_batchIdx1;
  for (int inmemorytablescan_localIdx1 = 0; inmemorytablescan_localIdx1 < inmemorytablescan_localEnd1; inmemorytablescan_localIdx1++) {
    int inmemorytablescan_rowIdx1 = inmemorytablescan_batchIdx1 + inmemorytablescan_localIdx1;
    long inmemorytablescan_value2 = inmemorytablescan_mutableStateArray2[1].getLong(inmemorytablescan_rowIdx1);
inmemorytablescan_mutableStateArray5[1].write(0, inmemorytablescan_value2);
append(inmemorytablescan_mutableStateArray3[1]);
    if (shouldStop()) { inmemorytablescan_batchIdx1 = inmemorytablescan_rowIdx1 + 1; return; }
  }
  inmemorytablescan_batchIdx1 = inmemorytablescan_numRows1;
  inmemorytablescan_mutableStateArray1[1] = null;
  inmemorytablescan_nextBatch1();
}
((org.apache.spark.sql.execution.metric.SQLMetric) references[3] /* scanTime */).add(inmemorytablescan_scanTime1 / (1000 * 1000));
inmemorytablescan_scanTime1 = 0;

// Example 2: ColumnarBatchScan with supportsBatch disabled

val q = Seq(Seq(1,2,3)).toDF("ids").cache
val plan = q.queryExecution.executedPlan

import org.apache.spark.sql.execution.columnar.InMemoryTableScanExec
val inmemoryScan = plan.collectFirst { case exec: InMemoryTableScanExec => exec }.get

assert(inmemoryScan.supportsBatch == false)

// NOTE: The following codegen won't work since supportsBatch is off and so is codegen
import org.apache.spark.sql.catalyst.expressions.codegen.CodegenContext
val ctx = new CodegenContext
import org.apache.spark.sql.execution.CodegenSupport
val parent = plan.asInstanceOf[CodegenSupport]
scala> val code = inmemoryScan.produce(ctx, parent)
java.lang.UnsupportedOperationException
  at org.apache.spark.sql.execution.CodegenSupport$class.doConsume(WholeStageCodegenExec.scala:315)
  at org.apache.spark.sql.execution.columnar.InMemoryTableScanExec.doConsume(InMemoryTableScanExec.scala:33)
  at org.apache.spark.sql.execution.CodegenSupport$class.constructDoConsumeFunction(WholeStageCodegenExec.scala:208)
  at org.apache.spark.sql.execution.CodegenSupport$class.consume(WholeStageCodegenExec.scala:179)
  at org.apache.spark.sql.execution.columnar.InMemoryTableScanExec.consume(InMemoryTableScanExec.scala:33)
  at org.apache.spark.sql.execution.ColumnarBatchScan$class.produceRows(ColumnarBatchScan.scala:166)
  at org.apache.spark.sql.execution.ColumnarBatchScan$class.doProduce(ColumnarBatchScan.scala:80)
  at org.apache.spark.sql.execution.columnar.InMemoryTableScanExec.doProduce(InMemoryTableScanExec.scala:33)
  at org.apache.spark.sql.execution.CodegenSupport$$anonfun$produce$1.apply(WholeStageCodegenExec.scala:88)
  at org.apache.spark.sql.execution.CodegenSupport$$anonfun$produce$1.apply(WholeStageCodegenExec.scala:83)
  at org.apache.spark.sql.execution.SparkPlan$$anonfun$executeQuery$1.apply(SparkPlan.scala:155)
  at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:151)
  at org.apache.spark.sql.execution.SparkPlan.executeQuery(SparkPlan.scala:152)
  at org.apache.spark.sql.execution.CodegenSupport$class.produce(WholeStageCodegenExec.scala:83)
  at org.apache.spark.sql.execution.columnar.InMemoryTableScanExec.produce(InMemoryTableScanExec.scala:33)
  ... 49 elided
----

=== [[produceRows]] Generating Java Source Code for Producing Rows -- `produceRows` Internal Method

[source, scala]
----
produceRows(ctx: CodegenContext, input: String): String
----

`produceRows` creates a new link:spark-sql-CodegenSupport.adoc#metricTerm[metric term] for the <<numOutputRows, numOutputRows>> metric.

`produceRows` creates a link:spark-sql-CodegenContext.adoc#freshName[fresh term name] for a `row` variable and assigns it as the name of the link:spark-sql-CodegenContext.adoc#INPUT_ROW[INPUT_ROW].

`produceRows` resets (`nulls`) link:spark-sql-CodegenContext.adoc#currentVars[currentVars].

For every link:spark-sql-catalyst-QueryPlan.adoc#output[output schema attribute], `produceRows` creates a link:spark-sql-Expression-BoundReference.adoc#creating-instance[BoundReference] and requests it to link:spark-sql-Expression.adoc#genCode[generate code for expression evaluation].

`produceRows` selects the name of the row term per <<needsUnsafeRowConversion, needsUnsafeRowConversion>> flag.

`produceRows` link:spark-sql-CodegenSupport.adoc#consume[generates the Java source code to consume generated columns or row from the current physical operator] and uses it to generate the final Java source code for producing rows.

[source, scala]
----
// Demo: ColumnarBatchScan.produceRows in Action
// 1. FileSourceScanExec as a ColumnarBatchScan
val q = spark.read.text("README.md")

val plan = q.queryExecution.executedPlan
import org.apache.spark.sql.execution.FileSourceScanExec
val scan = plan.collectFirst { case exec: FileSourceScanExec => exec }.get

// 2. supportsBatch is off
assert(scan.supportsBatch == false)

// 3. InMemoryTableScanExec.produce
import org.apache.spark.sql.catalyst.expressions.codegen.CodegenContext
val ctx = new CodegenContext
import org.apache.spark.sql.execution.CodegenSupport

import org.apache.spark.sql.execution.WholeStageCodegenExec
val wsce = plan.collectFirst { case exec: WholeStageCodegenExec => exec }.get

val code = scan.produce(ctx, parent = wsce)
scala> println(code)
// blank lines removed
while (scan_mutableStateArray[2].hasNext()) {
  InternalRow scan_row2 = (InternalRow) scan_mutableStateArray[2].next();
  ((org.apache.spark.sql.execution.metric.SQLMetric) references[2] /* numOutputRows */).add(1);
  append(scan_row2);
  if (shouldStop()) return;
}
----

NOTE: `produceRows` is used exclusively when `ColumnarBatchScan` is requested to <<doProduce, generate the Java source code for produce path in whole-stage code generation>> (when <<supportsBatch, supportsBatch>> flag is off).

=== [[vectorTypes]] Fully-Qualified Class Names (Types) of Concrete ColumnVectors -- `vectorTypes` Method

[source, scala]
----
vectorTypes: Option[Seq[String]] = None
----

`vectorTypes` defines the fully-qualified class names (_types_) of the concrete <<spark-sql-ColumnVector.adoc#, ColumnVectors>> for every column used in a columnar batch.

`vectorTypes` gives no vector types by default (`None`).

NOTE: `vectorTypes` is used exclusively when `ColumnarBatchScan` is requested to <<produceBatches, produceBatches>>.
