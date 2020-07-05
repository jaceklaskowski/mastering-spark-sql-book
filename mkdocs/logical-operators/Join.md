title: Join

# Join Logical Operator

`Join` is a link:spark-sql-LogicalPlan.adoc#BinaryNode[binary logical operator], i.e. works with two logical operators. `Join` has a join type and an optional expression condition for the join.

`Join` is <<creating-instance, created>> when...FIXME

NOTE: CROSS JOIN is just an INNER JOIN with no join condition.

[[output]]
`Join` has link:spark-sql-catalyst-QueryPlan.adoc#output[output schema attributes]...FIXME

=== [[creating-instance]] Creating Join Instance

`Join` takes the following when created:

* [[left]] link:spark-sql-LogicalPlan.adoc[Logical plan] of the left side
* [[right]] link:spark-sql-LogicalPlan.adoc[Logical plan] of the right side
* [[joinType]] link:spark-sql-joins.adoc#join-types[Join type]
* [[condition]] Join condition (if available) as a link:spark-sql-Expression.adoc[Catalyst expression]
