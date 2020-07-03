# TypeCoercion Object

`TypeCoercion` is a Scala object that defines the <<typeCoercionRules, type coercion rules>> for <<spark-sql-Analyzer.adoc#typeCoercionRules, Spark Analyzer>>.

=== [[typeCoercionRules]] Defining Type Coercion Rules (For Spark Analyzer) -- `typeCoercionRules` Method

[source, scala]
----
typeCoercionRules(conf: SQLConf): List[Rule[LogicalPlan]]
----

`typeCoercionRules` is a collection of <<spark-sql-catalyst-Rule.adoc#, Catalyst rules>> to transform <<spark-sql-LogicalPlan.adoc#, logical plans>> (in the order of execution):

. <<spark-sql-Analyzer-TypeCoercionRule-InConversion.adoc#, InConversion>>
. `WidenSetOperationTypes`
. `PromoteStrings`
. `DecimalPrecision`
. `BooleanEquality`
. `FunctionArgumentConversion`
. `ConcatCoercion`
. `EltCoercion`
. `CaseWhenCoercion`
. `IfCoercion`
. `StackCoercion`
. `Division`
. `ImplicitTypeCasts`
. `DateTimeOperations`
. <<spark-sql-Analyzer-TypeCoercionRule-WindowFrameCoercion.adoc#, WindowFrameCoercion>>

NOTE: `typeCoercionRules` is used exclusively when `Analyzer` is requested for <<spark-sql-Analyzer.adoc#batches, batches>>.
