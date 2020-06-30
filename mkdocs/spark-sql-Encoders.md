title: Encoders

# Encoders Factory Object

`Encoders` is a factory object that...FIXME

=== [[kryo]] Creating Encoder Using Kryo -- `kryo` Method

[source, scala]
----
kryo[T: ClassTag]: Encoder[T]
----

`kryo` simply <<genericSerializer, creates an encoder>> that serializes objects of type `T` using Kryo (i.e. the `useKryo` flag is enabled).

NOTE: `kryo` is used when...FIXME

=== [[javaSerialization]] Creating Encoder Using Java Serialization -- `javaSerialization` Method

[source, scala]
----
javaSerialization[T: ClassTag]: Encoder[T]
----

`javaSerialization` simply <<genericSerializer, creates an encoder>> that serializes objects of type `T` using the generic Java serialization (i.e. the `useKryo` flag is disabled).

NOTE: `javaSerialization` is used when...FIXME

=== [[genericSerializer]] Creating Generic Encoder -- `genericSerializer` Internal Method

[source, scala]
----
genericSerializer[T: ClassTag](useKryo: Boolean): Encoder[T]
----

`genericSerializer`...FIXME

NOTE: `genericSerializer` is used when `Encoders` is requested for a generic encoder using <<kryo, Kryo>> and <<javaSerialization, Java Serialization>>.
