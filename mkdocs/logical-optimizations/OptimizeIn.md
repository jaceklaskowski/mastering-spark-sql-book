# OptimizeIn Logical Optimization

`OptimizeIn` is a <<spark-sql-Optimizer.adoc#batches, base logical optimization>> that <<apply, transforms logical plans with In predicate expressions>> as follows:

. Replaces an `In` expression that has an link:spark-sql-Expression-In.adoc#list[empty list] and the link:spark-sql-Expression-In.adoc#value[value] expression not link:spark-sql-Expression.adoc#nullable[nullable] to `false`

. Eliminates duplicates of link:spark-sql-Expression-Literal.adoc[Literal] expressions in an link:spark-sql-Expression-In.adoc[In] predicate expression that is link:spark-sql-Expression-In.adoc#inSetConvertible[inSetConvertible]

. Replaces an `In` predicate expression that is link:spark-sql-Expression-In.adoc#inSetConvertible[inSetConvertible] with link:spark-sql-Expression-InSet.adoc[InSet] expressions when the number of link:spark-sql-Expression-Literal.adoc[literal] expressions in the link:spark-sql-Expression-In.adoc#list[list] expression is greater than link:spark-sql-properties.adoc#spark.sql.optimizer.inSetConversionThreshold[spark.sql.optimizer.inSetConversionThreshold] internal configuration property (default: `10`)

`OptimizeIn` is part of the <<spark-sql-Optimizer.adoc#Operator_Optimization_before_Inferring_Filters, Operator Optimization before Inferring Filters>> fixed-point batch in the standard batches of the <<spark-sql-Optimizer.adoc#, Catalyst Optimizer>>.

`OptimizeIn` is simply a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

[source, scala]
----
// Use Catalyst DSL to define a logical plan

// HACK: Disable symbolToColumn implicit conversion
// It is imported automatically in spark-shell (and makes demos impossible)
// implicit def symbolToColumn(s: Symbol): org.apache.spark.sql.ColumnName
trait ThatWasABadIdea
implicit def symbolToColumn(ack: ThatWasABadIdea) = ack

import org.apache.spark.sql.catalyst.dsl.expressions._
import org.apache.spark.sql.catalyst.dsl.plans._

import org.apache.spark.sql.catalyst.plans.logical.LocalRelation
val rel = LocalRelation('a.int, 'b.int, 'c.int)

import org.apache.spark.sql.catalyst.expressions.{In, Literal}
val plan = rel
  .where(In('a, Seq[Literal](1, 2, 3)))
  .analyze
scala> println(plan.numberedTreeString)
00 Filter a#6 IN (1,2,3)
01 +- LocalRelation <empty>, [a#6, b#7, c#8]

// In --> InSet
spark.conf.set("spark.sql.optimizer.inSetConversionThreshold", 0)

import org.apache.spark.sql.catalyst.optimizer.OptimizeIn
val optimizedPlan = OptimizeIn(plan)
scala> println(optimizedPlan.numberedTreeString)
00 Filter a#6 INSET (1,2,3)
01 +- LocalRelation <empty>, [a#6, b#7, c#8]
----

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply`...FIXME
