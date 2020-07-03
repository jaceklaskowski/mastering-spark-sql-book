# PreWriteCheck Extended Analysis Check

`PreWriteCheck` is an *extended analysis check* that verifies correctness of a <<spark-sql-LogicalPlan.adoc#, logical query plan>> with regard to <<InsertIntoTable.adoc#, InsertIntoTable>> unary logical operator (right before analysis can be considered complete).

`PreWriteCheck` is part of the <<spark-sql-Analyzer-CheckAnalysis.adoc#extendedCheckRules, extended analysis check rules>> of the logical <<spark-sql-Analyzer.adoc#, Analyzer>> in <<spark-sql-BaseSessionStateBuilder.adoc#analyzer, BaseSessionStateBuilder>> and link:hive/HiveSessionStateBuilder.adoc#analyzer[HiveSessionStateBuilder].

`PreWriteCheck` is simply a <<apply, function>> of <<spark-sql-LogicalPlan.adoc#, LogicalPlan>> that...FIXME

=== [[apply]] Executing Function -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): Unit
----

NOTE: `apply` is part of Scala's https://www.scala-lang.org/api/2.11.12/index.html#scala.Function1[scala.Function1] contract to create a function of one parameter.

`apply` <<spark-sql-catalyst-TreeNode.adoc#foreach, traverses>> the input <<spark-sql-LogicalPlan.adoc#, logical query plan>> and finds <<InsertIntoTable.adoc#, InsertIntoTable>> unary logical operators.


* [[apply-InsertableRelation]] For an `InsertIntoTable` with a <<spark-sql-LogicalPlan-LogicalRelation.adoc#, LogicalRelation>>...FIXME

* For any `InsertIntoTable`, `apply` throws a `AnalysisException` if the <<InsertIntoTable.adoc#table, logical plan for the table to insert into>> is neither a <<spark-sql-LogicalPlan-LeafNode.adoc#, LeafNode>> nor one of the following leaf logical operators: <<spark-sql-LogicalPlan-Range.adoc#, Range>>, <<spark-sql-LogicalPlan-OneRowRelation.adoc#, OneRowRelation>>, <<spark-sql-LogicalPlan-LocalRelation.adoc#, LocalRelation>>.
+
```
Inserting into an RDD-based table is not allowed.
```
