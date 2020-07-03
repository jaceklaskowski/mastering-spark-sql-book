# LookupFunctions Logical Rule -- Checking Whether UnresolvedFunctions Are Resolvable

`LookupFunctions` is a logical rule that the link:spark-sql-Analyzer.adoc#LookupFunctions[logical query plan analyzer] uses to <<apply, make sure that UnresolvedFunction expressions can be resolved>> in an entire logical query plan.

`LookupFunctions` is similar to link:spark-sql-Analyzer-ResolveFunctions.adoc[ResolveFunctions] logical resolution rule, but it is `ResolveFunctions` to resolve `UnresolvedFunction` expressions while `LookupFunctions` is just a sanity check that a future resolution is possible if tried.

Technically, `LookupFunctions` is just a link:spark-sql-catalyst-Rule.adoc[Catalyst rule] for transforming link:spark-sql-LogicalPlan.adoc[logical plans], i.e. `Rule[LogicalPlan]`.

NOTE: `LookupFunctions` does not however transform a logical plan.

`LookupFunctions` is part of link:spark-sql-Analyzer.adoc#Simple-Sanity-Check[Simple Sanity Check] one-off batch of rules.

NOTE: `LookupFunctions` is a Scala object inside link:spark-sql-Analyzer.adoc[Analyzer] class.

[[example]]
[source, scala]
----
// Using Catalyst DSL to create a logical plan with an unregistered function
import org.apache.spark.sql.catalyst.dsl.plans._
val t1 = table("t1")

import org.apache.spark.sql.catalyst.dsl.expressions._
val f1 = 'f1.function()

val plan = t1.select(f1)
scala> println(plan.numberedTreeString)
00 'Project [unresolvedalias('f1(), None)]
01 +- 'UnresolvedRelation `t1`

// Make sure the function f1 does not exist
import org.apache.spark.sql.catalyst.FunctionIdentifier
spark.sessionState.catalog.dropFunction(FunctionIdentifier("f1"), ignoreIfNotExists = true)

assert(spark.catalog.functionExists("f1") == false)

import spark.sessionState.analyzer.LookupFunctions
scala> LookupFunctions(plan)
org.apache.spark.sql.AnalysisException: Undefined function: 'f1'. This function is neither a registered temporary function nor a permanent function registered in the database 'default'.;
  at org.apache.spark.sql.catalyst.analysis.Analyzer$LookupFunctions$$anonfun$apply$15$$anonfun$applyOrElse$49.apply(Analyzer.scala:1198)
  at org.apache.spark.sql.catalyst.analysis.Analyzer$LookupFunctions$$anonfun$apply$15$$anonfun$applyOrElse$49.apply(Analyzer.scala:1198)
  at org.apache.spark.sql.catalyst.analysis.package$.withPosition(package.scala:48)
  at org.apache.spark.sql.catalyst.analysis.Analyzer$LookupFunctions$$anonfun$apply$15.applyOrElse(Analyzer.scala:1197)
  at org.apache.spark.sql.catalyst.analysis.Analyzer$LookupFunctions$$anonfun$apply$15.applyOrElse(Analyzer.scala:1195)
----

=== [[apply]] Applying LookupFunctions to Logical Plan -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of link:spark-sql-catalyst-Rule.adoc#apply[Rule Contract] to apply a rule to a link:spark-sql-LogicalPlan.adoc[logical plan].

`apply` finds all link:spark-sql-Expression-UnresolvedFunction.adoc[UnresolvedFunction] expressions (in every logical operator in the input link:spark-sql-LogicalPlan.adoc[logical plan]) and requests the link:spark-sql-Analyzer.adoc#catalog[SessionCatalog] to link:spark-sql-SessionCatalog.adoc#functionExists[check if their functions exist].

`apply` does nothing if a function exists or reports a `NoSuchFunctionException` (that fails logical analysis).

```
Undefined function: '[func]'. This function is neither a registered temporary function nor a permanent function registered in the database '[db]'.
```
