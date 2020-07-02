title: Star

# Star Expression Contract

`Star` is a <<contract, contract>> of link:spark-sql-Expression.adoc#LeafExpression[leaf] and link:spark-sql-Expression-NamedExpression.adoc[named expressions] that...FIXME

[[contract]]
[source, scala]
----
package org.apache.spark.sql.catalyst.analysis

abstract class Star extends LeafExpression with NamedExpression {
  // only required methods that have no implementation
  def expand(input: LogicalPlan, resolver: Resolver): Seq[NamedExpression]
}
----

.Star Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| `expand`
a| [[expand]] Used exclusively when `ResolveReferences` logical resolution rule is requested to expand `Star` expressions in the following logical operators:

* link:spark-sql-Analyzer-ResolveReferences.adoc#apply[ScriptTransformation]

* link:spark-sql-Analyzer-ResolveReferences.adoc#buildExpandedProjectList[Project and Aggregate]
|===

[[implementations]]
.Stars
[cols="1,2",options="header",width="100%"]
|===
| Star
| Description

| [[ResolvedStar]] link:spark-sql-Expression-ResolvedStar.adoc[ResolvedStar]
|

| [[UnresolvedRegex]] link:spark-sql-Expression-UnresolvedRegex.adoc[UnresolvedRegex]
|

| [[UnresolvedStar]] link:spark-sql-Expression-UnresolvedStar.adoc[UnresolvedStar]
|
|===
