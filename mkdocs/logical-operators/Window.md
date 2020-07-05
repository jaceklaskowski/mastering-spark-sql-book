title: Window

# Window Unary Logical Operator

`Window` is a link:spark-sql-LogicalPlan.adoc#UnaryNode[unary logical operator] that...FIXME

`Window` is <<creating-instance, created>> when:

* `ExtractWindowExpressions` logical resolution rule is <<spark-sql-Analyzer-ExtractWindowExpressions.adoc#apply, executed>>

* `CleanupAliases` logical analysis rule is <<spark-sql-Analyzer-CleanupAliases.adoc#apply, executed>>

[[output]]
When requested for <<spark-sql-catalyst-QueryPlan.adoc#output, output schema attributes>>, `Window` requests the <<child, child>> logical operator for them and adds the <<spark-sql-Expression-NamedExpression.adoc#toAttribute, attributes>> of the <<windowExpressions, window named expressions>>.

NOTE: `Window` logical operator is a subject of pruning unnecessary window expressions in <<spark-sql-Optimizer-ColumnPruning.adoc#, ColumnPruning>> logical optimization and collapsing window operators in <<spark-sql-Optimizer-CollapseWindow.adoc#, CollapseWindow>> logical optimization.

NOTE: `Window` logical operator is resolved to a <<spark-sql-SparkPlan-WindowExec.adoc#, WindowExec>> in <<spark-sql-SparkStrategy-BasicOperators.adoc#Window, BasicOperators>> execution planning strategy.

=== [[catalyst-dsl]] Catalyst DSL -- `window` Operator

[source, scala]
----
window(
  windowExpressions: Seq[NamedExpression],
  partitionSpec: Seq[Expression],
  orderSpec: Seq[SortOrder]): LogicalPlan
----

<<spark-sql-catalyst-dsl.adoc#window, window>> operator in xref:spark-sql-catalyst-dsl.adoc[Catalyst DSL] creates a <<creating-instance, Window>> logical operator, e.g. for testing or Spark SQL internals exploration.

[source, scala]
----
// FIXME: DEMO
----

=== [[creating-instance]] Creating Window Instance

`Window` takes the following when created:

* [[windowExpression]] Window link:spark-sql-Expression-NamedExpression.adoc[named expressions]
* [[partitionSpec]] Window partition specification link:spark-sql-Expression.adoc[expressions]
* [[orderSpec]] Window order specification (as a collection of `SortOrder` expressions)
* [[child]] Child <<spark-sql-LogicalPlan.adoc#, logical operator>>

=== [[windowOutputSet]] Creating AttributeSet with Window Expression Attributes -- `windowOutputSet` Method

[source, scala]
----
windowOutputSet: AttributeSet
----

`windowOutputSet` simply creates a `AttributeSet` with the <<spark-sql-Expression-NamedExpression.adoc#toAttribute, attributes>> of the <<windowExpressions, window named expressions>>.

[NOTE]
====
`windowOutputSet` is used when:

* `ColumnPruning` logical optimization is <<spark-sql-Optimizer-ColumnPruning.adoc#apply, executed>> (on a <<spark-sql-LogicalPlan-Project.adoc#, Project>> operator with a `Window` as the <<spark-sql-LogicalPlan-Project.adoc#child, child operator>>)

* `CollapseWindow` logical optimization is <<spark-sql-Optimizer-CollapseWindow.adoc#apply, executed>> (on a `Window` operator with another `Window` operator as the <<child, child>>)
====
