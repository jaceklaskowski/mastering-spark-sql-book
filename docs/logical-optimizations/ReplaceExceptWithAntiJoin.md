# ReplaceExceptWithAntiJoin Logical Optimization Rule -- Rewriting Except (DISTINCT) Operators

`ReplaceExceptWithAntiJoin` is a catalyst/Rule.md[Catalyst rule] for transforming spark-sql-LogicalPlan.md[logical plans] (i.e. `Rule[LogicalPlan]`).

[[apply]]
When catalyst/Rule.md#apply[executed], `ReplaceExceptWithAntiJoin` transforms an Except.md[Except (distinct)] logical operator to a `Distinct` unary logical operator with a left-anti Join.md[Join] operator. The output columns of the left and right child logical operators of the `Except` operator are used to build a logical `AND` join condition of [EqualNullSafe](../expressions/EqualNullSafe.md) expressions.

`ReplaceExceptWithAntiJoin` requires that the number of columns of the left- and right-side of the `Except` operator are the same or throws an `AssertionError`.

`ReplaceExceptWithAntiJoin` is a part of the [Replace Operators](../catalyst/Optimizer.md#Replace-Operators) fixed-point rule batch of the base [Logical Optimizer](../catalyst/Optimizer.md).

[[demo]]
.Demo: ReplaceExceptWithAntiJoin
```
import org.apache.spark.sql.catalyst.dsl.expressions._
import org.apache.spark.sql.catalyst.dsl.plans._

// Using hacks to disable two Catalyst DSL implicits
implicit def symbolToColumn(ack: ThatWasABadIdea) = ack
implicit class StringToColumn(val sc: StringContext) {}

import org.apache.spark.sql.catalyst.plans.logical.LocalRelation
val t1 = LocalRelation('a.int, 'b.int)
val t2 = LocalRelation('C.int, 'D.int)

val plan = t1.except(t2, isAll = false)

import org.apache.spark.sql.catalyst.optimizer.ReplaceExceptWithAntiJoin
val optimizedPlan = ReplaceExceptWithAntiJoin(plan)
scala> println(optimizedPlan.numberedTreeString)
00 Distinct
01 +- Join LeftAnti, ((a#14 <=> C#18) && (b#15 <=> D#19))
02    :- LocalRelation <empty>, [a#14, b#15]
03    +- LocalRelation <empty>, [C#18, D#19]
```
