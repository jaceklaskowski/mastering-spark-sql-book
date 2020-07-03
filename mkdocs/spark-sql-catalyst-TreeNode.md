# TreeNode -- Node in Catalyst Tree

`TreeNode` is the <<contract, contract>> of <<implementations, nodes>> in <<spark-sql-catalyst.adoc#, Catalyst>> tree with <<nodeName, name>> and zero or more <<children, children>>.

[[contract]]
[source, scala]
----
package org.apache.spark.sql.catalyst.trees

abstract class TreeNode[BaseType <: TreeNode[BaseType]] extends Product {
  self: BaseType =>

  // only required properties (vals and methods) that have no implementation
  // the others follow
  def children: Seq[BaseType]
  def verboseString: String
}
----

`TreeNode` is a recursive data structure that can have one or many <<children, children>> that are again `TreeNodes`.

TIP: Read up on `<:` type operator in Scala in https://docs.scala-lang.org/tour/upper-type-bounds.html[Upper Type Bounds].

Scala-specific, `TreeNode` is an abstract class that is the <<implementations, base class>> of Catalyst <<spark-sql-Expression.adoc#, Expression>> and <<spark-sql-catalyst-QueryPlan.adoc#, QueryPlan>> abstract classes.

`TreeNode` therefore allows for building entire trees of `TreeNodes`, e.g. generic <<spark-sql-catalyst-QueryPlan.adoc#, query plans>> with concrete <<spark-sql-LogicalPlan.adoc#, logical>> and <<spark-sql-SparkPlan.adoc#, physical>> operators that both use <<spark-sql-Expression.adoc#, Catalyst expressions>> (which are `TreeNodes` again).

NOTE: Spark SQL uses `TreeNode` for <<spark-sql-catalyst-QueryPlan.adoc#, query plans>> and <<spark-sql-Expression.adoc#, Catalyst expressions>> that can further be used together to build more advanced trees, e.g. Catalyst expressions can have query plans as <<spark-sql-subqueries.adoc#, subquery expressions>>.

`TreeNode` can itself be a node in a tree or a collection of nodes, i.e. itself and the <<children, children>> nodes. Not only does `TreeNode` come with the <<methods, methods>> that you may have used in https://docs.scala-lang.org/overviews/collections/overview.html[Scala Collection API] (e.g. <<map, map>>, <<flatMap, flatMap>>, <<collect, collect>>, <<collectFirst, collectFirst>>, <<foreach, foreach>>), but also specialized ones for more advanced tree manipulation, e.g. <<mapChildren, mapChildren>>, <<transform, transform>>, <<transformDown, transformDown>>, <<transformUp, transformUp>>, <<foreachUp, foreachUp>>, <<numberedTreeString, numberedTreeString>>, <<p, p>>, <<asCode, asCode>>, <<prettyJson, prettyJson>>.

[[methods]]
.TreeNode API (Public Methods)
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| <<apply, apply>>
a|

[source, scala]
----
apply(number: Int): TreeNode[_]
----

| <<argString, argString>>
a|

[source, scala]
----
argString: String
----

| <<asCode, asCode>>
a|

[source, scala]
----
asCode: String
----

| <<collect, collect>>
a|

[source, scala]
----
collect[B](pf: PartialFunction[BaseType, B]): Seq[B]
----

| <<collectFirst, collectFirst>>
a|

[source, scala]
----
collectFirst[B](pf: PartialFunction[BaseType, B]): Option[B]
----

| <<collectLeaves, collectLeaves>>
a|

[source, scala]
----
collectLeaves(): Seq[BaseType]
----

| <<fastEquals, fastEquals>>
a|

[source, scala]
----
fastEquals(other: TreeNode[_]): Boolean
----

| <<find, find>>
a|

[source, scala]
----
find(f: BaseType => Boolean): Option[BaseType]
----

| <<flatMap, flatMap>>
a|

[source, scala]
----
flatMap[A](f: BaseType => TraversableOnce[A]): Seq[A]
----

| <<foreach, foreach>>
a|

[source, scala]
----
foreach(f: BaseType => Unit): Unit
----

| <<foreachUp, foreachUp>>
a|

[source, scala]
----
foreachUp(f: BaseType => Unit): Unit
----

| <<generateTreeString, generateTreeString>>
a|

