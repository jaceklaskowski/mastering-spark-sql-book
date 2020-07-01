title: AvroDataToCatalyst

# AvroDataToCatalyst Unary Expression

`AvroDataToCatalyst` is a <<spark-sql-Expression-UnaryExpression.adoc#, unary expression>> that represents <<spark-sql-avro.adoc#from_avro, from_avro>> function in a structured query.

[[creating-instance]]
`AvroDataToCatalyst` takes the following when created:

* [[child]] <<spark-sql-Expression.adoc#, Catalyst expression>>
* [[jsonFormatSchema]] JSON-encoded Avro schema

`AvroDataToCatalyst` <<doGenCode, generates Java source code (as ExprCode) for code-generated expression evaluation>>.

=== [[doGenCode]] Generating Java Source Code (ExprCode) For Code-Generated Expression Evaluation -- `doGenCode` Method

[source, scala]
----
doGenCode(ctx: CodegenContext, ev: ExprCode): ExprCode
----

NOTE: `doGenCode` is part of <<spark-sql-Expression.adoc#doGenCode, Expression Contract>> to generate a Java source code (`ExprCode`) for code-generated expression evaluation.

`doGenCode` requests the `CodegenContext` to <<spark-sql-CodegenContext.adoc#addReferenceObj, generate code to reference this AvroDataToCatalyst instance>>.

In the end, `doGenCode` <<spark-sql-Expression-UnaryExpression.adoc#defineCodeGen, defineCodeGen>> with the function `f` that uses <<nullSafeEval, nullSafeEval>>.

=== [[nullSafeEval]] `nullSafeEval` Method

[source, scala]
----
nullSafeEval(input: Any): Any
----

NOTE: `nullSafeEval` is part of the <<spark-sql-Expression-UnaryExpression.adoc#nullSafeEval, UnaryExpression Contract>> to...FIXME.

`nullSafeEval`...FIXME
