title: SortMergeJoinExec

# SortMergeJoinExec Binary Physical Operator for Sort Merge Join

`SortMergeJoinExec` is a link:spark-sql-SparkPlan.adoc#BinaryExecNode[binary physical operator] to <<doExecute, execute>> a *sort merge join*.

`ShuffledHashJoinExec` is <<creating-instance, selected>> to represent a link:spark-sql-LogicalPlan-Join.adoc[Join] logical operator when link:spark-sql-SparkStrategy-JoinSelection.adoc[JoinSelection] execution planning strategy is executed for joins with <<leftKeys, left join keys>> that are <<orderable, orderable>>, i.e. that can be ordered (sorted).

[[orderable]]
[NOTE]
====
A join key is *orderable* when is of one of the following link:spark-sql-DataType.adoc[data types]:

* `NullType`
* link:spark-sql-DataType.adoc#AtomicType[AtomicType] (that represents all the available types except `NullType`, `StructType`, `ArrayType`, `UserDefinedType`, `MapType`, and `ObjectType`)
* `StructType` with orderable fields
* `ArrayType` of orderable type
* `UserDefinedType` of orderable type

Therefore, a join key is *not* orderable when is of the following data type:

* `MapType`
* `ObjectType`
====

[NOTE]
====
link:spark-sql-properties.adoc#spark.sql.join.preferSortMergeJoin[spark.sql.join.preferSortMergeJoin] is an internal configuration property and is enabled by default.

That means that link:spark-sql-SparkStrategy-JoinSelection.adoc[JoinSelection] execution planning strategy (and so Spark Planner) prefers sort merge join over link:spark-sql-SparkPlan-ShuffledHashJoinExec.adoc[shuffled hash join].
====

[[supportCodegen]]
`SortMergeJoinExec` supports link:spark-sql-CodegenSupport.adoc[Java code generation] (aka _codegen_) for inner and cross joins.

[TIP]
====
Enable `DEBUG` logging level for `org.apache.spark.sql.catalyst.planning.ExtractEquiJoinKeys` logger to see the join condition and the left and right join keys.
====

[source, scala]
----
// Disable auto broadcasting so Broadcast Hash Join won't take precedence
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", -1)

val tokens = Seq(
  (0, "playing"),
  (1, "with"),
  (2, "SortMergeJoinExec")
).toDF("id", "token")

// all data types are orderable
scala> tokens.printSchema
root
 |-- id: integer (nullable = false)
 |-- token: string (nullable = true)

// Spark Planner prefers SortMergeJoin over Shuffled Hash Join
scala> println(spark.conf.get("spark.sql.join.preferSortMergeJoin"))
true

