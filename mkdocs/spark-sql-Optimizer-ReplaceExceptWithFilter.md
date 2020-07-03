# ReplaceExceptWithFilter Logical Optimization Rule -- Rewriting Except (DISTINCT) Operators

`ReplaceExceptWithFilter` is a link:spark-sql-catalyst-Rule.adoc[Catalyst rule] for transforming link:spark-sql-LogicalPlan.adoc[logical plans] (i.e. `Rule[LogicalPlan]`).

[[apply]]
When link:spark-sql-catalyst-Rule.adoc#apply[executed], `ReplaceExceptWithFilter` transforms an link:spark-sql-LogicalPlan-Except.adoc[Except (distinct)] logical operator to...FIXME

`ReplaceExceptWithFilter` is a part of the link:spark-sql-Optimizer.adoc#Replace-Operators[Replace Operators] fixed-point rule batch of the base link:spark-sql-Optimizer.adoc[Catalyst Optimizer].

`ReplaceExceptWithFilter` can be turned off and on based on link:spark-sql-properties.adoc#spark.sql.optimizer.replaceExceptWithFilter[spark.sql.optimizer.replaceExceptWithFilter] configuration property.

[[demo]]
.Demo: ReplaceExceptWithFilter
```
import org.apache.spark.sql.catalyst.dsl.expressions._
import org.apache.spark.sql.catalyst.dsl.plans._

// Using hacks to disable two Catalyst DSL implicits
implicit def symbolToColumn(ack: ThatWasABadIdea) = ack
implicit class StringToColumn(val sc: StringContext) {}

import org.apache.spark.sql.catalyst.plans.logical.LocalRelation
val t1 = LocalRelation('a.int, 'b.int)
val t2 = t1.where('a > 5)

val plan = t1.except(t2, isAll = false)

import org.apache.spark.sql.catalyst.optimizer.ReplaceExceptWithFilter
val optimizedPlan = ReplaceExceptWithFilter(plan)
scala> println(optimizedPlan.numberedTreeString)
00 'Distinct
01 +- 'Filter NOT coalesce(('a > 5), false)
02    +- LocalRelation <empty>, [a#12, b#13]
```

=== [[isEligible]] `isEligible` Internal Predicate

[source, scala]
----
isEligible(
  left: LogicalPlan,
  right: LogicalPlan): Boolean
----

`isEligible` is positive (`true`) when the right logical operator is a link:spark-sql-LogicalPlan-Project.adoc[Project] with a link:spark-sql-LogicalPlan-Filter.adoc[Filter] child operator or simply a link:spark-sql-LogicalPlan-Filter.adoc[Filter] operator itself and <<verifyConditions, verifyConditions>>.

Otherwise, `isEligible` is negative (`false`).

NOTE: `isEligible` is used when `ReplaceExceptWithFilter` is <<apply, executed>>.

=== [[verifyConditions]] `verifyConditions` Internal Predicate

[source, scala]
----
verifyConditions(
  left: LogicalPlan,
  right: LogicalPlan): Boolean
----

`verifyConditions` is positive (`true`) when all of the following hold:

* FIXME

Otherwise, `verifyConditions` is negative (`false`).

NOTE: `verifyConditions` is used when `ReplaceExceptWithFilter` is <<apply, executed>> (when requested to <<isEligible, isEligible>>).
