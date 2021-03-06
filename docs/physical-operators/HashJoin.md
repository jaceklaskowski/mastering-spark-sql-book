# HashJoin &mdash; Hash-Based Join Physical Operators

`HashJoin` is an [extension](#contract) of the [BaseJoinExec](BaseJoinExec.md) abstraction for [hash-based join physical operators](#implementations) with support for [Java code generation](CodegenSupport.md).

## Contract

### <span id="buildSide"> buildSide

```scala
buildSide: BuildSide
```

### <span id="prepareRelation"> Preparing HashedRelation

```scala
prepareRelation(
  ctx: CodegenContext): HashedRelationInfo
```

Used when:

* `HashJoin` is requested to [codegenInner](#codegenInner), [codegenOuter](#codegenOuter), [codegenSemi](#codegenSemi), [codegenAnti](#codegenAnti), [codegenExistence](#codegenExistence)

## Implementations

* [BroadcastHashJoinExec](BroadcastHashJoinExec.md)
* [ShuffledHashJoinExec](ShuffledHashJoinExec.md)

## <span id="join"> join

```scala
join(
  streamedIter: Iterator[InternalRow],
  hashed: HashedRelation,
  numOutputRows: SQLMetric): Iterator[InternalRow]
```

`join` branches off per [JoinType](BaseJoinExec.md#joinType) to create an joined rows iterator (off the rows from the input `streamedIter` and `hashed`):

* [innerJoin](#innerJoin) for a [InnerLike](../joins.md#InnerLike) join

* [outerJoin](#outerJoin) for a [LeftOuter](../joins.md#LeftOuter) or a [RightOuter](../joins.md#RightOuter) join

* [semiJoin](#semiJoin) for a [LeftSemi](../joins.md#LeftSemi) join

* [antiJoin](#antiJoin) for a [LeftAnti](../joins.md#LeftAnti) join

* [existenceJoin](#existenceJoin) for a [ExistenceJoin](../joins.md#ExistenceJoin) join

`join` [creates a result projection](#createResultProjection).

In the end, for every row in the joined rows iterator `join` increments the input `numOutputRows` SQL metric and applies the result projection.

`join` reports an `IllegalArgumentException` for unsupported [JoinType](BaseJoinExec.md#joinType):

```text
HashJoin should not take [joinType] as the JoinType
```

`join` is used when:

* [BroadcastHashJoinExec](BroadcastHashJoinExec.md) and [ShuffledHashJoinExec](ShuffledHashJoinExec.md) physical operators are executed

## <span id="codegenAnti"> Generating Java Code for Anti Join

```scala
codegenAnti(
  ctx: CodegenContext,
  input: Seq[ExprCode]): String
```

`codegenAnti`...FIXME

`codegenAnti` is used when:

* `BroadcastHashJoinExec` physical operator is requested to [codegenAnti](BroadcastHashJoinExec.md#codegenAnti) (with the [isNullAwareAntiJoin](BroadcastHashJoinExec.md#isNullAwareAntiJoin) flag off)
* `HashJoin` is requested to [doConsume](#doConsume)
