# ComplexTypedAggregateExpression

`ComplexTypedAggregateExpression` is...FIXME

`ComplexTypedAggregateExpression` is <<creating-instance, created>> when...FIXME

=== [[creating-instance]] Creating ComplexTypedAggregateExpression Instance

`ComplexTypedAggregateExpression` takes the following when created:

* [[aggregator]] link:spark-sql-Aggregator.adoc[Aggregator]
* [[inputDeserializer]] Optional input deserializer link:spark-sql-Expression.adoc[expression]
* [[inputClass]] Optional Java class for the input
* [[inputSchema]] Optional link:spark-sql-StructType.adoc[schema] for the input
* [[bufferSerializer]] Buffer serializer (as a collection of link:spark-sql-Expression-NamedExpression.adoc[named expressions])
* [[bufferDeserializer]] Buffer deserializer link:spark-sql-Expression.adoc[expression]
* [[outputSerializer]] Output serializer (as a collection of link:spark-sql-Expression.adoc[expressions])
* [[dataType]] link:spark-sql-DataType.adoc[DataType]
* [[nullable]] `nullable` flag
* [[mutableAggBufferOffset]] `mutableAggBufferOffset` (default: `0`)
* [[inputAggBufferOffset]] `inputAggBufferOffset` (default: `0`)
