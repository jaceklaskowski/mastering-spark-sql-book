title: First

# First Aggregate Function Expression

`First` is a <<spark-sql-Expression-DeclarativeAggregate.adoc#, DeclarativeAggregate>> function expression that is <<creating-instance, created>> when:

* `AstBuilder` is requested to <<spark-sql-AstBuilder.adoc#visitFirst, parse a FIRST statement>>

* <<spark-sql-functions.adoc#first, first>> standard function is used

* `first` and `first_value` <<spark-sql-FunctionRegistry.adoc#, SQL functions>> are used

[source, scala]
----
val sqlText = "FIRST (organizationName IGNORE NULLS)"
val e = spark.sessionState.sqlParser.parseExpression(sqlText)
scala> :type e
org.apache.spark.sql.catalyst.expressions.Expression

import org.apache.spark.sql.catalyst.expressions.aggregate.AggregateExpression
val aggExpr = e.asInstanceOf[AggregateExpression]

import org.apache.spark.sql.catalyst.expressions.aggregate.First
val f = aggExpr.aggregateFunction
scala> println(f.simpleString)
first('organizationName) ignore nulls
----

[[evaluateExpression]]
When requested to <<spark-sql-Expression-DeclarativeAggregate.adoc#evaluateExpression, evaluate>> (and return the final value), `First` simply returns a <<spark-sql-Expression-AttributeReference.adoc#, AttributeReference>> (with `first` name and the <<spark-sql-Expression.adoc#dataType, data type>> of the <<child, child>> expression).

[[catalyst-dsl]]
TIP: Use <<first, first>> operator from the <<spark-sql-catalyst-dsl.adoc#, Catalyst DSL>> to create an `First` aggregate function expression, e.g. for testing or Spark SQL internals exploration.

=== [[first]] Catalyst DSL -- `first` Operator

[source, scala]
----
first(e: Expression): Expression
----

`first` <<creating-instance, creates>> a `First` expression and requests it to <<spark-sql-Expression-AggregateFunction.adoc#toAggregateExpression, convert to a AggregateExpression>>.

[source, scala]
----
import org.apache.spark.sql.catalyst.dsl.expressions._
val e = first('orgName)

scala> println(e.numberedTreeString)
00 first('orgName, false)
01 +- first('orgName)()
02    :- 'orgName
03    +- false

import org.apache.spark.sql.catalyst.expressions.aggregate.AggregateExpression
val aggExpr = e.asInstanceOf[AggregateExpression]

import org.apache.spark.sql.catalyst.expressions.aggregate.First
val f = aggExpr.aggregateFunction
scala> println(f.simpleString)
first('orgName)()
----

=== [[creating-instance]] Creating First Instance

`First` takes the following when created:

* [[child]] Child <<spark-sql-Expression.adoc#, expression>>
* [[ignoreNullsExpr]] `ignoreNullsExpr` flag <<spark-sql-Expression.adoc#, expression>>
