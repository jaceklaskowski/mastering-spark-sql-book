title: ListQuery

# ListQuery Subquery Expression

`ListQuery` is a link:spark-sql-Expression-SubqueryExpression.adoc[SubqueryExpression] that represents SQL's link:spark-sql-AstBuilder.adoc#withPredicate[IN predicate with a subquery], e.g. `NOT? IN '(' query ')'`.

[[Unevaluable]]
`ListQuery` link:spark-sql-Expression.adoc#Unevaluable[cannot be evaluated] and produce a value given an internal row.

[[resolved]]
`ListQuery` is link:spark-sql-Expression-SubqueryExpression.adoc#resolved[resolved] when:

. link:spark-sql-Expression.adoc#childrenResolved[Children are resolved]

. <<plan, Subquery logical plan>> is link:spark-sql-LogicalPlan.adoc#resolved[resolved]

. There is at least one <<childOutputs, child output attribute>>

=== [[creating-instance]] Creating ListQuery Instance

`ListQuery` takes the following when created:

* [[plan]] Subquery link:spark-sql-LogicalPlan.adoc[logical plan]
* [[children]] Child link:spark-sql-Expression.adoc[expressions]
* [[exprId]] Expression ID (as `ExprId` and defaults to a link:spark-sql-Expression-NamedExpression.adoc#newExprId[new ExprId])
* [[childOutputs]] Child output link:spark-sql-Expression-Attribute.adoc[attributes]
