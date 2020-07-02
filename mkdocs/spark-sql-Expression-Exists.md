title: Exists

# Exists -- Correlated Predicate Subquery Expression

`Exists` is a link:spark-sql-Expression-SubqueryExpression.adoc[SubqueryExpression] and a link:spark-sql-Expression.adoc#Predicate[predicate expression] (i.e. the result link:spark-sql-Expression.adoc#dataType[data type] is always link:spark-sql-DataType.adoc#BooleanType[boolean]).

`Exists` is <<creating-instance, created>> when:

. `ResolveSubquery` is requested to link:spark-sql-Analyzer-ResolveSubquery.adoc#resolveSubQueries[resolveSubQueries]

. `PullupCorrelatedPredicates` is requested to link:spark-sql-PullupCorrelatedPredicates.adoc#rewriteSubQueries[rewriteSubQueries]

. `AstBuilder` is requested to link:spark-sql-AstBuilder.adoc#visitExists[visitExists] (in SQL statements)

[[Unevaluable]]
`Exists` link:spark-sql-Expression.adoc#Unevaluable[cannot be evaluated], i.e. produce a value given an internal row.

[[eval]][[doGenCode]]
When requested to evaluate or `doGenCode`, `Exists` simply reports a `UnsupportedOperationException`.

```
Cannot evaluate expression: [this]
```

[[nullable]]
`Exists` is never link:spark-sql-Expression.adoc#nullable[nullable].

[[toString]]
`Exists` uses the following *text representation*:

```
exists#[exprId] [conditionString]
```

[[canonicalized]]
When requested for a link:spark-sql-BroadcastMode.adoc#canonicalized[canonicalized] version, `Exists` <<creating-instance, creates>> a new instance with...FIXME

=== [[creating-instance]] Creating Exists Instance

`Exists` takes the following when created:

* [[plan]] Subquery link:spark-sql-LogicalPlan.adoc[logical plan]
* [[children]] Child link:spark-sql-Expression.adoc[expressions]
* [[exprId]] `ExprId`