val q = tokens.join(tokens, Seq("id"), "inner")
scala> q.explain
== Physical Plan ==
*(3) Project [id#5, token#6, token#10]
+- *(3) SortMergeJoin [id#5], [id#9], Inner
   :- *(1) Sort [id#5 ASC NULLS FIRST], false, 0
   :  +- Exchange hashpartitioning(id#5, 200)
   :     +- LocalTableScan [id#5, token#6]
   +- *(2) Sort [id#9 ASC NULLS FIRST], false, 0
      +- ReusedExchange [id#9, token#10], Exchange hashpartitioning(id#5, 200)
----

[[metrics]]
.SortMergeJoinExec's Performance Metrics
[cols="1,2,2",options="header",width="100%"]
|===
| Key
| Name (in web UI)
| Description

| [[numOutputRows]] `numOutputRows`
| number of output rows
|
|===

.SortMergeJoinExec in web UI (Details for Query)
image::images/spark-sql-SortMergeJoinExec-webui-query-details.png[align="center"]

NOTE: The prefix for variable names for `SortMergeJoinExec` operators in link:spark-sql-CodegenSupport.adoc[CodegenSupport]-generated code is *smj*.

[source, scala]
----
scala> q.queryExecution.debug.codegen
Found 3 WholeStageCodegen subtrees.
== Subtree 1 / 3 ==
*Project [id#5, token#6, token#11]
+- *SortMergeJoin [id#5], [id#10], Inner
   :- *Sort [id#5 ASC NULLS FIRST], false, 0
   :  +- Exchange hashpartitioning(id#5, 200)
   :     +- LocalTableScan [id#5, token#6]
   +- *Sort [id#10 ASC NULLS FIRST], false, 0
      +- ReusedExchange [id#10, token#11], Exchange hashpartitioning(id#5, 200)

Generated code:
/* 001 */ public Object generate(Object[] references) {
/* 002 */   return new GeneratedIterator(references);
/* 003 */ }
/* 004 */
/* 005 */ final class GeneratedIterator extends org.apache.spark.sql.execution.BufferedRowIterator {
/* 006 */   private Object[] references;
/* 007 */   private scala.collection.Iterator[] inputs;
/* 008 */   private scala.collection.Iterator smj_leftInput;
/* 009 */   private scala.collection.Iterator smj_rightInput;
/* 010 */   private InternalRow smj_leftRow;
/* 011 */   private InternalRow smj_rightRow;
/* 012 */   private int smj_value2;
/* 013 */   private org.apache.spark.sql.execution.ExternalAppendOnlyUnsafeRowArray smj_matches;
/* 014 */   private int smj_value3;
/* 015 */   private int smj_value4;
/* 016 */   private UTF8String smj_value5;
/* 017 */   private boolean smj_isNull2;
/* 018 */   private org.apache.spark.sql.execution.metric.SQLMetric smj_numOutputRows;
/* 019 */   private UnsafeRow smj_result;
/* 020 */   private org.apache.spark.sql.catalyst.expressions.codegen.BufferHolder smj_holder;
/* 021 */   private org.apache.spark.sql.catalyst.expressions.codegen.UnsafeRowWriter smj_rowWriter;
...
----

[[output]]
The link:spark-sql-catalyst-QueryPlan.adoc#output[output schema] of a `SortMergeJoinExec` is...FIXME

[[outputPartitioning]]
The link:spark-sql-SparkPlan.adoc#outputPartitioning[outputPartitioning] of a `SortMergeJoinExec` is...FIXME

[[outputOrdering]]
The link:spark-sql-SparkPlan.adoc#outputOrdering[outputOrdering] of a `SortMergeJoinExec` is...FIXME

[[requiredChildDistribution]]
The link:spark-sql-SparkPlan.adoc#requiredChildDistribution[partitioning requirements] of the input of a `SortMergeJoinExec` (aka _child output distributions_) are link:spark-sql-Distribution-HashClusteredDistribution.adoc[HashClusteredDistributions] of <<leftKeys, left>> and <<rightKeys, right>> join keys.

.SortMergeJoinExec's Required Child Output Distributions
[cols="1,1",options="header",width="100%"]
|===
| Left Child
| Right Child

| link:spark-sql-Distribution-HashClusteredDistribution.adoc[HashClusteredDistribution] (per <<leftKeys, left join key expressions>>)
| link:spark-sql-Distribution-HashClusteredDistribution.adoc[HashClusteredDistribution] (per <<rightKeys, right join key expressions>>)
|===

[[requiredChildOrdering]]
The link:spark-sql-SparkPlan.adoc#requiredChildOrdering[ordering requirements] of the input of a `SortMergeJoinExec` (aka _child output ordering_) is...FIXME

NOTE: `SortMergeJoinExec` operator is chosen in link:spark-sql-SparkStrategy-JoinSelection.adoc[JoinSelection] execution planning strategy (after link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc[BroadcastHashJoinExec] and link:spark-sql-SparkPlan-ShuffledHashJoinExec.adoc[ShuffledHashJoinExec] physical join operators have not met the requirements).

=== [[doProduce]] Generating Java Source Code for Produce Path in Whole-Stage Code Generation -- `doProduce` Method

[source, scala]
----
doProduce(ctx: CodegenContext): String
----

NOTE: `doProduce` is part of <<spark-sql-CodegenSupport.adoc#doProduce, CodegenSupport Contract>> to generate the Java source code for <<spark-sql-whole-stage-codegen.adoc#produce-path, produce path>> in Whole-Stage Code Generation.

`doProduce`...FIXME

=== [[doExecute]] Executing Physical Operator (Generating RDD[InternalRow]) -- `doExecute` Method

[source, scala]
----
doExecute(): RDD[InternalRow]
----

NOTE: `doExecute` is part of <<spark-sql-SparkPlan.adoc#doExecute, SparkPlan Contract>> to generate the runtime representation of a structured query as a distributed computation over <<spark-sql-InternalRow.adoc#, internal binary rows>> on Apache Spark (i.e. `RDD[InternalRow]`).

`doExecute`...FIXME

=== [[creating-instance]] Creating SortMergeJoinExec Instance

`SortMergeJoinExec` takes the following when created:

* [[leftKeys]] Left join key link:spark-sql-Expression.adoc[expressions]
* [[rightKeys]] Right join key link:spark-sql-Expression.adoc[expressions]
* [[joinType]] link:spark-sql-joins.adoc#join-types[Join type]
* [[condition]] Optional join condition link:spark-sql-Expression.adoc[expression]
* [[left]] Left link:spark-sql-SparkPlan.adoc[physical operator]
* [[right]] Right link:spark-sql-SparkPlan.adoc[physical operator]
