title: Projection

# Projection -- Functions to Produce InternalRow for InternalRow

`Projection` is a <<contract, contract>> of Scala functions that produce an link:spark-sql-InternalRow.adoc[internal binary row] for a given internal row.

[source, scala]
----
Projection: InternalRow => InternalRow
----

[[initialize]]
`Projection` can optionally be *initialized* with the current partition index (which by default does nothing).

[source, scala]
----
initialize(partitionIndex: Int): Unit = {}
----

NOTE: `initialize` is overriden by link:spark-sql-InterpretedProjection.adoc#initialize[InterpretedProjection] and `InterpretedMutableProjection` projections that are used in link:spark-sql-Expression.adoc#eval[interpreted expression evaluation].

[[implementations]]
.Projections
[cols="1,2",options="header",width="100%"]
|===
| Projection
| Description

| [[UnsafeProjection]] link:spark-sql-UnsafeProjection.adoc[UnsafeProjection]
|

| [[InterpretedProjection]] link:spark-sql-InterpretedProjection.adoc[InterpretedProjection]
|

| [[IdentityProjection]] `IdentityProjection`
|

| [[MutableProjection]] `MutableProjection`
|

| [[InterpretedMutableProjection]] `InterpretedMutableProjection`
| _Appears not to be used anymore_
|===
