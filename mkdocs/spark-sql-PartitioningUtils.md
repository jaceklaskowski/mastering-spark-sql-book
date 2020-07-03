# PartitioningUtils Helper Object

`PartitioningUtils` is...FIXME

=== [[validatePartitionColumn]] `validatePartitionColumn` Method

[source, scala]
----
validatePartitionColumn(
  schema: StructType,
  partitionColumns: Seq[String],
  caseSensitive: Boolean): Unit
----

`validatePartitionColumn`...FIXME

NOTE: `validatePartitionColumn` is used when...FIXME

=== [[parsePartitions]] `parsePartitions` Method

[source, scala]
----
parsePartitions(
  paths: Seq[Path],
  typeInference: Boolean,
  basePaths: Set[Path],
  timeZoneId: String): PartitionSpec  // <1>
parsePartitions(
  paths: Seq[Path],
  typeInference: Boolean,
  basePaths: Set[Path],
  timeZone: TimeZone): PartitionSpec
----
<1> Uses `parsePartitions` with `timeZoneId` mapped to a `TimeZone`

`parsePartitions`...FIXME

NOTE: `parsePartitions` is used when...FIXME
