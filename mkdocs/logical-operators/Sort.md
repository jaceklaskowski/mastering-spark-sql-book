title: Sort

# Sort Unary Logical Operator

`Sort` is a <<spark-sql-LogicalPlan.adoc#UnaryNode, unary logical operator>> that represents the following in a logical plan:

* `ORDER BY`, `SORT BY`, `SORT BY ... DISTRIBUTE BY` and `CLUSTER BY` clauses (when `AstBuilder` is requested to <<spark-sql-AstBuilder.adoc#withQueryResultClauses, parse a query>>)

* <<spark-sql-dataset-operators.adoc#sortWithinPartitions, Dataset.sortWithinPartitions>>, <<spark-sql-dataset-operators.adoc#sort, Dataset.sort>> and <<spark-sql-dataset-operators.adoc#randomSplit, Dataset.randomSplit>> operators

[source, scala]
----
// Using the feature of ordinal literal
val ids = Seq(1,3,2).toDF("id").sort(lit(1))
val logicalPlan = ids.queryExecution.logical
scala> println(logicalPlan.numberedTreeString)
00 Sort [1 ASC NULLS FIRST], true
01 +- AnalysisBarrier
02       +- Project [value#22 AS id#24]
03          +- LocalRelation [value#22]

import org.apache.spark.sql.catalyst.plans.logical.Sort
val sortOp = logicalPlan.collect { case s: Sort => s }.head
scala> println(sortOp.numberedTreeString)
00 Sort [1 ASC NULLS FIRST], true
01 +- AnalysisBarrier
02       +- Project [value#22 AS id#24]
03          +- LocalRelation [value#22]
----

[source, scala]
----
val nums = Seq((0, "zero"), (1, "one")).toDF("id", "name")
// Creates a Sort logical operator:
// - descending sort direction for id column (specified explicitly)
// - name column is wrapped with ascending sort direction
val numsOrdered = nums.sort('id.desc, 'name)
val logicalPlan = numsOrdered.queryExecution.logical
scala> println(logicalPlan.numberedTreeString)
00 'Sort ['id DESC NULLS LAST, 'name ASC NULLS FIRST], true
01 +- Project [_1#11 AS id#14, _2#12 AS name#15]
02    +- LocalRelation [_1#11, _2#12]
----

[[creating-instance]]
`Sort` takes the following when created:

* [[order]] <<spark-sql-Expression-SortOrder.adoc#, SortOrder>> ordering expressions
* [[global]] `global` flag for global (`true`) or partition-only (`false`) sorting
* [[child]] Child <<spark-sql-LogicalPlan.adoc#, logical plan>>

[[output]]
The <<spark-sql-catalyst-QueryPlan.adoc#output, output schema>> of a `Sort` operator is the output of the <<child, child>> logical operator.

[[maxRows]]
The <<spark-sql-LogicalPlan.adoc#maxRows, maxRows>> of a `Sort` operator is the `maxRows` of the <<child, child>> logical operator.

[[catalyst-dsl]]
TIP: Use <<orderBy, orderBy>> or <<sortBy, sortBy>> operators from the <<spark-sql-catalyst-dsl.adoc#, Catalyst DSL>> to create a `Sort` logical operator, e.g. for testing or Spark SQL internals exploration.

NOTE: Sorting is supported for columns of orderable type only (which is enforced at analysis when `CheckAnalysis` is requested to <<spark-sql-Analyzer-CheckAnalysis.adoc#checkAnalysis, checkAnalysis>>).

NOTE: `Sort` logical operator is resolved to <<spark-sql-SparkPlan-SortExec.adoc#, SortExec>> unary physical operator when <<spark-sql-SparkStrategy-BasicOperators.adoc#Sort, BasicOperators>> execution planning strategy is executed.

=== [[orderBy]][[sortBy]] Catalyst DSL -- `orderBy` and `sortBy` Operators

[source, scala]
----
orderBy(sortExprs: SortOrder*): LogicalPlan
sortBy(sortExprs: SortOrder*): LogicalPlan
----

`orderBy` and `sortBy` <<creating-instance, create>> a `Sort` logical operator with the <<global, global>> flag on and off, respectively.

[source, scala]
----
import org.apache.spark.sql.catalyst.dsl.plans._
val t1 = table("t1")

import org.apache.spark.sql.catalyst.dsl.expressions._
val globalSortById = t1.orderBy('id.asc_nullsLast)

// Note true for the global flag
scala> println(globalSortById.numberedTreeString)
00 'Sort ['id ASC NULLS LAST], true
01 +- 'UnresolvedRelation `t1`

// Note false for the global flag
val partitionOnlySortById = t1.sortBy('id.asc_nullsLast)
scala> println(partitionOnlySortById.numberedTreeString)
00 'Sort ['id ASC NULLS LAST], false
01 +- 'UnresolvedRelation `t1`
----
