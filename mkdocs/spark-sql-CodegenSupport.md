# CodegenSupport -- Physical Operators with Java Code Generation

`CodegenSupport` is the <<contract, contract>> of <<implementations, physical operators>> that want to support *Java code generation* and participate in the <<spark-sql-whole-stage-codegen.adoc#, Whole-Stage Java Code Generation (Whole-Stage CodeGen)>>.

[[contract]]
[source, scala]
----
package org.apache.spark.sql.execution

trait CodegenSupport extends SparkPlan {
  // only required properties (vals and methods) that have no implementation
  // the others follow
  def doProduce(ctx: CodegenContext): String
  def inputRDDs(): Seq[RDD[InternalRow]]

  // ...except the following that throws an UnsupportedOperationException by default
  def doConsume(ctx: CodegenContext, input: Seq[ExprCode], row: ExprCode): String
}
----

.(Subset of) CodegenSupport Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| `doConsume`
| [[doConsume]] Generating a plain Java source code for link:spark-sql-whole-stage-codegen.adoc#consume-path[whole-stage "consume" path code generation]

Used exclusively when `CodegenSupport` is requested for the Java source code to <<consume, consume>> the generated columns or a row from a physical operator.

| `doProduce`
| [[doProduce]] Generating a plain Java source code (as a text) for the link:spark-sql-whole-stage-codegen.adoc#produce-path["produce" path] in whole-stage Java code generation.

Used exclusively when a physical operator is requested to <<produce, generate the Java source code for produce code path>>, i.e. a Java code that reads the rows from the input RDDs, processes them to produce output rows that are then the input rows to downstream physical operators.

| `inputRDDs`
a| [[inputRDDs]] Input RDDs of a physical operator

NOTE: Up to two input RDDs are supported only.

Used when `WholeStageCodegenExec` unary physical operator is <<spark-sql-SparkPlan-WholeStageCodegenExec.adoc#doExecute, executed>>.
|===

`CodegenSupport` has the <<final-methods, final methods>> that are used to generate the Java source code in different phases of Whole-Stage Java Code Generation.

[[final-methods]]
.SparkPlan's Final Methods
[cols="1,3",options="header",width="100%"]
|===
| Name
| Description

| <<consume, consume>>
a| Code for consuming generated columns or a row from a physical operator

[source, scala]
----
consume(ctx: CodegenContext, outputVars: Seq[ExprCode], row: String = null): String
----

| <<produce, produce>>
a| Code for "produce" code path

[source, scala]
----
produce(ctx: CodegenContext, parent: CodegenSupport): String
----
|===

`CodegenSupport` allows physical operators to <<supportCodegen, disable Java code generation>>.

TIP: Use link:spark-sql-debugging-query-execution.adoc#debugCodegen[debugCodegen] or link:spark-sql-QueryExecution.adoc#debug[QueryExecution.debug.codegen] methods to access the generated Java source code for a structured query.

[[variablePrefix]]
`variablePrefix` is...FIXME

`CodegenSupport` uses a <<parent, parent>> physical operator (with `CodegenSupport`) for...FIXME

[source, scala]
----
val q = spark.range(1)

import org.apache.spark.sql.execution.debug._
scala> q.debugCodegen
Found 1 WholeStageCodegen subtrees.
== Subtree 1 / 1 ==
*Range (0, 1, step=1, splits=8)

Generated code:
...

// The above is equivalent to the following method chain
scala> q.queryExecution.debug.codegen
Found 1 WholeStageCodegen subtrees.
== Subtree 1 / 1 ==
*Range (0, 1, step=1, splits=8)

Generated code:
...
----

[[implementations]]
.CodegenSupports
[cols="1m,2",options="header",width="100%"]
|===
| CodegenSupport
| Description

| <<spark-sql-SparkPlan-BaseLimitExec.adoc#, BaseLimitExec>>
| [[BaseLimitExec]]

