# CodeGeneratorWithInterpretedFallback

`CodeGeneratorWithInterpretedFallback` is the <<contract, base>> of <<extensions, codegen object generators>> that can create objects for <<createCodeGeneratedObject, codegen>> and <<createInterpretedObject, interpreted>> evaluation paths.

[[contract]]
.CodeGeneratorWithInterpretedFallback Contract
[cols="1m,2",options="header",width="100%"]
|===
| Method
| Description

| createCodeGeneratedObject
a| [[createCodeGeneratedObject]]

[source, scala]
----
createCodeGeneratedObject(in: IN): OUT
----

Used when...FIXME

| createInterpretedObject
a| [[createInterpretedObject]]

[source, scala]
----
createInterpretedObject(in: IN): OUT
----

Used when...FIXME
|===

[[extensions]]
NOTE: <<spark-sql-UnsafeProjection.adoc#, UnsafeProjection>> is the one and only known implementation of the <<contract, CodeGeneratorWithInterpretedFallback Contract>> in Apache Spark.

[[IN]][[OUT]]
[NOTE]
====
`CodeGeneratorWithInterpretedFallback` is a Scala type constructor (aka _generic type_) that accepts two types referred as `IN` and `OUT`.

[source, scala]
----
abstract class CodeGeneratorWithInterpretedFallback[IN, OUT]
----
====

=== [[createObject]] `createObject` Method

[source, scala]
----
createObject(in: IN): OUT
----

`createObject`...FIXME

NOTE: `createObject` is used exclusively when `UnsafeProjection` is requested to <<spark-sql-UnsafeProjection.adoc#create, create an UnsafeProjection for Catalyst expressions>>.
