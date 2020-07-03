# PhysicalOperation -- Scala Extractor for Destructuring Logical Query Plans

`PhysicalOperation` is a Scala extractor to <<unapply, destructure a logical query plan>> into a tuple with the following elements:

. link:spark-sql-Expression-NamedExpression.adoc[Named expressions] (aka _projects_)

. link:spark-sql-Expression.adoc[Expressions] (aka _filters_)

. link:spark-sql-LogicalPlan.adoc[Logical operator] (aka _leaf node_)

[[ReturnType]]
.ReturnType
[source, scala]
----
(Seq[NamedExpression], Seq[Expression], LogicalPlan)
----

The following idiom is often used in `Strategy` implementations (e.g. link:hive/HiveTableScans.adoc#apply[HiveTableScans], link:spark-sql-SparkStrategy-InMemoryScans.adoc#apply[InMemoryScans], link:spark-sql-SparkStrategy-DataSourceStrategy.adoc#apply[DataSourceStrategy], <<FileSourceStrategy, FileSourceStrategy>>):

[source, scala]
----
def apply(plan: LogicalPlan): Seq[SparkPlan] = plan match {
  case PhysicalOperation(projections, predicates, plan) =>
    // do something
  case _ => Nil
}
----

Whenever used to pattern match to a `LogicalPlan`, ``PhysicalOperation``'s `unapply` is called.

=== [[unapply]] `unapply` Method

[source, scala]
----
type ReturnType = (Seq[NamedExpression], Seq[Expression], LogicalPlan)

unapply(plan: LogicalPlan): Option[ReturnType]
----

`unapply`...FIXME

NOTE: `unapply` is _almost_ <<collectProjectsAndFilters, collectProjectsAndFilters>> method itself (with some manipulations of the return value).

[NOTE]
====
`unapply` is used when...FIXME
====
