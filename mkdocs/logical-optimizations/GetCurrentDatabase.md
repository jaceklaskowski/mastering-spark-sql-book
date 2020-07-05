# GetCurrentDatabase Logical Optimization

`GetCurrentDatabase` is a <<spark-sql-Optimizer.adoc#batches, base logical optimization>> that <<apply, gives the current database>> for `current_database` SQL function.

`GetCurrentDatabase` is part of the <<spark-sql-Optimizer.adoc#GetCurrentDatabase, Finish Analysis>> once-executed batch in the standard batches of the <<spark-sql-Optimizer.adoc#, Catalyst Optimizer>>.

`GetCurrentDatabase` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

[source, scala]
----
val q = sql("SELECT current_database() AS db")
val analyzedPlan = q.queryExecution.analyzed

scala> println(analyzedPlan.numberedTreeString)
00 Project [current_database() AS db#22]
01 +- OneRowRelation

import org.apache.spark.sql.catalyst.optimizer.GetCurrentDatabase

val afterGetCurrentDatabase = GetCurrentDatabase(spark.sessionState.catalog)(analyzedPlan)
scala> println(afterGetCurrentDatabase.numberedTreeString)
00 Project [default AS db#22]
01 +- OneRowRelation
----

[NOTE]
====
`GetCurrentDatabase` corresponds to SQL's `current_database()` function.

You can access the current database in Scala using

```
scala> val database = spark.catalog.currentDatabase
database: String = default
```
====

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply`...FIXME
