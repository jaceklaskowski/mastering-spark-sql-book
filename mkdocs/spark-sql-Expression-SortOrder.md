title: SortOrder

# SortOrder Unevaluable Unary Expression

`SortOrder` is a <<spark-sql-Expression-UnaryExpression.adoc#, unary expression>> that represents the following operators in a logical plan:

* `AstBuilder` is requested to <<spark-sql-AstBuilder.adoc#visitSortItem, parse ORDER BY or SORT BY sort specifications>>

* <<spark-sql-column-operators.adoc#asc, Column.asc>>, <<spark-sql-column-operators.adoc#asc_nulls_first, Column.asc_nulls_first>>, <<spark-sql-column-operators.adoc#asc_nulls_last, Column.asc_nulls_last>>, <<spark-sql-column-operators.adoc#desc, Column.desc>>, <<spark-sql-column-operators.adoc#desc_nulls_first, Column.desc_nulls_first>>, and <<spark-sql-column-operators.adoc#desc_nulls_last, Column.desc_nulls_last>> operators are used

`SortOrder` is used to specify the <<spark-sql-SparkPlan.adoc#, output data ordering requirements>> of a physical operator.

`SortOrder` is an <<spark-sql-Expression.adoc#Unevaluable, unevaluable expression>> and cannot be evaluated (i.e. produce a value given an internal row).

NOTE: An <<spark-sql-Expression.adoc#Unevaluable, unevaluable expression>> cannot be evaluated to produce a value (neither in <<spark-sql-Expression.adoc#eval, interpreted>> nor <<spark-sql-Expression.adoc#doGenCode, code-generated>> expression evaluations) and has to be resolved (replaced) to some other expressions or logical operators at <<spark-sql-QueryExecution.adoc#analyzed, analysis>> or <<spark-sql-QueryExecution.adoc#optimizedPlan, optimization>> phases or they fail analysis.

[[foldable]]
`SortOrder` is never <<spark-sql-Expression.adoc#foldable, foldable>> (as an unevaluable expression with no evaluation).

[[catalyst-dsl]]
TIP: Use <<asc, asc>>, <<asc_nullsLast, asc_nullsLast>>, <<desc, desc>> or <<desc_nullsFirst, desc_nullsFirst>> operators from the <<spark-sql-catalyst-dsl.adoc#, Catalyst DSL>> to create a `SortOrder` expression, e.g. for testing or Spark SQL internals exploration.

NOTE: <<spark-sql-dataset-operators.adoc#repartitionByRange, Dataset.repartitionByRange>>, <<spark-sql-dataset-operators.adoc#sortWithinPartitions, Dataset.sortWithinPartitions>>, <<spark-sql-dataset-operators.adoc#sort, Dataset.sort>> and <<spark-sql-WindowSpec.adoc#orderBy, WindowSpec.orderBy>> default to <<Ascending, Ascending>> sort direction.

=== [[apply]] Creating SortOrder Instance -- `apply` Factory Method

[source, scala]
----
apply(
  child: Expression,
  direction: SortDirection,
  sameOrderExpressions: Set[Expression] = Set.empty): SortOrder
----

`apply` is a convenience method to create a <<SortOrder, SortOrder>> with the `defaultNullOrdering` of the <<SortDirection, SortDirection>>.

NOTE: `apply` is used exclusively in link:spark-sql-functions-datetime.adoc#window[window] function.

=== [[asc]][[asc_nullsLast]][[desc]][[desc_nullsFirst]] Catalyst DSL -- `asc`, `asc_nullsLast`, `desc` and `desc_nullsFirst` Operators

[source, scala]
----
asc: SortOrder
asc_nullsLast: SortOrder
desc: SortOrder
desc_nullsFirst: SortOrder
----

`asc`, `asc_nullsLast`, `desc` and `desc_nullsFirst` <<creating-instance, create>> a `SortOrder` expression with the `Ascending` or `Descending` sort direction, respectively.

[source, scala]
----
import org.apache.spark.sql.catalyst.dsl.expressions._
val sortNullsLast = 'id.asc_nullsLast
scala> println(sortNullsLast.sql)
`id` ASC NULLS LAST
----

=== [[creating-instance]] Creating SortOrder Instance

`SortOrder` takes the following when created:

* [[child]] Child <<spark-sql-Expression.adoc#, expression>>
* [[direction]] <<SortDirection, SortDirection>>
* [[nullOrdering]] `NullOrdering`
* [[sameOrderExpressions]] "Same Order" <<spark-sql-Expression.adoc#, expressions>>

=== [[SortDirection]] SortDirection Contract

`SortDirection` is the <<SortDirection-contract, base>> of <<SortDirection-extensions, sort directions>>.

[[SortDirection-contract]]
.SortDirection Contract
[cols="1m,2",options="header",width="100%"]
|===
| Method
| Description

| defaultNullOrdering
a| [[defaultNullOrdering]]

[source, scala]
----
defaultNullOrdering: NullOrdering
----

Used when...FIXME

| sql
a| [[sql]]

[source, scala]
----
sql: String
----

Used when...FIXME
|===

==== [[SortDirection-extensions]][[Ascending]][[Descending]] Ascending and Descending Sort Directions

There are two <<SortDirection, sorting directions>> available, i.e. `Ascending` and `Descending`.
