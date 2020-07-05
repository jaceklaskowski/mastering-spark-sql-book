# ComputeCurrentTime Logical Optimization

`ComputeCurrentTime` is a <<spark-sql-Optimizer.adoc#batches, base logical optimization>> that <<apply, computes the current date and timestamp>>.

`ComputeCurrentTime` is part of the <<spark-sql-Optimizer.adoc#ComputeCurrentTime, Finish Analysis>> once-executed batch in the standard batches of the <<spark-sql-Optimizer.adoc#, Catalyst Optimizer>>.

`ComputeCurrentTime` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

[source, scala]
----
// Query with two current_date's
import org.apache.spark.sql.functions.current_date
val q = spark.range(1).select(current_date() as "d1", current_date() as "d2")
val analyzedPlan = q.queryExecution.analyzed

scala> println(analyzedPlan.numberedTreeString)
00 Project [current_date(Some(Europe/Warsaw)) AS d1#12, current_date(Some(Europe/Warsaw)) AS d2#13]
01 +- Range (0, 1, step=1, splits=Some(8))

import org.apache.spark.sql.catalyst.optimizer.ComputeCurrentTime

val afterComputeCurrentTime = ComputeCurrentTime(analyzedPlan)
scala> println(afterComputeCurrentTime.numberedTreeString)
00 Project [17773 AS d1#12, 17773 AS d2#13]
01 +- Range (0, 1, step=1, splits=Some(8))

// Another query with two current_timestamp's
// Here the millis play a bigger role so it is easier to notice the results
import org.apache.spark.sql.functions.current_timestamp
val q = spark.range(1).select(current_timestamp() as "ts1", current_timestamp() as "ts2")
val analyzedPlan = q.queryExecution.analyzed
val afterComputeCurrentTime = ComputeCurrentTime(analyzedPlan)
scala> println(afterComputeCurrentTime.numberedTreeString)
00 Project [1535629687768000 AS ts1#18, 1535629687768000 AS ts2#19]
01 +- Range (0, 1, step=1, splits=Some(8))
----

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply`...FIXME
