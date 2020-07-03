# PushDownOperatorsToDataSource Logical Optimization

`PushDownOperatorsToDataSource` is a *logical optimization* that <<apply, pushes down operators to underlying data sources>> (i.e. <<spark-sql-LogicalPlan-DataSourceV2Relation.adoc#, DataSourceV2Relations>>) (before planning so that data source can report statistics more accurately).

Technically, `PushDownOperatorsToDataSource` is a <<spark-sql-catalyst-Rule.adoc#, Catalyst rule>> for transforming <<spark-sql-LogicalPlan.adoc#, logical plans>>, i.e. `Rule[LogicalPlan]`.

`PushDownOperatorsToDataSource` is part of the <<spark-sql-SparkOptimizer.adoc#PushDownOperatorsToDataSource, Push down operators to data source scan>> once-executed rule batch of the <<spark-sql-SparkOptimizer.adoc#, SparkOptimizer>>.

=== [[apply]] Executing Rule -- `apply` Method

[source, scala]
----
apply(plan: LogicalPlan): LogicalPlan
----

NOTE: `apply` is part of the <<spark-sql-catalyst-Rule.adoc#apply, Rule Contract>> to execute (apply) a rule on a <<spark-sql-catalyst-TreeNode.adoc#, TreeNode>> (e.g. <<spark-sql-LogicalPlan.adoc#, LogicalPlan>>).

`apply`...FIXME

=== [[pushDownRequiredColumns]] `pushDownRequiredColumns` Internal Method

[source, scala]
----
pushDownRequiredColumns(plan: LogicalPlan, requiredByParent: AttributeSet): LogicalPlan
----

`pushDownRequiredColumns` branches off per the input <<spark-sql-LogicalPlan.adoc#, logical operator>> (that is supposed to have at least one child node):

. For <<spark-sql-LogicalPlan-Project.adoc#, Project>> unary logical operator, `pushDownRequiredColumns` takes the <<spark-sql-Expression.adoc#references, references>> of the <<spark-sql-LogicalPlan-Project.adoc#projectList, project expressions>> as the required columns (attributes) and executes itself recursively on the <<spark-sql-LogicalPlan-Project.adoc#child, child logical operator>>
+
Note that the input `requiredByParent` attributes are not considered in the required columns.

. For <<spark-sql-LogicalPlan-Filter.adoc#, Filter>> unary logical operator, `pushDownRequiredColumns` adds the <<spark-sql-Expression.adoc#references, references>> of the <<spark-sql-LogicalPlan-Filter.adoc#condition, filter condition>> to the input `requiredByParent` attributes and executes itself recursively on the <<spark-sql-LogicalPlan-Filter.adoc#child, child logical operator>>

. For <<spark-sql-LogicalPlan-DataSourceV2Relation.adoc#, DataSourceV2Relation>> unary logical operator, `pushDownRequiredColumns`...FIXME

. For other logical operators, `pushDownRequiredColumns` simply executes itself (using <<spark-sql-catalyst-TreeNode.adoc#mapChildren, TreeNode.mapChildren>>) recursively on the <<spark-sql-catalyst-TreeNode.adoc#children, child nodes>> (logical operators)

NOTE: `pushDownRequiredColumns` is used exclusively when `PushDownOperatorsToDataSource` logical optimization is requested to <<apply, execute>>.

=== [[FilterAndProject]][[unapply]] Destructuring Logical Operator -- `FilterAndProject.unapply` Method

[source, scala]
----
unapply(plan: LogicalPlan): Option[(Seq[NamedExpression], Expression, DataSourceV2Relation)]
----

`unapply` is part of `FilterAndProject` extractor object to destructure the input <<spark-sql-LogicalPlan.adoc#, logical operator>> into a tuple with...FIXME

`unapply` works with (matches) the following logical operators:

. For a <<spark-sql-LogicalPlan-Filter.adoc#, Filter>> with a <<spark-sql-LogicalPlan-DataSourceV2Relation.adoc#, DataSourceV2Relation>> leaf logical operator, `unapply`...FIXME

. For a <<spark-sql-LogicalPlan-Filter.adoc#, Filter>> with a <<spark-sql-LogicalPlan-Project.adoc#, Project>> over a <<spark-sql-LogicalPlan-DataSourceV2Relation.adoc#, DataSourceV2Relation>> leaf logical operator, `unapply`...FIXME

. For others, `unapply` returns `None` (i.e. does nothing / does not match)

NOTE: `unapply` is used exclusively when `PushDownOperatorsToDataSource` logical optimization is requested to <<apply, execute>>.
