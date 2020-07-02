title: UnresolvedOrdinal

# UnresolvedOrdinal Unevaluable Leaf Expression

`UnresolvedOrdinal` is a <<spark-sql-Expression.adoc#LeafExpression, leaf expression>> that represents a single integer literal in <<spark-sql-LogicalPlan-Sort.adoc#, Sort>> logical operators (in <<spark-sql-LogicalPlan-Sort.adoc#order, SortOrder>> ordering expressions) and in <<spark-sql-LogicalPlan-Aggregate.adoc#, Aggregate>> logical operators (in <<spark-sql-LogicalPlan-Aggregate.adoc#groupingExpressions, grouping expressions>>) in a logical plan.

`UnresolvedOrdinal` is <<creating-instance, created>> when `SubstituteUnresolvedOrdinals` logical resolution rule is executed.

[source, scala]
----
// Note "order by 1" clause
val sqlText = "select id from VALUES 1, 2, 3 t1(id) order by 1"
val logicalPlan = spark.sql(sqlText).queryExecution.logical
scala> println(logicalPlan.numberedTreeString)
00 'Sort [1 ASC NULLS FIRST], true
01 +- 'Project ['id]
02    +- 'SubqueryAlias t1
03       +- 'UnresolvedInlineTable [id], [List(1), List(2), List(3)]

import org.apache.spark.sql.catalyst.analysis.SubstituteUnresolvedOrdinals
val rule = new SubstituteUnresolvedOrdinals(spark.sessionState.conf)

val logicalPlanWithUnresolvedOrdinals = rule.apply(logicalPlan)
scala> println(logicalPlanWithUnresolvedOrdinals.numberedTreeString)
00 'Sort [unresolvedordinal(1) ASC NULLS FIRST], true
01 +- 'Project ['id]
02    +- 'SubqueryAlias t1
03       +- 'UnresolvedInlineTable [id], [List(1), List(2), List(3)]

import org.apache.spark.sql.catalyst.plans.logical.Sort
val sortOp = logicalPlanWithUnresolvedOrdinals.collect { case s: Sort => s }.head
val sortOrder = sortOp.order.head

import org.apache.spark.sql.catalyst.analysis.UnresolvedOrdinal
val unresolvedOrdinalExpr = sortOrder.child.asInstanceOf[UnresolvedOrdinal]
scala> println(unresolvedOrdinalExpr)
unresolvedordinal(1)
----

[[creating-instance]]
[[ordinal]]
`UnresolvedOrdinal` takes a single `ordinal` integer when created.

`UnresolvedOrdinal` is an <<spark-sql-Expression.adoc#Unevaluable, unevaluable expression>> and cannot be evaluated (i.e. produce a value given an internal row).

NOTE: An <<spark-sql-Expression.adoc#Unevaluable, unevaluable expression>> cannot be evaluated to produce a value (neither in <<spark-sql-Expression.adoc#eval, interpreted>> nor <<spark-sql-Expression.adoc#doGenCode, code-generated>> expression evaluations) and has to be resolved (replaced) to some other expressions or logical operators at <<spark-sql-QueryExecution.adoc#analyzed, analysis>> or <<spark-sql-QueryExecution.adoc#optimizedPlan, optimization>> phases or they fail analysis.

[[resolved]]
`UnresolvedOrdinal` can never be <<spark-sql-Expression.adoc#resolved, resolved>> (and is replaced at <<analysis-phase, analysis phase>>).

[[analysis-phase]]
NOTE: `UnresolvedOrdinal` is resolved when <<spark-sql-Analyzer-ResolveOrdinalInOrderByAndGroupBy.adoc#, ResolveOrdinalInOrderByAndGroupBy>> logical resolution rule is executed.

[[NonSQLExpression]]
`UnresolvedOrdinal` has <<spark-sql-Expression.adoc#NonSQLExpression, no representation in SQL>>.

NOTE: `UnresolvedOrdinal` in GROUP BY ordinal position is not allowed for a select list with a star (`*`).
