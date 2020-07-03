# PredicateHelper Scala Trait

`PredicateHelper` defines the <<methods, methods>> that are used to work with predicates (mainly).

[[methods]]
.PredicateHelper's Methods
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| <<splitConjunctivePredicates, splitConjunctivePredicates>>
|

| <<splitDisjunctivePredicates, splitDisjunctivePredicates>>
|

| <<replaceAlias, replaceAlias>>
|

| <<canEvaluate, canEvaluate>>
|

| <<canEvaluateWithinJoin, canEvaluateWithinJoin>>
|
|===

=== [[splitConjunctivePredicates]] Splitting Conjunctive Predicates -- `splitConjunctivePredicates` Method

[source, scala]
----
splitConjunctivePredicates(condition: Expression): Seq[Expression]
----

`splitConjunctivePredicates` takes the input condition link:spark-sql-Expression.adoc[expression] and splits it to two expressions if they are children of a `And` binary expression.

`splitConjunctivePredicates` splits the child expressions recursively down the child expressions until no conjunctive `And` binary expressions exist.

=== [[splitDisjunctivePredicates]] `splitDisjunctivePredicates` Method

[source, scala]
----
splitDisjunctivePredicates(condition: Expression): Seq[Expression]
----

`splitDisjunctivePredicates`...FIXME

NOTE: `splitDisjunctivePredicates` is used when...FIXME

=== [[replaceAlias]] `replaceAlias` Method

[source, scala]
----
replaceAlias(
  condition: Expression,
  aliases: AttributeMap[Expression]): Expression
----

`replaceAlias`...FIXME

NOTE: `replaceAlias` is used when...FIXME

=== [[canEvaluate]] `canEvaluate` Method

[source, scala]
----
canEvaluate(expr: Expression, plan: LogicalPlan): Boolean
----

`canEvaluate`...FIXME

NOTE: `canEvaluate` is used when...FIXME

=== [[canEvaluateWithinJoin]] `canEvaluateWithinJoin` Method

[source, scala]
----
canEvaluateWithinJoin(expr: Expression): Boolean
----

`canEvaluateWithinJoin` indicates whether a link:spark-sql-Expression.adoc[Catalyst expression] _can be evaluated within a join_, i.e. when one of the following conditions holds:

* Expression is link:spark-sql-Expression.adoc#deterministic[deterministic]

* Expression is not `Unevaluable`, `ListQuery` or `Exists`

* Expression is a `SubqueryExpression` with no child expressions

* Expression is a `AttributeReference`

* Any expression with child expressions that meet one of the above conditions

[NOTE]
====
`canEvaluateWithinJoin` is used when:

* `PushPredicateThroughJoin` logical optimization rule is link:spark-sql-Optimizer-PushPredicateThroughJoin.adoc#apply[executed]

* `ReorderJoin` logical optimization rule does link:spark-sql-Optimizer-ReorderJoin.adoc#createOrderedJoin[createOrderedJoin]
====
