# SimpleTypedAggregateExpression

`SimpleTypedAggregateExpression` is...FIXME

`SimpleTypedAggregateExpression` is <<creating-instance, created>> when...FIXME

[[internal-registries]]
.SimpleTypedAggregateExpression's Internal Properties (e.g. Registries, Counters and Flags)
[cols="1m,2",options="header",width="100%"]
|===
| Name
| Description

| evaluateExpression
| [[evaluateExpression]] <<spark-sql-Expression.adoc#, Expression>>

| resultObjToRow
| [[resultObjToRow]] <<spark-sql-UnsafeProjection.adoc#, UnsafeProjection>>
|===

=== [[creating-instance]] Creating SimpleTypedAggregateExpression Instance

`SimpleTypedAggregateExpression` takes the following when created:

* [[aggregator]] link:spark-sql-Aggregator.adoc[Aggregator]
* [[inputDeserializer]] Optional input deserializer link:spark-sql-Expression.adoc[expression]
* [[inputClass]] Optional Java class for the input
* [[inputSchema]] Optional link:spark-sql-StructType.adoc[schema] for the input
* [[bufferSerializer]] Buffer serializer (as a collection of link:spark-sql-Expression-NamedExpression.adoc[named expressions])
* [[bufferDeserializer]] Buffer deserializer link:spark-sql-Expression.adoc[expression]
* [[outputSerializer]] Output serializer (as a collection of link:spark-sql-Expression.adoc[expressions])
* [[dataType]] link:spark-sql-DataType.adoc[DataType]
* [[nullable]] `nullable` flag
