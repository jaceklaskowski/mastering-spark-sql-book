title: ResolvedHint

# ResolvedHint Unary Logical Operator

`ResolvedHint` is a link:spark-sql-LogicalPlan.adoc#UnaryNode[unary logical operator] that...FIXME

`ResolvedHint` is <<creating-instance, created>> when...FIXME

[[output]]
When requested for link:spark-sql-catalyst-QueryPlan.adoc#output[output schema], `ResolvedHint` uses the output of the child logical operator.

[[doCanonicalize]]
`ResolvedHint` simply requests the <<child, child logical operator>> for the <<spark-sql-catalyst-QueryPlan.adoc#doCanonicalize, canonicalized version>>.

=== [[creating-instance]] Creating ResolvedHint Instance

`ResolvedHint` takes the following when created:

* [[child]] Child link:spark-sql-LogicalPlan.adoc[logical operator]
* [[hints]] link:spark-sql-HintInfo.adoc[Query hints]
