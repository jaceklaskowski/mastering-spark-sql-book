title: AnalysisBarrier

# AnalysisBarrier Leaf Logical Operator -- Hiding Child Query Plan in Analysis

`AnalysisBarrier` is a <<spark-sql-LogicalPlan-LeafNode.adoc#, leaf logical operator>> that is a wrapper of an <<child, analyzed logical plan>> to hide it from the Spark Analyzer. The purpose of `AnalysisBarrier` is to prevent the child logical plan from being analyzed again (and increasing the time spent on query analysis).

`AnalysisBarrier` is <<creating-instance, created>> when:

* `ResolveReferences` logical resolution rule is requested to <<spark-sql-Analyzer-ResolveReferences.adoc#dedupRight, dedupRight>>

* `ResolveMissingReferences` logical resolution rule is requested to <<spark-sql-Analyzer-ResolveMissingReferences.adoc#resolveExprsAndAddMissingAttrs, resolveExprsAndAddMissingAttrs>>

* `Dataset` is <<spark-sql-Dataset.adoc#planWithBarrier, created>>

* `DataFrameWriter` is requested to <<spark-sql-DataFrameWriter.adoc#saveToV1Source, execute a logical command for writing to a data source V1>> (when `DataFrameWriter` is requested to <<spark-sql-DataFrameWriter.adoc#save, save the rows of a structured query (a DataFrame) to a data source>>)

* `KeyValueGroupedDataset` is requested for the <<spark-sql-KeyValueGroupedDataset.adoc#logicalPlan, logical query plan>>

[[child]]
[[creating-instance]]
`AnalysisBarrier` takes a single `child` <<spark-sql-LogicalPlan.adoc#, logical query plan>> when created.

[[innerChildren]]
`AnalysisBarrier` returns the <<child, child logical query plan>> when requested for the <<spark-sql-catalyst-TreeNode.adoc#innerChildren, inner nodes>> (that should be shown as an inner nested tree of this node).

[[output]]
`AnalysisBarrier` simply requests the <<child, child logical query plan>> for the <<spark-sql-catalyst-QueryPlan.adoc#output, output schema attributes>>.

[[isStreaming]]
`AnalysisBarrier` simply requests the <<child, child logical query plan>> for the <<spark-sql-LogicalPlan.adoc#isStreaming, isStreaming>> flag.

[[doCanonicalize]]
`AnalysisBarrier` simply requests the <<child, child logical operator>> for the <<spark-sql-catalyst-QueryPlan.adoc#doCanonicalize, canonicalized version>>.
