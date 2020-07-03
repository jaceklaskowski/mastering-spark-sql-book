# CreateStruct Function Builder

`CreateStruct` is a <<spark-sql-FunctionRegistry.adoc#expressions, function builder>> (e.g. `Seq[Expression] => Expression`) that can <<apply, create CreateNamedStruct expressions>> and is the <<registryEntry, metadata>> of the <<spark-sql-FunctionRegistry.adoc#struct, struct>> function.

=== [[registryEntry]] Metadata of struct Function -- `registryEntry` Property

[source, scala]
----
registryEntry: (String, (ExpressionInfo, FunctionBuilder))
----

`registryEntry`...FIXME

NOTE: `registryEntry` is used exclusively when `FunctionRegistry` is requested for the <<spark-sql-FunctionRegistry.adoc#expressions, function expression registry>>.

=== [[apply]] Creating CreateNamedStruct Expression -- `apply` Method

[source, scala]
----
apply(children: Seq[Expression]): CreateNamedStruct
----

NOTE: `apply` is part of Scala's https://www.scala-lang.org/api/2.11.12/index.html#scala.Function1[scala.Function1] contract to create a function of one parameter (e.g. `Seq[Expression]`).

`apply` creates a <<spark-sql-Expression-CreateNamedStruct.adoc#creating-instance, CreateNamedStruct>> expression with the input `children` <<spark-sql-Expression.adoc#, expressions>> as follows:

* For <<spark-sql-Expression-NamedExpression.adoc#, NamedExpression>> expressions that are <<spark-sql-Expression.adoc#resolved, resolved>>, `apply` creates a pair of a <<spark-sql-Expression-Literal.adoc#apply, Literal>> expression (with the <<spark-sql-Expression-NamedExpression.adoc#name, name>> of the `NamedExpression`) and the `NamedExpression` itself

* For <<spark-sql-Expression-NamedExpression.adoc#, NamedExpression>> expressions that are not <<spark-sql-Expression.adoc#resolved, resolved>> yet, `apply` creates a pair of a `NamePlaceholder` expression and the `NamedExpression` itself

* For all other <<spark-sql-Expression.adoc#, expressions>>, `apply` creates a pair of a <<spark-sql-Expression-Literal.adoc#apply, Literal>> expression (with the value as `col[index]`) and the `Expression` itself

[NOTE]
====
`apply` is used when:

* `ResolveReferences` logical resolution rule is requested to <<spark-sql-Analyzer-ResolveReferences.adoc#expandStarExpression, expandStarExpression>>

* `InConversion` type coercion rule is requested to <<spark-sql-Analyzer-TypeCoercionRule-InConversion.adoc#coerceTypes, coerceTypes>>

* `ExpressionEncoder` is requested to <<spark-sql-ExpressionEncoder.adoc#tuple, create an ExpressionEncoder for a tuple>>

* `Stack` generator expression is requested to <<spark-sql-Expression-Stack.adoc#doGenCode, generate a Java source code>>

* `AstBuilder` is requested to parse a <<spark-sql-AstBuilder.adoc#visitStruct, struct>> and <<spark-sql-AstBuilder.adoc#visitRowConstructor, row constructor>>

* `ColumnStat` is requested to <<spark-sql-ColumnStat.adoc#statExprs, statExprs>>

* `KeyValueGroupedDataset` is requested to <<spark-sql-KeyValueGroupedDataset.adoc#aggUntyped, aggUntyped>> (when <<spark-sql-KeyValueGroupedDataset.adoc#agg, KeyValueGroupedDataset.agg>> typed operator is used)

* <<spark-sql-dataset-operators.adoc#joinWith, Dataset.joinWith>> typed transformation is used

* <<spark-sql-functions.adoc#struct, struct>> standard function is used

* `SimpleTypedAggregateExpression` expression is requested for the <<spark-sql-Expression-SimpleTypedAggregateExpression.adoc#evaluateExpression, evaluateExpression>> and <<spark-sql-Expression-SimpleTypedAggregateExpression.adoc#resultObjToRow, resultObjToRow>>
====
