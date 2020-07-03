# PhysicalAggregation -- Scala Extractor for Destructuring Aggregate Logical Operators

`PhysicalAggregation` is a Scala extractor to <<unapply, destructure an Aggregate logical operator>> into a four-element tuple with the following elements:

. Grouping link:spark-sql-Expression-NamedExpression.adoc[named expressions]

. link:spark-sql-Expression-AggregateExpression.adoc[AggregateExpressions]

. Result link:spark-sql-Expression-NamedExpression.adoc[named expressions]

. Child link:spark-sql-LogicalPlan.adoc[logical operator]

[[ReturnType]]
.ReturnType
[source, scala]
----
(Seq[NamedExpression], Seq[AggregateExpression], Seq[NamedExpression], LogicalPlan)
----

TIP: See the document about http://docs.scala-lang.org/tutorials/tour/extractor-objects.html[Scala extractor objects].

=== [[unapply]] Destructuring Aggregate Logical Operator -- `unapply` Method

[source, scala]
----
type ReturnType =
  (Seq[NamedExpression], Seq[AggregateExpression], Seq[NamedExpression], LogicalPlan)

unapply(a: Any): Option[ReturnType]
----

`unapply` destructures the input `a` link:spark-sql-LogicalPlan-Aggregate.adoc[Aggregate] logical operator into a four-element <<ReturnType, ReturnType>>.

[NOTE]
====
`unapply` is used when...FIXME
====
