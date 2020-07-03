# ExtractEquiJoinKeys -- Scala Extractor for Destructuring Join Logical Operators

`ExtractEquiJoinKeys` is a Scala extractor to <<unapply, destructure a Join logical operator>> into a tuple with the following elements:

. link:spark-sql-joins.adoc#join-types[Join type]

. Left and right keys (for non-empty join keys in the link:spark-sql-LogicalPlan-Join.adoc#condition[condition] of the `Join` operator)

. Join condition (i.e. a Catalyst expression that could be used as a new join condition)

. The link:spark-sql-LogicalPlan-Join.adoc#left[left] and the link:spark-sql-LogicalPlan-Join.adoc#right[right] logical operators

[[ReturnType]]
.ReturnType
[source, scala]
----
(JoinType, Seq[Expression], Seq[Expression], Option[Expression], LogicalPlan, LogicalPlan)
----

`unapply` gives `None` (aka _nothing_) when no join keys were found or the logical plan is not a Join logical operator.

NOTE: `ExtractEquiJoinKeys` is a Scala object with `unapply` method.

```
val left = Seq((0, 1, "zero"), (1, 2, "one")).toDF("k1", "k2", "name")
val right = Seq((0, 0, "0"), (1, 1, "1")).toDF("k1", "k2", "name")
val q = left.join(right, Seq("k1", "k2", "name")).
  where(left("k1") > 3)
import org.apache.spark.sql.catalyst.plans.logical.Join
val join = q.queryExecution.optimizedPlan.p(1).asInstanceOf[Join]

// make sure the join condition is available
scala> join.condition.get
res1: org.apache.spark.sql.catalyst.expressions.Expression = (((k1#148 = k1#161) && (k2#149 = k2#162)) && (name#150 = name#163))

// Enable DEBUG logging level
import org.apache.log4j.{Level, Logger}
Logger.getLogger("org.apache.spark.sql.catalyst.planning.ExtractEquiJoinKeys").setLevel(Level.DEBUG)

import org.apache.spark.sql.catalyst.planning.ExtractEquiJoinKeys
scala> ExtractEquiJoinKeys.unapply(join)
2018-03-14 12:02:14 DEBUG ExtractEquiJoinKeys:58 - Considering join on: Some((((k1#148 = k1#161) && (k2#149 = k2#162)) && (name#150 = name#163)))
2018-03-14 12:02:14 DEBUG ExtractEquiJoinKeys:58 - leftKeys:List(k1#148, k2#149, name#150) | rightKeys:List(k1#161, k2#162, name#163)
res3: Option[org.apache.spark.sql.catalyst.planning.ExtractEquiJoinKeys.ReturnType] =
Some((Inner,List(k1#148, k2#149, name#150),List(k1#161, k2#162, name#163),None,Project [_1#144 AS k1#148, _2#145 AS k2#149, _3#146 AS name#150]
+- Filter ((_1#144 > 3) && isnotnull(_3#146))
   +- LocalRelation [_1#144, _2#145, _3#146]
,Project [_1#157 AS k1#161, _2#158 AS k2#162, _3#159 AS name#163]
+- Filter ((_1#157 > 3) && isnotnull(_3#159))
   +- LocalRelation [_1#157, _2#158, _3#159]
))
```

[[logging]]
[TIP]
====
Enable `DEBUG` logging level for `org.apache.spark.sql.catalyst.planning.ExtractEquiJoinKeys` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.catalyst.planning.ExtractEquiJoinKeys=DEBUG
```

Refer to link:spark-logging.adoc[Logging].
====

=== [[unapply]] Destructuring Logical Plan -- `unapply` Method

[source, scala]
----
type ReturnType =
  (JoinType, Seq[Expression], Seq[Expression], Option[Expression], LogicalPlan, LogicalPlan)

unapply(plan: LogicalPlan): Option[ReturnType]
----

Internally, `unapply` prints out the following DEBUG message to the logs:

```
Considering join on: [condition]
```

`unapply` then splits `condition` at `And` expression points (if there are any) to have a list of predicate expressions.

CAUTION: FIXME Example with a condition with multiple predicates separated by ANDs.

`unapply` finds `EqualTo` and `EqualNullSafe` binary expressions to collect the join keys (for the left and right side).

CAUTION: FIXME 5 examples for the different cases of `EqualTo` and `EqualNullSafe` binary expressions.

`unapply` takes the expressions that...FIXME...to build `otherPredicates`.

In the end, `unapply` splits the pairs of join keys into collections of left and right join keys. `unapply` prints out the following DEBUG message to the logs:

```
leftKeys:[leftKeys] | rightKeys:[rightKeys]
```

[NOTE]
====
`unapply` is used when:

* `JoinEstimation` is requested to link:spark-sql-JoinEstimation.adoc#estimateInnerOuterJoin[estimateInnerOuterJoin]

* `JoinSelection` execution planning strategy is link:spark-sql-SparkStrategy-JoinSelection.adoc#apply[executed]

* (Spark Structured Streaming) `StreamingJoinStrategy` execution planning strategy is executed

* (Spark Structured Streaming) `StreamingJoinHelper` is requested to find the watermark in the join keys
====
