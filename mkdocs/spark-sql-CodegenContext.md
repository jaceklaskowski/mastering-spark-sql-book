# CodegenContext

`CodegenContext` is...FIXME

[[creating-instance]]
`CodegenContext` takes no input parameters.

[source, scala]
----
import org.apache.spark.sql.catalyst.expressions.codegen.CodegenContext
val ctx = new CodegenContext
----

`CodegenContext` is <<creating-instance, created>> when:

* `WholeStageCodegenExec` physical operator is requested to link:spark-sql-SparkPlan-WholeStageCodegenExec.adoc#doCodeGen[generate a Java source code for the child operator] (when `WholeStageCodegenExec` is link:spark-sql-SparkPlan-WholeStageCodegenExec.adoc#doExecute[executed])

* `CodeGenerator` is requested for a link:spark-sql-CodeGenerator.adoc#newCodeGenContext[new CodegenContext]

* `GenerateUnsafeRowJoiner` is requested for a `UnsafeRowJoiner`

`CodegenContext` stores expressions that don't support codegen.

.Example of CodegenContext.subexpressionElimination (through CodegenContext.generateExpressions)
[source, scala]
----
import org.apache.spark.sql.catalyst.expressions.codegen.CodegenContext
val ctx = new CodegenContext

// Use Catalyst DSL
import org.apache.spark.sql.catalyst.dsl.expressions._
val expressions = "hello".expr.as("world") :: "hello".expr.as("world") :: Nil

// FIXME Use a real-life query to extract the expressions

// CodegenContext.subexpressionElimination (where the elimination all happens) is a private method
// It is used exclusively in CodegenContext.generateExpressions which is public
// and does the elimination when it is enabled

// Note the doSubexpressionElimination flag is on
// Triggers the subexpressionElimination private method
ctx.generateExpressions(expressions, doSubexpressionElimination = true)

// subexpressionElimination private method uses ctx.equivalentExpressions
val commonExprs = ctx.equivalentExpressions.getAllEquivalentExprs

assert(commonExprs.length > 0, "No common expressions found")
----

[[internal-registries]]
.CodegenContext's Internal Properties (e.g. Registries, Counters and Flags)
[cols="1,2",options="header",width="100%"]
|===
| Name
| Description

| `classFunctions`
| [[classFunctions]] Mutable Scala `Map` with function names, their Java source code and a class name

New entries are added when `CodegenContext` is requested to <<addClass, addClass>> and <<addNewFunctionToClass, addNewFunctionToClass>>

Used when `CodegenContext` is requested to <<declareAddedFunctions, declareAddedFunctions>>

| `equivalentExpressions`
a| [[equivalentExpressions]] link:spark-sql-EquivalentExpressions.adoc[EquivalentExpressions]

Expressions are link:spark-sql-EquivalentExpressions.adoc#addExprTree[added] and then link:spark-sql-EquivalentExpressions.adoc#getAllEquivalentExprs[fetched as equivalent sets] when `CodegenContext` is requested to <<subexpressionElimination, subexpressionElimination>> (for <<generateExpressions, generateExpressions>> with link:spark-sql-subexpression-elimination.adoc#spark.sql.subexpressionElimination.enabled[subexpression elimination] enabled)

| `currentVars`
| [[currentVars]] The list of generated columns as input of current operator

| `INPUT_ROW`
| [[INPUT_ROW]] The variable name of the input row of the current operator

| `placeHolderToComments`
| [[placeHolderToComments]][[getPlaceHolderToComments]]

Placeholders and their comments

Used when...FIXME

| `references`
a| [[references]] References that are used to generate classes in the following code generators:

* link:spark-sql-GenerateMutableProjection.adoc#create[GenerateMutableProjection]

* link:spark-sql-GenerateOrdering.adoc#create[GenerateOrdering]

* link:spark-sql-GeneratePredicate.adoc#create[GeneratePredicate]

* link:spark-sql-GenerateSafeProjection.adoc#create[GenerateSafeProjection]

