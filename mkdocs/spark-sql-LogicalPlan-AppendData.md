title: AppendData

# AppendData Logical Operator -- Appending Data to DataSourceV2

`AppendData` is a <<spark-sql-LogicalPlan.adoc#, logical operator>> that represents appending data (the result of executing a <<query, structured query>>) to a <<table, table>> (with the <<isByName, columns matching>> by <<byName, name>> or <<byPosition, position>>) in <<spark-sql-data-source-api-v2.adoc#, Data Source API V2>>.

`AppendData` is <<creating-instance, created>> (indirectly via <<byName, byName>> or <<byPosition, byPosition>> factory methods) only for tests.

NOTE: `AppendData` has replaced the deprecated <<spark-sql-LogicalPlan-WriteToDataSourceV2.adoc#, WriteToDataSourceV2>> logical operator.

[[creating-instance]]
`AppendData` takes the following to be created:

* [[table]] `NamedRelation` for the table (to append data to)
* [[query]] <<spark-sql-LogicalPlan.adoc#, Logical operator>> (for the query)
* [[isByName]] `isByName` flag

[[children]]
`AppendData` has a <<spark-sql-catalyst-TreeNode.adoc#children, single child logical operator>> that is exactly the <<query, logical operator>>.

`AppendData` is resolved using <<spark-sql-Analyzer-ResolveOutputRelation.adoc#, ResolveOutputRelation>> logical resolution rule.

`AppendData` is planned (_replaced_) to <<spark-sql-SparkPlan-WriteToDataSourceV2Exec.adoc#, WriteToDataSourceV2Exec>> physical operator (when the <<table, table>> is a <<spark-sql-LogicalPlan-DataSourceV2Relation.adoc#, DataSourceV2Relation>> logical operator).

=== [[byName]] `byName` Factory Method

[source, scala]
----
byName(table: NamedRelation, df: LogicalPlan): AppendData
----

`byName` simply creates a <<AppendData, AppendData>> logical operator with the <<isByName, isByName>> flag on (`true`).

NOTE: `byName` seems used only for tests.

=== [[byPosition]] `byPosition` Factory Method

[source, scala]
----
byPosition(table: NamedRelation, query: LogicalPlan): AppendData
----

`byPosition` simply creates a <<AppendData, AppendData>> logical operator with the <<isByName, isByName>> flag off (`false`).

NOTE: `byPosition` seems used only for tests.
