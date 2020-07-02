title: BoundReference

# BoundReference Leaf Expression -- Reference to Value in Internal Binary Row

`BoundReference` is a link:spark-sql-Expression.adoc#LeafExpression[leaf expression] that <<eval, evaluates to a value in an internal binary row>> at a specified <<ordinal, position>> and of a given <<dataType, data type>>.

[[creating-instance]]
`BoundReference` takes the following when created:

* [[ordinal]] Ordinal, i.e. the position
* [[dataType]] link:spark-sql-DataType.adoc[Data type] of the value
* [[nullable]] `nullable` flag that controls whether the value can be `null` or not

[source, scala]
----
import org.apache.spark.sql.catalyst.expressions.BoundReference
import org.apache.spark.sql.types.LongType
val boundRef = BoundReference(ordinal = 0, dataType = LongType, nullable = true)

scala> println(boundRef.toString)
input[0, bigint, true]

import org.apache.spark.sql.catalyst.InternalRow
val row = InternalRow(1L, "hello")

val value = boundRef.eval(row).asInstanceOf[Long]
----

You can also create a `BoundReference` using Catalyst DSL's link:spark-sql-catalyst-dsl.adoc#at[at] method.

[source, scala]
----
import org.apache.spark.sql.catalyst.dsl.expressions._
val boundRef = 'hello.string.at(4)
scala> println(boundRef)
input[4, string, true]
----

=== [[eval]] Evaluating Expression -- `eval` Method

[source, scala]
----
eval(input: InternalRow): Any
----

NOTE: `eval` is part of <<spark-sql-Expression.adoc#eval, Expression Contract>> for the *interpreted (non-code-generated) expression evaluation*, i.e. evaluating a Catalyst expression to a JVM object for a given <<spark-sql-InternalRow.adoc#, internal binary row>>.

`eval` gives the value at <<ordinal, position>> from the `input` link:spark-sql-InternalRow.adoc[internal binary row] that is of a correct type.

Internally, `eval` returns `null` if the value at the <<ordinal, position>> is `null`.

Otherwise, `eval` uses the methods of `InternalRow` per the defined <<dataType, data type>> to access the value.

.eval's DataType to InternalRow's Methods Mapping (in execution order)
[cols="1,m",options="header",width="100%"]
|===
| DataType
| InternalRow's Method

| link:spark-sql-DataType.adoc#BooleanType[BooleanType]
| getBoolean

| link:spark-sql-DataType.adoc#ByteType[ByteType]
| getByte

| link:spark-sql-DataType.adoc#ShortType[ShortType]
| getShort

| link:spark-sql-DataType.adoc#IntegerType[IntegerType] or link:spark-sql-DataType.adoc#DateType[DateType]
| getInt

| link:spark-sql-DataType.adoc#LongType[LongType] or link:spark-sql-DataType.adoc#TimestampType[TimestampType]
| getLong

| link:spark-sql-DataType.adoc#FloatType[FloatType]
| getFloat

| link:spark-sql-DataType.adoc#DoubleType[DoubleType]
| getDouble

| link:spark-sql-DataType.adoc#StringType[StringType]
| getUTF8String

| link:spark-sql-DataType.adoc#BinaryType[BinaryType]
| getBinary

| link:spark-sql-DataType.adoc#CalendarIntervalType[CalendarIntervalType]
| getInterval

| link:spark-sql-DataType.adoc#DecimalType[DecimalType]
| getDecimal

| link:spark-sql-DataType.adoc#StructType[StructType]
| getStruct

| link:spark-sql-DataType.adoc#ArrayType[ArrayType]
| getArray

| link:spark-sql-DataType.adoc#MapType[MapType]
| getMap

| _others_
| get(ordinal, dataType)
|===

=== [[doGenCode]] Generating Java Source Code (ExprCode) For Code-Generated Expression Evaluation -- `doGenCode` Method

[source, scala]
----
doGenCode(ctx: CodegenContext, ev: ExprCode): ExprCode
----

NOTE: `doGenCode` is part of <<spark-sql-Expression.adoc#doGenCode, Expression Contract>> to generate a Java source code (ExprCode) for code-generated expression evaluation.

`doGenCode`...FIXME

=== [[BindReferences]][[bindReference]] `BindReferences.bindReference` Method

[source, scala]
----
bindReference[A <: Expression](
  expression: A,
  input: AttributeSeq,
  allowFailures: Boolean = false): A
----

`bindReference`...FIXME

NOTE: `bindReference` is used when...FIXME
