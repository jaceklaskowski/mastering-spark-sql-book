title: CatalystDataToAvro

# CatalystDataToAvro Unary Expression

`CatalystDataToAvro` is a <<spark-sql-Expression-UnaryExpression.adoc#, unary expression>> that represents <<spark-sql-avro.adoc#to_avro, to_avro>> function in a structured query.

[[creating-instance]]
[[child]]
`CatalystDataToAvro` takes a single <<spark-sql-Expression.adoc#, Catalyst expression>> when created.

`CatalystDataToAvro` <<doGenCode, generates Java source code (as ExprCode) for code-generated expression evaluation>>.

```
import org.apache.spark.sql.avro.CatalystDataToAvro
val catalystDataToAvro = CatalystDataToAvro($"id".expr)

import org.apache.spark.sql.catalyst.expressions.codegen.{CodegenContext, ExprCode}
val ctx = new CodegenContext
// doGenCode is used when Expression.genCode is executed
// FIXME The following won't work due to https://issues.apache.org/jira/browse/SPARK-26063
val ExprCode(code, _, _) = catalystDataToAvro.genCode(ctx)

// Helper methods
def trim(code: String): String = {
  code.trim.split("\n").map(_.trim).filter(line => line.nonEmpty).mkString("\n")
}
def prettyPrint(code: String) = println(trim(code))
// END: Helper methods

scala> println(trim(code))
// FIXME: Finish me once https://issues.apache.org/jira/browse/SPARK-26063 is fixed
// See the following example
```

```
// Let's use a workaround to create a CatalystDataToAvro expression
// with the child resolved
val q = spark.range(1).withColumn("to_avro_id", to_avro('id))
import org.apache.spark.sql.avro.CatalystDataToAvro
val analyzedPlan = q.queryExecution.analyzed
val catalystDataToAvro = analyzedPlan.expressions.drop(1).head.children.head.asInstanceOf[CatalystDataToAvro]

import org.apache.spark.sql.catalyst.expressions.codegen.{CodegenContext, ExprCode}
val ctx = new CodegenContext
val ExprCode(code, _, _) = catalystDataToAvro.genCode(ctx)

// Doh! It does not work either
// java.lang.UnsupportedOperationException: Cannot evaluate expression: id#38L

// Let's try something else (more serious)

import org.apache.spark.sql.catalyst.expressions.{BindReferences, Expression}
val boundExprs = analyzedPlan.expressions.map { e =>
  BindReferences.bindReference[Expression](e, analyzedPlan.children.head.output)
}
// That should trigger doGenCode
val codes = ctx.generateExpressions(boundExprs)

// The following corresponds to catalystDataToAvro.genCode(ctx)
val ExprCode(code, _, _) = codes.tail.head

// Helper methods
def trim(code: String): String = {
  code.trim.split("\n").map(_.trim).filter(line => line.nonEmpty).mkString("\n")
}
def prettyPrint(code: String) = println(trim(code))
// END: Helper methods

scala> println(trim(code.toString))
long value_7 = i.getLong(0);
byte[] value_6 = null;
value_6 = (byte[]) ((org.apache.spark.sql.avro.CatalystDataToAvro) references[2] /* this */).nullSafeEval(value_7);
```

=== [[doGenCode]] Generating Java Source Code (ExprCode) For Code-Generated Expression Evaluation -- `doGenCode` Method

[source, scala]
----
doGenCode(ctx: CodegenContext, ev: ExprCode): ExprCode
----

NOTE: `doGenCode` is part of <<spark-sql-Expression.adoc#doGenCode, Expression Contract>> to generate a Java source code (`ExprCode`) for code-generated expression evaluation.

`doGenCode` requests the `CodegenContext` to <<spark-sql-CodegenContext.adoc#addReferenceObj, generate code to reference this CatalystDataToAvro instance>>.

In the end, `doGenCode` <<spark-sql-Expression-UnaryExpression.adoc#defineCodeGen, defineCodeGen>> with the function `f` that uses <<nullSafeEval, nullSafeEval>>.

=== [[nullSafeEval]] `nullSafeEval` Method

[source, scala]
----
nullSafeEval(input: Any): Any
----

NOTE: `nullSafeEval` is part of the <<spark-sql-Expression-UnaryExpression.adoc#nullSafeEval, UnaryExpression Contract>> to...FIXME.

`nullSafeEval`...FIXME
