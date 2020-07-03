# EliminateView Logical Optimization

`EliminateView` is a <<spark-sql-Optimizer.adoc#batches, base logical optimization>> that <<apply, removes (eliminates) View logical operators from a logical query plan>>.

`EliminateView` is part of the <<spark-sql-Optimizer.adoc#Finish_Analysis, Finish Analysis>> once-executed batch in the standard batches of the <<spark-sql-Optimizer.adoc#, Catalyst Optimizer>>.

`EliminateView` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

[source, scala]
----
val name = "demo_view"
sql(s"CREATE OR REPLACE VIEW $name COMMENT 'demo view' AS VALUES 1,2")
assert(spark.catalog.tableExists(name))

val q = spark.table(name)

val analyzedPlan = q.queryExecution.analyzed
scala> println(analyzedPlan.numberedTreeString)
00 SubqueryAlias demo_view
01 +- View (`default`.`demo_view`, [col1#37])
02    +- Project [cast(col1#38 as int) AS col1#37]
03       +- LocalRelation [col1#38]

import org.apache.spark.sql.catalyst.analysis.EliminateView
val afterEliminateView = EliminateView(analyzedPlan)
// Notice no View operator
scala> println(afterEliminateView.numberedTreeString)
00 SubqueryAlias demo_view
01 +- Project [cast(col1#38 as int) AS col1#37]
02    +- LocalRelation [col1#38]

// TIP: You may also want to use EliminateSubqueryAliases to eliminate SubqueryAliases
----

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply` simply removes (eliminates) <<spark-sql-LogicalPlan-View.adoc#, View>> unary logical operators from the input <<spark-sql-LogicalPlan.adoc#, logical plan>> and replaces them with their <<spark-sql-LogicalPlan-View.adoc#child, child>> logical operator.

`apply` throws an `AssertionError` when the <<spark-sql-LogicalPlan-View.adoc#output, output schema>> of the `View` operator does not match the <<spark-sql-catalyst-QueryPlan.adoc#output, output schema>> of the <<spark-sql-LogicalPlan-View.adoc#child, child>> logical operator.

```
assertion failed: The output of the child [output] is different from the view output [output]
```

NOTE: The assertion should not really happen since <<spark-sql-Analyzer-AliasViewChild.adoc#, AliasViewChild>> logical analysis rule is executed earlier and takes care of not allowing for such difference in the output schema (by throwing an `AnalysisException` earlier).
