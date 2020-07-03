# ScalaReflection

`ScalaReflection` is the contract and the only implementation of the contract with...FIXME

=== [[serializerFor]] `serializerFor` Object Method

[source, scala]
----
serializerFor[T : TypeTag](inputObject: Expression): CreateNamedStruct
----

`serializerFor` firstly finds the <<localTypeOf, local type of>> the input type `T` and then the <<getClassNameFromType, class name>>.

`serializerFor` uses the <<serializerFor-internal, internal version of itself>> with the input `inputObject` expression, the `tpe` type and the `walkedTypePath` with the class name found earlier (of the input type `T`).

```
- root class: "[clsName]"
```

In the end, `serializerFor` returns one of the following:

* The <<spark-sql-Expression-CreateNamedStruct.adoc#, CreateNamedStruct>> expression from the false value of the `If` expression returned only if the type `T` is <<definedByConstructorParams, definedByConstructorParams>>

* Creates a <<spark-sql-Expression-CreateNamedStruct.adoc#creating-instance, CreateNamedStruct>> expression with the <<spark-sql-Expression-Literal.adoc#, Literal>> with the <<spark-sql-Expression-Literal.adoc#apply, value>> as `"value"` and the expression returned

[source, scala]
----
import org.apache.spark.sql.functions.lit
val inputObject = lit(1).expr

import org.apache.spark.sql.catalyst.ScalaReflection
val serializer = ScalaReflection.serializerFor(inputObject)
scala> println(serializer)
named_struct(value, 1)
----

NOTE: `serializerFor` is used when...FIXME

==== [[serializerFor-internal]] `serializerFor` Internal Method

[source, scala]
----
serializerFor(
  inputObject: Expression,
  tpe: `Type`,
  walkedTypePath: Seq[String],
  seenTypeSet: Set[`Type`] = Set.empty): Expression
----

`serializerFor`...FIXME

NOTE: `serializerFor` is used exclusively when `ScalaReflection` is requested to <<serializerFor, serializerFor>>.

=== [[localTypeOf]] `localTypeOf` Object Method

[source, scala]
----
localTypeOf[T: TypeTag]: `Type`
----

`localTypeOf`...FIXME

[source, scala]
----
import org.apache.spark.sql.catalyst.ScalaReflection
val tpe = ScalaReflection.localTypeOf[Int]
scala> :type tpe
org.apache.spark.sql.catalyst.ScalaReflection.universe.Type

scala> println(tpe)
Int
----

NOTE: `localTypeOf` is used when...FIXME

=== [[getClassNameFromType]] `getClassNameFromType` Object Method

[source, scala]
----
getClassNameFromType(tpe: `Type`): String
----

`getClassNameFromType`...FIXME

[source, scala]
----
import org.apache.spark.sql.catalyst.ScalaReflection
val tpe = ScalaReflection.localTypeOf[java.time.LocalDateTime]
val className = ScalaReflection.getClassNameFromType(tpe)
scala> println(className)
java.time.LocalDateTime
----

NOTE: `getClassNameFromType` is used when...FIXME

=== [[definedByConstructorParams]] `definedByConstructorParams` Object Method

[source, scala]
----
definedByConstructorParams(tpe: Type): Boolean
----

`definedByConstructorParams`...FIXME

NOTE: `definedByConstructorParams` is used when...FIXME