| <<spark-sql-SparkPlan-BroadcastHashJoinExec.adoc#, BroadcastHashJoinExec>>
| [[BroadcastHashJoinExec]]

| <<spark-sql-ColumnarBatchScan.adoc#, ColumnarBatchScan>>
| [[ColumnarBatchScan]]

| <<spark-sql-SparkPlan-DataSourceScanExec.adoc#, DataSourceScanExec>>
| [[DataSourceScanExec]]

| <<spark-sql-SparkPlan-DebugExec.adoc#, DebugExec>>
| [[DebugExec]]

| <<spark-sql-SparkPlan-DeserializeToObjectExec.adoc#, DeserializeToObjectExec>>
| [[DeserializeToObjectExec]]

| <<spark-sql-SparkPlan-ExpandExec.adoc#, ExpandExec>>
| [[ExpandExec]]

| <<spark-sql-SparkPlan-FilterExec.adoc#, FilterExec>>
| [[FilterExec]]

| <<spark-sql-SparkPlan-GenerateExec.adoc#, GenerateExec>>
| [[GenerateExec]]

| <<spark-sql-SparkPlan-HashAggregateExec.adoc#, HashAggregateExec>>
| [[HashAggregateExec]]

| <<spark-sql-SparkPlan-InputAdapter.adoc#, InputAdapter>>
| [[InputAdapter]]

| <<spark-sql-SparkPlan-MapElementsExec.adoc#, MapElementsExec>>
| [[MapElementsExec]]

| <<spark-sql-SparkPlan-ProjectExec.adoc#, ProjectExec>>
| [[ProjectExec]]

| <<spark-sql-SparkPlan-RangeExec.adoc#, RangeExec>>
| [[RangeExec]]

| <<spark-sql-SparkPlan-SampleExec.adoc#, SampleExec>>
| [[SampleExec]]

| <<spark-sql-SparkPlan-SerializeFromObjectExec.adoc#, SerializeFromObjectExec>>
| [[SerializeFromObjectExec]]

| <<spark-sql-SparkPlan-SortExec.adoc#, SortExec>>
| [[SortExec]]

| <<spark-sql-SparkPlan-SortMergeJoinExec.adoc#, SortMergeJoinExec>>
| [[SortMergeJoinExec]]

| <<spark-sql-SparkPlan-WholeStageCodegenExec.adoc#, WholeStageCodegenExec>>
| [[WholeStageCodegenExec]]
|===

=== [[supportCodegen]] `supportCodegen` Flag

[source, scala]
----
supportCodegen: Boolean = true
----

`supportCodegen` flag is to select between `InputAdapter` or `WholeStageCodegenExec` physical operators when `CollapseCodegenStages` is link:spark-sql-CollapseCodegenStages.adoc#apply[executed] (and link:spark-sql-CollapseCodegenStages.adoc#supportCodegen[checks whether a physical operator meets the requirements of whole-stage Java code generation or not]).

`supportCodegen` flag is turned on by default.

[NOTE]
====
`supportCodegen` is turned off in the following physical operators:

* link:spark-sql-SparkPlan-GenerateExec.adoc[GenerateExec]
* link:spark-sql-SparkPlan-HashAggregateExec.adoc[HashAggregateExec] with link:spark-sql-Expression-ImperativeAggregate.adoc[ImperativeAggregates]
* link:spark-sql-SparkPlan-SortMergeJoinExec.adoc[SortMergeJoinExec] for all link:spark-sql-joins.adoc#join-types[join types] except `INNER` and `CROSS`
====

=== [[produce]] Generating Java Source Code for Produce Code Path -- `produce` Final Method

[source, scala]
----
produce(ctx: CodegenContext, parent: CodegenSupport): String
----

`produce` generates the Java source code for link:spark-sql-whole-stage-codegen.adoc#produce-path[whole-stage-codegen produce code path] for processing the rows from the <<inputRDDs, input RDDs>>, i.e. a Java code that reads the rows from the input RDDs, processes them to produce output rows that are then the input rows to downstream physical operators.

