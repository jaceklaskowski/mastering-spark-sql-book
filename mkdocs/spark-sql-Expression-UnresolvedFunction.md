title: UnresolvedFunction

# UnresolvedFunction Unevaluable Expression -- Logical Representation of Functions in Queries

`UnresolvedFunction` is an link:spark-sql-Expression.adoc[Catalyst expression] that represents a function (application) in a logical query plan.

`UnresolvedFunction` is <<creating-instance, created>> as a result of the following:

* link:spark-sql-functions.adoc#callUDF[callUDF] standard function

* link:spark-sql-RelationalGroupedDataset.adoc#agg[RelationalGroupedDataset.agg] operator with aggregation functions specified by name (that link:spark-sql-RelationalGroupedDataset.adoc#strToExpr[converts function names to UnresolvedFunction expressions])

* `AstBuilder` is requested to link:spark-sql-AstBuilder.adoc#visitFunctionCall[visitFunctionCall] (in SQL queries)

[[resolved]]
`UnresolvedFunction` can never be link:spark-sql-Expression.adoc#resolved[resolved] (and is replaced at analysis phase).

NOTE: `UnresolvedFunction` is first looked up in link:spark-sql-Analyzer-LookupFunctions.adoc[LookupFunctions] logical rule and then resolved in link:spark-sql-Analyzer-ResolveFunctions.adoc[ResolveFunctions] logical resolution rule.

[[Unevaluable]][[eval]][[doGenCode]]
Given `UnresolvedFunction` can never be resolved it should not come as a surprise that it link:spark-sql-Expression.adoc#Unevaluable[cannot be evaluated] either (i.e. produce a value given an internal row). When requested to evaluate, `UnresolvedFunction` simply reports a `UnsupportedOperationException`.

```
Cannot evaluate expression: [this]
```

NOTE: link:spark-sql-Expression.adoc#Unevaluable[Unevaluable expressions] are expressions that have to be replaced by some other expressions during link:spark-sql-Analyzer.adoc[analysis] or link:spark-sql-Optimizer.adoc[optimization] (or they fail analysis).

TIP: Use Catalyst DSL's link:spark-sql-catalyst-dsl.adoc#function[function] or link:spark-sql-catalyst-dsl.adoc#distinctFunction[distinctFunction] to create a `UnresolvedFunction` with <<isDistinct, isDistinct>> flag off and on, respectively.

[source, scala]
----
// Using Catalyst DSL to create UnresolvedFunctions
import org.apache.spark.sql.catalyst.dsl.expressions._

// Scala Symbols supported only
val f = 'f.function()
scala> :type f
org.apache.spark.sql.catalyst.analysis.UnresolvedFunction

scala> f.isDistinct
res0: Boolean = false

val g = 'g.distinctFunction()
scala> g.isDistinct
res1: Boolean = true
----

=== [[apply]] Creating UnresolvedFunction (With Database Undefined) -- `apply` Factory Method

[source, scala]
----
apply(name: String, children: Seq[Expression], isDistinct: Boolean): UnresolvedFunction
----

`apply` creates a `FunctionIdentifier` with the `name` and no database first and then creates a <<UnresolvedFunction, UnresolvedFunction>> with the `FunctionIdentifier`, `children` and `isDistinct` flag.

[NOTE]
====
`apply` is used when:

* link:spark-sql-functions.adoc#callUDF[callUDF] standard function is used

* `RelationalGroupedDataset` is requested to link:spark-sql-RelationalGroupedDataset.adoc#agg[agg] with aggregation functions specified by name (and link:spark-sql-RelationalGroupedDataset.adoc#strToExpr[converts function names to UnresolvedFunction expressions])
====

=== [[creating-instance]] Creating UnresolvedFunction Instance

`UnresolvedFunction` takes the following when created:

* [[name]] `FunctionIdentifier`
* [[children]] Child link:spark-sql-Expression.adoc[expressions]
* [[isDistinct]] `isDistinct` flag
