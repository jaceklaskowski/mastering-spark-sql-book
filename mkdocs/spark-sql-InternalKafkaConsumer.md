# InternalKafkaConsumer

`InternalKafkaConsumer` is...FIXME

=== [[get]] Getting Single Kafka ConsumerRecord -- `get` Method

[source, scala]
----
get(
  offset: Long,
  untilOffset: Long,
  pollTimeoutMs: Long,
  failOnDataLoss: Boolean): ConsumerRecord[Array[Byte], Array[Byte]]
----

`get`...FIXME

NOTE: `get` is used when...FIXME

=== [[getAvailableOffsetRange]] Getting Single AvailableOffsetRange -- `getAvailableOffsetRange` Method

[source, scala]
----
getAvailableOffsetRange(): AvailableOffsetRange
----

`getAvailableOffsetRange`...FIXME

NOTE: `getAvailableOffsetRange` is used when...FIXME