Internally, `produce` link:spark-sql-SparkPlan.adoc#executeQuery[prepares a physical operator for query execution] and then generates a Java source code with the result of <<doProduce, doProduce>>.

While generating the Java source code, `produce` annotates code blocks with `PRODUCE` markers that are link:spark-sql-catalyst-QueryPlan.adoc#simpleString[simple descriptions] of the physical operators in a structured query.

TIP: Enable `spark.sql.codegen.comments` Spark SQL property to have `PRODUCE` markers in the generated Java source code.

[source, scala]
----
// ./bin/spark-shell --conf spark.sql.codegen.comments=true
import org.apache.spark.sql.execution.debug._
val q = Seq((0 to 4).toList).toDF.
  select(explode('value) as "id").
  join(spark.range(1), "id")
scala> q.debugCodegen
Found 2 WholeStageCodegen subtrees.
== Subtree 1 / 2 ==
*Range (0, 1, step=1, splits=8)
...
/* 080 */   protected void processNext() throws java.io.IOException {
/* 081 */     // PRODUCE: Range (0, 1, step=1, splits=8)
/* 082 */     // initialize Range
/* 083 */     if (!range_initRange) {
...
== Subtree 2 / 2 ==
*Project [id#6]
+- *BroadcastHashJoin [cast(id#6 as bigint)], [id#9L], Inner, BuildRight
   :- Generate explode(value#1), false, false, [id#6]
   :  +- LocalTableScan [value#1]
   +- BroadcastExchange HashedRelationBroadcastMode(List(input[0, bigint, false]))
      +- *Range (0, 1, step=1, splits=8)
...
/* 062 */   protected void processNext() throws java.io.IOException {
/* 063 */     // PRODUCE: Project [id#6]
/* 064 */     // PRODUCE: BroadcastHashJoin [cast(id#6 as bigint)], [id#9L], Inner, BuildRight
/* 065 */     // PRODUCE: InputAdapter
/* 066 */     while (inputadapter_input.hasNext() && !stopEarly()) {
...
----

[NOTE]
====
`produce` is used when:

* (most importantly) `WholeStageCodegenExec` is requested to <<spark-sql-SparkPlan-WholeStageCodegenExec.adoc#doCodeGen, generate the Java source code for a child physical plan subtree>> (i.e. a physical operator and its children)

* A physical operator (with `CodegenSupport`) is requested to <<doProduce, generate a Java source code for the produce path in whole-stage Java code generation>> that usually looks as follows:
+
[source, scala]
----
protected override def doProduce(ctx: CodegenContext): String = {
  child.asInstanceOf[CodegenSupport].produce(ctx, this)
}
----
====

=== [[prepareRowVar]] `prepareRowVar` Internal Method

[source, scala]
----
prepareRowVar(ctx: CodegenContext, row: String, colVars: Seq[ExprCode]): ExprCode
----

`prepareRowVar`...FIXME

NOTE: `prepareRowVar` is used exclusively when `CodegenSupport` is requested to <<consume, consume>> (and <<constructDoConsumeFunction, constructDoConsumeFunction>> with link:spark-sql-properties.adoc#spark.sql.codegen.splitConsumeFuncByOperator[spark.sql.codegen.splitConsumeFuncByOperator] enabled).

=== [[constructDoConsumeFunction]] `constructDoConsumeFunction` Internal Method

[source, scala]
----
constructDoConsumeFunction(
  ctx: CodegenContext,
  inputVars: Seq[ExprCode],
  row: String): String
----

`constructDoConsumeFunction`...FIXME

NOTE: `constructDoConsumeFunction` is used exclusively when `CodegenSupport` is requested to <<consume, consume>>.

=== [[registerComment]] `registerComment` Method

[source, scala]
----
registerComment(text: => String): String
----

`registerComment`...FIXME

NOTE: `registerComment` is used when...FIXME

=== [[metricTerm]] `metricTerm` Method

[source, scala]
----
metricTerm(ctx: CodegenContext, name: String): String
----

`metricTerm`...FIXME

NOTE: `metricTerm` is used when...FIXME

=== [[usedInputs]] `usedInputs` Method

[source, scala]
----
usedInputs: AttributeSet
----

`usedInputs` returns the link:spark-sql-catalyst-QueryPlan.adoc#references[expression references].

NOTE: Physical operators can mark it as empty to defer evaluation of attribute expressions until they are actually used (in the <<spark-sql-CodegenSupport.adoc#consume, generated Java source code for consume path>>).

NOTE: `usedInputs` is used exclusively when `CodegenSupport` is requested to <<consume, generate a Java source code for consume path>>.

=== [[consume]] Generating Java Source Code to Consume Generated Columns or Row From Current Physical Operator -- `consume` Final Method

[source, scala]
----
consume(ctx: CodegenContext, outputVars: Seq[ExprCode], row: String = null): String
----

NOTE: `consume` is a final method that cannot be changed and is the foundation of codegen support.

`consume` creates the `ExprCodes` for the input variables (aka `inputVars`).

* If `outputVars` is defined, `consume` makes sure that their number is exactly the length of the link:spark-sql-catalyst-QueryPlan.adoc#output[output] and copies them. In other words, `inputVars` is exactly `outputVars`.

* If `outputVars` is not defined, `consume` makes sure that `row` is defined. `consume` sets link:spark-sql-CodegenContext.adoc#currentVars[currentVars] of the `CodegenContext` to `null` while link:spark-sql-CodegenContext.adoc#INPUT_ROW[INPUT_ROW] to the `row`. For every attribute in the link:spark-sql-catalyst-QueryPlan.adoc#output[output], `consume` creates a link:spark-sql-Expression-BoundReference.adoc#creating-instance[BoundReference] and requests it to link:spark-sql-Expression.adoc#genCode[generate code for expression evaluation].

`consume` <<prepareRowVar, creates a row variable>>.

`consume` sets the following in the `CodegenContext`:

* link:spark-sql-CodegenContext.adoc#currentVars[currentVars] as the `inputVars`

* link:spark-sql-CodegenContext.adoc#INPUT_ROW[INPUT_ROW] as `null`

* link:spark-sql-CodegenContext.adoc#freshNamePrefix[freshNamePrefix] as the <<variablePrefix, variablePrefix>> of the <<parent, parent CodegenSupport operator>>.

`consume` <<evaluateRequiredVariables, evaluateRequiredVariables>> (with the `output`, `inputVars` and <<usedInputs, usedInputs>> of the <<parent, parent CodegenSupport operator>>) and creates so-called `evaluated`.

`consume` creates a so-called `consumeFunc` by <<constructDoConsumeFunction, constructDoConsumeFunction>> when the following are all met:

. link:spark-sql-properties.adoc#spark.sql.codegen.splitConsumeFuncByOperator[spark.sql.codegen.splitConsumeFuncByOperator] internal configuration property is enabled

. <<usedInputs, usedInputs>> of the <<parent, parent CodegenSupport operator>> contains all link:spark-sql-catalyst-QueryPlan.adoc#output[output attributes]

. `paramLength` is correct (FIXME)

Otherwise, `consume` requests the <<parent, parent CodegenSupport operator>> to <<doConsume, doConsume>>.

In the end, `consume` gives the plain Java source code with the comment `CONSUME: [parent]`:

```
[evaluated]
[consumeFunc]
```

TIP: Enable link:spark-sql-properties.adoc#spark.sql.codegen.comments[spark.sql.codegen.comments] Spark SQL property to have `CONSUME` markers in the generated Java source code.

[source, scala]
----
// ./bin/spark-shell --conf spark.sql.codegen.comments=true
import org.apache.spark.sql.execution.debug._
val q = Seq((0 to 4).toList).toDF.
  select(explode('value) as "id").
  join(spark.range(1), "id")
scala> q.debugCodegen
Found 2 WholeStageCodegen subtrees.
...
== Subtree 2 / 2 ==
*Project [id#6]
+- *BroadcastHashJoin [cast(id#6 as bigint)], [id#9L], Inner, BuildRight
   :- Generate explode(value#1), false, false, [id#6]
   :  +- LocalTableScan [value#1]
   +- BroadcastExchange HashedRelationBroadcastMode(List(input[0, bigint, false]))
      +- *Range (0, 1, step=1, splits=8)
...
/* 066 */     while (inputadapter_input.hasNext() && !stopEarly()) {
/* 067 */       InternalRow inputadapter_row = (InternalRow) inputadapter_input.next();
/* 068 */       // CONSUME: BroadcastHashJoin [cast(id#6 as bigint)], [id#9L], Inner, BuildRight
/* 069 */       // input[0, int, false]
/* 070 */       int inputadapter_value = inputadapter_row.getInt(0);
...
/* 079 */       // find matches from HashedRelation
/* 080 */       UnsafeRow bhj_matched = bhj_isNull ? null: (UnsafeRow)bhj_relation.getValue(bhj_value);
/* 081 */       if (bhj_matched != null) {
/* 082 */         {
/* 083 */           bhj_numOutputRows.add(1);
/* 084 */
/* 085 */           // CONSUME: Project [id#6]
/* 086 */           // CONSUME: WholeStageCodegen
/* 087 */           project_rowWriter.write(0, inputadapter_value);
/* 088 */           append(project_result);
/* 089 */
/* 090 */         }
/* 091 */       }
/* 092 */       if (shouldStop()) return;
...
----

[NOTE]
====
`consume` is used when:

* link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc#doConsume[BroadcastHashJoinExec], `BaseLimitExec`, `DeserializeToObjectExec`, `ExpandExec`, <<spark-sql-SparkPlan-FilterExec.adoc#doConsume, FilterExec>>, link:spark-sql-SparkPlan-GenerateExec.adoc#doConsume[GenerateExec], link:spark-sql-SparkPlan-ProjectExec.adoc#doConsume[ProjectExec], `SampleExec`, `SerializeFromObjectExec`, `MapElementsExec`, `DebugExec` physical operators are requested to generate the Java source code for link:spark-sql-whole-stage-codegen.adoc#consume-path["consume" path] in whole-stage code generation

* link:spark-sql-ColumnarBatchScan.adoc#doProduce[ColumnarBatchScan], link:spark-sql-SparkPlan-HashAggregateExec.adoc#doProduce[HashAggregateExec], link:spark-sql-SparkPlan-InputAdapter.adoc#doProduce[InputAdapter], link:spark-sql-SparkPlan-RowDataSourceScanExec.adoc#doProduce[RowDataSourceScanExec], link:spark-sql-SparkPlan-RangeExec.adoc#doProduce[RangeExec], link:spark-sql-SparkPlan-SortExec.adoc#doProduce[SortExec], link:spark-sql-SparkPlan-SortMergeJoinExec.adoc#doProduce[SortMergeJoinExec] physical operators are requested to generate the Java source code for the link:spark-sql-whole-stage-codegen.adoc#produce-path["produce" path] in whole-stage code generation
====

=== [[parent]] `parent` Internal Variable Property

[source, scala]
----
parent: CodegenSupport
----

`parent` is a <<CodegenSupport, physical operator that supports whole-stage Java code generation>>.

`parent` starts empty, (i.e. defaults to `null` value) and is assigned a physical operator (with `CodegenContext`) only when `CodegenContext` is requested to <<produce, generate a Java source code for produce code path>>. The physical operator is passed in as an input argument for the <<produce, produce>> code path.

NOTE: `parent` is used when...FIXME
