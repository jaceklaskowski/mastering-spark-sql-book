title: SQLConf

# SQLConf -- Internal Configuration Store

`SQLConf` is an *internal key-value configuration store* for <<parameters, parameters and hints>> used in Spark SQL.

[NOTE]
====
`SQLConf` is an internal part of Spark SQL and is not supposed to be used directly.

Spark SQL configuration is available through <<spark-sql-RuntimeConfig.adoc#, RuntimeConfig>> (the user-facing configuration management interface) that you can access using link:spark-sql-SparkSession.adoc#conf[SparkSession].

[source, scala]
----
scala> :type spark
org.apache.spark.sql.SparkSession

scala> :type spark.conf
org.apache.spark.sql.RuntimeConfig
----
====

You can access a `SQLConf` using:

. <<get, SQLConf.get>> (preferred) - the `SQLConf` of the current active `SparkSession`

. <<spark-sql-SparkSession.adoc#sessionState, SessionState>> - direct access through <<spark-sql-SparkSession.adoc#sessionState, SessionState>> of the `SparkSession` of your choice (that gives more flexibility on what `SparkSession` is used that can be different from the current active `SparkSession`)

[source, scala]
----
import org.apache.spark.sql.internal.SQLConf

// Use type-safe access to configuration properties
// using SQLConf.get.getConf
val parallelFileListingInStatsComputation = SQLConf.get.getConf(SQLConf.PARALLEL_FILE_LISTING_IN_STATS_COMPUTATION)

// or even simpler
SQLConf.get.parallelFileListingInStatsComputation
----

`SQLConf` offers methods to <<get, get>>, <<set, set>>, <<unset, unset>> or <<clear, clear>> values of configuration properties, but has also the <<accessor-methods, accessor methods>> to read the current value of a configuration property or hint.

[source, scala]
----
scala> :type spark
org.apache.spark.sql.SparkSession

// Direct access to the session SQLConf
val sqlConf = spark.sessionState.conf
scala> :type sqlConf
org.apache.spark.sql.internal.SQLConf

scala> println(sqlConf.offHeapColumnVectorEnabled)
false

// Or simply import the conf value
import spark.sessionState.conf

// accessing properties through accessor methods
scala> conf.numShufflePartitions
res1: Int = 200

// Prefer SQLConf.get (over direct access)
import org.apache.spark.sql.internal.SQLConf
val cc = SQLConf.get
scala> cc == conf
res4: Boolean = true

// setting properties using aliases
import org.apache.spark.sql.internal.SQLConf.SHUFFLE_PARTITIONS
conf.setConf(SHUFFLE_PARTITIONS, 2)
scala> conf.numShufflePartitions
res2: Int = 2

// unset aka reset properties to the default value
conf.unsetConf(SHUFFLE_PARTITIONS)
scala> conf.numShufflePartitions
res3: Int = 200
----

[[accessor-methods]]
.SQLConf's Accessor Methods
[cols="1m,1,1",options="header",width="100%"]
|===
| Name
| Parameter
| Description

| adaptiveExecutionEnabled
| link:spark-sql-properties.adoc#spark.sql.adaptive.enabled[spark.sql.adaptive.enabled]
| [[adaptiveExecutionEnabled]] Used exclusively when `EnsureRequirements` link:spark-sql-EnsureRequirements.adoc#withExchangeCoordinator[adds an ExchangeCoordinator] (for link:spark-sql-adaptive-query-execution.adoc[adaptive query execution])

| autoBroadcastJoinThreshold
| link:spark-sql-properties.adoc#spark.sql.autoBroadcastJoinThreshold[spark.sql.autoBroadcastJoinThreshold]
| [[autoBroadcastJoinThreshold]] Used exclusively in link:spark-sql-SparkStrategy-JoinSelection.adoc[JoinSelection] execution planning strategy

| autoSizeUpdateEnabled
| link:spark-sql-properties.adoc#spark.sql.statistics.size.autoUpdate.enabled[spark.sql.statistics.size.autoUpdate.enabled]
a| [[autoSizeUpdateEnabled]] Used when:

* `CommandUtils` is requested for link:spark-sql-CommandUtils.adoc#updateTableStats[updating existing table statistics]

* `AlterTableAddPartitionCommand` is executed

| avroCompressionCodec
| <<spark-sql-properties.adoc#spark.sql.avro.compression.codec, spark.sql.avro.compression.codec>>
| [[avroCompressionCodec]] Used exclusively when `AvroOptions` is requested for the <<spark-sql-AvroOptions.adoc#compression, compression>> configuration property (and it was not set explicitly)

