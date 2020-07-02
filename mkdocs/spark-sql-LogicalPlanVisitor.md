title: LogicalPlanVisitor

# LogicalPlanVisitor -- Contract for Computing Statistic Estimates and Query Hints of Logical Plan

`LogicalPlanVisitor` is the <<contract, contract>> that uses the <<visit, visitor design pattern>> to scan a logical query plan and compute link:spark-sql-Statistics.adoc[estimates of plan statistics and query hints].

TIP: Read about the *visitor design pattern* in https://en.wikipedia.org/wiki/Visitor_pattern[Wikipedia].

[[visit]]
`LogicalPlanVisitor` defines `visit` method that dispatches computing the statistics of a logical plan to the <<handlers, corresponding handler methods>>.

[source, scala]
----
visit(p: LogicalPlan): T
----

NOTE: `T` stands for the type of a result to be computed (while visiting the query plan tree) and is currently always link:spark-sql-Statistics.adoc[Statistics] only.

The <<implementations, concrete>> `LogicalPlanVisitor` is chosen per link:spark-sql-cost-based-optimization.adoc#spark.sql.cbo.enabled[spark.sql.cbo.enabled] configuration property. When turned on (i.e. `true`), `LogicalPlanStats` link:spark-sql-LogicalPlanStats.adoc#stats[uses] <<BasicStatsPlanVisitor, BasicStatsPlanVisitor>> while <<SizeInBytesOnlyStatsPlanVisitor, SizeInBytesOnlyStatsPlanVisitor>> otherwise.

NOTE: link:spark-sql-properties.adoc#spark.sql.cbo.enabled[spark.sql.cbo.enabled] configuration property is off, i.e. `false` by default.

[[implementations]]
.LogicalPlanVisitors
[cols="1,2",options="header",width="100%"]
|===
| LogicalPlanVisitor
| Description

| [[BasicStatsPlanVisitor]] link:spark-sql-BasicStatsPlanVisitor.adoc[BasicStatsPlanVisitor]
|

| [[SizeInBytesOnlyStatsPlanVisitor]] link:spark-sql-SizeInBytesOnlyStatsPlanVisitor.adoc[SizeInBytesOnlyStatsPlanVisitor]
|
|===

[[contract]]
[[handlers]]
.LogicalPlanVisitor's Logical Operators and Their Handlers
[cols="1,2",options="header",width="100%"]
|===
| Logical Operator
| Handler

| [[Aggregate]] link:spark-sql-LogicalPlan-Aggregate.adoc[Aggregate]
| [[visitAggregate]] `visitAggregate`

| [[Distinct]] `Distinct`
| `visitDistinct`

| [[Except]] `Except`
| `visitExcept`

| [[Expand]] link:spark-sql-LogicalPlan-Expand.adoc[Expand]
| `visitExpand`

| [[Filter]] `Filter`
| [[visitFilter]] `visitFilter`

| [[Generate]] link:spark-sql-LogicalPlan-Generate.adoc[Generate]
| `visitGenerate`

| [[GlobalLimit]] `GlobalLimit`
| `visitGlobalLimit`

| [[Intersect]] `Intersect`
| [[visitIntersect]] `visitIntersect`

| [[Join]] link:spark-sql-LogicalPlan-Join.adoc[Join]
| [[visitJoin]] `visitJoin`

| [[LocalLimit]] `LocalLimit`
| `visitLocalLimit`

| [[Pivot]] link:spark-sql-LogicalPlan-Pivot.adoc[Pivot]
| `visitPivot`

| [[Project]] link:spark-sql-LogicalPlan-Project.adoc[Project]
| [[visitProject]] `visitProject`

| [[Repartition]] link:spark-sql-LogicalPlan-Repartition-RepartitionByExpression.adoc[Repartition]
| `visitRepartition`

| [[RepartitionByExpression]] link:spark-sql-LogicalPlan-Repartition-RepartitionByExpression.adoc[RepartitionByExpression]
| `visitRepartitionByExpr`

| [[ResolvedHint]] link:spark-sql-LogicalPlan-ResolvedHint.adoc[ResolvedHint]
| `visitHint`

| [[Sample]] `Sample`
| `visitSample`

| [[ScriptTransformation]] `ScriptTransformation`
| `visitScriptTransform`

| [[Union]] `Union`
| `visitUnion`

| [[Window]] link:spark-sql-LogicalPlan-Window.adoc[Window]
| `visitWindow`

| [[LogicalPlan]] Other link:spark-sql-LogicalPlan.adoc[logical operators]
| `default`
|===
