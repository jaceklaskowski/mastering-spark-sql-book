# Window Aggregation

*Window Aggregation* is...FIXME

=== From Structured Query to Physical Plan

<<spark-sql-Analyzer.adoc#, Spark Analyzer>> uses <<spark-sql-Analyzer-ExtractWindowExpressions.adoc#, ExtractWindowExpressions>> logical resolution rule to replace (extract) <<spark-sql-Expression-WindowExpression.adoc#, WindowExpression>> expressions with <<spark-sql-LogicalPlan-Window.adoc#, Window>> logical operators in a <<spark-sql-LogicalPlan.adoc#, logical query plan>>.

NOTE: Window —> (BasicOperators) —> WindowExec —> WindowExec.adoc#doExecute (and windowExecBufferInMemoryThreshold + windowExecBufferSpillThreshold)
