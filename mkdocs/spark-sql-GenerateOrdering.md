# GenerateOrdering

`GenerateOrdering` is...FIXME

=== [[create]] Creating BaseOrdering -- `create` Method

[source, scala]
----
create(ordering: Seq[SortOrder]): BaseOrdering
create(schema: StructType): BaseOrdering
----

NOTE: `create` is part of link:spark-sql-CodeGenerator.adoc#create[CodeGenerator Contract] to...FIXME.

`create`...FIXME

=== [[genComparisons]] `genComparisons` Method

[source, scala]
----
genComparisons(ctx: CodegenContext, schema: StructType): String
----

`genComparisons`...FIXME

NOTE: `genComparisons` is used when...FIXME
