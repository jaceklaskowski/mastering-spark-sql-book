title: LocalDateTimeEncoder

# LocalDateTimeEncoder -- Custom ExpressionEncoder for java.time.LocalDateTime

Spark SQL does not support `java.time.LocalDateTime` values in a `Dataset`.

```
import java.time.LocalDateTime

val data = Seq((0, LocalDateTime.now))
scala> val times = data.toDF("time")
java.lang.UnsupportedOperationException: No Encoder found for java.time.LocalDateTime
- field (class: "java.time.LocalDateTime", name: "_2")
- root class: "scala.Tuple2"
  at org.apache.spark.sql.catalyst.ScalaReflection$$anonfun$org$apache$spark$sql$catalyst$ScalaReflection$$serializerFor$1.apply(ScalaReflection.scala:643)
  at org.apache.spark.sql.catalyst.ScalaReflection$$anonfun$org$apache$spark$sql$catalyst$ScalaReflection$$serializerFor$1.apply(ScalaReflection.scala:445)
  at scala.reflect.internal.tpe.TypeConstraints$UndoLog.undo(TypeConstraints.scala:56)
  at org.apache.spark.sql.catalyst.ScalaReflection$class.cleanUpReflectionObjects(ScalaReflection.scala:824)
  at org.apache.spark.sql.catalyst.ScalaReflection$.cleanUpReflectionObjects(ScalaReflection.scala:39)
  at org.apache.spark.sql.catalyst.ScalaReflection$.org$apache$spark$sql$catalyst$ScalaReflection$$serializerFor(ScalaReflection.scala:445)
  at org.apache.spark.sql.catalyst.ScalaReflection$$anonfun$org$apache$spark$sql$catalyst$ScalaReflection$$serializerFor$1$$anonfun$8.apply(ScalaReflection.scala:637)
  at org.apache.spark.sql.catalyst.ScalaReflection$$anonfun$org$apache$spark$sql$catalyst$ScalaReflection$$serializerFor$1$$anonfun$8.apply(ScalaReflection.scala:625)
  at scala.collection.TraversableLike$$anonfun$flatMap$1.apply(TraversableLike.scala:241)
  at scala.collection.TraversableLike$$anonfun$flatMap$1.apply(TraversableLike.scala:241)
  at scala.collection.immutable.List.foreach(List.scala:381)
  at scala.collection.TraversableLike$class.flatMap(TraversableLike.scala:241)
  at scala.collection.immutable.List.flatMap(List.scala:344)
  at org.apache.spark.sql.catalyst.ScalaReflection$$anonfun$org$apache$spark$sql$catalyst$ScalaReflection$$serializerFor$1.apply(ScalaReflection.scala:625)
  at org.apache.spark.sql.catalyst.ScalaReflection$$anonfun$org$apache$spark$sql$catalyst$ScalaReflection$$serializerFor$1.apply(ScalaReflection.scala:445)
  at scala.reflect.internal.tpe.TypeConstraints$UndoLog.undo(TypeConstraints.scala:56)
  at org.apache.spark.sql.catalyst.ScalaReflection$class.cleanUpReflectionObjects(ScalaReflection.scala:824)
  at org.apache.spark.sql.catalyst.ScalaReflection$.cleanUpReflectionObjects(ScalaReflection.scala:39)
  at org.apache.spark.sql.catalyst.ScalaReflection$.org$apache$spark$sql$catalyst$ScalaReflection$$serializerFor(ScalaReflection.scala:445)
  at org.apache.spark.sql.catalyst.ScalaReflection$.serializerFor(ScalaReflection.scala:434)
  at org.apache.spark.sql.catalyst.encoders.ExpressionEncoder$.apply(ExpressionEncoder.scala:71)
  at org.apache.spark.sql.Encoders$.product(Encoders.scala:275)
  at org.apache.spark.sql.LowPrioritySQLImplicits$class.newProductEncoder(SQLImplicits.scala:248)
  at org.apache.spark.sql.SQLImplicits.newProductEncoder(SQLImplicits.scala:34)
  ... 50 elided
```

As it is clearly said in the exception, the root cause is no <<spark-sql-Encoder.adoc#, Encoder>> found for `java.time.LocalDateTime` (as there is not one available in Spark SQL).

You could define one using <<spark-sql-ExpressionEncoder.adoc#, ExpressionEncoder>>, but that does not seem to work either.

```
import org.apache.spark.sql.catalyst.encoders.ExpressionEncoder
scala> ExpressionEncoder[java.time.LocalDateTime]
java.lang.UnsupportedOperationException: No Encoder found for java.time.LocalDateTime
- root class: "java.time.LocalDateTime"
  at org.apache.spark.sql.catalyst.ScalaReflection$$anonfun$org$apache$spark$sql$catalyst$ScalaReflection$$serializerFor$1.apply(ScalaReflection.scala:643)
  at org.apache.spark.sql.catalyst.ScalaReflection$$anonfun$org$apache$spark$sql$catalyst$ScalaReflection$$serializerFor$1.apply(ScalaReflection.scala:445)
  at scala.reflect.internal.tpe.TypeConstraints$UndoLog.undo(TypeConstraints.scala:56)
  at org.apache.spark.sql.catalyst.ScalaReflection$class.cleanUpReflectionObjects(ScalaReflection.scala:824)
  at org.apache.spark.sql.catalyst.ScalaReflection$.cleanUpReflectionObjects(ScalaReflection.scala:39)
  at org.apache.spark.sql.catalyst.ScalaReflection$.org$apache$spark$sql$catalyst$ScalaReflection$$serializerFor(ScalaReflection.scala:445)
  at org.apache.spark.sql.catalyst.ScalaReflection$.serializerFor(ScalaReflection.scala:434)
  at org.apache.spark.sql.catalyst.encoders.ExpressionEncoder$.apply(ExpressionEncoder.scala:71)
  ... 50 elided
```