[source, scala]
----
generateTreeString(
  depth: Int,
  lastChildren: Seq[Boolean],
  builder: StringBuilder,
  verbose: Boolean,
  prefix: String = "",
  addSuffix: Boolean = false): StringBuilder
----

| <<map, map>>
a|

[source, scala]
----
map[A](f: BaseType => A): Seq[A]
----

| <<mapChildren, mapChildren>>
a|

[source, scala]
----
mapChildren(f: BaseType => BaseType): BaseType
----

| <<nodeName, nodeName>>
a|

[source, scala]
----
nodeName: String
----

| <<numberedTreeString, numberedTreeString>>
a|

[source, scala]
----
numberedTreeString: String
----

| <<p, p>>
a|

[source, scala]
----
p(number: Int): BaseType
----

| <<prettyJson, prettyJson>>
a|

[source, scala]
----
prettyJson: String
----

| <<simpleString, simpleString>>
a|

[source, scala]
----
simpleString: String
----

| <<toJSON, toJSON>>
a|

[source, scala]
----
toJSON: String
----

| <<transform, transform>>
a|

[source, scala]
----
transform(rule: PartialFunction[BaseType, BaseType]): BaseType
----

| <<transformDown, transformDown>>
a|

[source, scala]
----
transformDown(rule: PartialFunction[BaseType, BaseType]): BaseType
----

| <<transformUp, transformUp>>
a|

[source, scala]
----
transformUp(rule: PartialFunction[BaseType, BaseType]): BaseType
----

| <<treeString, treeString>>
a|

[source, scala]
----
treeString: String
treeString(verbose: Boolean, addSuffix: Boolean = false): String
----

| <<verboseString, verboseString>>
a|

[source, scala]
----
verboseString: String
----

| <<verboseStringWithSuffix, verboseStringWithSuffix>>
a|

[source, scala]
----
verboseStringWithSuffix: String
----

| <<withNewChildren, withNewChildren>>
a|

[source, scala]
----
withNewChildren(newChildren: Seq[BaseType]): BaseType
----
|===

.(Subset of) TreeNode Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| `children`
| [[children]] Child nodes

| `verboseString`
| [[verboseString]] One-line *verbose description*

Used when `TreeNode` is requested for <<generateTreeString, generateTreeString>> (with `verbose` flag enabled) and <<verboseStringWithSuffix, verboseStringWithSuffix>>
|===

[[implementations]]
.TreeNodes
[cols="1,2",options="header",width="100%"]
|===
| TreeNode
| Description

| <<spark-sql-Expression.adoc#, Expression>>
| [[Expression]]

| <<spark-sql-catalyst-QueryPlan.adoc#, QueryPlan>>
| [[QueryPlan]]
|===

[TIP]
====
`TreeNode` abstract type is a fairly advanced Scala type definition (at least comparing to the other Scala types in Spark) so understanding its behaviour even outside Spark might be worthwhile by itself.

[source, scala]
----
abstract class TreeNode[BaseType <: TreeNode[BaseType]] extends Product {
  self: BaseType =>

  // ...
}
----
====

=== [[withNewChildren]] `withNewChildren` Method

[source, scala]
----
withNewChildren(newChildren: Seq[BaseType]): BaseType
----

`withNewChildren`...FIXME

NOTE: `withNewChildren` is used when...FIXME

=== [[simpleString]] Simple Node Description -- `simpleString` Method

[source, scala]
----
simpleString: String
----

`simpleString` gives a simple one-line description of a `TreeNode`.

Internally, `simpleString` is the <<nodeName, nodeName>> followed by <<argString, argString>> separated by a single white space.

NOTE: `simpleString` is used when `TreeNode` is requested for <<argString, argString>> (of child nodes) and <<generateTreeString, tree text representation>> (with `verbose` flag off).

=== [[numberedTreeString]] Numbered Text Representation -- `numberedTreeString` Method

[source, scala]
----
numberedTreeString: String
----

`numberedTreeString` adds numbers to the <<treeString, text representation of all the nodes>>.

NOTE: `numberedTreeString` is used primarily for interactive debugging using <<apply, apply>> and <<p, p>> methods.

=== [[apply]] Getting n-th TreeNode in Tree (for Interactive Debugging) -- `apply` Method

[source, scala]
----
apply(number: Int): TreeNode[_]
----

