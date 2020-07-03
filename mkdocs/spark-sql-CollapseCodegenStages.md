# CollapseCodegenStages Physical Query Optimization -- Collapsing Physical Operators for Whole-Stage Java Code Generation (aka Whole-Stage CodeGen)

`CollapseCodegenStages` is a *physical query optimization* (aka _physical query preparation rule_ or simply _preparation rule_) that <<apply, collapses physical operators and generates a Java source code for their execution>>.

When <<apply, executed>> (with <<spark-sql-whole-stage-codegen.adoc#spark.sql.codegen.wholeStage, whole-stage code generation enabled>>), `CollapseCodegenStages` <<insertWholeStageCodegen, inserts WholeStageCodegenExec or InputAdapter physical operators>> to a physical plan. `CollapseCodegenStages` uses so-called *control gates* before deciding whether a <<spark-sql-SparkPlan.adoc#, physical operator>> supports the <<spark-sql-whole-stage-codegen.adoc#, whole-stage Java code generation>> or not (and what physical operator to insert):

. Factors in physical operators with <<spark-sql-CodegenSupport.adoc#, CodegenSupport>> only

. Enforces the <<supportCodegen, supportCodegen>> custom requirements on a physical operator, i.e.
.. <<spark-sql-CodegenSupport.adoc#supportCodegen, supportCodegen>> flag turned on (`true`)
.. No <<spark-sql-Expression.adoc#, Catalyst expressions>> are <<spark-sql-Expression-CodegenFallback.adoc#, CodegenFallback>>
.. <<spark-sql-catalyst-QueryPlan.adoc#schema, Output schema>> is *neither wide nor deep* and  <<spark-sql-SparkPlan-WholeStageCodegenExec.adoc#isTooManyFields, uses just enough fields (including nested fields)>>
.. <<spark-sql-catalyst-TreeNode.adoc#children, Children>> use output schema that is also <<spark-sql-SparkPlan-WholeStageCodegenExec.adoc#isTooManyFields, neither wide nor deep>>

[NOTE]
====
link:spark-sql-properties.adoc#spark.sql.codegen.maxFields[spark.sql.codegen.maxFields] Spark internal property controls the total number of fields in a schema that is acceptable for whole-stage code generation.

The number is `100` by default.
====

Technically, `CollapseCodegenStages` is just a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-SparkPlan.adoc#, physical query plans>>, i.e. `Rule[SparkPlan]`.

`CollapseCodegenStages` is part of <<spark-sql-QueryExecution.adoc#preparations, preparations>> batch of physical query plan rules and is executed when `QueryExecution` is requested for the <<spark-sql-QueryExecution.adoc#executedPlan, optimized physical query plan>> (i.e. in *executedPlan* phase of a query execution).

[source, scala]
----
val q = spark.range(3).groupBy('id % 2 as "gid").count

// Let's see where and how many "stars" does this query get
scala> q.explain
== Physical Plan ==
*(2) HashAggregate(keys=[(id#0L % 2)#9L], functions=[count(1)])
+- Exchange hashpartitioning((id#0L % 2)#9L, 200)
   +- *(1) HashAggregate(keys=[(id#0L % 2) AS (id#0L % 2)#9L], functions=[partial_count(1)])
      +- *(1) Range (0, 3, step=1, splits=8)

// There are two stage IDs: 1 and 2 (see the round brackets)
// Looks like Exchange physical operator does not support codegen
// Let's walk through the query execution phases and see it ourselves

// sparkPlan phase is just before CollapseCodegenStages physical optimization is applied
val sparkPlan = q.queryExecution.sparkPlan
scala> println(sparkPlan.numberedTreeString)
00 HashAggregate(keys=[(id#0L % 2)#12L], functions=[count(1)], output=[gid#2L, count#5L])
01 +- HashAggregate(keys=[(id#0L % 2) AS (id#0L % 2)#12L], functions=[partial_count(1)], output=[(id#0L % 2)#12L, count#11L])
02    +- Range (0, 3, step=1, splits=8)

// Compare the above with the executedPlan phase
// which happens to be after CollapseCodegenStages physical optimization
scala> println(q.queryExecution.executedPlan.numberedTreeString)
00 *(2) HashAggregate(keys=[(id#0L % 2)#12L], functions=[count(1)], output=[gid#2L, count#5L])
01 +- Exchange hashpartitioning((id#0L % 2)#12L, 200)
02    +- *(1) HashAggregate(keys=[(id#0L % 2) AS (id#0L % 2)#12L], functions=[partial_count(1)], output=[(id#0L % 2)#12L, count#11L])
03       +- *(1) Range (0, 3, step=1, splits=8)

// Let's apply the CollapseCodegenStages rule ourselves
import org.apache.spark.sql.execution.CollapseCodegenStages
val ccsRule = CollapseCodegenStages(spark.sessionState.conf)
scala> val planAfterCCS = ccsRule.apply(sparkPlan)
planAfterCCS: org.apache.spark.sql.execution.SparkPlan =
*(1) HashAggregate(keys=[(id#0L % 2)#12L], functions=[count(1)], output=[gid#2L, count#5L])
+- *(1) HashAggregate(keys=[(id#0L % 2) AS (id#0L % 2)#12L], functions=[partial_count(1)], output=[(id#0L % 2)#12L, count#11L])
   +- *(1) Range (0, 3, step=1, splits=8)

// The number of stage IDs do not match
// Looks like the above misses one or more rules
// EnsureRequirements optimization rule?
// It is indeed executed before CollapseCodegenStages
import org.apache.spark.sql.execution.exchange.EnsureRequirements
val erRule = EnsureRequirements(spark.sessionState.conf)
val planAfterER = erRule.apply(sparkPlan)
scala> println(planAfterER.numberedTreeString)
00 HashAggregate(keys=[(id#0L % 2)#12L], functions=[count(1)], output=[gid#2L, count#5L])
01 +- Exchange hashpartitioning((id#0L % 2)#12L, 200)
02    +- HashAggregate(keys=[(id#0L % 2) AS (id#0L % 2)#12L], functions=[partial_count(1)], output=[(id#0L % 2)#12L, count#11L])
03       +- Range (0, 3, step=1, splits=8)

// Time for CollapseCodegenStages
val planAfterCCS = ccsRule.apply(planAfterER)
scala> println(planAfterCCS.numberedTreeString)
00 *(2) HashAggregate(keys=[(id#0L % 2)#12L], functions=[count(1)], output=[gid#2L, count#5L])
01 +- Exchange hashpartitioning((id#0L % 2)#12L, 200)
02    +- *(1) HashAggregate(keys=[(id#0L % 2) AS (id#0L % 2)#12L], functions=[partial_count(1)], output=[(id#0L % 2)#12L, count#11L])
03       +- *(1) Range (0, 3, step=1, splits=8)

assert(planAfterCCS == q.queryExecution.executedPlan, "Plan after ER and CCS rules should match the executedPlan plan")

// Bingo!
// The result plan matches the executedPlan plan

// HashAggregateExec and Range physical operators support codegen (is a CodegenSupport)
// - HashAggregateExec disables codegen for ImperativeAggregate aggregate functions
// ShuffleExchangeExec does not support codegen (is not a CodegenSupport)

// The top-level physical operator should be WholeStageCodegenExec
import org.apache.spark.sql.execution.WholeStageCodegenExec
val wsce = planAfterCCS.asInstanceOf[WholeStageCodegenExec]

// The single child operator should be HashAggregateExec
import org.apache.spark.sql.execution.aggregate.HashAggregateExec
val hae = wsce.child.asInstanceOf[HashAggregateExec]

// Since ShuffleExchangeExec does not support codegen, the child of HashAggregateExec is InputAdapter
import org.apache.spark.sql.execution.InputAdapter
val ia = hae.child.asInstanceOf[InputAdapter]

// And it's only now when we can get at ShuffleExchangeExec
import org.apache.spark.sql.execution.exchange.ShuffleExchangeExec
val se = ia.child.asInstanceOf[ShuffleExchangeExec]
----

With link:spark-sql-properties.adoc#spark.sql.codegen.wholeStage[spark.sql.codegen.wholeStage] Spark internal property enabled (which is on by default), `CollapseCodegenStages` <<insertWholeStageCodegen, finds physical operators with CodegenSupport>> for which <<supportCodegen, whole-stage codegen requirements hold>> and collapses them together as `WholeStageCodegenExec` physical operator (possibly with link:spark-sql-SparkPlan-InputAdapter.adoc[InputAdapter] in-between for physical operators with no support for Java code generation).

[NOTE]
====
`InputAdapter` link:spark-sql-SparkPlan-InputAdapter.adoc#generateTreeString[shows itself with no star in the output] of link:spark-sql-dataset-operators.adoc#explain[explain] (or link:spark-sql-catalyst-TreeNode.adoc#numberedTreeString[TreeNode.numberedTreeString]).

[source, scala]
----
val q = spark.range(1).groupBy("id").count
scala> q.explain
== Physical Plan ==
*HashAggregate(keys=[id#16L], functions=[count(1)])
+- Exchange hashpartitioning(id#16L, 200)
   +- *HashAggregate(keys=[id#16L], functions=[partial_count(1)])
      +- *Range (0, 1, step=1, splits=8)
----
====

[[conf]]
`CollapseCodegenStages` takes a link:spark-sql-SQLConf.adoc[SQLConf] when created.

[NOTE]
====
You can disable `CollapseCodegenStages` (and so whole-stage Java code generation) by turning link:spark-sql-properties.adoc#spark.sql.codegen.wholeStage[spark.sql.codegen.wholeStage] Spark internal property off.

`spark.sql.codegen.wholeStage` property is enabled by default.

[source, scala]
----
import org.apache.spark.sql.internal.SQLConf.WHOLESTAGE_CODEGEN_ENABLED
scala> spark.conf.get(WHOLESTAGE_CODEGEN_ENABLED)
res0: String = true
----

Use link:spark-sql-SQLConf.adoc#wholeStageEnabled[SQLConf.wholeStageEnabled] method to access the current value.

[source, scala]
----
scala> spark.sessionState.conf.wholeStageEnabled
res1: Boolean = true
----
====

TIP: Import `CollapseCodegenStages` and apply the rule directly to a physical plan to learn how the rule works.

[source, scala]
----
import org.apache.spark.sql.SparkSession
val spark: SparkSession = ...
// Just a structured query with explode Generator expression that supports codegen "partially"
// i.e. explode extends CodegenSupport but codegenSupport flag is off
val q = spark.range(2)
  .filter($"id" === 0)
  .select(explode(lit(Array(0,1,2))) as "exploded")
  .join(spark.range(2))
  .where($"exploded" === $"id")
scala> q.show
+--------+---+
|exploded| id|
+--------+---+
|       0|  0|
|       1|  1|
+--------+---+

// the final physical plan (after CollapseCodegenStages applied and the other optimization rules)
scala> q.explain
== Physical Plan ==
*BroadcastHashJoin [cast(exploded#34 as bigint)], [id#37L], Inner, BuildRight
:- *Filter isnotnull(exploded#34)
:  +- Generate explode([0,1,2]), false, false, [exploded#34]
:     +- *Project
:        +- *Filter (id#29L = 0)
:           +- *Range (0, 2, step=1, splits=8)
+- BroadcastExchange HashedRelationBroadcastMode(List(input[0, bigint, false]))
   +- *Range (0, 2, step=1, splits=8)

// Control when CollapseCodegenStages is applied to a query plan
// Take sparkPlan that is a physical plan before optimizations, incl. CollapseCodegenStages
val plan = q.queryExecution.sparkPlan

// Is wholeStageEnabled enabled?
// It is by default
scala> println(spark.sessionState.conf.wholeStageEnabled)
true

import org.apache.spark.sql.execution.CollapseCodegenStages
val ccs = CollapseCodegenStages(conf = spark.sessionState.conf)

scala> ccs.ruleName
res0: String = org.apache.spark.sql.execution.CollapseCodegenStages

// Before CollapseCodegenStages
scala> println(plan.numberedTreeString)
00 BroadcastHashJoin [cast(exploded#34 as bigint)], [id#37L], Inner, BuildRight
01 :- Filter isnotnull(exploded#34)
02 :  +- Generate explode([0,1,2]), false, false, [exploded#34]
03 :     +- Project
04 :        +- Filter (id#29L = 0)
05 :           +- Range (0, 2, step=1, splits=8)
06 +- Range (0, 2, step=1, splits=8)

// After CollapseCodegenStages
// Note the stars (that WholeStageCodegenExec.generateTreeString gives)
val execPlan = ccs.apply(plan)
scala> println(execPlan.numberedTreeString)
00 *BroadcastHashJoin [cast(exploded#34 as bigint)], [id#37L], Inner, BuildRight
01 :- *Filter isnotnull(exploded#34)
02 :  +- Generate explode([0,1,2]), false, false, [exploded#34]
03 :     +- *Project
04 :        +- *Filter (id#29L = 0)
05 :           +- *Range (0, 2, step=1, splits=8)
06 +- *Range (0, 2, step=1, splits=8)

// The first star is from WholeStageCodegenExec physical operator
import org.apache.spark.sql.execution.WholeStageCodegenExec
val wsc = execPlan(0).asInstanceOf[WholeStageCodegenExec]
scala> println(wsc.numberedTreeString)
00 *BroadcastHashJoin [cast(exploded#34 as bigint)], [id#37L], Inner, BuildRight
01 :- *Filter isnotnull(exploded#34)
02 :  +- Generate explode([0,1,2]), false, false, [exploded#34]
03 :     +- *Project
04 :        +- *Filter (id#29L = 0)
05 :           +- *Range (0, 2, step=1, splits=8)
06 +- *Range (0, 2, step=1, splits=8)

// Let's disable wholeStage codegen
// CollapseCodegenStages becomes a noop
// It is as if we were not applied Spark optimizations to a physical plan
// We're selective as we only disable whole-stage codegen
val newSpark = spark.newSession()
import org.apache.spark.sql.internal.SQLConf.WHOLESTAGE_CODEGEN_ENABLED
newSpark.sessionState.conf.setConf(WHOLESTAGE_CODEGEN_ENABLED, false)
scala> println(newSpark.sessionState.conf.wholeStageEnabled)
false

// Whole-stage codegen is disabled
// So regardless whether you do apply Spark optimizations or not
// Java code generation won't take place
val ccsWholeStageDisabled = CollapseCodegenStages(conf = newSpark.sessionState.conf)
val execPlan = ccsWholeStageDisabled.apply(plan)
// Note no stars in the output
scala> println(execPlan.numberedTreeString)
00 BroadcastHashJoin [cast(exploded#34 as bigint)], [id#37L], Inner, BuildRight
01 :- Filter isnotnull(exploded#34)
02 :  +- Generate explode([0,1,2]), false, false, [exploded#34]
03 :     +- Project
04 :        +- Filter (id#29L = 0)
05 :           +- Range (0, 2, step=1, splits=8)
06 +- Range (0, 2, step=1, splits=8)
----

=== [[apply]] Executing Rule (Inserting WholeStageCodegenExec or InputAdapter into Physical Query Plan for Whole-Stage Java Code Generation) -- `apply` Method

[source, scala]
----
apply(plan: SparkPlan): SparkPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to apply a rule to a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. link:spark-sql-SparkPlan.adoc[physical plan]).

`apply` starts <<insertWholeStageCodegen, inserting WholeStageCodegenExec (with InputAdapter)>> in the input `plan` physical plan only when link:spark-sql-properties.adoc#spark.sql.codegen.wholeStage[spark.sql.codegen.wholeStage] Spark internal property is turned on.

Otherwise, `apply` does nothing at all (i.e. passes the input physical plan through unchanged).

=== [[insertWholeStageCodegen]] Inserting WholeStageCodegenExec Physical Operator For Codegen Stages -- `insertWholeStageCodegen` Internal Method

[source, scala]
----
insertWholeStageCodegen(plan: SparkPlan): SparkPlan
----

Internally, `insertWholeStageCodegen` branches off per <<spark-sql-SparkPlan.adoc#, physical operator>>:

. For physical operators with a single <<spark-sql-catalyst-QueryPlan.adoc#output, output schema attribute>> of type `ObjectType`, `insertWholeStageCodegen` requests the operator for the <<spark-sql-catalyst-TreeNode.adoc#children, child>> physical operators and tries to <<insertWholeStageCodegen, insertWholeStageCodegen>> on them only.

[[insertWholeStageCodegen-CodegenSupport]]
. For physical operators that support <<spark-sql-CodegenSupport.adoc#, Java code generation>> and meets the <<supportCodegen, additional requirements for codegen>>, `insertWholeStageCodegen` <<insertInputAdapter, insertInputAdapter>> (with the operator), requests `WholeStageCodegenId` for the `getNextStageId` and then uses both to return a new <<spark-sql-SparkPlan-WholeStageCodegenExec.adoc#creating-instance, WholeStageCodegenExec>> physical operator.

. For any other physical operators, `insertWholeStageCodegen` requests the operator for the <<spark-sql-catalyst-TreeNode.adoc#children, child>> physical operators and tries to <<insertWholeStageCodegen, insertWholeStageCodegen>> on them only.

[source, scala]
----
// FIXME: DEMO
// Step 1. The top-level physical operator is CodegenSupport with supportCodegen enabled
// Step 2. The top-level operator is CodegenSupport with supportCodegen disabled
// Step 3. The top-level operator is not CodegenSupport
// Step 4. "plan.output.length == 1 && plan.output.head.dataType.isInstanceOf[ObjectType]"
----

[[insertWholeStageCodegen-ObjectType]]
NOTE: `insertWholeStageCodegen` explicitly skips physical operators with a single-attribute <<spark-sql-catalyst-QueryPlan.adoc#output, output schema>> with the type of the attribute being `ObjectType` type.

[NOTE]
====
`insertWholeStageCodegen` is used recursively when `CollapseCodegenStages` is requested for the following:

* <<apply, Executes>> (and walks down a physical plan)

* <<insertInputAdapter, Inserts InputAdapter physical operator>>
====

=== [[insertInputAdapter]] Inserting InputAdapter Unary Physical Operator -- `insertInputAdapter` Internal Method

[source, scala]
----
insertInputAdapter(plan: SparkPlan): SparkPlan
----

`insertInputAdapter` inserts an link:spark-sql-SparkPlan-InputAdapter.adoc[InputAdapter] physical operator in a physical plan.

* For link:spark-sql-SparkPlan-SortMergeJoinExec.adoc[SortMergeJoinExec] (with inner and outer joins) <<insertWholeStageCodegen, inserts an InputAdapter operator>> for both children physical operators individually

* For <<supportCodegen, codegen-unsupported>> operators <<insertWholeStageCodegen, inserts an InputAdapter operator>>

* For other operators (except `SortMergeJoinExec` operator above or for which <<supportCodegen, Java code cannot be generated>>) <<insertWholeStageCodegen, inserts a WholeStageCodegenExec operator>> for every child operator

CAUTION: FIXME Examples for every case + screenshots from web UI

NOTE: `insertInputAdapter` is used exclusively when `CollapseCodegenStages` <<insertWholeStageCodegen, inserts WholeStageCodegenExec physical operator>> and recursively down the physical plan.

=== [[supportCodegen]] Enforcing Whole-Stage CodeGen Requirements For Physical Operators -- `supportCodegen` Internal Predicate

[source, scala]
----
supportCodegen(plan: SparkPlan): Boolean
----

`supportCodegen` is positive (`true`) when the input <<spark-sql-SparkPlan.adoc#, physical operator>> is as follows:

. link:spark-sql-CodegenSupport.adoc[CodegenSupport] and the <<spark-sql-CodegenSupport.adoc#supportCodegen, supportCodegen>> flag is turned on
+
NOTE: link:spark-sql-CodegenSupport.adoc#supportCodegen[supportCodegen] flag is turned on by default.

. No <<supportCodegen-Expression, Catalyst expressions are CodegenFallback (except LeafExpressions)>>

. Output schema is *neither wide not deep*, i.e. <<spark-sql-SparkPlan-WholeStageCodegenExec.adoc#isTooManyFields, uses just enough fields (including nested fields)>>
+
NOTE: link:spark-sql-properties.adoc#spark.sql.codegen.maxFields[spark.sql.codegen.maxFields] Spark internal property defaults to `100`.

. <<spark-sql-catalyst-TreeNode.adoc#children, Children>> also have the output schema that is <<spark-sql-SparkPlan-WholeStageCodegenExec.adoc#isTooManyFields, neither wide nor deep>>

Otherwise, `supportCodegen` is negative (`false`).

[source, scala]
----
import org.apache.spark.sql.SparkSession
val spark: SparkSession = ...
// both where and select operators support codegen
// the plan tree (with the operators and expressions) meets the requirements
// That's why the plan has WholeStageCodegenExec inserted
// That you can see as stars (*) in the output of explain
val q = Seq((1,2,3)).toDF("id", "c0", "c1").where('id === 0).select('c0)
scala> q.explain
== Physical Plan ==
*Project [_2#89 AS c0#93]
+- *Filter (_1#88 = 0)
   +- LocalTableScan [_1#88, _2#89, _3#90]

// CollapseCodegenStages is only used in QueryExecution.executedPlan
// Use sparkPlan then so we avoid CollapseCodegenStages
val plan = q.queryExecution.sparkPlan
import org.apache.spark.sql.execution.ProjectExec
val pe = plan.asInstanceOf[ProjectExec]

scala> pe.supportCodegen
res1: Boolean = true

scala> pe.schema.fields.size
res2: Int = 1

scala> pe.children.map(_.schema).map(_.size).sum
res3: Int = 3
----

[source, scala]
----
import org.apache.spark.sql.SparkSession
val spark: SparkSession = ...
// both where and select support codegen
// let's break the requirement of spark.sql.codegen.maxFields
val newSpark = spark.newSession()
import org.apache.spark.sql.internal.SQLConf.WHOLESTAGE_MAX_NUM_FIELDS
newSpark.sessionState.conf.setConf(WHOLESTAGE_MAX_NUM_FIELDS, 2)

scala> println(newSpark.sessionState.conf.wholeStageMaxNumFields)
2

import newSpark.implicits._
// the same query as above but created in SparkSession with WHOLESTAGE_MAX_NUM_FIELDS as 2
val q = Seq((1,2,3)).toDF("id", "c0", "c1").where('id === 0).select('c0)

// Note that there are no stars in the output of explain
// No WholeStageCodegenExec operator in the plan => whole-stage codegen disabled
scala> q.explain
== Physical Plan ==
Project [_2#122 AS c0#126]
+- Filter (_1#121 = 0)
   +- LocalTableScan [_1#121, _2#122, _3#123]
----

[NOTE]
====
`supportCodegen` is used when `CollapseCodegenStages` does the following:

* <<insertInputAdapter, Inserts InputAdapter physical operator>> for physical plans that do not support whole-stage Java code generation (i.e. `supportCodegen` is turned off).

* <<insertWholeStageCodegen, Inserts WholeStageCodegenExec physical operator>> for physical operators that do support whole-stage Java code generation (i.e. `supportCodegen` is turned on).
====

=== [[supportCodegen-Expression]] Enforcing Whole-Stage CodeGen Requirements For Catalyst Expressions -- `supportCodegen` Internal Predicate

[source, scala]
----
supportCodegen(e: Expression): Boolean
----

`supportCodegen` is positive (`true`) when the input link:spark-sql-Expression.adoc[Catalyst expression] is the following (in the order of verification):

. link:spark-sql-Expression.adoc#LeafExpression[LeafExpression]

. non-<<spark-sql-Expression.adoc#CodegenFallback, CodegenFallback>>

Otherwise, `supportCodegen` is negative (`false`).

NOTE: `supportCodegen` (for <<spark-sql-Expression.adoc#, Catalyst expressions>>) is used exclusively when `CollapseCodegenStages` physical optimization is requested to <<supportCodegen, enforce whole-stage codegen requirements for a physical operator>>.