The simplest solution is to transform the `Dataset` with `java.time.LocalDateTime` to a supported type that Spark SQL offers an encoder for.

A much better solution would be to provide a custom `Encoder` that would expand the types supported in Spark SQL.

`LocalDateTimeEncoder` is an _attempt_ to develop a custom <<spark-sql-ExpressionEncoder.adoc#, ExpressionEncoder>> for Java's https://docs.oracle.com/javase/8/docs/api/java/time/LocalDateTime.html[java.time.LocalDateTime] so you don't have to map values to another supported type.

====
public final class *LocalDateTime*

A date-time without a time-zone in the ISO-8601 calendar system, such as `2007-12-03T10:15:30`.

`LocalDateTime` is an immutable date-time object that represents a date-time, often viewed as year-month-day-hour-minute-second.
====

```
// A very fresh attempt at creating an Encoder for java.time.LocalDateTime

// See ExpressionEncoder.apply
import org.apache.spark.sql.catalyst.expressions.BoundReference
import org.apache.spark.sql.catalyst.ScalaReflection
import java.time.LocalDateTime
val inputObject = BoundReference(0, ScalaReflection.dataTypeFor[LocalDateTime], nullable = true)

// ScalaReflection.serializerFor[LocalDateTime](inputObject)
import org.apache.spark.sql.catalyst.expressions.{CreateNamedStruct, Literal}
import org.apache.spark.sql.catalyst.expressions.objects.StaticInvoke
import org.apache.spark.sql.catalyst.util.DateTimeUtils
import org.apache.spark.sql.types.DateType
// Simply invokes DateTimeUtils.fromJavaDate
// fromJavaDate(date: Date): Int
// serializing a Date to an Int

// Util object to do conversion (serialization)
object LocalDateTimeUtils {
  import java.time.LocalDateTime
  def fromLocalDateTime(date: LocalDateTime): Long = date.toEpochSecond(java.time.ZoneOffset.UTC)
}

val other = StaticInvoke(
  LocalDateTimeUtils.getClass,
  DateType,
  "fromLocalDateTime",
  inputObject :: Nil,
  returnNullable = false)
val serializer = CreateNamedStruct(Literal("value") :: other :: Nil)
val schema = serializer.dataType

// ScalaReflection.deserializerFor[T]
// FIXME Create it as for ScalaReflection.serializerFor above
val deserializer = serializer // FIXME

import scala.reflect.ClassTag
import scala.reflect.runtime.universe.{typeTag, TypeTag}
val mirror = ScalaReflection.mirror
val tpe = typeTag[java.time.LocalDateTime].in(mirror).tpe
val cls = mirror.runtimeClass(tpe)

import org.apache.spark.sql.catalyst.encoders.ExpressionEncoder
val localDateTimeEncoder = new ExpressionEncoder[java.time.LocalDateTime](
  schema,
  flat = true,
  serializer.flatten,
  deserializer,
  ClassTag[java.time.LocalDateTime](cls))

import org.apache.spark.sql.Encoder
implicit val encLocalDateTime: Encoder[java.time.LocalDateTime] = localDateTimeEncoder

// DEMO
val data = Seq(LocalDateTime.now)
val times = spark.createDataset(data) // (encLocalDateTime)
```

