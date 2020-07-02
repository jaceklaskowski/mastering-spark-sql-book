title: UnresolvedStar

# UnresolvedStar Expression

`UnresolvedStar` is a link:spark-sql-Expression-Star.adoc[Star] expression that represents a star (i.e. all) expression in a logical query plan.

`UnresolvedStar` is <<creating-instance, created>> when:

* `Column` is link:spark-sql-Column.adoc#star[created] with `*`

* `AstBuilder` is requested to link:spark-sql-AstBuilder.adoc#visitStar[visitStar]

[source, scala]
----
val q = spark.range(5).select("*")
val plan = q.queryExecution.logical
scala> println(plan.numberedTreeString)
00 'Project [*]
01 +- AnalysisBarrier
02       +- Range (0, 5, step=1, splits=Some(8))

import org.apache.spark.sql.catalyst.analysis.UnresolvedStar
val starExpr = plan.expressions.head.asInstanceOf[UnresolvedStar]

val namedExprs = starExpr.expand(input = q.queryExecution.analyzed, spark.sessionState.analyzer.resolver)
scala> println(namedExprs.head.numberedTreeString)
00 id#0: bigint
----

[[resolved]]
`UnresolvedStar` can never be link:spark-sql-Expression.adoc#resolved[resolved], and is <<expand, expanded>> at analysis (when link:spark-sql-Analyzer-ResolveReferences.adoc[ResolveReferences] logical resolution rule is executed).

NOTE: `UnresolvedStar` can only be used in link:spark-sql-Expression-Star.adoc#expand[Project, Aggregate or ScriptTransformation logical operators].

[[Unevaluable]][[eval]][[doGenCode]]
Given `UnresolvedStar` can never be <<resolved, resolved>> it should not come as a surprise that it link:spark-sql-Expression.adoc#Unevaluable[cannot be evaluated] either (i.e. produce a value given an internal row). When requested to evaluate, `UnresolvedStar` simply reports a `UnsupportedOperationException`.

```
Cannot evaluate expression: [this]
```

[[creating-instance]]
[[target]]
When created, `UnresolvedStar` takes *name parts* that, once concatenated, is the target of the star expansion.

[source, scala]
----
import org.apache.spark.sql.catalyst.analysis.UnresolvedStar
scala> val us = UnresolvedStar(None)
us: org.apache.spark.sql.catalyst.analysis.UnresolvedStar = *

scala> val ab = UnresolvedStar(Some("a" :: "b" :: Nil))
ab: org.apache.spark.sql.catalyst.analysis.UnresolvedStar = List(a, b).*
----

[TIP]
====
Use `star` operator from Catalyst DSL's link:spark-sql-catalyst-dsl.adoc#expressions[expressions] to create an `UnresolvedStar`.

[source, scala]
----
import org.apache.spark.sql.catalyst.dsl.expressions._
val s = star()
scala> :type s
org.apache.spark.sql.catalyst.expressions.Expression

import org.apache.spark.sql.catalyst.analysis.UnresolvedStar
assert(s.isInstanceOf[UnresolvedStar])

val s = star("a", "b")
scala> println(s)
WrappedArray(a, b).*
----

You could also use `$"*"` or `'*` to create an `UnresolvedStar`, but that requires `sbt console` (with Spark libraries defined in `build.sbt`) as the Catalyst DSL `expressions` implicits interfere with the Spark implicits to create columns.
====

[NOTE]
====
`AstBuilder` link:spark-sql-AstBuilder.adoc#visitFunctionCall[replaces] `count(*)` (with no `DISTINCT` keyword) to `count(1)`.

```
val q = sql("SELECT COUNT(*) FROM RANGE(1,2,3)")
scala> println(q.queryExecution.logical.numberedTreeString)
00 'Project [unresolvedalias('count(1), None)]
01 +- 'UnresolvedTableValuedFunction range, [1, 2, 3]

val q = sql("SELECT COUNT(DISTINCT *) FROM RANGE(1,2,3)")
scala> println(q.queryExecution.logical.numberedTreeString)
00 'Project [unresolvedalias('COUNT(*), None)]
01 +- 'UnresolvedTableValuedFunction RANGE, [1, 2, 3]
```
====

=== [[expand]] Star Expansion -- `expand` Method

[source, scala]
----
expand(input: LogicalPlan, resolver: Resolver): Seq[NamedExpression]
----

NOTE: `expand` is part of link:spark-sql-Expression-Star.adoc#expand[Star Contract] to...FIXME.

`expand` first expands to named expressions per <<target, target>>:

* For unspecified <<target, target>>, `expand` gives the link:spark-sql-catalyst-QueryPlan.adoc#output[output] schema of the `input` logical query plan (that assumes that the star refers to a relation / table)

* For <<target, target>> with one element, `expand` gives the table (attribute) in the link:spark-sql-catalyst-QueryPlan.adoc#output[output] schema of the `input` logical query plan (using link:spark-sql-Expression-NamedExpression.adoc#qualifier[qualifiers]) if available

With no result earlier, `expand` then requests the `input` logical query plan to link:spark-sql-LogicalPlan.adoc#resolve[resolve] the <<target, target>> name parts to a named expression.

For a named expression of link:spark-sql-StructType.adoc[StructType] data type, `expand` creates an link:spark-sql-Expression-Alias.adoc#creating-instance[Alias] expression with a link:spark-sql-Expression-GetStructField.adoc#creating-instance[GetStructField] unary expression (with the resolved named expression and the field index).

[source, scala]
----
val q = Seq((0, "zero")).toDF("id", "name").select(struct("id", "name") as "s")
val analyzedPlan = q.queryExecution.analyzed

import org.apache.spark.sql.catalyst.analysis.UnresolvedStar
import org.apache.spark.sql.catalyst.dsl.expressions._
val s = star("s").asInstanceOf[UnresolvedStar]
val exprs = s.expand(input = analyzedPlan, spark.sessionState.analyzer.resolver)

// star("s") should expand to two Alias(GetStructField) expressions
// s is a struct of id and name in the query

import org.apache.spark.sql.catalyst.expressions.{Alias, GetStructField}
val getStructFields = exprs.collect { case Alias(g: GetStructField, _) => g }.map(_.sql)
scala> getStructFields.foreach(println)
`s`.`id`
`s`.`name`
----

`expand` reports a `AnalysisException` when:

* The link:spark-sql-Expression.adoc#dataType[data type] of the named expression (when the `input` logical plan was requested to link:spark-sql-LogicalPlan.adoc#resolve[resolve] the <<target, target>>) is not a link:spark-sql-StructType.adoc[StructType].
+
```
Can only star expand struct data types. Attribute: `[target]`
```

* Earlier attempts gave no results
+
```
cannot resolve '[target].*' given input columns '[from]'
```
