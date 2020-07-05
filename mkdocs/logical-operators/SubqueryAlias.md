title: SubqueryAlias

# SubqueryAlias Unary Logical Operator

`SubqueryAlias` is a <<spark-sql-LogicalPlan.adoc#UnaryNode, unary logical operator>> that represents an *aliased subquery* (i.e. the <<child, child>> logical query plan with the <<alias, alias>> in the <<output, output schema>>).

`SubqueryAlias` is <<creating-instance, created>> when:

* `AstBuilder` is requested to parse a <<spark-sql-AstBuilder.adoc#visitNamedQuery, named>> or <<spark-sql-AstBuilder.adoc#visitAliasedQuery, aliased>> query, <<spark-sql-AstBuilder.adoc#aliasPlan, aliased query plan>> and <<spark-sql-AstBuilder.adoc#mayApplyAliasPlan, mayApplyAliasPlan>> in a SQL statement

* <<spark-sql-dataset-operators.adoc#as, Dataset.as>> operator is used

* `SessionCatalog` is requested to <<spark-sql-SessionCatalog.adoc#lookupRelation, find a table or view in catalogs>>

* `RewriteCorrelatedScalarSubquery` logical optimization is requested to <<spark-sql-Optimizer-RewriteCorrelatedScalarSubquery.adoc#constructLeftJoins, constructLeftJoins>> (when <<spark-sql-Optimizer-RewriteCorrelatedScalarSubquery.adoc#apply, applied>> to <<spark-sql-LogicalPlan-Aggregate.adoc#, Aggregate>>, <<spark-sql-LogicalPlan-Project.adoc#, Project>> or <<spark-sql-LogicalPlan-Filter.adoc#, Filter>> logical operators with correlated scalar subqueries)

[[doCanonicalize]]
`SubqueryAlias` simply requests the <<child, child logical operator>> for the <<spark-sql-catalyst-QueryPlan.adoc#doCanonicalize, canonicalized version>>.

[[output]]
When requested for <<spark-sql-catalyst-QueryPlan.adoc#output, output schema attributes>>, `SubqueryAlias` requests the <<child, child>> logical operator for them and adds the <<alias, alias>> as a <<spark-sql-Expression-Attribute.adoc#withQualifier, qualifier>>.

NOTE: <<spark-sql-Optimizer-EliminateSubqueryAliases.adoc#, EliminateSubqueryAliases>> logical optimization eliminates (removes) `SubqueryAlias` operators from a logical query plan.

NOTE: <<spark-sql-Optimizer-RewriteCorrelatedScalarSubquery.adoc#, RewriteCorrelatedScalarSubquery>> logical optimization rewrites correlated scalar subqueries with `SubqueryAlias` operators.

=== [[catalyst-dsl]][[subquery]][[as]] Catalyst DSL -- `subquery` And `as` Operators

[source, scala]
----
as(alias: String): LogicalPlan
subquery(alias: Symbol): LogicalPlan
----

<<spark-sql-catalyst-dsl.adoc#subquery, subquery>> and <<spark-sql-catalyst-dsl.adoc#as, as>> operators in xref:spark-sql-catalyst-dsl.adoc[Catalyst DSL] create a <<creating-instance, SubqueryAlias>> logical operator, e.g. for testing or Spark SQL internals exploration.

[source, scala]
----
import org.apache.spark.sql.catalyst.dsl.plans._
val t1 = table("t1")

val plan = t1.subquery('a)
scala> println(plan.numberedTreeString)
00 'SubqueryAlias a
01 +- 'UnresolvedRelation `t1`

val plan = t1.as("a")
scala> println(plan.numberedTreeString)
00 'SubqueryAlias a
01 +- 'UnresolvedRelation `t1`
----

=== [[creating-instance]] Creating SubqueryAlias Instance

`SubqueryAlias` takes the following when created:

* [[alias]] Alias
* [[child]] Child <<spark-sql-LogicalPlan.adoc#, logical plan>>