```
// $ SPARK_SUBMIT_OPTS="-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005" ./bin/spark-shell --conf spark.rpc.askTimeout=5m

import java.time.LocalDateTime
import org.apache.spark.sql.Encoder
import org.apache.spark.sql.catalyst.encoders.ExpressionEncoder

import org.apache.spark.sql.types._
val schema = StructType(
  $"year".int :: $"month".int :: $"day".int :: Nil)
import org.apache.spark.sql.catalyst.expressions.Expression
import org.apache.spark.sql.catalyst.expressions.objects.StaticInvoke

import org.apache.spark.sql.types.ObjectType
import org.apache.spark.sql.catalyst.expressions.BoundReference
val clazz = classOf[java.time.LocalDateTime]
val inputObject = BoundReference(0, ObjectType(clazz), nullable = true)
val nullSafeInput = inputObject

import org.apache.spark.sql.types.TimestampType
val staticInvoke = StaticInvoke(
  classOf[java.time.LocalDateTime],
  TimestampType,
  "parse",
  inputObject :: Nil))

// Based on UDTRegistration
val clazz = classOf[java.time.LocalDateTime]
import org.apache.spark.sql.catalyst.expressions.objects.NewInstance
val obj = NewInstance(
  cls = clazz,
  arguments = Nil,
  dataType = ObjectType(clazz))
import org.apache.spark.sql.catalyst.expressions.objects.Invoke

// the following would be nice to have
// FIXME How to bind them all up into one BoundReference?
import org.apache.spark.sql.types.IntegerType
val yearRef = BoundReference(0, IntegerType, nullable = true)
val monthRef = BoundReference(1, IntegerType, nullable = true)
val dayOfMonthRef = BoundReference(2, IntegerType, nullable = true)
val hourRef = BoundReference(3, IntegerType, nullable = true)
val minuteRef = BoundReference(4, IntegerType, nullable = true)

import org.apache.spark.sql.types.ArrayType
val inputObject = BoundReference(0, ArrayType(IntegerType), nullable = true)

def invoke(inputObject: Expression, fieldName: String) = Invoke(
  targetObject = inputObject,
  functionName = fieldName,
  dataType = IntegerType)

import org.apache.spark.sql.catalyst.expressions.CreateNamedStruct
import org.apache.spark.sql.catalyst.expressions.Literal
import org.apache.spark.sql.catalyst.expressions.GetArrayItem
val year = GetArrayItem(inputObject, Literal(0))
val month = GetArrayItem(inputObject, Literal(1))
val day = GetArrayItem(inputObject, Literal(2))
val hour = GetArrayItem(inputObject, Literal(3))
val minute = GetArrayItem(inputObject, Literal(4))

// turn LocalDateTime into InternalRow
// by saving LocalDateTime in parts
val serializer = CreateNamedStruct(
  Literal("year") :: year ::
  Literal("month") :: month ::
  Literal("day") :: day ::
  Literal("hour") :: hour ::
  Literal("minute") :: minute ::
  Nil)

import org.apache.spark.sql.catalyst.expressions.objects.StaticInvoke
import org.apache.spark.sql.catalyst.util.DateTimeUtils
val getPath: Expression = Literal("value")
val deserializer: Expression =
  StaticInvoke(
    DateTimeUtils.getClass,
    ObjectType(classOf[java.time.LocalDateTime]),
    "toJavaTimestamp",
    getPath :: Nil)

// we ask serializer about the schema
val schema: StructType = serializer.dataType

import scala.reflect._
implicit def scalaLocalDateTime: Encoder[java.time.LocalDateTime] =
  new ExpressionEncoder[java.time.LocalDateTime](
    schema,
    flat = false,  // serializer.size == 1
    serializer.flatten,
    deserializer,
    classTag[java.time.LocalDateTime])

// the above leads to the following exception
// Add log4j.logger.org.apache.spark.sql.catalyst.expressions.codegen.CodeGenerator=DEBUG to see the code
scala> scalaLocalDateTime.asInstanceOf[ExpressionEncoder[LocalDateTime]].toRow(LocalDateTime.now)
java.lang.RuntimeException: Error while encoding: java.lang.ClassCastException: java.time.LocalDateTime cannot be cast to org.apache.spark.sql.catalyst.util.ArrayData
input[0, array<int>, true][0] AS year#0
input[0, array<int>, true][1] AS month#1
input[0, array<int>, true][2] AS day#2
input[0, array<int>, true][3] AS hour#3
input[0, array<int>, true][4] AS minute#4
  at org.apache.spark.sql.catalyst.encoders.ExpressionEncoder.toRow(ExpressionEncoder.scala:291)
  ... 52 elided
Caused by: java.lang.ClassCastException: java.time.LocalDateTime cannot be cast to org.apache.spark.sql.catalyst.util.ArrayData
  at org.apache.spark.sql.catalyst.expressions.BaseGenericInternalRow$class.getArray(rows.scala:48)
  at org.apache.spark.sql.catalyst.expressions.GenericInternalRow.getArray(rows.scala:194)
  at org.apache.spark.sql.catalyst.expressions.GeneratedClass$SpecificUnsafeProjection.apply_0$(Unknown Source)
  at org.apache.spark.sql.catalyst.expressions.GeneratedClass$SpecificUnsafeProjection.apply(Unknown Source)
  at org.apache.spark.sql.catalyst.encoders.ExpressionEncoder.toRow(ExpressionEncoder.scala:288)
  ... 52 more

// and so the following won't work either
val times = Seq(LocalDateTime.now).toDF("time")
```

## Open Questions

. `ScalaReflection.serializerFor` passes `ObjectType` objects through

. `ScalaReflection.serializerFor` uses `StaticInvoke` for `java.sql.Timestamp` and `java.sql.Date`.
+
```
case t if t <:< localTypeOf[java.sql.Timestamp] =>
  StaticInvoke(
    DateTimeUtils.getClass,
    TimestampType,
    "fromJavaTimestamp",
    inputObject :: Nil)

case t if t <:< localTypeOf[java.sql.Date] =>
  StaticInvoke(
    DateTimeUtils.getClass,
    DateType,
    "fromJavaDate",
    inputObject :: Nil)
```

. How could `SQLUserDefinedType` and `UDTRegistration` help here?
