# ResolveFunctions Logical Resolution Rule -- Resolving grouping__id UnresolvedAttribute, UnresolvedGenerator And UnresolvedFunction Expressions

`ResolveFunctions` is a logical resolution rule that the link:spark-sql-Analyzer.adoc#ResolveFunctions[logical query plan analyzer] uses to <<apply, resolve grouping__id UnresolvedAttribute, UnresolvedGenerator and UnresolvedFunction expressions>> in an entire logical query plan.

Technically, `ResolveReferences` is just a link:spark-sql-catalyst-Rule.adoc[Catalyst rule] for transforming link:spark-sql-LogicalPlan.adoc[logical plans], i.e. `Rule[LogicalPlan]`.

`ResolveFunctions` is part of link:spark-sql-Analyzer.adoc#Resolution[Resolution] fixed-point batch of rules.

NOTE: `ResolveFunctions` is a Scala object inside link:spark-sql-Analyzer.adoc[Analyzer] class.

[[example]]
[source, scala]
----
import spark.sessionState.analyzer.ResolveFunctions

// Example: UnresolvedAttribute with VirtualColumn.hiveGroupingIdName (grouping__id) => Alias
import org.apache.spark.sql.catalyst.expressions.VirtualColumn
import org.apache.spark.sql.catalyst.analysis.UnresolvedAttribute
val groupingIdAttr = UnresolvedAttribute(VirtualColumn.hiveGroupingIdName)
scala> println(groupingIdAttr.sql)
`grouping__id`

// Using Catalyst DSL to create a logical plan with grouping__id
import org.apache.spark.sql.catalyst.dsl.plans._
val t1 = table("t1")
val plan = t1.select(groupingIdAttr)
scala> println(plan.numberedTreeString)
00 'Project ['grouping__id]
01 +- 'UnresolvedRelation `t1`

val resolvedPlan = ResolveFunctions(plan)
scala> println(resolvedPlan.numberedTreeString)
00 'Project [grouping_id() AS grouping__id#0]
01 +- 'UnresolvedRelation `t1`

import org.apache.spark.sql.catalyst.expressions.Alias
val alias = resolvedPlan.expressions.head.asInstanceOf[Alias]
scala> println(alias.sql)
grouping_id() AS `grouping__id`

// Example: UnresolvedGenerator => a) Generator or b) analysis failure
// Register a function so a function resolution works
import org.apache.spark.sql.catalyst.FunctionIdentifier
import org.apache.spark.sql.catalyst.catalog.CatalogFunction
val f1 = CatalogFunction(FunctionIdentifier(funcName = "f1"), "java.lang.String", resources = Nil)
import org.apache.spark.sql.catalyst.expressions.{Expression, Stack}
// FIXME What happens when looking up a function with the functionBuilder None in registerFunction?
// Using Stack as ResolveFunctions requires that the function to be resolved is a Generator
// You could roll your own, but that's a demo, isn't it? (don't get too carried away)
spark.sessionState.catalog.registerFunction(
  funcDefinition = f1,
  overrideIfExists = true,
  functionBuilder = Some((children: Seq[Expression]) => Stack(children = Nil)))

import org.apache.spark.sql.catalyst.analysis.UnresolvedGenerator
import org.apache.spark.sql.catalyst.FunctionIdentifier
val ungen = UnresolvedGenerator(name = FunctionIdentifier("f1"), children = Seq.empty)
val plan = t1.select(ungen)
scala> println(plan.numberedTreeString)
00 'Project [unresolvedalias('f1(), None)]
01 +- 'UnresolvedRelation `t1`

val resolvedPlan = ResolveFunctions(plan)
scala> println(resolvedPlan.numberedTreeString)
00 'Project [unresolvedalias(stack(), None)]
01 +- 'UnresolvedRelation `t1`

CAUTION: FIXME

// Example: UnresolvedFunction => a) AggregateWindowFunction with(out) isDistinct, b) AggregateFunction, c) other with(out) isDistinct
val plan = ???
val resolvedPlan = ResolveFunctions(plan)
----

=== [[apply]] Resolving grouping__id UnresolvedAttribute, UnresolvedGenerator and UnresolvedFunction Expressions In Entire Query Plan (Applying ResolveFunctions to Logical Plan) -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of link:spark-sql-catalyst-Rule.adoc#apply[Rule Contract] to apply a rule to a link:spark-sql-LogicalPlan.adoc[logical plan].

`apply` takes a link:spark-sql-LogicalPlan.adoc[logical plan] and transforms each expression (for every logical operator found in the query plan) as follows:

* For link:spark-sql-Expression-UnresolvedAttribute.adoc[UnresolvedAttributes] with link:spark-sql-Expression-UnresolvedAttribute.adoc#name[names] as `grouping__id`, `apply` creates a link:spark-sql-Expression-Alias.adoc#creating-instance[Alias] (with a `GroupingID` child expression and `grouping__id` name).
+
That case seems mostly for compatibility with Hive as `grouping__id` attribute name is used by Hive.

* For link:spark-sql-Expression-UnresolvedGenerator.adoc[UnresolvedGenerators], `apply` requests the link:spark-sql-Analyzer.adoc#catalog[SessionCatalog] to link:spark-sql-SessionCatalog.adoc#lookupFunction[find a Generator function by name].
+
If some other non-generator function is found for the name, `apply` fails the analysis phase by reporting an `AnalysisException`:
+
```
[name] is expected to be a generator. However, its class is [className], which is not a generator.
```

* For link:spark-sql-Expression-UnresolvedFunction.adoc[UnresolvedFunctions], `apply` requests the link:spark-sql-Analyzer.adoc#catalog[SessionCatalog] to link:spark-sql-SessionCatalog.adoc#lookupFunction[find a function by name].

* link:spark-sql-Expression-AggregateWindowFunction.adoc[AggregateWindowFunctions] are returned directly or `apply` fails the analysis phase by reporting an `AnalysisException` when the `UnresolvedFunction` has link:spark-sql-Expression-UnresolvedFunction.adoc#isDistinct[isDistinct] flag enabled.
+
```
[name] does not support the modifier DISTINCT
```

* link:spark-sql-Expression-AggregateFunction.adoc[AggregateFunctions] are wrapped in a link:spark-sql-Expression-AggregateExpression.adoc[AggregateExpression] (with `Complete` aggregate mode)

* All other functions are returned directly or `apply` fails the analysis phase by reporting an `AnalysisException` when the `UnresolvedFunction` has link:spark-sql-Expression-UnresolvedFunction.adoc#isDistinct[isDistinct] flag enabled.
+
```
[name] does not support the modifier DISTINCT
```

`apply` skips link:spark-sql-Expression.adoc#childrenResolved[unresolved expressions].
