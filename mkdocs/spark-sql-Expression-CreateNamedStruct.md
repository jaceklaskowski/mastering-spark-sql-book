title: CreateNamedStruct

# CreateNamedStruct Expression

`CreateNamedStruct` is a <<spark-sql-Expression-CreateNamedStructLike.adoc#, CreateNamedStructLike>> expression that...FIXME

[[prettyName]]
`CreateNamedStruct` uses *named_struct* for the <<spark-sql-Expression.adoc#prettyName, user-facing name>>.

[source, scala]
----
// Using Catalyst DSL
import org.apache.spark.sql.catalyst.dsl.expressions._
val s = namedStruct("*")
scala> println(s)
named_struct(*)
----

`CreateNamedStruct` is registered in <<spark-sql-FunctionRegistry.adoc#expressions, FunctionRegistry>> under the name of `named_struct` SQL function.

[source, scala]
----
import org.apache.spark.sql.catalyst.FunctionIdentifier
val fid = FunctionIdentifier(funcName = "named_struct")
val className = spark.sessionState.functionRegistry.lookupFunction(fid).get.getClassName
scala> println(className)
org.apache.spark.sql.catalyst.expressions.CreateNamedStruct

val q = sql("SELECT named_struct('id', 0)")
// analyzed so the function is resolved already (using FunctionRegistry)
val analyzedPlan = q.queryExecution.analyzed
scala> println(analyzedPlan.numberedTreeString)
00 Project [named_struct(id, 0) AS named_struct(id, 0)#7]
01 +- OneRowRelation

val e = analyzedPlan.expressions.head.children.head
import org.apache.spark.sql.catalyst.expressions.CreateNamedStruct
assert(e.isInstanceOf[CreateNamedStruct])
----

`CreateNamedStruct` is <<creating-instance, created>> when:

* <<spark-sql-ScalaReflection.adoc#serializerFor, ScalaReflection>>, <<spark-sql-RowEncoder.adoc#serializerFor, RowEncoder>> and `JavaTypeInference` are requested for a serializer of a type

* <<spark-sql-Analyzer-TimeWindowing.adoc#apply, TimeWindowing>> and <<spark-sql-Analyzer-ResolveCreateNamedStruct.adoc#apply, ResolveCreateNamedStruct>> logical resolution rules are executed

* `CreateStruct` is requested to <<spark-sql-CreateStruct.adoc#apply, create a CreateNamedStruct expression>>

[[children]]
[[creating-instance]]
`CreateNamedStruct` takes a collection of <<spark-sql-Expression.adoc#, Catalyst expressions>> when created.

`CreateNamedStruct` <<doGenCode, generates Java source code (as ExprCode) for code-generated expression evaluation>>.

[source, scala]
----
// You could also use Seq("*")
import org.apache.spark.sql.functions.lit
val exprs = Seq("a", 1).map(lit).map(_.expr)

import org.apache.spark.sql.catalyst.expressions.CreateNamedStruct
val createNamedStruct = CreateNamedStruct(exprs)

import org.apache.spark.sql.catalyst.expressions.codegen.{CodegenContext, ExprCode}
val ctx = new CodegenContext
// doGenCode is used when Expression.genCode is executed
val ExprCode(code, _, _) = createNamedStruct.genCode(ctx)

// Helper methods
def trim(code: String): String = {
  code.trim.split("\n").map(_.trim).filter(line => line.nonEmpty).mkString("\n")
}
def prettyPrint(code: String) = println(trim(code))
// END: Helper methods

scala> println(trim(code))
Object[] values_0 = new Object[1];
if (false) {
values_0[0] = null;
} else {
values_0[0] = 1;
}
final InternalRow value_0 = new org.apache.spark.sql.catalyst.expressions.GenericInternalRow(values_0);
values_0 = null;
----

[TIP]
====
Use `namedStruct` operator from Catalyst DSL's link:spark-sql-catalyst-dsl.adoc#expressions[expressions] to create a `CreateNamedStruct` expression.

[source, scala]
----
import org.apache.spark.sql.catalyst.dsl.expressions._
val s = namedStruct()
scala> :type s
org.apache.spark.sql.catalyst.expressions.Expression

import org.apache.spark.sql.catalyst.expressions.CreateNamedStruct
assert(s.isInstanceOf[CreateNamedStruct])

val s = namedStruct("*")
scala> println(s)
named_struct(*)
----
====

=== [[doGenCode]] Generating Java Source Code (ExprCode) For Code-Generated Expression Evaluation -- `doGenCode` Method

[source, scala]
----
doGenCode(ctx: CodegenContext, ev: ExprCode): ExprCode
----

NOTE: `doGenCode` is part of <<spark-sql-Expression.adoc#doGenCode, Expression Contract>> to generate a Java source code (ExprCode) for code-generated expression evaluation.

`doGenCode`...FIXME
