# ResolveSQLOnFile Logical Evaluation Rule for...FIXME

`ResolveSQLOnFile` is...FIXME

=== [[maybeSQLFile]] `maybeSQLFile` Internal Method

[source, scala]
----
maybeSQLFile(u: UnresolvedRelation): Boolean
----

`maybeSQLFile` is enabled (i.e. `true`) where the following all hold:

1. FIXME

NOTE: `maybeSQLFile` is used exclusively when...FIXME

=== [[apply]] Applying Rule to Logical Plan -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of link:spark-sql-catalyst-Rule.adoc#apply[Rule Contract] to apply a rule to a link:spark-sql-LogicalPlan.adoc[logical plan].

`apply`...FIXME