| broadcastTimeout
| link:spark-sql-properties.adoc#spark.sql.broadcastTimeout[spark.sql.broadcastTimeout]
| [[broadcastTimeout]] Used exclusively in link:spark-sql-SparkPlan-BroadcastExchangeExec.adoc[BroadcastExchangeExec] (for broadcasting a table to executors).

| bucketingEnabled
| link:spark-sql-properties.adoc#spark.sql.sources.bucketing.enabled[spark.sql.sources.bucketing.enabled]
| [[bucketingEnabled]] Used when `FileSourceScanExec` is requested for the link:spark-sql-SparkPlan-FileSourceScanExec.adoc#inputRDD[input RDD] and to determine link:spark-sql-SparkPlan-FileSourceScanExec.adoc#outputPartitioning[output partitioning] and link:spark-sql-SparkPlan-FileSourceScanExec.adoc#outputOrdering[ordering]

| cacheVectorizedReaderEnabled
| link:spark-sql-properties.adoc#spark.sql.inMemoryColumnarStorage.enableVectorizedReader[spark.sql.inMemoryColumnarStorage.enableVectorizedReader]
| [[cacheVectorizedReaderEnabled]] Used exclusively when `InMemoryTableScanExec` physical operator is requested for link:spark-sql-SparkPlan-InMemoryTableScanExec.adoc#supportsBatch[supportsBatch] flag.

| caseSensitiveAnalysis
| link:spark-sql-properties.adoc#spark.sql.caseSensitive[spark.sql.caseSensitive]
a| [[caseSensitiveAnalysis]]

| cboEnabled
| link:spark-sql-properties.adoc#spark.sql.cbo.enabled[spark.sql.cbo.enabled]
a| [[cboEnabled]] Used in:

* link:spark-sql-Optimizer-ReorderJoin.adoc[ReorderJoin] logical plan optimization (and indirectly in `StarSchemaDetection` for `reorderStarJoins`)
* link:spark-sql-Optimizer-CostBasedJoinReorder.adoc[CostBasedJoinReorder] logical plan optimization

| columnBatchSize
| link:spark-sql-properties.adoc#spark.sql.inMemoryColumnarStorage.batchSize[spark.sql.inMemoryColumnarStorage.batchSize]
| [[columnBatchSize]] Used when...FIXME

| constraintPropagationEnabled
| link:spark-sql-properties.adoc#spark.sql.constraintPropagation.enabled[spark.sql.constraintPropagation.enabled]
a| [[constraintPropagationEnabled]][[CONSTRAINT_PROPAGATION_ENABLED]] Used when:

* link:spark-sql-Optimizer-InferFiltersFromConstraints.adoc[InferFiltersFromConstraints] logical optimization is executed

* `QueryPlanConstraints` is requested for the constraints

