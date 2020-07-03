# ResolveWindowFrame Logical Resolution Rule

`ResolveWindowFrame` is a logical resolution rule that the link:spark-sql-Analyzer.adoc[logical query plan analyzer] uses to <<apply, validate and resolve WindowExpression expressions>> in an entire logical query plan.

Technically, `ResolveWindowFrame` is just a link:spark-sql-catalyst-Rule.adoc[Catalyst rule] for transforming link:spark-sql-LogicalPlan.adoc[logical plans], i.e. `Rule[LogicalPlan]`.

`ResolveWindowFrame` is part of link:spark-sql-Analyzer.adoc#Resolution[Resolution] fixed-point batch of rules.

[[transformations]]
`ResolveWindowFrame` takes a link:spark-sql-LogicalPlan.adoc[logical plan] and does the following:

. Makes sure that the window frame of a `WindowFunction` is unspecified or matches the `SpecifiedWindowFrame` of the link:spark-sql-Expression-WindowSpecDefinition.adoc[WindowSpecDefinition] expression.
+
Reports a `AnalysisException` when the frames do not match:
+
```
Window Frame [f] must match the required frame [frame]
```

. Copies the frame specification of `WindowFunction` to link:spark-sql-Expression-WindowSpecDefinition.adoc[WindowSpecDefinition]

. Creates a new `SpecifiedWindowFrame` for `WindowExpression` with the resolved Catalyst expression and `UnspecifiedFrame`

NOTE: `ResolveWindowFrame` is a Scala object inside link:spark-sql-Analyzer.adoc[Analyzer] class.

[[example]]
[source, scala]
----
import import org.apache.spark.sql.expressions.Window
// cume_dist requires ordered windows
val q = spark.
  range(5).
  withColumn("cume_dist", cume_dist() over Window.orderBy("id"))
import org.apache.spark.sql.catalyst.plans.logical.LogicalPlan
val planBefore: LogicalPlan = q.queryExecution.logical

// Before ResolveWindowFrame
scala> println(planBefore.numberedTreeString)
00 'Project [*, cume_dist() windowspecdefinition('id ASC NULLS FIRST, UnspecifiedFrame) AS cume_dist#39]
01 +- Range (0, 5, step=1, splits=Some(8))

import spark.sessionState.analyzer.ResolveWindowFrame
val planAfter = ResolveWindowFrame.apply(plan)

// After ResolveWindowFrame
scala> println(planAfter.numberedTreeString)
00 'Project [*, cume_dist() windowspecdefinition('id ASC NULLS FIRST, RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cume_dist#31]
01 +- Range (0, 5, step=1, splits=Some(8))
----

=== [[apply]] Applying ResolveWindowFrame to Logical Plan -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of link:spark-sql-catalyst-Rule.adoc#apply[Rule Contract] to apply a rule to a link:spark-sql-LogicalPlan.adoc[logical plan].

`apply`...FIXME