`apply` gives `number`-th tree node in a tree.

NOTE: `apply` can be used for interactive debugging.

Internally, `apply` <<getNodeNumbered, gets the node>> at `number` position or `null`.

=== [[p]] Getting n-th BaseType in Tree (for Interactive Debugging) -- `p` Method

[source, scala]
----
p(number: Int): BaseType
----

`p` gives `number`-th tree node in a tree as `BaseType` for interactive debugging.

NOTE: `p` can be used for interactive debugging.

[NOTE]
====
`BaseType` is the base type of a tree and in Spark SQL can be:

* link:spark-sql-LogicalPlan.adoc[LogicalPlan] for logical plan trees

* link:spark-sql-SparkPlan.adoc[SparkPlan] for physical plan trees

* link:spark-sql-Expression.adoc[Expression] for expression trees
====

=== [[toString]] Text Representation -- `toString` Method

[source, scala]
----
toString: String
----

NOTE: `toString` is part of Java's link:++https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#toString--++[Object Contract] for the string representation of an object, e.g. `TreeNode`.

`toString` simply returns the <<treeString, text representation of all nodes in the tree>>.

=== [[treeString]] Text Representation of All Nodes in Tree -- `treeString` Method

[source, scala]
----
treeString: String  // <1>
treeString(verbose: Boolean, addSuffix: Boolean = false): String
----
<1> Turns verbose flag on

`treeString` gives the string representation of all the nodes in the `TreeNode`.

[source, scala]
----
import org.apache.spark.sql.{functions => f}
val q = spark.range(10).withColumn("rand", f.rand())
val executedPlan = q.queryExecution.executedPlan

val output = executedPlan.treeString(verbose = true)

