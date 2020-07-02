title: UnresolvedAttribute

# UnresolvedAttribute Leaf Expression

[[name]]
`UnresolvedAttribute` is a named link:spark-sql-Expression-Attribute.adoc[Attribute] leaf expression (i.e. it has a name) that represents a reference to an entity in a logical query plan.

`UnresolvedAttribute` is <<creating-instance, created>> when:

* `AstBuilder` is requested to link:spark-sql-AstBuilder.adoc#visitDereference[visitDereference]

* `LogicalPlan` is requested to link:spark-sql-LogicalPlan.adoc#resolve[resolve an attribute by name parts]

* `DescribeColumnCommand` is link:spark-sql-LogicalPlan-DescribeColumnCommand.adoc#run[executed]

[[resolved]]
`UnresolvedAttribute` can never be link:spark-sql-Expression.adoc#resolved[resolved] (and is replaced at <<analysis-phase, analysis phase>>).

[[analysis-phase]]
[NOTE]
====
`UnresolvedAttribute` is resolved when `Analyzer` is link:spark-sql-Analyzer.adoc#execute[executed] by the following link:spark-sql-Analyzer.adoc#Resolution[logical resolution rules]:

* link:spark-sql-Analyzer-ResolveReferences.adoc#resolve[ResolveReferences]

* `ResolveMissingReferences`

* `ResolveDeserializer`

* `ResolveSubquery`
====

[[Unevaluable]][[eval]][[doGenCode]]
Given `UnresolvedAttribute` can never be resolved it should not come as a surprise that it link:spark-sql-Expression.adoc#Unevaluable[cannot be evaluated] either (i.e. produce a value given an internal row). When requested to evaluate, `UnresolvedAttribute` simply reports a `UnsupportedOperationException`.

```
Cannot evaluate expression: [this]
```

[[creating-instance]]
[[nameParts]]
`UnresolvedAttribute` takes *name parts* when created.

[source, scala]
----
import org.apache.spark.sql.catalyst.analysis.UnresolvedAttribute

scala> val t1 = UnresolvedAttribute("t1")
t1: org.apache.spark.sql.catalyst.analysis.UnresolvedAttribute = 't1

scala> val t2 = UnresolvedAttribute("db1.t2")
t2: org.apache.spark.sql.catalyst.analysis.UnresolvedAttribute = 'db1.t2

scala> println(s"Number of name parts: ${t2.nameParts.length}")
Number of name parts: 2

scala> println(s"Name parts: ${t2.nameParts.mkString(",")}")
Name parts: db1,t2
----

[[apply]]
`UnresolvedAttribute` can be created with a fully-qualified name with dots to separate name parts.

[source, scala]
----
apply(name: String): UnresolvedAttribute
----

[TIP]
====
Use backticks (````) around names with dots (`.`) to disable them as separators.

The following is a two-part attribute name with `a.b` and `c` name parts.

```
`a.b`.c
```
====

[[quoted]]
`UnresolvedAttribute` can also be created without the dots with the special meaning.

[source, scala]
----
import org.apache.spark.sql.catalyst.analysis.UnresolvedAttribute
val attr1 = UnresolvedAttribute.quoted("a.b.c")
scala> println(s"Number of name parts: ${attr1.nameParts.length}")
Number of name parts: 1
----

[NOTE]
====
Catalyst DSL defines two Scala implicits to create an `UnresolvedAttribute`:

* `StringToAttributeConversionHelper` is a Scala implicit class that converts `$"colName"` into an `UnresolvedAttribute`

* `symbolToUnresolvedAttribute` is a Scala implicit method that converts `'colName` into an `UnresolvedAttribute`

Both implicits are part of link:spark-sql-catalyst-dsl.adoc#ExpressionConversions[ExpressionConversions] Scala trait of Catalyst DSL.

Import `expressions` object to get access to the expression conversions.

[source, scala]
----
// Use `sbt console` with Spark libraries defined (in `build.sbt`)
import org.apache.spark.sql.catalyst.dsl.expressions._
scala> :type $"name"
org.apache.spark.sql.catalyst.analysis.UnresolvedAttribute

import org.apache.spark.sql.catalyst.analysis.UnresolvedAttribute
val nameAttr: UnresolvedAttribute = 'name

scala> :type nameAttr
org.apache.spark.sql.catalyst.analysis.UnresolvedAttribute
----
====

[NOTE]
====
A `UnresolvedAttribute` can be replaced by (_resolved_) a `NamedExpression` using an link:spark-sql-LogicalPlan.adoc#resolveQuoted[analyzed logical plan] (of the structured query the attribute is part of).

[source, scala]
----
val analyzedPlan = Seq((0, "zero")).toDF("id", "name").queryExecution.analyzed

import org.apache.spark.sql.catalyst.analysis.UnresolvedAttribute
val nameAttr = UnresolvedAttribute("name")
val nameResolved = analyzedPlan.resolveQuoted(
  name = nameAttr.name,
  resolver = spark.sessionState.analyzer.resolver).getOrElse(nameAttr)

scala> println(nameResolved.numberedTreeString)
00 name#47: string

scala> :type nameResolved
org.apache.spark.sql.catalyst.expressions.NamedExpression
----
====
