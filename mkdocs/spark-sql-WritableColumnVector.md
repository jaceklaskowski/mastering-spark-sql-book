# WritableColumnVector

`WritableColumnVector` is the <<contract, extension>> of the <<spark-sql-ColumnVector.adoc#, ColumnVector contract>> for <<implementations, writable column vectors>> that <<FIXME, FIXME>>.

[[contract]]
.WritableColumnVector Contract (Abstract Methods Only)
[cols="1m,3",options="header",width="100%"]
|===
| Method
| Description

| getArrayLength
a| [[getArrayLength]]

[source, java]
----
int getArrayLength(int rowId)
----

Used when...FIXME

| getArrayOffset
a| [[getArrayOffset]]

[source, java]
----
int getArrayOffset(int rowId)
----

Used when...FIXME

| getBytesAsUTF8String
a| [[getBytesAsUTF8String]]

[source, java]
----
UTF8String getBytesAsUTF8String(
  int rowId,
  int count)
----

Used when...FIXME

| getDictId
a| [[getDictId]]

[source, java]
----
int getDictId(int rowId)
----

Used when...FIXME

| putArray
a| [[putArray]]

[source, java]
----
void putArray(
  int rowId,
  int offset,
  int length)
----

Used when...FIXME

| putBoolean
a| [[putBoolean]]

[source, java]
----
void putBoolean(
  int rowId,
  boolean value)
----

Used when...FIXME

| putBooleans
a| [[putBooleans]]

[source, java]
----
void putBooleans(
  int rowId,
  int count,
  boolean value)
----

Used when...FIXME

| putByte
a| [[putByte]]

[source, java]
----
void putByte(
  int rowId,
  byte value)
----

Used when...FIXME

| putByteArray
a| [[putByteArray]]

[source, java]
----
int putByteArray(
  int rowId,
  byte[] value,
  int offset,
  int count)
----

Used when...FIXME

| putBytes
a| [[putBytes]]

[source, java]
----
void putBytes(
  int rowId,
  int count,
  byte value)
void putBytes(
  int rowId,
  int count,
  byte[] src,
  int srcIndex)
----

Used when...FIXME

| putDouble
a| [[putDouble]]

[source, java]
----
void putDouble(
  int rowId,
  double value)
----

Used when...FIXME

| putDoubles
a| [[putDoubles]]

[source, java]
----
void putDoubles(
  int rowId,
  int count,
  byte[] src,
  int srcIndex)
void putDoubles(
  int rowId,
  int count,
  double value)
void putDoubles(
  int rowId,
  int count,
  double[] src,
  int srcIndex)
----

Used when...FIXME

| putFloat
a| [[putFloat]]

[source, java]
----
void putFloat(
  int rowId,
  float value)
----

Used when...FIXME

| putFloats
a| [[putFloats]]

[source, java]
----
void putFloats(
  int rowId,
  int count,
  byte[] src,
  int srcIndex)
void putFloats(
  int rowId,
  int count,
  float value)
void putFloats(
  int rowId,
  int count,
  float[] src,
  int srcIndex)
----

Used when...FIXME

| putInt
a| [[putInt]]

[source, java]
----
void putInt(
  int rowId,
  int value)
----

Used when...FIXME

| putInts
a| [[putInts]]

[source, java]
----
void putInts(
  int rowId,
  int count,
  byte[] src,
  int srcIndex)
void putInts(
  int rowId,
  int count,
  int value)
void putInts(
  int rowId,
  int count,
  int[] src,
  int srcIndex)
----

Used when...FIXME

| putIntsLittleEndian
a| [[putIntsLittleEndian]]

[source, java]
----
void putIntsLittleEndian(
  int rowId,
  int count,
  byte[] src,
  int srcIndex)
----

Used when...FIXME

| putLong
a| [[putLong]]

[source, java]
----
void putLong(
  int rowId,
  long value)
----

Used when...FIXME

| putLongs
a| [[putLongs]]

[source, java]
----
void putLongs(
  int rowId,
  int count,
  byte[] src,
  int srcIndex)
void putLongs(
  int rowId,
  int count,
  long value)
void putLongs(
  int rowId,
  int count,
  long[] src,
  int srcIndex)
----

Used when...FIXME

| putLongsLittleEndian
a| [[putLongsLittleEndian]]