scala> println(output)
*(1) Project [id#0L, rand(6790207094253656854) AS rand#2]
+- *(1) Range (0, 10, step=1, splits=8)
----

[NOTE]
====
`treeString` is used when:

* `TreeNode` is requested for the <<numberedTreeString, numbered text representation>> and the <<toString, text representation>>

* `QueryExecution` is requested for link:spark-sql-QueryExecution.adoc#simpleString[simple], link:spark-sql-QueryExecution.adoc#toString[extended] and link:spark-sql-QueryExecution.adoc#stringWithStats[with statistics] text representations
====

=== [[verboseStringWithSuffix]] Verbose Description with Suffix -- `verboseStringWithSuffix` Method

[source, scala]
----
verboseStringWithSuffix: String
----

`verboseStringWithSuffix` simply returns <<verboseString, verbose description>>.

NOTE: `verboseStringWithSuffix` is used exclusively when `TreeNode` is requested to <<generateTreeString, generateTreeString>> (with `verbose` and `addSuffix` flags enabled).

=== [[generateTreeString]] Generating Text Representation of Inner and Regular Child Nodes -- `generateTreeString` Method

[source, scala]
----
generateTreeString(
  depth: Int,
  lastChildren: Seq[Boolean],
  builder: StringBuilder,
  verbose: Boolean,
  prefix: String = "",
  addSuffix: Boolean = false): StringBuilder
----

Internally, `generateTreeString` appends the following node descriptions per the `verbose` and `addSuffix` flags:

* <<verboseStringWithSuffix, verbose description with suffix>> when both are enabled (i.e. `verbose` and `addSuffix` flags are all `true`)

* <<verboseString, verbose description>> when `verbose` is enabled (i.e. `verbose` is `true` and `addSuffix` is `false`)

* <<simpleString, simple description>> when `verbose` is disabled (i.e. `verbose` is `false`)

In the end, `generateTreeString` calls itself recursively for the <<innerChildren, innerChildren>> and the <<children, child nodes>>.

NOTE: `generateTreeString` is used exclusively when `TreeNode` is requested for <<treeString, text representation of all nodes in the tree>>.

=== [[innerChildren]] Inner Child Nodes -- `innerChildren` Method

[source, scala]
----
innerChildren: Seq[TreeNode[_]]
----

`innerChildren` returns the inner nodes that should be shown as an inner nested tree of this node.

`innerChildren` simply returns an empty collection of `TreeNodes`.

NOTE: `innerChildren` is used when `TreeNode` is requested to <<generateTreeString, generate the text representation of inner and regular child nodes>>, <<allChildren, allChildren>> and <<getNodeNumbered, getNodeNumbered>>.

=== [[allChildren]] `allChildren` Property

[source, scala]
----
allChildren: Set[TreeNode[_]]
----

NOTE: `allChildren` is a Scala lazy value which is computed once when accessed and cached afterwards.

`allChildren`...FIXME

NOTE: `allChildren` is used when...FIXME

=== [[getNodeNumbered]] `getNodeNumbered` Internal Method

[source, scala]
----
getNodeNumbered(number: MutableInt): Option[TreeNode[_]]
----

`getNodeNumbered`...FIXME

NOTE: `getNodeNumbered` is used when...FIXME

=== [[foreach]] `foreach` Method

[source, scala]
----
foreach(f: BaseType => Unit): Unit
----

`foreach` applies the input function `f` to itself (`this`) first and then (recursively) to the <<children, children>>.

=== [[collect]] `collect` Method

[source, scala]
----
collect[B](pf: PartialFunction[BaseType, B]): Seq[B]
----

`collect`...FIXME

=== [[collectFirst]] `collectFirst` Method

[source, scala]
----
collectFirst[B](pf: PartialFunction[BaseType, B]): Option[B]
----

`collectFirst`...FIXME

=== [[collectLeaves]] `collectLeaves` Method

[source, scala]
----
collectLeaves(): Seq[BaseType]
----

`collectLeaves`...FIXME

=== [[find]] `find` Method

[source, scala]
----
find(f: BaseType => Boolean): Option[BaseType]
----

`find`...FIXME

=== [[flatMap]] `flatMap` Method

[source, scala]
----
flatMap[A](f: BaseType => TraversableOnce[A]): Seq[A]
----

`flatMap`...FIXME

=== [[foreachUp]] `foreachUp` Method

[source, scala]
----
foreachUp(f: BaseType => Unit): Unit
----

`foreachUp`...FIXME

=== [[map]] `map` Method

[source, scala]
----
map[A](f: BaseType => A): Seq[A]
----

`map`...FIXME

=== [[mapChildren]] `mapChildren` Method

[source, scala]
----
mapChildren(f: BaseType => BaseType): BaseType
----

`mapChildren`...FIXME

=== [[transform]] `transform` Method

[source, scala]
----
transform(rule: PartialFunction[BaseType, BaseType]): BaseType
----

`transform`...FIXME

=== [[transformDown]] Transforming Nodes Downwards -- `transformDown` Method

[source, scala]
----
transformDown(rule: PartialFunction[BaseType, BaseType]): BaseType
----

`transformDown`...FIXME

=== [[transformUp]] `transformUp` Method

[source, scala]
----
transformUp(rule: PartialFunction[BaseType, BaseType]): BaseType
----

`transformUp`...FIXME

=== [[asCode]] `asCode` Method

[source, scala]
----
asCode: String
----

`asCode`...FIXME

=== [[prettyJson]] `prettyJson` Method

[source, scala]
----
prettyJson: String
----

`prettyJson`...FIXME

NOTE: `prettyJson` is used when...FIXME

=== [[toJSON]] `toJSON` Method

[source, scala]
----
toJSON: String
----

`toJSON`...FIXME

NOTE: `toJSON` is used when...FIXME

=== [[argString]] `argString` Method

[source, scala]
----
argString: String
----

`argString`...FIXME

NOTE: `argString` is used when...FIXME

=== [[nodeName]] `nodeName` Method

[source, scala]
----
nodeName: String
----

`nodeName` returns the name of the class with `Exec` suffix removed (that is used as a naming convention for the class name of <<spark-sql-SparkPlan.adoc#, physical operators>>).

NOTE: `nodeName` is used when `TreeNode` is requested for <<simpleString, simpleString>> and <<asCode, asCode>>.

=== [[fastEquals]] `fastEquals` Method

[source, scala]
----
fastEquals(other: TreeNode[_]): Boolean
----

`fastEquals`...FIXME

NOTE: `fastEquals` is used when...FIXME

=== [[shouldConvertToJson]] `shouldConvertToJson` Internal Method

[source, scala]
----
shouldConvertToJson(
  product: Product): Boolean
----

`shouldConvertToJson`...FIXME

NOTE: `shouldConvertToJson` is used when...FIXME
