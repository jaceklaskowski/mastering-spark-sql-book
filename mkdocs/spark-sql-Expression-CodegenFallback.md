title: CodegenFallback

# CodegenFallback -- Catalyst Expressions with Fallback Code Generation Mode

`CodegenFallback` is the <<contract, contract>> of <<implementations, Catalyst expressions>> that do not support a Java code generation and want to <<doGenCode, fall back to interpreted mode>> (aka _fallback mode_).

`CodegenFallback` is used when `CollapseCodegenStages` physical optimization is requested to <<spark-sql-CollapseCodegenStages.adoc#apply, execute>> (and <<spark-sql-CollapseCodegenStages.adoc#supportCodegen-Expression, enforce whole-stage codegen requirements for Catalyst expressions>>).

[[contract]]
[source, scala]
----
package org.apache.spark.sql.catalyst.expressions.codegen

trait CodegenFallback extends Expression {
  // No properties (vals and methods) that have no implementation
}
----

[[implementations]]
.(Some Examples of) CodegenFallbacks
[cols="1,2",options="header",width="100%"]
|===
| CodegenFallback
| Description

| `CurrentDate`
| [[CurrentDate]]

| `CurrentTimestamp`
| [[CurrentTimestamp]]

| `Cube`
| [[Cube]]

| <<spark-sql-Expression-JsonToStructs.adoc#, JsonToStructs>>
| [[JsonToStructs]]

| `Rollup`
| [[Rollup]]

| `StructsToJson`
| [[StructsToJson]]
|===

.Example -- CurrentTimestamp Expression with nullable flag disabled
[source, scala]
----
import org.apache.spark.sql.catalyst.expressions.CurrentTimestamp
val currTimestamp = CurrentTimestamp()

import org.apache.spark.sql.catalyst.expressions.codegen.CodegenFallback
assert(currTimestamp.isInstanceOf[CodegenFallback], "CurrentTimestamp should be a CodegenFallback")

assert(currTimestamp.nullable == false, "CurrentTimestamp should not be nullable")

import org.apache.spark.sql.catalyst.expressions.codegen.{CodegenContext, ExprCode}
val ctx = new CodegenContext
// doGenCode is used when Expression.genCode is executed
val ExprCode(code, _, _) = currTimestamp.genCode(ctx)

scala> println(code)

Object obj_0 = ((Expression) references[0]).eval(null);
        long value_0 = (Long) obj_0;
----

=== [[doGenCode]] Generating Java Source Code (ExprCode) For Code-Generated Expression Evaluation -- `doGenCode` Method

[source, scala]
----
doGenCode(ctx: CodegenContext, ev: ExprCode): ExprCode
----

NOTE: `doGenCode` is part of <<spark-sql-Expression.adoc#doGenCode, Expression Contract>> to generate a Java source code (ExprCode) for code-generated expression evaluation.

`doGenCode` requests the input `CodegenContext` to add itself to the <<spark-sql-CodegenContext.adoc#references, references>>.

`doGenCode` <<spark-sql-catalyst-TreeNode.adoc#foreach, walks down the expression tree>> to find <<spark-sql-Expression-Nondeterministic.adoc#, Nondeterministic>> expressions and for every `Nondeterministic` expression does the following:

. Requests the input `CodegenContext` to add it to the <<spark-sql-CodegenContext.adoc#references, references>>

. Requests the input `CodegenContext` to <<spark-sql-CodegenContext.adoc#addPartitionInitializationStatement, addPartitionInitializationStatement>> that is a Java code block as follows:
+
[source, scala]
----
((Nondeterministic) references[[childIndex]])
  .initialize(partitionIndex);
----

In the end, `doGenCode` generates a plain Java source code block that is one of the following code blocks per the <<spark-sql-Expression.adoc#nullable, nullable>> flag. `doGenCode` copies the input `ExprCode` with the code block added (as the `code` property).

.`doGenCode` Code Block for `nullable` flag enabled
[source, scala]
----
[placeHolder]
Object [objectTerm] = ((Expression) references[[idx]]).eval([input]);
boolean [isNull] = [objectTerm] == null;
[javaType] [value] = [defaultValue];
if (![isNull]) {
  [value] = ([boxedType]) [objectTerm];
}
----

.`doGenCode` Code Block for `nullable` flag disabled
[source, scala]
----
[placeHolder]
Object [objectTerm] = ((Expression) references[[idx]]).eval([input]);
[javaType] [value] = ([boxedType]) [objectTerm];
----
