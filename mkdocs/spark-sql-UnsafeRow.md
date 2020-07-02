title: UnsafeRow

# UnsafeRow -- Mutable Raw-Memory Unsafe Binary Row Format

`UnsafeRow` is a concrete link:spark-sql-InternalRow.adoc[InternalRow] that represents a mutable internal raw-memory (and hence unsafe) binary row format.

In other words, `UnsafeRow` is an `InternalRow` that is backed by raw memory instead of Java objects.

[source, scala]
----
// Use ExpressionEncoder for simplicity
import org.apache.spark.sql.catalyst.encoders.ExpressionEncoder
val stringEncoder = ExpressionEncoder[String]
val row = stringEncoder.toRow("hello world")

import org.apache.spark.sql.catalyst.expressions.UnsafeRow
val unsafeRow = row match { case ur: UnsafeRow => ur }

scala> unsafeRow.getBytes
res0: Array[Byte] = Array(0, 0, 0, 0, 0, 0, 0, 0, 11, 0, 0, 0, 16, 0, 0, 0, 104, 101, 108, 108, 111, 32, 119, 111, 114, 108, 100, 0, 0, 0, 0, 0)

scala> unsafeRow.getUTF8String(0)
res1: org.apache.spark.unsafe.types.UTF8String = hello world
----

[[sizeInBytes]]
`UnsafeRow` knows its *size in bytes*.

[source, scala]
----
scala> println(unsafeRow.getSizeInBytes)
32
----

`UnsafeRow` supports Java's <<Externalizable, Externalizable>> and Kryo's <<KryoSerializable, KryoSerializable>> serialization/deserialization protocols.

The fields of a data row are placed using *field offsets*.

[[mutableFieldTypes]]
[[mutable-types]]
`UnsafeRow` considers a <<spark-sql-DataType.adoc#, data type>> mutable if it is one of the following:

* <<spark-sql-DataType.adoc#BooleanType, BooleanType>>
* <<spark-sql-DataType.adoc#ByteType, ByteType>>
* <<spark-sql-DataType.adoc#DateType, DateType>>
* <<spark-sql-DataType.adoc#DecimalType, DecimalType>> (see <<isMutable, isMutable>>)
* <<spark-sql-DataType.adoc#DoubleType, DoubleType>>
* <<spark-sql-DataType.adoc#FloatType, FloatType>>
* <<spark-sql-DataType.adoc#IntegerType, IntegerType>>
* <<spark-sql-DataType.adoc#LongType, LongType>>
* <<spark-sql-DataType.adoc#NullType, NullType>>
* <<spark-sql-DataType.adoc#ShortType, ShortType>>
* <<spark-sql-DataType.adoc#TimestampType, TimestampType>>

`UnsafeRow` is composed of three regions:

. Null Bit Set Bitmap Region (1 bit/field) for tracking null values
. Fixed-Length 8-Byte Values Region
. Variable-Length Data Section

That gives the property of rows being always 8-byte word aligned and so their size is always a multiple of 8 bytes.

Equality comparision and hashing of rows can be performed on raw bytes since if two rows are identical so should be their bit-wise representation. No type-specific interpretation is required.

=== [[isMutable]] `isMutable` Static Predicate

[source, java]
----
static boolean isMutable(DataType dt)
----

`isMutable` is enabled (`true`) when the input <<spark-sql-DataType.adoc#, DataType>> is among the <<mutableFieldTypes, mutable field types>> or a <<spark-sql-DataType.adoc#DecimalType, DecimalType>>.

Otherwise, `isMutable` is disabled (`false`).

[NOTE]
====
`isMutable` is used when:

* `UnsafeFixedWidthAggregationMap` is requested to <<spark-sql-UnsafeFixedWidthAggregationMap.adoc#supportsAggregationBufferSchema, supportsAggregationBufferSchema>>

* `SortBasedAggregationIterator` is requested for <<spark-sql-SortBasedAggregationIterator.adoc#newBuffer, newBuffer>>
====

=== [[KryoSerializable]] Kryo's KryoSerializable SerDe Protocol

TIP: Read up on https://github.com/EsotericSoftware/kryo#kryoserializable[KryoSerializable].

==== [[write]] Serializing JVM Object -- KryoSerializable's `write` Method

[source, java]
----
void write(Kryo kryo, Output out)
----

==== [[read]] Deserializing Kryo-Managed Object -- KryoSerializable's `read` Method

[source, java]
----
void read(Kryo kryo, Input in)
----

=== [[Externalizable]] Java's Externalizable SerDe Protocol

TIP: Read up on https://docs.oracle.com/javase/8/docs/api/java/io/Externalizable.html[java.io.Externalizable].

==== [[writeExternal]] Serializing JVM Object -- Externalizable's `writeExternal` Method

[source, java]
----
void writeExternal(ObjectOutput out)
throws IOException
----

==== [[readExternal]] Deserializing Java-Externalized Object -- Externalizable's `readExternal` Method

[source, java]
----
void readExternal(ObjectInput in)
throws IOException, ClassNotFoundException
----

=== [[pointTo]] `pointTo` Method

[source, java]
----
void pointTo(Object baseObject, long baseOffset, int sizeInBytes)
----

`pointTo`...FIXME

NOTE: `pointTo` is used when...FIXME
