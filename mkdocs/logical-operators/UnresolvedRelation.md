title: UnresolvedRelation

# UnresolvedRelation Leaf Logical Operator for Table Reference

[[tableIdentifier]][[creating-instance]]
`UnresolvedRelation` is a link:spark-sql-LogicalPlan-LeafNode.adoc[leaf logical operator] to represent a *table reference* in a logical query plan that has yet to be resolved (i.e. looked up in a catalog).

[NOTE]
====
If after link:spark-sql-Analyzer.adoc[Analyzer] has finished analyzing a logical query plan the plan has still a `UnresolvedRelation` it link:spark-sql-Analyzer-CheckAnalysis.adoc#UnresolvedRelation[fails the analyze phase] with the following `AnalysisException`:

```
Table or view not found: [tableIdentifier]
```
====

`UnresolvedRelation` is <<creating-instance, created>> when:

* `SparkSession` is requested to link:spark-sql-SparkSession.adoc#table[create a DataFrame from a table]

* `DataFrameWriter` is requested to link:spark-sql-DataFrameWriter.adoc#insertInto[insert a DataFrame into a table]

* `INSERT INTO (TABLE)` or `INSERT OVERWRITE TABLE` SQL commands are link:InsertIntoTable.adoc#INSERT_INTO_TABLE[executed]

* link:hive/CreateHiveTableAsSelectCommand.adoc[CreateHiveTableAsSelectCommand] logical command is executed

[TIP]
====
Use `table` operator from link:spark-sql-catalyst-dsl.adoc#plans[Catalyst DSL] to create a `UnresolvedRelation` logical operator, e.g. for testing or Spark SQL internals exploration.

[source, scala]
----
import org.apache.spark.sql.catalyst.dsl.plans._
val plan = table(db = "myDB", ref = "t1")
scala> println(plan.numberedTreeString)
00 'UnresolvedRelation `myDB`.`t1`
----
====

NOTE: `UnresolvedRelation` is resolved to...FIXME
