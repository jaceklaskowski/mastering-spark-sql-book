# Data Source API V2

*Data Source API V2* (_DataSource API V2_ or _DataSource V2_) is a new API for data sources in Spark SQL with the following abstractions (_contracts_):

* <<spark-sql-DataSourceV2.adoc#, DataSourceV2>> marker interface

* <<spark-sql-ReadSupport.adoc#, ReadSupport>>

* <<spark-sql-DataSourceReader.adoc#, DataSourceReader>>

* <<spark-sql-WriteSupport.adoc#, WriteSupport>>

* <<spark-sql-DataSourceWriter.adoc#, DataSourceWriter>>

* <<spark-sql-SessionConfigSupport.adoc#, SessionConfigSupport>>

* <<spark-sql-DataSourceV2StringFormat.adoc#, DataSourceV2StringFormat>>

* <<spark-sql-InputPartition.adoc#, InputPartition>>

NOTE: The work on Data Source API V2 was tracked under https://issues.apache.org/jira/browse/SPARK-15689[SPARK-15689 Data source API v2] that was fixed in Apache Spark 2.3.0.

NOTE: Data Source API V2 is already heavily used in Spark Structured Streaming.

=== Query Planning and Execution

Data Source API V2 relies on the <<spark-sql-SparkStrategy-DataSourceV2Strategy.adoc#, DataSourceV2Strategy>> execution planning strategy for query planning.

==== Data Reading

Data Source API V2 uses <<spark-sql-LogicalPlan-DataSourceV2Relation.adoc#, DataSourceV2Relation>> logical operator to represent data reading (aka _data scan_).

`DataSourceV2Relation` is planned (_translated_) to a <<spark-sql-SparkPlan-ProjectExec.adoc#, ProjectExec>> with a <<spark-sql-SparkPlan-DataSourceV2ScanExec.adoc#, DataSourceV2ScanExec>> physical operator (possibly under the <<spark-sql-SparkPlan-FilterExec.adoc#, FilterExec>> operator) when <<spark-sql-SparkStrategy-DataSourceV2Strategy.adoc#, DataSourceV2Strategy>> execution planning strategy is requested to <<spark-sql-SparkStrategy-DataSourceV2Strategy.adoc#apply-DataSourceV2Relation, plan a logical plan>>.

<<spark-sql-SparkPlan-DataSourceV2ScanExec.adoc#doExecute, At execution>>, `DataSourceV2ScanExec` physical operator creates a <<spark-sql-DataSourceRDD.adoc#, DataSourceRDD>> (or a `ContinuousReader` for Spark Structured Streaming).

`DataSourceRDD` uses <<spark-sql-InputPartition.adoc#, InputPartitions>> for <<spark-sql-DataSourceRDD.adoc#getPartitions, partitions>>, <<spark-sql-DataSourceRDD.adoc#getPreferredLocations, preferred locations>>, and <<spark-sql-DataSourceRDD.adoc#compute, computing partitions>>.

==== Data Writing

Data Source API V2 uses <<spark-sql-LogicalPlan-WriteToDataSourceV2.adoc#, WriteToDataSourceV2>> and <<spark-sql-LogicalPlan-AppendData.adoc#, AppendData>> logical operators to represent data writing (over a <<spark-sql-LogicalPlan-DataSourceV2Relation.adoc#, DataSourceV2Relation>> logical operator). As of Spark SQL 2.4.0, `WriteToDataSourceV2` operator was deprecated for the more specific `AppendData` operator (compare _"data writing"_ to _"data append"_ which is certainly more specific).

NOTE: One of the differences between `WriteToDataSourceV2` and `AppendData` logical operators is that the former (`WriteToDataSourceV2`) uses <<spark-sql-LogicalPlan-WriteToDataSourceV2.adoc#writer, DataSourceWriter>> directly while the latter (`AppendData`) uses <<spark-sql-LogicalPlan-AppendData.adoc#table, DataSourceV2Relation>> to <<spark-sql-LogicalPlan-DataSourceV2Relation.adoc#newWriter, get the DataSourceWriter from>>.

<<spark-sql-LogicalPlan-WriteToDataSourceV2.adoc#, WriteToDataSourceV2>> and <<spark-sql-LogicalPlan-AppendData.adoc#, AppendData>> (with <<spark-sql-LogicalPlan-DataSourceV2Relation.adoc#, DataSourceV2Relation>>) logical operators are planned as (_translated to_) a <<spark-sql-SparkPlan-WriteToDataSourceV2Exec.adoc#, WriteToDataSourceV2Exec>> physical operator.

<<spark-sql-SparkPlan-WriteToDataSourceV2Exec.adoc#doExecute, At execution>>, `WriteToDataSourceV2Exec` physical operator...FIXME

=== [[filter-pushdown]] Filter Pushdown Performance Optimization

Data Source API V2 supports *filter pushdown* performance optimization for <<spark-sql-DataSourceReader.adoc#, DataSourceReaders>> with <<spark-sql-SupportsPushDownFilters.adoc#, SupportsPushDownFilters>> (that is applied when <<spark-sql-SparkStrategy-DataSourceV2Strategy.adoc#, DataSourceV2Strategy>> execution planning strategy is requested to plan a <<spark-sql-SparkStrategy-DataSourceV2Strategy.adoc#apply-DataSourceV2Relation, DataSourceV2Relation>> logical operator).

(From https://drill.apache.org/docs/parquet-filter-pushdown/[Parquet Filter Pushdown] in Apache Drill's documentation) Filter pushdown is a performance optimization that prunes extraneous data while reading from a data source to reduce the amount of data to scan and read for queries with <<spark-sql-SparkStrategy-DataSourceStrategy.adoc#translateFilter, supported filter expressions>>. Pruning data reduces the I/O, CPU, and network overhead to optimize query performance.

TIP: Enable INFO logging level for the <<spark-sql-SparkStrategy-DataSourceV2Strategy.adoc#logging, DataSourceV2Strategy logger>> to be told <<spark-sql-SparkStrategy-DataSourceV2Strategy.adoc#apply-DataSourceV2Relation, what the pushed filters are>>.

=== [[i-want-more]] Further Reading and Watching

. (video) https://databricks.com/session/apache-spark-data-source-v2[Apache Spark Data Source V2 by Wenchen Fan and Gengliang Wang]
