# ResolveCreateNamedStruct Logical Resolution Rule -- Resolving NamePlaceholders In CreateNamedStruct Expressions

`ResolveCreateNamedStruct` is a <<spark-sql-Analyzer.adoc#batches, logical resolution rule>> that <<apply, replaces NamePlaceholders with Literals for the names in CreateNamedStruct expressions>> in an entire logical query plan.

`ResolveCreateNamedStruct` is part of the <<spark-sql-Analyzer.adoc#Resolution, Resolution>> fixed-point batch in the standard batches of the <<spark-sql-Analyzer.adoc#, Analyzer>>.

`ResolveCreateNamedStruct` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

[source, scala]
----
scala> :type spark
org.apache.spark.sql.SparkSession

val q = spark.range(1).select(struct($"id"))
val logicalPlan = q.queryExecution.logical
scala> println(logicalPlan.numberedTreeString)
00 'Project [unresolvedalias(named_struct(NamePlaceholder, 'id), None)]
01 +- AnalysisBarrier
02       +- Range (0, 1, step=1, splits=Some(8))

// Let's resolve references first
import spark.sessionState.analyzer.ResolveReferences
val planWithRefsResolved = ResolveReferences(logicalPlan)

import org.apache.spark.sql.catalyst.analysis.ResolveCreateNamedStruct
val afterResolveCreateNamedStruct = ResolveCreateNamedStruct(planWithRefsResolved)
scala> println(afterResolveCreateNamedStruct.numberedTreeString)
00 'Project [unresolvedalias(named_struct(id, id#4L), None)]
01 +- AnalysisBarrier
02       +- Range (0, 1, step=1, splits=Some(8))
----

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply` <<spark-sql-catalyst-QueryPlan.adoc#transformAllExpressions, traverses all Catalyst expressions>> (in the input <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>) that are <<spark-sql-Expression-CreateNamedStruct.adoc#, CreateNamedStruct>> expressions which are not <<spark-sql-Expression.adoc#resolved, resolved>> yet and replaces `NamePlaceholders` with <<spark-sql-Expression-Literal.adoc#, Literal>> expressions.

In other words, `apply` finds unresolved <<spark-sql-Expression-CreateNamedStruct.adoc#, CreateNamedStruct>> expressions with `NamePlaceholder` expressions in the <<spark-sql-Expression-CreateNamedStruct.adoc#children, children>> and replaces them with the <<spark-sql-Expression-NamedExpression.adoc#name, name>> of corresponding <<spark-sql-Expression-NamedExpression.adoc#, NamedExpression>>, but only if the `NamedExpression` is resolved.

In the end, `apply` creates a <<spark-sql-Expression-CreateNamedStruct.adoc#creating-instance, CreateNamedStruct>> with new children.
