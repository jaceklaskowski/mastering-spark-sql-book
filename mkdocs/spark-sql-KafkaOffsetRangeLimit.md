# KafkaOffsetRangeLimit

`KafkaOffsetRangeLimit` is the desired <<extensions, offset range limits>> for starting, ending, and specific offsets.

[[extensions]]
.KafkaOffsetRangeLimits
[cols="1m,2",options="header",width="100%"]
|===
| KafkaOffsetRangeLimit
| Description

| EarliestOffsetRangeLimit
| [[EarliestOffsetRangeLimit]] Bind to the *earliest* offset

| LatestOffsetRangeLimit
| [[LatestOffsetRangeLimit]] Bind to the *latest* offset

| SpecificOffsetRangeLimit
| [[SpecificOffsetRangeLimit]] Bind to specific offsets

Takes `partitionOffsets` (as `Map[TopicPartition, Long]`) when created.
|===

`KafkaOffsetRangeLimit` is "created" (i.e. mapped to from a human-readable text representation) when `KafkaSourceProvider` is requested to <<spark-sql-KafkaSourceProvider.adoc#getKafkaOffsetRangeLimit, getKafkaOffsetRangeLimit>>.

`KafkaOffsetRangeLimit` defines two constants to denote offset range limits that are resolved via Kafka:

* [[LATEST]] `-1L` for the latest offset

* [[EARLIEST]] `-2L` for the earliest offset

NOTE: `KafkaOffsetRangeLimit` is a Scala *sealed trait* which means that all the <<extensions, extensions>> are in the same compilation unit (a single file).
