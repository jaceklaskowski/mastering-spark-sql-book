# UnaryExpression

`UnaryExpression` is...FIXME

=== [[defineCodeGen]] `defineCodeGen` Method

[source, scala]
----
defineCodeGen(
  ctx: CodegenContext,
  ev: ExprCode,
  f: String => String): ExprCode
----

`defineCodeGen`...FIXME

NOTE: `defineCodeGen` is used when...FIXME

=== [[nullSafeEval]] `nullSafeEval` Method

[source, scala]
----
nullSafeEval(input: Any): Any
----

`nullSafeEval` simply fails with the following error (and is expected to be overrided to save null-check code):

```
UnaryExpressions must override either eval or nullSafeEval
```

NOTE: `nullSafeEval` is used exclusively when `UnaryExpression` is requested to <<eval, eval>>.

=== [[eval]] Evaluating Expression -- `eval` Method

[source, scala]
----
eval(input: InternalRow): Any
----

NOTE: `eval` is part of <<spark-sql-Expression.adoc#eval, Expression Contract>> for the *interpreted (non-code-generated) expression evaluation*, i.e. evaluating a Catalyst expression to a JVM object for a given <<spark-sql-InternalRow.adoc#, internal binary row>>.

`eval`...FIXME