* link:spark-sql-GenerateUnsafeProjection.adoc#create[GenerateUnsafeProjection]

* link:spark-sql-WholeStageCodegenExec.adoc#doExecute[WholeStageCodegenExec]

* Elements are added when:
** `CodegenContext` is requested to <<addReferenceObj, addReferenceObj>>
** `CodegenFallback` is requested to link:spark-sql-Expression-CodegenFallback.adoc#doGenCode[doGenCode]

| `subExprEliminationExprs`
| [[subExprEliminationExprs]] `SubExprEliminationStates` by link:spark-sql-Expression.adoc[Expression]

Used when...FIXME

| `subexprFunctions`
| [[subexprFunctions]] Names of the functions that...FIXME
|===

=== [[generateExpressions]] Generating Java Source Code For Code-Generated Evaluation of Multiple Expressions (With Optional Subexpression Elimination) -- `generateExpressions` Method

[source, scala]
----
generateExpressions(
  expressions: Seq[Expression],
  doSubexpressionElimination: Boolean = false): Seq[ExprCode]
----

(only with link:spark-sql-subexpression-elimination.adoc#spark.sql.subexpressionElimination.enabled[subexpression elimination] enabled) `generateExpressions` does <<subexpressionElimination, subexpressionElimination>> of the input `expressions`.

In the end, `generateExpressions` requests every expressions to link:spark-sql-Expression.adoc#genCode[generate the Java source code for code-generated (non-interpreted) expression evaluation].

[NOTE]
====
`generateExpressions` is used when:

* `GenerateMutableProjection` is requested to link:spark-sql-GenerateMutableProjection.adoc#create[create a MutableProjection]

* `GenerateUnsafeProjection` is requested to link:spark-sql-GenerateUnsafeProjection.adoc#createCode[create an ExprCode for Catalyst expressions]

* `HashAggregateExec` is requested to link:spark-sql-SparkPlan-HashAggregateExec.adoc#doConsumeWithKeys[generate the Java source code for whole-stage consume path with grouping keys]
====

=== [[addReferenceObj]] `addReferenceObj` Method

[source, scala]
----
addReferenceObj(objName: String, obj: Any, className: String = null): String
----

`addReferenceObj`...FIXME

NOTE: `addReferenceObj` is used when...FIXME

=== [[subexpressionEliminationForWholeStageCodegen]] `subexpressionEliminationForWholeStageCodegen` Method

[source, scala]
----
subexpressionEliminationForWholeStageCodegen(expressions: Seq[Expression]): SubExprCodes
----

`subexpressionEliminationForWholeStageCodegen`...FIXME

NOTE: `subexpressionEliminationForWholeStageCodegen` is used exclusively when `HashAggregateExec` is requested to link:spark-sql-SparkPlan-HashAggregateExec.adoc#doConsume[generate a Java source code for whole-stage consume path] (link:spark-sql-SparkPlan-HashAggregateExec.adoc#doConsumeWithKeys[with grouping keys] or link:spark-sql-SparkPlan-HashAggregateExec.adoc#doConsumeWithoutKeys[not]).

=== [[addNewFunction]] Adding Function to Generated Class -- `addNewFunction` Method

[source, scala]
----
addNewFunction(
  funcName: String,
  funcCode: String,
  inlineToOuterClass: Boolean = false): String
----

`addNewFunction`...FIXME

NOTE: `addNewFunction` is used when...FIXME

=== [[subexpressionElimination]] `subexpressionElimination` Internal Method

[source, scala]
----
subexpressionElimination(expressions: Seq[Expression]): Unit
----

`subexpressionElimination` requests <<equivalentExpressions, EquivalentExpressions>> to link:spark-sql-EquivalentExpressions.adoc#addExprTree[addExprTree] for every expression (in the input `expressions`).

`subexpressionElimination` requests <<equivalentExpressions, EquivalentExpressions>> for the link:spark-sql-EquivalentExpressions.adoc#getAllEquivalentExprs[equivalent sets of expressions] with at least two equivalent expressions (aka _common expressions_).

For every equivalent expression set, `subexpressionElimination` does the following:

. Takes the first expression and requests it to link:spark-sql-Expression.adoc#genCode[generate a Java source code] for the expression tree

. <<addNewFunction, addNewFunction>> and adds it to <<subexprFunctions, subexprFunctions>>

. Creates a `SubExprEliminationState` and adds it with every common expression in the equivalent expression set to <<subExprEliminationExprs, subExprEliminationExprs>>

NOTE: `subexpressionElimination` is used exclusively when `CodegenContext` is requested to <<generateExpressions, generateExpressions>> (with link:spark-sql-subexpression-elimination.adoc#spark.sql.subexpressionElimination.enabled[subexpression elimination] enabled).

=== [[addMutableState]] Adding Mutable State -- `addMutableState` Method

[source, scala]
----
addMutableState(
  javaType: String,
  variableName: String,
  initFunc: String => String = _ => "",
  forceInline: Boolean = false,
  useFreshName: Boolean = true): String
----

`addMutableState`...FIXME

[source, scala]
----
val input = ctx.addMutableState("scala.collection.Iterator", "input", v => s"$v = inputs[0];")
----

NOTE: `addMutableState` is used when...FIXME

=== [[addImmutableStateIfNotExists]] Adding Immutable State (Unless Exists Already) -- `addImmutableStateIfNotExists` Method

[source, scala]
----
addImmutableStateIfNotExists(
  javaType: String,
  variableName: String,
  initFunc: String => String = _ => ""): Unit
----

`addImmutableStateIfNotExists`...FIXME

[source, scala]
----
val ctx: CodegenContext = ???
val partitionMaskTerm = "partitionMask"
ctx.addImmutableStateIfNotExists(ctx.JAVA_LONG, partitionMaskTerm)
----

NOTE: `addImmutableStateIfNotExists` is used when...FIXME

=== [[freshName]] `freshName` Method

[source, scala]
----
freshName(name: String): String
----

`freshName`...FIXME

NOTE: `freshName` is used when...FIXME

=== [[addNewFunctionToClass]] `addNewFunctionToClass` Internal Method

[source, scala]
----
addNewFunctionToClass(
  funcName: String,
  funcCode: String,
  className: String): mutable.Map[String, mutable.Map[String, String]]
----

`addNewFunctionToClass`...FIXME

NOTE: `addNewFunctionToClass` is used when...FIXME

=== [[addClass]] `addClass` Internal Method

[source, scala]
----
addClass(className: String, classInstance: String): Unit
----

`addClass`...FIXME

NOTE: `addClass` is used when...FIXME

=== [[declareAddedFunctions]] `declareAddedFunctions` Method

[source, scala]
----
declareAddedFunctions(): String
----

`declareAddedFunctions`...FIXME

NOTE: `declareAddedFunctions` is used when...FIXME

=== [[declareMutableStates]] `declareMutableStates` Method

[source, scala]
----
declareMutableStates(): String
----

`declareMutableStates`...FIXME

NOTE: `declareMutableStates` is used when...FIXME

=== [[initMutableStates]] `initMutableStates` Method

[source, scala]
----
initMutableStates(): String
----

`initMutableStates`...FIXME

NOTE: `initMutableStates` is used when...FIXME

=== [[initPartition]] `initPartition` Method

[source, scala]
----
initPartition(): String
----

`initPartition`...FIXME

NOTE: `initPartition` is used when...FIXME

=== [[emitExtraCode]] `emitExtraCode` Method

[source, scala]
----
emitExtraCode(): String
----

`emitExtraCode`...FIXME

NOTE: `emitExtraCode` is used when...FIXME

=== [[addPartitionInitializationStatement]] `addPartitionInitializationStatement` Method

[source, scala]
----
addPartitionInitializationStatement(statement: String): Unit
----

`addPartitionInitializationStatement`...FIXME

NOTE: `addPartitionInitializationStatement` is used when...FIXME
