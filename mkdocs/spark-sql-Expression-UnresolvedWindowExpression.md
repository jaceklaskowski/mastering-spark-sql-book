title: UnresolvedWindowExpression

# UnresolvedWindowExpression Unevaluable Expression -- WindowExpression With Unresolved Window Specification Reference

`UnresolvedWindowExpression` is an <<spark-sql-Expression.adoc#Unevaluable, unevaluable expression>> that represents...FIXME

NOTE: An <<spark-sql-Expression.adoc#Unevaluable, unevaluable expression>> cannot be evaluated to produce a value (neither in <<spark-sql-Expression.adoc#eval, interpreted>> nor <<spark-sql-Expression.adoc#doGenCode, code-generated>> expression evaluations) and has to be resolved (replaced) to some other expressions or logical operators at <<spark-sql-QueryExecution.adoc#analyzed, analysis>> or <<spark-sql-QueryExecution.adoc#optimizedPlan, optimization>> phases or they fail analysis.

`UnresolvedWindowExpression` is <<creating-instance, created>> when:

* FIXME

[[child]]
`UnresolvedWindowExpression` is created to represent a `child` link:spark-sql-Expression.adoc[expression] and `WindowSpecReference` (with an identifier for the window reference) when `AstBuilder` link:spark-sql-AstBuilder.adoc#visitFunctionCall-UnresolvedWindowExpression[parses a function evaluated in a windowed context with a `WindowSpecReference`].

`UnresolvedWindowExpression` is resolved to a <<WindowExpression, WindowExpression>> when `Analyzer` link:spark-sql-Analyzer.adoc#WindowsSubstitution[resolves `UnresolvedWindowExpressions`].

[source, scala]
----
import spark.sessionState.sqlParser

scala> sqlParser.parseExpression("foo() OVER windowSpecRef")
res1: org.apache.spark.sql.catalyst.expressions.Expression = unresolvedwindowexpression('foo(), WindowSpecReference(windowSpecRef))
----

[[properties]]
.UnresolvedWindowExpression's Properties
[width="100%",cols="1,2",options="header"]
|===
| Name
| Description

| `dataType`
| Reports a `UnresolvedException`

| `foldable`
| Reports a `UnresolvedException`

| `nullable`
| Reports a `UnresolvedException`

| `resolved`
| Disabled (i.e. `false`)
|===
