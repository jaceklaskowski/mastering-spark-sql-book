# ReplaceExpressions Logical Optimization

`ReplaceExpressions` is a <<spark-sql-Optimizer.adoc#batches, base logical optimization>> that <<apply, replaces RuntimeReplaceable expressions with their single child expression>>.

`ReplaceExpressions` is part of the <<spark-sql-Optimizer.adoc#Finish_Analysis, Finish Analysis>> once-executed batch in the standard batches of the <<spark-sql-Optimizer.adoc#, Catalyst Optimizer>>.

`ReplaceExpressions` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

[source, scala]
----
val query = sql("select ifnull(NULL, array('2')) from values 1")
val analyzedPlan = query.queryExecution.analyzed
scala> println(analyzedPlan.numberedTreeString)
00 Project [ifnull(null, array(2)) AS ifnull(NULL, array('2'))#3]
01 +- LocalRelation [col1#2]

import org.apache.spark.sql.catalyst.optimizer.ReplaceExpressions
val optimizedPlan = ReplaceExpressions(analyzedPlan)
scala> println(optimizedPlan.numberedTreeString)
00 Project [coalesce(cast(null as array<string>), cast(array(2) as array<string>)) AS ifnull(NULL, array('2'))#3]
01 +- LocalRelation [col1#2]
----

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply` <<spark-sql-catalyst-QueryPlan.adoc#transformAllExpressions, traverses all Catalyst expressions>> (in the input <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>) and replaces a link:spark-sql-Expression-RuntimeReplaceable.adoc[RuntimeReplaceable] expression into its single child.