[source, java]
----
void putLongsLittleEndian(
  int rowId,
  int count,
  byte[] src,
  int srcIndex)
----

Used when...FIXME

| putNotNull
a| [[putNotNull]]

[source, java]
----
void putNotNull(int rowId)
----

Used when `WritableColumnVector` is requested to <<reset, reset>> and <<appendNotNulls, appendNotNulls>>

| putNotNulls
a| [[putNotNulls]]

[source, java]
----
void putNotNulls(
  int rowId,
  int count)
----

Used when...FIXME

| putNull
a| [[putNull]]

[source, java]
----
void putNull(int rowId)
----

Used when...FIXME

| putNulls
a| [[putNulls]]

[source, java]
----
void putNulls(
  int rowId,
  int count)
----

Used when...FIXME

| putShort
a| [[putShort]]

[source, java]
----
void putShort(
  int rowId,
  short value)
----

Used when...FIXME

| putShorts
a| [[putShorts]]

[source, java]
----
void putShorts(
  int rowId,
  int count,
  byte[] src,
  int srcIndex)
void putShorts(
  int rowId,
  int count,
  short value)
void putShorts(
  int rowId,
  int count,
  short[] src,
  int srcIndex)
----

Used when...FIXME

| reserveInternal
a| [[reserveInternal]]

[source, java]
----
void reserveInternal(int capacity)
----

Used when:

* <<spark-sql-OffHeapColumnVector.adoc#, OffHeapColumnVector>> and <<spark-sql-OnHeapColumnVector.adoc#, OnHeapColumnVector>> are created

* `WritableColumnVector` is requested to <<reserve, reserve memory of a given required capacity>>

| reserveNewColumn
a| [[reserveNewColumn]]

[source, java]
----
WritableColumnVector reserveNewColumn(
  int capacity,
  DataType type)
----

Used when...FIXME

|===

[[implementations]]
.WritableColumnVectors
[cols="1,3",options="header",width="100%"]
|===
| WritableColumnVector
| Description

| <<spark-sql-OffHeapColumnVector.adoc#, OffHeapColumnVector>>
| [[OffHeapColumnVector]]

| <<spark-sql-OnHeapColumnVector.adoc#, OnHeapColumnVector>>
| [[OnHeapColumnVector]]

|===

[[creating-instance]]
`WritableColumnVector` takes the following to be created:

* [[capacity]] Number of rows to hold in a vector (aka `capacity`)
* [[type]] link:spark-sql-DataType.adoc[Data type] of the rows stored

NOTE: `WritableColumnVector` is a Java abstract class and cannot be <<creating-instance, created>> directly. It is created indirectly for the <<implementations, concrete WritableColumnVectors>>.

=== [[reset]] `reset` Method

[source, java]
----
void reset()
----

`reset`...FIXME

[NOTE]
====
`reset` is used when:

* `OrcColumnarBatchReader` is requested to `nextBatch`

* `VectorizedParquetRecordReader` is requested to <<spark-sql-VectorizedParquetRecordReader.adoc#nextBatch, read next rows into a columnar batch>>

* <<spark-sql-OffHeapColumnVector.adoc#, OffHeapColumnVector>> and <<spark-sql-OnHeapColumnVector.adoc#, OnHeapColumnVector>> are created

* `WritableColumnVector` is requested to <<reserveDictionaryIds, reserveDictionaryIds>>
====

=== [[reserve]] Reserving Memory Of Required Capacity -- `reserve` Method

[source, java]
----
void reserve(int requiredCapacity)
----

`reserve`...FIXME

[NOTE]
====
`reserve` is used when:

* `OrcColumnarBatchReader` is requested to `putRepeatingValues`, `putNonNullValues`, `putValues`, and `putDecimalWritables`

* `WritableColumnVector` is requested to _append values_
====

=== [[reserveDictionaryIds]] `reserveDictionaryIds` Method

[source, java]
----
WritableColumnVector reserveDictionaryIds(int capacity)
----

`reserveDictionaryIds`...FIXME

NOTE: `reserveDictionaryIds` is used when...FIXME

=== [[appendNotNulls]] `appendNotNulls` Final Method

[source, java]
----
int appendNotNulls(int count)
----

`appendNotNulls`...FIXME

NOTE: `appendNotNulls` is used for testing purposes only.
