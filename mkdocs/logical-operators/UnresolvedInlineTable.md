title: UnresolvedInlineTable

# UnresolvedInlineTable Logical Operator

`UnresolvedInlineTable` is a <<spark-sql-LogicalPlan.adoc#UnaryNode, unary logical operator>> that represents an inline table (aka _virtual table_ in Apache Hive).

`UnresolvedInlineTable` is <<creating-instance, created>> when `AstBuilder` is requested to <<spark-sql-AstBuilder.adoc#visitInlineTable, parse an inline table>> in a SQL statement.

[source, scala]
----
// `tableAlias` undefined so columns default to `colN`
val q = sql("VALUES 1, 2, 3")
scala> println(q.queryExecution.logical.numberedTreeString)
00 'UnresolvedInlineTable [col1], [List(1), List(2), List(3)]

// CreateNamedStruct with `tableAlias`
val q = sql("VALUES (1, 'a'), (2, 'b') AS t1(a, b)")
scala> println(q.queryExecution.logical.numberedTreeString)
00 'SubqueryAlias t1
01 +- 'UnresolvedInlineTable [a, b], [List(1, a), List(2, b)]
----

[[resolved]]
`UnresolvedInlineTable` is never <<spark-sql-LogicalPlan.adoc#resolved, resolved>> (and is converted to a <<spark-sql-LogicalPlan-LocalRelation.adoc#, LocalRelation>> in <<spark-sql-Analyzer-ResolveInlineTables.adoc#, ResolveInlineTables>> logical resolution rule).

[[output]]
`UnresolvedInlineTable` uses no <<spark-sql-catalyst-QueryPlan.adoc#output, output schema attributes>>.

[[expressionsResolved]]
`UnresolvedInlineTable` uses `expressionsResolved` flag that is on (`true`) only when all the Catalyst expressions in the <<rows, rows>> are <<spark-sql-Expression.adoc#resolved, resolved>>.

=== [[creating-instance]] Creating UnresolvedInlineTable Instance

`UnresolvedInlineTable` takes the following when created:

* [[names]] Column names
* [[rows]] Rows (as <<spark-sql-Expression.adoc#, Catalyst expressions>> for every row, i.e. `Seq[Seq[Expression]]`)
