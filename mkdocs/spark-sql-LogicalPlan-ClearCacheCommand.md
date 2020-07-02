title: ClearCacheCommand

# ClearCacheCommand Logical Command

`ClearCacheCommand` is a link:spark-sql-LogicalPlan-RunnableCommand.adoc[logical command] to link:spark-sql-Catalog.adoc#clearCache[remove all cached tables from the in-memory cache].

`ClearCacheCommand` corresponds to `CLEAR CACHE` SQL statement.

NOTE: `ClearCacheCommand` is described by `clearCache` labeled alternative in `statement` expression in `SqlBase.g4` and parsed using link:spark-sql-SparkSqlParser.adoc[SparkSqlParser].
