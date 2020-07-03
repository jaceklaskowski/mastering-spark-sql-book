# AliasViewChild Logical Analysis Rule

`AliasViewChild` is a <<spark-sql-Analyzer.adoc#batches, logical analysis rule>> that <<apply, transforms a logical query plan>> with <<spark-sql-LogicalPlan-View.adoc#, View>> unary logical operators and adds <<spark-sql-LogicalPlan-Project.adoc#, Project>> logical operator (possibly with <<spark-sql-Expression-Alias.adoc#, Alias>> expressions) when the outputs of a view and the underlying table do not match (and therefore require aliasing and projection).

`AliasViewChild` is part of the <<spark-sql-Analyzer.adoc#View, View>> once-executed batch in the standard batches of the <<spark-sql-Analyzer.adoc#, Analyzer>>.

`AliasViewChild` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

[[conf]]
[[creating-instance]]
`AliasViewChild` takes a <<spark-sql-SQLConf.adoc#, SQLConf>> when created.

[source, scala]
----
// Sample view for the demo
// The order of the column names do not match
// In View: name, id
// In VALUES: id, name
sql("""
  CREATE OR REPLACE VIEW v (name COMMENT 'First name only', id COMMENT 'Identifier') COMMENT 'Permanent view'
  AS VALUES (1, 'Jacek'), (2, 'Agata') AS t1(id, name)
  """)

scala> :type spark
org.apache.spark.sql.SparkSession

val q = spark.table("v")

val plan = q.queryExecution.logical
scala> println(plan.numberedTreeString)
00 'UnresolvedRelation `v`

// Resolve UnresolvedRelation first
// since AliasViewChild work with View operators only
import spark.sessionState.analyzer.ResolveRelations
val resolvedPlan = ResolveRelations(plan)
scala> println(resolvedPlan.numberedTreeString)
00 SubqueryAlias v
01 +- View (`default`.`v`, [name#32,id#33])
02    +- SubqueryAlias t1
03       +- LocalRelation [id#34, name#35]

scala> :type spark.sessionState.conf
org.apache.spark.sql.internal.SQLConf

import org.apache.spark.sql.catalyst.analysis.AliasViewChild
val rule = AliasViewChild(spark.sessionState.conf)

// Notice that resolvedPlan is used (not plan)
val planAfterAliasViewChild = rule(resolvedPlan)
scala> println(planAfterAliasViewChild.numberedTreeString)
00 SubqueryAlias v
01 +- View (`default`.`v`, [name#32,id#33])
02    +- Project [cast(id#34 as int) AS name#32, cast(name#35 as string) AS id#33]
03       +- SubqueryAlias t1
04          +- LocalRelation [id#34, name#35]
----

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply`...FIXME