| CONVERT_METASTORE_ORC
| link:hive/configuration-properties.adoc#spark.sql.hive.convertMetastoreOrc[spark.sql.hive.convertMetastoreOrc]
| [[CONVERT_METASTORE_ORC]] Used when `RelationConversions` logical post-hoc evaluation rule is executed (and requested to link:hive/RelationConversions.adoc#isConvertible[isConvertible])

| CONVERT_METASTORE_PARQUET
| link:hive/configuration-properties.adoc#spark.sql.hive.convertMetastoreParquet[spark.sql.hive.convertMetastoreParquet]
| [[CONVERT_METASTORE_PARQUET]] Used when `RelationConversions` logical post-hoc evaluation rule is executed (and requested to link:hive/RelationConversions.adoc#isConvertible[isConvertible])

| dataFramePivotMaxValues
| link:spark-sql-properties.adoc#spark.sql.pivotMaxValues[spark.sql.pivotMaxValues]
| [[dataFramePivotMaxValues]] Used exclusively in link:spark-sql-RelationalGroupedDataset.adoc#pivot[pivot] operator.

| dataFrameRetainGroupColumns
| link:spark-sql-properties.adoc#spark.sql.retainGroupColumns[spark.sql.retainGroupColumns]
| [[dataFrameRetainGroupColumns]] Used exclusively in link:spark-sql-RelationalGroupedDataset.adoc[RelationalGroupedDataset] when creating the result `Dataset` (after `agg`, `count`, `mean`, `max`, `avg`, `min`, and `sum` operators).

| defaultSizeInBytes
| link:spark-sql-properties.adoc#spark.sql.defaultSizeInBytes[spark.sql.defaultSizeInBytes]
a| [[defaultSizeInBytes]] Used when:

* `DetermineTableStats` logical resolution rule could not compute the table size or <<spark.sql.statistics.fallBackToHdfs, spark.sql.statistics.fallBackToHdfs>> is turned off

* link:spark-sql-LogicalPlan-ExternalRDD.adoc#computeStats[ExternalRDD], link:spark-sql-LogicalPlan-LogicalRDD.adoc#computeStats[LogicalRDD] and `DataSourceV2Relation` are requested for statistics (i.e. `computeStats`)

*  (Spark Structured Streaming) `StreamingRelation`, `StreamingExecutionRelation`, `StreamingRelationV2` and `ContinuousExecutionRelation` are requested for statistics (i.e. `computeStats`)

* `DataSource` link:spark-sql-DataSource.adoc#resolveRelation[creates a HadoopFsRelation for FileFormat data source] (and builds a CatalogFileIndex when no table statistics are available)

* `BaseRelation` is requested for link:spark-sql-BaseRelation.adoc#sizeInBytes[an estimated size of this relation] (in bytes)

| enableRadixSort
| <<spark-sql-properties.adoc#spark.sql.sort.enableRadixSort, spark.sql.sort.enableRadixSort>>
a| [[enableRadixSort]] Used exclusively when `SortExec` physical operator is requested for a <<spark-sql-SparkPlan-SortExec.adoc#createSorter, UnsafeExternalRowSorter>>.

| exchangeReuseEnabled
| link:spark-sql-properties.adoc#spark.sql.exchange.reuse[spark.sql.exchange.reuse]
a| [[exchangeReuseEnabled]] Used when link:spark-sql-ReuseSubquery.adoc#apply[ReuseSubquery] and link:spark-sql-ReuseExchange.adoc#apply[ReuseExchange] physical optimizations are executed

NOTE: When disabled (i.e. `false`), `ReuseSubquery` and `ReuseExchange` physical optimizations do no optimizations.

| fallBackToHdfsForStatsEnabled
| link:spark-sql-properties.adoc#spark.sql.statistics.fallBackToHdfs[spark.sql.statistics.fallBackToHdfs]
| [[fallBackToHdfsForStatsEnabled]] Used exclusively when `DetermineTableStats` logical resolution rule is executed.

| fileCommitProtocolClass
| link:spark-sql-properties.adoc#spark.sql.sources.commitProtocolClass[spark.sql.sources.commitProtocolClass]
a| [[fileCommitProtocolClass]] Used (to instantiate a `FileCommitProtocol`) when:

* `SaveAsHiveFile` is requested to <<hive/SaveAsHiveFile.adoc#saveAsHiveFile, saveAsHiveFile>>

* <<spark-sql-LogicalPlan-InsertIntoHadoopFsRelationCommand.adoc#, InsertIntoHadoopFsRelationCommand>> logical command is executed

| filesMaxPartitionBytes
| <<spark-sql-properties.adoc#spark.sql.files.maxPartitionBytes, spark.sql.files.maxPartitionBytes>>
a| [[filesMaxPartitionBytes]] Used exclusively when <<spark-sql-SparkPlan-FileSourceScanExec.adoc#, FileSourceScanExec>> leaf physical operator is requested to <<spark-sql-SparkPlan-FileSourceScanExec.adoc#createNonBucketedReadRDD, create an RDD for non-bucketed reads>>

| filesOpenCostInBytes
| <<spark-sql-properties.adoc#spark.sql.files.openCostInBytes, spark.sql.files.openCostInBytes>>
a| [[filesOpenCostInBytes]] Used exclusively when <<spark-sql-SparkPlan-FileSourceScanExec.adoc#, FileSourceScanExec>> leaf physical operator is requested to <<spark-sql-SparkPlan-FileSourceScanExec.adoc#createNonBucketedReadRDD, create an RDD for non-bucketed reads>>

| histogramEnabled
| link:spark-sql-properties.adoc#spark.sql.statistics.histogram.enabled[spark.sql.statistics.histogram.enabled]
| [[histogramEnabled]] Used exclusively when `AnalyzeColumnCommand` logical command is link:spark-sql-LogicalPlan-AnalyzeColumnCommand.adoc#run[executed].

| histogramNumBins
| link:spark-sql-properties.adoc#spark.sql.statistics.histogram.numBins[spark.sql.statistics.histogram.numBins]
| [[histogramNumBins]] Used exclusively when `AnalyzeColumnCommand` is link:spark-sql-LogicalPlan-AnalyzeColumnCommand.adoc#run[executed] with link:spark-sql-properties.adoc#spark.sql.statistics.histogram.enabled[spark.sql.statistics.histogram.enabled] turned on (and link:spark-sql-LogicalPlan-AnalyzeColumnCommand.adoc#computePercentiles[calculates percentiles]).

| hugeMethodLimit
| link:spark-sql-properties.adoc#spark.sql.codegen.hugeMethodLimit[spark.sql.codegen.hugeMethodLimit]
| [[hugeMethodLimit]] Used exclusively when `WholeStageCodegenExec` unary physical operator is requested to <<spark-sql-SparkPlan-WholeStageCodegenExec.adoc#doExecute, execute>> (and generate a `RDD[InternalRow]`), i.e. when the compiled function exceeds this threshold, the whole-stage codegen is deactivated for this subtree of the query plan.

| ignoreCorruptFiles
| link:spark-sql-properties.adoc#spark.sql.files.ignoreCorruptFiles[spark.sql.files.ignoreCorruptFiles]
a| [[ignoreCorruptFiles]] Used when:

* `FileScanRDD` is link:spark-sql-FileScanRDD.adoc#ignoreCorruptFiles[created] (and then to link:spark-sql-FileScanRDD.adoc#compute[compute a partition])

* `OrcFileFormat` is requested to link:spark-sql-OrcFileFormat.adoc#inferSchema[inferSchema] and link:spark-sql-OrcFileFormat.adoc#buildReader[buildReader]

* `ParquetFileFormat` is requested to link:spark-sql-ParquetFileFormat.adoc#mergeSchemasInParallel[mergeSchemasInParallel]

| ignoreMissingFiles
| link:spark-sql-properties.adoc#spark.sql.files.ignoreMissingFiles[spark.sql.files.ignoreMissingFiles]
| [[ignoreMissingFiles]] Used exclusively when `FileScanRDD` is link:spark-sql-FileScanRDD.adoc#ignoreMissingFiles[created] (and then to link:spark-sql-FileScanRDD.adoc#compute[compute a partition])

| inMemoryPartitionPruning
| link:spark-sql-properties.adoc#spark.sql.inMemoryColumnarStorage.partitionPruning[spark.sql.inMemoryColumnarStorage.partitionPruning]
| [[inMemoryPartitionPruning]] Used exclusively when `InMemoryTableScanExec` physical operator is requested for link:spark-sql-SparkPlan-InMemoryTableScanExec.adoc#filteredCachedBatches[filtered cached column batches] (as a `RDD[CachedBatch]`).

| isParquetBinaryAsString
| link:spark-sql-properties.adoc#spark.sql.parquet.binaryAsString[spark.sql.parquet.binaryAsString]
| [[isParquetBinaryAsString]]

| isParquetINT96AsTimestamp
| link:spark-sql-properties.adoc#spark.sql.parquet.int96AsTimestamp[spark.sql.parquet.int96AsTimestamp]
| [[isParquetINT96AsTimestamp]]

| isParquetINT96TimestampConversion
| link:spark-sql-properties.adoc#spark.sql.parquet.int96TimestampConversion[spark.sql.parquet.int96TimestampConversion]
| [[isParquetINT96TimestampConversion]] Used exclusively when `ParquetFileFormat` is requested to link:spark-sql-ParquetFileFormat.adoc#buildReaderWithPartitionValues[build a data reader with partition column values appended].

| joinReorderEnabled
| link:spark-sql-properties.adoc#spark.sql.cbo.joinReorder.enabled[spark.sql.cbo.joinReorder.enabled]
| [[joinReorderEnabled]] Used exclusively in link:spark-sql-Optimizer-CostBasedJoinReorder.adoc[CostBasedJoinReorder] logical plan optimization

| limitScaleUpFactor
| link:spark-sql-properties.adoc#spark.sql.limit.scaleUpFactor[spark.sql.limit.scaleUpFactor]
| [[limitScaleUpFactor]] Used exclusively when a physical operator is requested link:spark-sql-SparkPlan.adoc#executeTake[the first n rows as an array].

| manageFilesourcePartitions
| link:hive/configuration-properties.adoc#spark.sql.hive.manageFilesourcePartitions[spark.sql.hive.manageFilesourcePartitions]
a| [[manageFilesourcePartitions]][[HIVE_MANAGE_FILESOURCE_PARTITIONS]] Used when:

* `HiveMetastoreCatalog` is requested to link:hive/HiveMetastoreCatalog.adoc#convertToLogicalRelation[convert a HiveTableRelation to a LogicalRelation over a HadoopFsRelation]

* <<spark-sql-LogicalPlan-CreateDataSourceTableCommand.adoc#, CreateDataSourceTableCommand>>, <<spark-sql-LogicalPlan-CreateDataSourceTableAsSelectCommand.adoc#, CreateDataSourceTableAsSelectCommand>> and <<spark-sql-LogicalPlan-InsertIntoHadoopFsRelationCommand.adoc#, InsertIntoHadoopFsRelationCommand>> logical commands are executed

* `DDLUtils` utility is used to link:spark-sql-DDLUtils.adoc#verifyPartitionProviderIsHive[verifyPartitionProviderIsHive]

* `DataSource` is requested to <<spark-sql-DataSource.adoc#resolveRelation, resolve a relation>> (for file-based data source tables and creates a `HadoopFsRelation`)

* `FileStatusCache` is requested to `getOrCreate`

| maxRecordsPerFile
| <<spark-sql-properties.adoc#spark.sql.files.maxRecordsPerFile, spark.sql.files.maxRecordsPerFile>>
a| [[maxRecordsPerFile]][[MAX_RECORDS_PER_FILE]] Used when `FileFormatWriter` utility is used to <<spark-sql-FileFormatWriter.adoc#write, write the result of a structured query>>

| metastorePartitionPruning
| link:spark-sql-properties.adoc#spark.sql.hive.metastorePartitionPruning[spark.sql.hive.metastorePartitionPruning]
a| [[metastorePartitionPruning]][[HIVE_METASTORE_PARTITION_PRUNING]] Used when link:hive/HiveTableScanExec.adoc[HiveTableScanExec] physical operator is executed with a partitioned table (and requested for link:HiveTableScanExec.adoc#rawPartitions[rawPartitions])

| minNumPostShufflePartitions
| <<spark-sql-properties.adoc#spark.sql.adaptive.minNumPostShufflePartitions, spark.sql.adaptive.minNumPostShufflePartitions>>
a| [[minNumPostShufflePartitions]] Used exclusively when `EnsureRequirements` physical query optimization is requested to <<spark-sql-EnsureRequirements.adoc#withExchangeCoordinator, add an ExchangeCoordinator>> for <<spark-sql-adaptive-query-execution.adoc#, Adaptive Query Execution>>.

| numShufflePartitions
| link:spark-sql-properties.adoc#spark.sql.shuffle.partitions[spark.sql.shuffle.partitions]
a| [[numShufflePartitions]] Used in:

* Dataset's link:spark-sql-dataset-operators.adoc#repartition[repartition] operator (for a link:spark-sql-LogicalPlan-Repartition-RepartitionByExpression.adoc#RepartitionByExpression[RepartitionByExpression] logical operator)
* link:spark-sql-SparkSqlAstBuilder.adoc#withRepartitionByExpression[SparkSqlAstBuilder] (for a link:spark-sql-LogicalPlan-Repartition-RepartitionByExpression.adoc#RepartitionByExpression[RepartitionByExpression] logical operator)
* link:spark-sql-SparkStrategy-JoinSelection.adoc#canBuildLocalHashMap[JoinSelection] execution planning strategy
* link:spark-sql-LogicalPlan-RunnableCommand.adoc#SetCommand[SetCommand] logical command
* link:spark-sql-EnsureRequirements.adoc#defaultNumPreShufflePartitions[EnsureRequirements] physical plan optimization

| offHeapColumnVectorEnabled
| link:spark-sql-properties.adoc#spark.sql.columnVector.offheap.enabled[spark.sql.columnVector.offheap.enabled]
a| [[offHeapColumnVectorEnabled]] Used when:

* `InMemoryTableScanExec` is requested for the link:spark-sql-SparkPlan-InMemoryTableScanExec.adoc#vectorTypes[vectorTypes] and the link:spark-sql-SparkPlan-InMemoryTableScanExec.adoc#inputRDD[input RDD]

* `OrcFileFormat` is requested to link:spark-sql-OrcFileFormat.adoc#buildReaderWithPartitionValues[build a data reader with partition column values appended]

* `ParquetFileFormat` is requested for link:spark-sql-SparkPlan-ParquetFileFormat.adoc#vectorTypes[vectorTypes] and link:spark-sql-SparkPlan-ParquetFileFormat.adoc#buildReaderWithPartitionValues[build a data reader with partition column values appended]

| optimizerExcludedRules
| <<spark-sql-properties.adoc#spark.sql.optimizer.excludedRules, spark.sql.optimizer.excludedRules>>
a| [[optimizerExcludedRules]] Used exclusively when `Optimizer` is requested for the <<spark-sql-Optimizer.adoc#batches, optimization batches>>

| optimizerInSetConversionThreshold
| link:spark-sql-properties.adoc#spark.sql.optimizer.inSetConversionThreshold[spark.sql.optimizer.inSetConversionThreshold]
| [[optimizerInSetConversionThreshold]] Used exclusively when `OptimizeIn` logical query optimization is link:spark-sql-Optimizer-OptimizeIn.adoc#apply[applied to a logical plan] (and replaces an link:spark-sql-Expression-In.adoc[In] predicate expression with an link:spark-sql-Expression-InSet.adoc[InSet])

|
|
a| [[ORC_IMPLEMENTATION]]

Supported values:

* `native` for xref:spark-sql-OrcFileFormat.adoc[OrcFileFormat]
* `hive` for `org.apache.spark.sql.hive.orc.OrcFileFormat`

| parallelFileListingInStatsComputation
| <<spark-sql-properties.adoc#spark.sql.statistics.parallelFileListingInStatsComputation.enabled, spark.sql.statistics.parallelFileListingInStatsComputation.enabled>>
a| [[parallelFileListingInStatsComputation]] Used exclusively when `CommandUtils` helper object is requested to <<calculateTotalSize, calculate the total size of a table (with partitions)>> (for <<spark-sql-LogicalPlan-AnalyzeColumnCommand.adoc#, AnalyzeColumnCommand>> and <<spark-sql-LogicalPlan-AnalyzeTableCommand.adoc#, AnalyzeTableCommand>> commands)

| parquetFilterPushDown
| link:spark-sql-properties.adoc#spark.sql.parquet.filterPushdown[spark.sql.parquet.filterPushdown]
| [[parquetFilterPushDown]] Used exclusively when `ParquetFileFormat` is requested to link:spark-sql-ParquetFileFormat.adoc#buildReaderWithPartitionValues[build a data reader with partition column values appended].

| parquetFilterPushDownDate
| <<spark-sql-properties.adoc#spark.sql.parquet.filterPushdown.date, spark.sql.parquet.filterPushdown.date>>
| [[parquetFilterPushDownDate]] Used exclusively when `ParquetFileFormat` is requested to <<spark-sql-ParquetFileFormat.adoc#buildReaderWithPartitionValues, build a data reader with partition column values appended>>.

| parquetRecordFilterEnabled
| link:spark-sql-properties.adoc#spark.sql.parquet.recordLevelFilter.enabled[spark.sql.parquet.recordLevelFilter.enabled]
| [[parquetRecordFilterEnabled]] Used exclusively when `ParquetFileFormat` is requested to link:spark-sql-ParquetFileFormat.adoc#buildReaderWithPartitionValues[build a data reader with partition column values appended].

| parquetVectorizedReaderBatchSize
| <<spark-sql-properties.adoc#spark.sql.parquet.columnarReaderBatchSize, spark.sql.parquet.columnarReaderBatchSize>>
a| [[parquetVectorizedReaderBatchSize]] Used exclusively when `ParquetFileFormat` is requested for a <<spark-sql-ParquetFileFormat.adoc#buildReaderWithPartitionValues, data reader>> (and creates a <<spark-sql-VectorizedParquetRecordReader.adoc#, VectorizedParquetRecordReader>> for <<spark-sql-vectorized-parquet-reader.adoc#, Vectorized Parquet Decoding>>)

| parquetVectorizedReaderEnabled
| link:spark-sql-properties.adoc#spark.sql.parquet.enableVectorizedReader[spark.sql.parquet.enableVectorizedReader]
a| [[parquetVectorizedReaderEnabled]] Used when:

* `FileSourceScanExec` is requested for link:spark-sql-SparkPlan-FileSourceScanExec.adoc#needsUnsafeRowConversion[needsUnsafeRowConversion] flag

* `ParquetFileFormat` is requested for link:spark-sql-ParquetFileFormat.adoc#supportBatch[supportBatch] flag and link:spark-sql-ParquetFileFormat.adoc#buildReaderWithPartitionValues[build a data reader with partition column values appended]

| partitionOverwriteMode
| <<spark-sql-properties.adoc#spark.sql.sources.partitionOverwriteMode, spark.sql.sources.partitionOverwriteMode>>
a| [[partitionOverwriteMode]] Used exclusively when <<spark-sql-LogicalPlan-InsertIntoHadoopFsRelationCommand.adoc#, InsertIntoHadoopFsRelationCommand>> logical command is executed

| preferSortMergeJoin
| link:spark-sql-properties.adoc#spark.sql.join.preferSortMergeJoin[spark.sql.join.preferSortMergeJoin]
| [[preferSortMergeJoin]] Used exclusively in link:spark-sql-SparkStrategy-JoinSelection.adoc[JoinSelection] execution planning strategy to prefer sort merge join over shuffle hash join.

| replaceDatabricksSparkAvroEnabled
| xref:spark-sql-properties.adoc#spark.sql.legacy.replaceDatabricksSparkAvro.enabled[spark.sql.legacy.replaceDatabricksSparkAvro.enabled]
| [[replaceDatabricksSparkAvroEnabled]][[LEGACY_REPLACE_DATABRICKS_SPARK_AVRO_ENABLED]]

| replaceExceptWithFilter
| link:spark-sql-properties.adoc#spark.sql.optimizer.replaceExceptWithFilter[spark.sql.optimizer.replaceExceptWithFilter]
| [[replaceExceptWithFilter]][[REPLACE_EXCEPT_WITH_FILTER]] Used when link:spark-sql-Optimizer-ReplaceExceptWithFilter.adoc[ReplaceExceptWithFilter] is executed

| runSQLonFile
| link:spark-sql-properties.adoc#spark.sql.runSQLOnFiles[spark.sql.runSQLOnFiles]
a| [[runSQLonFile]] Used when:

* `ResolveRelations` does link:spark-sql-Analyzer-ResolveRelations.adoc#isRunningDirectlyOnFiles[isRunningDirectlyOnFiles]

* `ResolveSQLOnFile` does link:spark-sql-Analyzer-ResolveSQLOnFile.adoc#maybeSQLFile[maybeSQLFile]

| sessionLocalTimeZone
| <<spark-sql-properties.adoc#spark.sql.session.timeZone, spark.sql.session.timeZone>>
a| [[sessionLocalTimeZone]]

| starSchemaDetection
| link:spark-sql-properties.adoc#spark.sql.cbo.starSchemaDetection[spark.sql.cbo.starSchemaDetection]
| [[starSchemaDetection]] Used exclusively in link:spark-sql-Optimizer-ReorderJoin.adoc[ReorderJoin] logical plan optimization (and indirectly in `StarSchemaDetection`)

| stringRedactionPattern
| link:spark-sql-properties.adoc#spark.sql.redaction.string.regex[spark.sql.redaction.string.regex]
a| [[stringRedactionPattern]] Used when:

* `DataSourceScanExec` is requested to link:spark-sql-SparkPlan-DataSourceScanExec.adoc#redact[redact sensitive information] (in text representations)

* `QueryExecution` is requested to link:spark-sql-QueryExecution.adoc#withRedaction[redact sensitive information] (in text representations)

| subexpressionEliminationEnabled
| link:spark-sql-properties.adoc#spark.sql.subexpressionElimination.enabled[spark.sql.subexpressionElimination.enabled]
| [[subexpressionEliminationEnabled]] Used exclusively when `SparkPlan` is requested for link:spark-sql-SparkPlan.adoc#subexpressionEliminationEnabled[subexpressionEliminationEnabled] flag.

| supportQuotedRegexColumnName
| link:spark-sql-properties.adoc#spark.sql.parser.quotedRegexColumnNames[spark.sql.parser.quotedRegexColumnNames]
a| [[supportQuotedRegexColumnName]] Used when:

* <<spark-sql-Dataset-untyped-transformations.adoc#col, Dataset.col>> operator is used

* `AstBuilder` is requested to parse a <<spark-sql-AstBuilder.adoc#visitDereference, dereference>> and <<spark-sql-AstBuilder.adoc#visitColumnReference, column reference>> in a SQL statement

| targetPostShuffleInputSize
| <<spark-sql-properties.adoc#spark.sql.adaptive.shuffle.targetPostShuffleInputSize, spark.sql.adaptive.shuffle.targetPostShuffleInputSize>>
| [[targetPostShuffleInputSize]] Used exclusively when `EnsureRequirements` physical query optimization is requested to <<spark-sql-EnsureRequirements.adoc#withExchangeCoordinator, add an ExchangeCoordinator>> for <<spark-sql-adaptive-query-execution.adoc#, Adaptive Query Execution>>.

| truncateTableIgnorePermissionAcl
| xref:spark-sql-properties.adoc#spark.sql.truncateTable.ignorePermissionAcl.enabled[spark.sql.truncateTable.ignorePermissionAcl.enabled]
| [[truncateTableIgnorePermissionAcl]][[TRUNCATE_TABLE_IGNORE_PERMISSION_ACL]] Used when xref:spark-sql-LogicalPlan-TruncateTableCommand.adoc[TruncateTableCommand] is executed

| useCompression
| link:spark-sql-properties.adoc#spark.sql.inMemoryColumnarStorage.compressed[spark.sql.inMemoryColumnarStorage.compressed]
| [[useCompression]] Used when...FIXME

| wholeStageEnabled
| link:spark-sql-properties.adoc#spark.sql.codegen.wholeStage[spark.sql.codegen.wholeStage]
a| [[wholeStageEnabled]] Used in:

* link:spark-sql-CollapseCodegenStages.adoc[CollapseCodegenStages] to control codegen
* link:spark-sql-ParquetFileFormat.adoc[ParquetFileFormat] to control row batch reading

| wholeStageFallback
| link:spark-sql-properties.adoc#spark.sql.codegen.fallback[spark.sql.codegen.fallback]
| [[wholeStageFallback]] Used exclusively when `WholeStageCodegenExec` is link:spark-sql-SparkPlan-WholeStageCodegenExec.adoc#doExecute[executed].

| wholeStageMaxNumFields
| link:spark-sql-properties.adoc#spark.sql.codegen.maxFields[spark.sql.codegen.maxFields]
a| [[wholeStageMaxNumFields]] Used in:

* link:spark-sql-CollapseCodegenStages.adoc[CollapseCodegenStages] to control codegen
* link:spark-sql-ParquetFileFormat.adoc[ParquetFileFormat] to control row batch reading

| wholeStageSplitConsumeFuncByOperator
| link:spark-sql-properties.adoc#spark.sql.codegen.splitConsumeFuncByOperator[spark.sql.codegen.splitConsumeFuncByOperator]
| [[wholeStageSplitConsumeFuncByOperator]] Used exclusively when `CodegenSupport` is requested to link:spark-sql-CodegenSupport.adoc#consume[consume]

| wholeStageUseIdInClassName
| link:spark-sql-properties.adoc#spark.sql.codegen.useIdInClassName[spark.sql.codegen.useIdInClassName]
| [[wholeStageUseIdInClassName]] Used exclusively when `WholeStageCodegenExec` is requested to <<spark-sql-SparkPlan-WholeStageCodegenExec.adoc#doCodeGen, generate the Java source code for the child physical plan subtree>> (when <<spark-sql-SparkPlan-WholeStageCodegenExec.adoc#creating-instance, created>>)

| windowExecBufferInMemoryThreshold
| link:spark-sql-properties.adoc#spark.sql.windowExec.buffer.in.memory.threshold[spark.sql.windowExec.buffer.in.memory.threshold]
| [[windowExecBufferInMemoryThreshold]] Used exclusively when `WindowExec` unary physical operator is <<spark-sql-SparkPlan-WindowExec.adoc#doExecute, executed>>.

| windowExecBufferSpillThreshold
| link:spark-sql-properties.adoc#spark.sql.windowExec.buffer.spill.threshold[spark.sql.windowExec.buffer.spill.threshold]
| [[windowExecBufferSpillThreshold]] Used exclusively when `WindowExec` unary physical operator is <<spark-sql-SparkPlan-WindowExec.adoc#doExecute, executed>>.

| useObjectHashAggregation
| link:spark-sql-properties.adoc#spark.sql.execution.useObjectHashAggregateExec[spark.sql.execution.useObjectHashAggregateExec]
| [[useObjectHashAggregation]] Used exclusively when `Aggregation` execution planning strategy is <<spark-sql-SparkStrategy-Aggregation.adoc#apply, executed>> (and uses `AggUtils` to <<spark-sql-AggUtils.adoc#createAggregate, create an aggregation physical operator>>).
|===

=== [[getConfString]][[getConf]][[getAllConfs]][[getAllDefinedConfs]] Getting Parameters and Hints

You can get the current parameters and hints using the following family of `get` methods.

[source, scala]
----
getConf[T](entry: ConfigEntry[T], defaultValue: T): T
getConf[T](entry: ConfigEntry[T]): T
getConf[T](entry: OptionalConfigEntry[T]): Option[T]
getConfString(key: String): String
getConfString(key: String, defaultValue: String): String
getAllConfs: immutable.Map[String, String]
getAllDefinedConfs: Seq[(String, String, String)]
----

=== [[set]] Setting Parameters and Hints

You can set parameters and hints using the following family of `set` methods.

[source, scala]
----
setConf(props: Properties): Unit
setConfString(key: String, value: String): Unit
setConf[T](entry: ConfigEntry[T], value: T): Unit
----

=== [[unset]] Unsetting Parameters and Hints

You can unset parameters and hints using the following family of `unset` methods.

[source, scala]
----
unsetConf(key: String): Unit
unsetConf(entry: ConfigEntry[_]): Unit
----

=== [[clear]] Clearing All Parameters and Hints

[source, scala]
----
clear(): Unit
----

You can use `clear` to remove all the parameters and hints in `SQLConf`.

=== [[redactOptions]] Redacting Data Source Options with Sensitive Information -- `redactOptions` Method

[source, scala]
----
redactOptions(options: Map[String, String]): Map[String, String]
----

`redactOptions` takes the values of the <<spark-sql-properties.adoc#spark.sql.redaction.options.regex, spark.sql.redaction.options.regex>> and `spark.redaction.regex` configuration properties.

For every regular expression (in the order), `redactOptions` redacts sensitive information, i.e. finds the first match of a regular expression pattern in every option key or value and if either matches replaces the value with `*********(redacted)`.

NOTE: `redactOptions` is used exclusively when `SaveIntoDataSourceCommand` logical command is requested for the <<spark-sql-LogicalPlan-SaveIntoDataSourceCommand.adoc#simpleString, simple description>>.
