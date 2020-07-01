# SupportsScanColumnarBatch

`SupportsScanColumnarBatch` is the <<contract, contract>>...FIXME

[[contract]]
[source, java]
----
package org.apache.spark.sql.sources.v2.reader;

public interface SupportsScanColumnarBatch extends DataSourceReader {
  // only required methods that have no implementation
  // the others follow
  List<DataReaderFactory<ColumnarBatch>> createBatchDataReaderFactories();
}
----

[NOTE]
====
`SupportsScanColumnarBatch` is an `Evolving` contract that is evolving towards becoming a stable API, but is not a stable API yet and can change from one feature release to another release.

In other words, using the contract is as treading on thin ice.
====

.(Subset of) SupportsScanColumnarBatch Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| [[createBatchDataReaderFactories]] `createBatchDataReaderFactories`
| Used when...FIXME
|===

[[implementations]]
NOTE: No custom `SupportsScanColumnarBatch` are part of Spark 2.3.

=== [[enableBatchRead]] `enableBatchRead` Method

[source, java]
----
default boolean enableBatchRead()
----

`enableBatchRead` flag is always enabled (i.e. `true`) unless overrode by <<implementations, custom SupportsScanColumnarBatches>>.

NOTE: `enableBatchRead` is used when...FIXME
