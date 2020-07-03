# Debugging Query Execution

`debug` package object contains <<methods, tools>> for *debugging query execution*, i.e. a full analysis of structured queries (as <<spark-sql-Dataset.adoc#, Datasets>>).

[[methods]]
.Debugging Query Execution Tools (debug Methods)
[cols="1,3",options="header",width="100%"]
|===
| Method
| Description

| <<debug, debug>>
a| Debugging a structured query

[source, scala]
----
debug(): Unit
----

| <<debugCodegen, debugCodegen>>
a| Displays the Java source code generated for a structured query in <<spark-sql-whole-stage-codegen.adoc#, whole-stage code generation>> (i.e. the output of each <<spark-sql-SparkPlan-WholeStageCodegenExec.adoc#, WholeStageCodegen subtree>> in a query plan).

[source, scala]
----
debugCodegen(): Unit
----
|===

`debug` package object is in `org.apache.spark.sql.execution.debug` package that you have to import before you can use the <<debug, debug>> and <<debugCodegen, debugCodegen>> methods.

[source, scala]
----
// Import the package object
import org.apache.spark.sql.execution.debug._

// Every Dataset (incl. DataFrame) has now the debug and debugCodegen methods
val q: DataFrame = ...
q.debug
q.debugCodegen
----

TIP: Read up on https://www.scala-lang.org/docu/files/packageobjects/packageobjects.html[Package Objects] in the Scala programming language.

[[DebugQuery]]
[[query]]
Internally, `debug` package object uses `DebugQuery` implicit class that "extends" `Dataset[_]` Scala type with the <<methods, debug methods>>.

[source, scala]
----
implicit class DebugQuery(query: Dataset[_]) {
  def debug(): Unit = ...
  def debugCodegen(): Unit = ...
}
----

TIP: Read up on https://docs.scala-lang.org/overviews/core/implicit-classes.html[Implicit Classes] in the official documentation of the Scala programming language.

=== [[debug]] Debugging Dataset -- `debug` Method

[source, scala]
----
debug(): Unit
----

`debug` requests the <<spark-sql-Dataset.adoc#queryExecution, QueryExecution>> (of the <<query, structured query>>) for the <<spark-sql-QueryExecution.adoc#executedPlan, optimized physical query plan>>.

`debug` <<spark-sql-catalyst-TreeNode.adoc#transform, transforms>> the optimized physical query plan to add a new <<spark-sql-SparkPlan-DebugExec.adoc#, DebugExec>> physical operator for every physical operator.

`debug` requests the query plan to <<spark-sql-SparkPlan.adoc#execute, execute>> and then counts the number of rows in the result. It prints out the following message:

```
Results returned: [count]
```

In the end, `debug` requests every `DebugExec` physical operator (in the query plan) to <<spark-sql-SparkPlan-DebugExec.adoc#dumpStats, dumpStats>>.

[source, scala]
----
val q = spark.range(10).where('id === 4)

scala> :type q
org.apache.spark.sql.Dataset[Long]

// Extend Dataset[Long] with debug and debugCodegen methods
import org.apache.spark.sql.execution.debug._

scala> q.debug
Results returned: 1
== WholeStageCodegen ==
Tuples output: 1
 id LongType: {java.lang.Long}
== Filter (id#0L = 4) ==
Tuples output: 0
 id LongType: {}
== Range (0, 10, step=1, splits=8) ==
Tuples output: 0
 id LongType: {}
----

=== [[debugCodegen]] Displaying Java Source Code Generated for Structured Query in Whole-Stage Code Generation ("Debugging" Codegen) -- `debugCodegen` Method

[source, scala]
----
debugCodegen(): Unit
----

`debugCodegen` requests the <<spark-sql-Dataset.adoc#queryExecution, QueryExecution>> (of the <<query, structured query>>) for the <<spark-sql-QueryExecution.adoc#executedPlan, optimized physical query plan>>.

In the end, `debugCodegen` simply <<codegenString, codegenString>> the query plan and prints it out to the standard output.

[source, scala]
----
import org.apache.spark.sql.execution.debug._

scala> spark.range(10).where('id === 4).debugCodegen
Found 1 WholeStageCodegen subtrees.
== Subtree 1 / 1 ==
*Filter (id#29L = 4)
+- *Range (0, 10, splits=8)

Generated code:
/* 001 */ public Object generate(Object[] references) {
/* 002 */   return new GeneratedIterator(references);
/* 003 */ }
/* 004 */
/* 005 */ final class GeneratedIterator extends org.apache.spark.sql.execution.BufferedRowIterator {
/* 006 */   private Object[] references;
...
----

[NOTE]
====
`debugCodegen` is equivalent to using `debug` interface of the link:spark-sql-Dataset.adoc#queryExecution[QueryExecution].

[source, scala]
----
val q = spark.range(1, 1000).select('id+1+2+3, 'id+4+5+6)
scala> q.queryExecution.debug.codegen
Found 1 WholeStageCodegen subtrees.
== Subtree 1 / 1 ==
*Project [(id#3L + 6) AS (((id + 1) + 2) + 3)#6L, (id#3L + 15) AS (((id + 4) + 5) + 6)#7L]
+- *Range (1, 1000, step=1, splits=8)

Generated code:
/* 001 */ public Object generate(Object[] references) {
/* 002 */   return new GeneratedIterator(references);
/* 003 */ }
/* 004 */
/* 005 */ final class GeneratedIterator extends org.apache.spark.sql.execution.BufferedRowIterator {
...
----
====

=== [[codegenToSeq]] `codegenToSeq` Method

[source, scala]
----
codegenToSeq(): Seq[(String, String)]
----

`codegenToSeq`...FIXME

NOTE: `codegenToSeq` is used when...FIXME

=== [[codegenString]] `codegenString` Method

[source, scala]
----
codegenString(plan: SparkPlan): String
----

`codegenString`...FIXME

NOTE: `codegenString` is used when...FIXME
