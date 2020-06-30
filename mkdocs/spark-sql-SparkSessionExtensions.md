# SparkSessionExtensions

`SparkSessionExtensions` is an <<methods, interface>> that a Spark developer can use to extend a <<spark-sql-SparkSession.adoc#extensions, SparkSession>> with custom query execution rules and a relational entity parser.

As a Spark developer, you use <<spark-sql-SparkSession-Builder.adoc#withExtensions, Builder.withExtensions>> method (while building a new <<spark-sql-SparkSession.adoc#, SparkSession>>) to access the session-bound `SparkSessionExtensions`.

[[methods]]
.SparkSessionExtensions API
[cols="1,3",options="header",width="100%"]
|===
| Method
| Description

| <<injectCheckRule, injectCheckRule>>
a|

[source, scala]
----
injectCheckRule(builder: SparkSession => LogicalPlan => Unit): Unit
----

| <<injectOptimizerRule, injectOptimizerRule>>
a| Registering a custom operator optimization rule

[source, scala]
----
injectOptimizerRule(builder: SparkSession => Rule[LogicalPlan]): Unit
----

| <<injectParser, injectParser>>
a|

[source, scala]
----
injectParser(builder: (SparkSession, ParserInterface) => ParserInterface): Unit
----

| <<injectPlannerStrategy, injectPlannerStrategy>>
a|

[source, scala]
----
injectPlannerStrategy(builder: SparkSession => Strategy): Unit
----

| <<injectPostHocResolutionRule, injectPostHocResolutionRule>>
a|

[source, scala]
----
injectPostHocResolutionRule(builder: SparkSession => Rule[LogicalPlan]): Unit
----

| <<injectResolutionRule, injectResolutionRule>>
a|

[source, scala]
----
injectResolutionRule(builder: SparkSession => Rule[LogicalPlan]): Unit
----
|===

`SparkSessionExtensions` is an integral part of <<spark-sql-SparkSession.adoc#extensions, SparkSession>> (and is indirectly required to create one).

[[internal-registries]]
.SparkSessionExtensions's Internal Properties (e.g. Registries, Counters and Flags)
[cols="1m,2",options="header",width="100%"]
|===
| Name
| Description

| optimizerRules
a| [[optimizerRules]] Collection of `RuleBuilder` functions (i.e. `SparkSession => Rule[LogicalPlan]`)

Used when `SparkSessionExtensions` is requested to:

* <<buildOptimizerRules, Associate custom operator optimization rules with SparkSession>>

* <<injectOptimizerRule, Register a custom operator optimization rule>>
|===

=== [[buildOptimizerRules]] Associating Custom Operator Optimization Rules with SparkSession -- `buildOptimizerRules` Method

[source, scala]
----
buildOptimizerRules(session: SparkSession): Seq[Rule[LogicalPlan]]
----

`buildOptimizerRules` gives the <<optimizerRules, optimizerRules>> logical rules that are associated with the input <<spark-sql-SparkSession.adoc#, SparkSession>>.

NOTE: `buildOptimizerRules` is used exclusively when `BaseSessionStateBuilder` is requested for the <<spark-sql-BaseSessionStateBuilder.adoc#customOperatorOptimizationRules, custom operator optimization rules to add to the base Operator Optimization batch>>.

=== [[injectCheckRule]] Registering Custom Check Analysis Rule (Builder) -- `injectCheckRule` Method

[source, scala]
----
injectCheckRule(builder: SparkSession => LogicalPlan => Unit): Unit
----

`injectCheckRule`...FIXME

=== [[injectOptimizerRule]] Registering Custom Operator Optimization Rule (Builder) -- `injectOptimizerRule` Method

[source, scala]
----
injectOptimizerRule(builder: SparkSession => Rule[LogicalPlan]): Unit
----

`injectOptimizerRule` simply registers a custom operator optimization rule (as a `RuleBuilder` function) to the <<optimizerRules, optimizerRules>> internal registry.

=== [[injectParser]] Registering Custom Parser (Builder) -- `injectParser` Method

[source, scala]
----
injectParser(builder: (SparkSession, ParserInterface) => ParserInterface): Unit
----

`injectParser`...FIXME

=== [[injectPlannerStrategy]] Registering Custom Planner Strategy (Builder) -- `injectPlannerStrategy` Method

[source, scala]
----
injectPlannerStrategy(builder: SparkSession => Strategy): Unit
----

`injectPlannerStrategy`...FIXME

=== [[injectPostHocResolutionRule]] Registering Custom Post-Hoc Resolution Rule (Builder) -- `injectPostHocResolutionRule` Method

[source, scala]
----
injectPostHocResolutionRule(builder: SparkSession => Rule[LogicalPlan]): Unit
----

`injectPostHocResolutionRule`...FIXME

=== [[injectResolutionRule]] Registering Custom Resolution Rule (Builder) -- `injectResolutionRule` Method

[source, scala]
----
injectResolutionRule(builder: SparkSession => Rule[LogicalPlan]): Unit
----

`injectResolutionRule`...FIXME
