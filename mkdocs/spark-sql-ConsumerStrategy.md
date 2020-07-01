title: ConsumerStrategy

# ConsumerStrategy -- Kafka Consumer Providers

`ConsumerStrategy` is the <<contract, contract>> for <<implementations, Kafka Consumer providers>> that can <<createConsumer, create a Kafka Consumer>> given Kafka parameters.

[[contract]]
[source, scala]
----
package org.apache.spark.sql.kafka010

sealed trait ConsumerStrategy {
  def createConsumer(kafkaParams: ju.Map[String, Object]): Consumer[Array[Byte], Array[Byte]]
}
----

.ConsumerStrategy Contract
[cols="1m,2",options="header",width="100%"]
|===
| Property
| Description

| createConsumer
| [[createConsumer]] Creates a Kafka https://kafka.apache.org/20/javadoc/org/apache/kafka/clients/consumer/Consumer.html[Consumer] (of keys and values of type `Array[Byte]`)

Used exclusively when `KafkaOffsetReader` is requested to <<spark-sql-KafkaOffsetReader.adoc#createConsumer, creating a Kafka Consumer>>
|===

[[implementations]]
.ConsumerStrategies
[cols="1,2",options="header",width="100%"]
|===
| ConsumerStrategy
| createConsumer

| `AssignStrategy`
| [[AssignStrategy]] Uses link:++http://kafka.apache.org/20/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#assign-java.util.Collection-++[KafkaConsumer.assign(Collection<TopicPartition> partitions)]

| `SubscribeStrategy`
| [[SubscribeStrategy]] Uses link:++http://kafka.apache.org/20/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#subscribe-java.util.Collection-++[KafkaConsumer.subscribe(Collection<String> topics)]

| `SubscribePatternStrategy`
a| [[SubscribePatternStrategy]] Uses link:++http://kafka.apache.org/20/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#subscribe-java.util.regex.Pattern-org.apache.kafka.clients.consumer.ConsumerRebalanceListener-++[KafkaConsumer.subscribe(Pattern pattern, ConsumerRebalanceListener listener)] with `NoOpConsumerRebalanceListener`.

TIP: Refer to http://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html[java.util.regex.Pattern] for the format of supported topic subscription regex patterns.
|===

NOTE: `ConsumerStrategy` is a Scala *sealed trait* which means that all the <<implementations, implementations>> are in the same compilation unit (a single file).
