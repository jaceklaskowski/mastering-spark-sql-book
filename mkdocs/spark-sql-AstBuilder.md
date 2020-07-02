title: AstBuilder

# AstBuilder -- ANTLR-based SQL Parser

`AstBuilder` converts SQL statements into Spark SQL's relational entities (i.e. link:spark-sql-DataType.adoc[data types], link:spark-sql-Expression.adoc[Catalyst expressions], link:spark-sql-LogicalPlan.adoc[logical plans] or `TableIdentifiers`) using <<visit-callbacks, visit callback methods>>.

`AstBuilder` is the link:spark-sql-AbstractSqlParser.adoc#astBuilder[AST builder] of `AbstractSqlParser` (i.e. the base SQL parsing infrastructure in Spark SQL).

[TIP]
====
Spark SQL supports SQL statements as described in https://github.com/apache/spark/blob/master/sql/catalyst/src/main/antlr4/org/apache/spark/sql/catalyst/parser/SqlBase.g4[SqlBase.g4]. Using the file can tell you (almost) exactly what Spark SQL supports at any given time.

_"Almost"_ being that although the grammar accepts a SQL statement it can be reported as not allowed by `AstBuilder`, e.g.

```
scala> sql("EXPLAIN FORMATTED SELECT * FROM myTable").show
org.apache.spark.sql.catalyst.parser.ParseException:
Operation not allowed: EXPLAIN FORMATTED(line 1, pos 0)

== SQL ==
EXPLAIN FORMATTED SELECT * FROM myTable
^^^

  at org.apache.spark.sql.catalyst.parser.ParserUtils$.operationNotAllowed(ParserUtils.scala:39)
  at org.apache.spark.sql.execution.SparkSqlAstBuilder$$anonfun$visitExplain$1.apply(SparkSqlParser.scala:275)
  at org.apache.spark.sql.execution.SparkSqlAstBuilder$$anonfun$visitExplain$1.apply(SparkSqlParser.scala:273)
  at org.apache.spark.sql.catalyst.parser.ParserUtils$.withOrigin(ParserUtils.scala:93)
  at org.apache.spark.sql.execution.SparkSqlAstBuilder.visitExplain(SparkSqlParser.scala:273)
  at org.apache.spark.sql.execution.SparkSqlAstBuilder.visitExplain(SparkSqlParser.scala:53)
  at org.apache.spark.sql.catalyst.parser.SqlBaseParser$ExplainContext.accept(SqlBaseParser.java:480)
  at org.antlr.v4.runtime.tree.AbstractParseTreeVisitor.visit(AbstractParseTreeVisitor.java:42)
  at org.apache.spark.sql.catalyst.parser.AstBuilder$$anonfun$visitSingleStatement$1.apply(AstBuilder.scala:66)
  at org.apache.spark.sql.catalyst.parser.AstBuilder$$anonfun$visitSingleStatement$1.apply(AstBuilder.scala:66)
  at org.apache.spark.sql.catalyst.parser.ParserUtils$.withOrigin(ParserUtils.scala:93)
  at org.apache.spark.sql.catalyst.parser.AstBuilder.visitSingleStatement(AstBuilder.scala:65)
  at org.apache.spark.sql.catalyst.parser.AbstractSqlParser$$anonfun$parsePlan$1.apply(ParseDriver.scala:62)
  at org.apache.spark.sql.catalyst.parser.AbstractSqlParser$$anonfun$parsePlan$1.apply(ParseDriver.scala:61)
  at org.apache.spark.sql.catalyst.parser.AbstractSqlParser.parse(ParseDriver.scala:90)
  at org.apache.spark.sql.execution.SparkSqlParser.parse(SparkSqlParser.scala:46)
  at org.apache.spark.sql.catalyst.parser.AbstractSqlParser.parsePlan(ParseDriver.scala:61)
  at org.apache.spark.sql.SparkSession.sql(SparkSession.scala:617)
  ... 48 elided
```
====

`AstBuilder` is a ANTLR `AbstractParseTreeVisitor` (as `SqlBaseBaseVisitor`) that is generated from https://github.com/apache/spark/blob/master/sql/catalyst/src/main/antlr4/org/apache/spark/sql/catalyst/parser/SqlBase.g4[SqlBase.g4] ANTLR grammar for Spark SQL.

[NOTE]
====
`SqlBaseBaseVisitor` is a ANTLR-specific base class that is auto-generated at build time from a ANTLR grammar in `SqlBase.g4`.

`SqlBaseBaseVisitor` is an ANTLR http://www.antlr.org/api/Java/org/antlr/v4/runtime/tree/AbstractParseTreeVisitor.html[AbstractParseTreeVisitor].
====

[[visit-callbacks]]
.AstBuilder's Visit Callback Methods
[cols="1m,1,3",options="header",width="100%"]
|===
| Callback Method
| ANTLR rule / labeled alternative
| Spark SQL Entity

| visitAliasedQuery
|
| [[visitAliasedQuery]]

| visitColumnReference
|
| [[visitColumnReference]]

| visitDereference
|
| [[visitDereference]]

| visitExists
| `#exists` labeled alternative
| [[visitExists]] link:spark-sql-Expression-Exists.adoc[Exists] expression

| visitExplain
| `explain` rule
a| [[visitExplain]] link:spark-sql-LogicalPlan-ExplainCommand.adoc[ExplainCommand]

[NOTE]
====
Can be a `OneRowRelation` for an `EXPLAIN` for an unexplainable link:spark-sql-LogicalPlan-DescribeTableCommand.adoc[DescribeTableCommand] logical command as created from <<visitDescribeTable, DESCRIBE TABLE>> SQL statement.

```
val q = sql("EXPLAIN DESCRIBE TABLE t")
scala> println(q.queryExecution.logical.numberedTreeString)
scala> println(q.queryExecution.logical.numberedTreeString)
00 ExplainCommand OneRowRelation$, false, false, false
```
====

| visitFirst
| `#first` labeled alternative
a| [[visitFirst]] <<spark-sql-Expression-First.adoc#, First>> aggregate function expression

```
FIRST '(' expression (IGNORE NULLS)? ')'
```

| visitFromClause
| `fromClause`
a| [[visitFromClause]] link:spark-sql-LogicalPlan.adoc[LogicalPlan]

Supports multiple comma-separated relations (that all together build a condition-less INNER JOIN) with optional link:spark-sql-Expression-Generator.adoc#lateral-view[LATERAL VIEW].

A relation can be one of the following or a combination thereof:

* Table identifier
* Inline table using `VALUES exprs AS tableIdent`
* Table-valued function (currently only `range` is supported)

| visitFunctionCall
| `functionCall` labeled alternative
a| [[visitFunctionCall]]

* link:spark-sql-Expression-UnresolvedFunction.adoc[UnresolvedFunction] for a bare function (with no window specification)

* [[visitFunctionCall-UnresolvedWindowExpression]] <<spark-sql-Expression-UnresolvedWindowExpression.adoc#, UnresolvedWindowExpression>> for a function evaluated in a windowed context with a `WindowSpecReference`

* link:spark-sql-Expression-WindowExpression.adoc[WindowExpression] for a function over a window

TIP: See the <<function-examples, function examples>> below.

| visitInlineTable
| `inlineTable` rule
a| [[visitInlineTable]] <<spark-sql-LogicalPlan-UnresolvedInlineTable.adoc#, UnresolvedInlineTable>> unary logical operator (as the child of <<spark-sql-LogicalPlan-SubqueryAlias.adoc#, SubqueryAlias>> for `tableAlias`)

```
VALUES expression (',' expression)* tableAlias
```

`expression` can be as follows:

* <<spark-sql-Expression-CreateNamedStruct.adoc#, CreateNamedStruct>> expression for multiple-column tables

* Any <<spark-sql-Expression.adoc#, Catalyst expression>> for one-column tables

`tableAlias` can be specified explicitly or defaults to `colN` for every column (starting from `1` for `N`).

| visitInsertIntoTable
| `#insertIntoTable` labeled alternative
a| [[visitInsertIntoTable]] <<InsertIntoTable.adoc#, InsertIntoTable>> (indirectly)

A 3-element tuple with a `TableIdentifier`, optional partition keys and the `exists` flag disabled

```
INSERT INTO TABLE? tableIdentifier partitionSpec?
```

NOTE: `insertIntoTable` is part of `insertInto` that is in turn used only as a helper labeled alternative in <<singleInsertQuery, singleInsertQuery>> and <<multiInsertQueryBody, multiInsertQueryBody>> rules.

| visitInsertOverwriteTable
| `#insertOverwriteTable` labeled alternative
a| [[visitInsertOverwriteTable]] <<InsertIntoTable.adoc#, InsertIntoTable>> (indirectly)

A 3-element tuple with a `TableIdentifier`, optional partition keys and the `exists` flag

```
INSERT OVERWRITE TABLE tableIdentifier (partitionSpec (IF NOT EXISTS)?)?
```

In a way, `visitInsertOverwriteTable` is simply a more general version of the <<visitInsertIntoTable, visitInsertIntoTable>> with the `exists` flag on or off per `IF NOT EXISTS` used or not. The main difference is that <<spark-sql-dynamic-partition-inserts.adoc#dynamic-partitions, dynamic partitions>> are used with no `IF NOT EXISTS`.

NOTE: `insertOverwriteTable` is part of `insertInto` that is in turn used only as a helper labeled alternative in <<singleInsertQuery, singleInsertQuery>> and <<multiInsertQueryBody, multiInsertQueryBody>> rules.

| `visitMultiInsertQuery`
| [[multiInsertQueryBody]] `multiInsertQueryBody`
a| [[visitMultiInsertQuery]] A logical operator with a link:InsertIntoTable.adoc[InsertIntoTable] (and link:spark-sql-LogicalPlan-UnresolvedRelation.adoc[UnresolvedRelation] leaf operator)

```
FROM relation (',' relation)* lateralView*
INSERT OVERWRITE TABLE ...

FROM relation (',' relation)* lateralView*
INSERT INTO TABLE? ...
```

| visitNamedExpression
| `namedExpression`
a| [[visitNamedExpression]]

* `Alias` (for a single alias)
* `MultiAlias` (for a parenthesis enclosed alias list
* a bare link:spark-sql-Expression.adoc[Expression]

| visitNamedQuery
|
| [[visitNamedQuery]] <<spark-sql-LogicalPlan-SubqueryAlias.adoc#, SubqueryAlias>>

| `visitQuerySpecification`
| `querySpecification`
a| [[visitQuerySpecification]] `OneRowRelation` or link:spark-sql-LogicalPlan.adoc[LogicalPlan]

[NOTE]
====
`visitQuerySpecification` creates a `OneRowRelation` for a `SELECT` without a `FROM` clause.

```
val q = sql("select 1")
scala> println(q.queryExecution.logical.numberedTreeString)
00 'Project [unresolvedalias(1, None)]
01 +- OneRowRelation$
```
====

| visitPredicated
| `predicated`
| [[visitPredicated]] link:spark-sql-Expression.adoc[Expression]

| visitRelation
| `relation`
| [[visitRelation]] link:spark-sql-LogicalPlan.adoc[LogicalPlan] for a `FROM` clause.

| visitRowConstructor
|
| [[visitRowConstructor]]

| visitSetOperation
| `setOperation`
| [[visitSetOperation]]

| visitSingleDataType
| `singleDataType`
| [[visitSingleDataType]] link:spark-sql-DataType.adoc[DataType]

| visitSingleExpression
| `singleExpression`
| [[visitSingleExpression]] link:spark-sql-Expression.adoc[Expression]

Takes the named expression and relays to <<visitNamedExpression, visitNamedExpression>>

| visitSingleInsertQuery
| [[singleInsertQuery]] `#singleInsertQuery` labeled alternative
a| [[visitSingleInsertQuery]] A logical operator with a link:InsertIntoTable.adoc[InsertIntoTable]

```
INSERT INTO TABLE? tableIdentifier partitionSpec? #insertIntoTable

INSERT OVERWRITE TABLE tableIdentifier (partitionSpec (IF NOT EXISTS)?)? #insertOverwriteTable
```

| visitSortItem
| `sortItem`
a| [[visitSortItem]] <<spark-sql-Expression-SortOrder.adoc#, SortOrder>> unevaluable unary expression

```
sortItem
    : expression ordering=(ASC | DESC)? (NULLS nullOrder=(LAST | FIRST))?
    ;

// queryOrganization
ORDER BY order+=sortItem (',' order+=sortItem)*
SORT BY sort+=sortItem (',' sort+=sortItem)*

// windowSpec
(ORDER | SORT) BY sortItem (',' sortItem)*)?
```

| visitSingleStatement
| `singleStatement`
a| [[visitSingleStatement]] link:spark-sql-LogicalPlan.adoc[LogicalPlan] from a single statement

NOTE: A single statement can be quite involved.

| visitSingleTableIdentifier
| `singleTableIdentifier`
| [[visitSingleTableIdentifier]] `TableIdentifier`

| visitStar
| `#star` labeled alternative
| [[visitStar]] link:spark-sql-Expression-UnresolvedStar.adoc[UnresolvedStar]

| visitStruct
|
| [[visitStruct]]

| visitSubqueryExpression
| `#subqueryExpression` labeled alternative
| [[visitSubqueryExpression]] link:spark-sql-Expression-SubqueryExpression-ScalarSubquery.adoc[ScalarSubquery]

| visitWindowDef
| `windowDef` labeled alternative
a| [[visitWindowDef]] link:spark-sql-Expression-WindowSpecDefinition.adoc[WindowSpecDefinition]

```
// CLUSTER BY with window frame
'(' CLUSTER BY partition+=expression (',' partition+=expression)*) windowFrame? ')'

// PARTITION BY and ORDER BY with window frame
'(' ((PARTITION \| DISTRIBUTE) BY partition+=expression (',' partition+=expression)*)?
  ((ORDER \| SORT) BY sortItem (',' sortItem)*)?)
  windowFrame? ')'
```
|===

[[with-methods]]
.AstBuilder's Parsing Handlers
[cols="1m,3",options="header",width="100%"]
|===
| Parsing Handler
| LogicalPlan Added

| withAggregation
a| [[withAggregation]]

* link:spark-sql-LogicalPlan-GroupingSets.adoc[GroupingSets] for `GROUP BY &hellip; GROUPING SETS (&hellip;)`

* link:spark-sql-LogicalPlan-Aggregate.adoc[Aggregate] for `GROUP BY &hellip; (WITH CUBE \| WITH ROLLUP)?`

| withGenerate
| [[withGenerate]] link:spark-sql-Expression-Generator.adoc[Generate] with a link:spark-sql-Expression-UnresolvedGenerator.adoc[UnresolvedGenerator] and link:spark-sql-LogicalPlan-Generate.adoc#join[join] flag turned on for `LATERAL VIEW` (in `SELECT` or `FROM` clauses).

| withHints
a| [[withHints]] link:spark-sql-LogicalPlan-Hint.adoc[Hint] for `/*+ hint */` in `SELECT` queries.

TIP: Note `+` (plus) between `/\*` and `*/`

`hint` is of the format `name` or `name (param1, param2, ...)`.

```
/*+ BROADCAST (table) */
```

| withInsertInto
a| [[withInsertInto]]

* link:InsertIntoTable.adoc[InsertIntoTable] for <<visitSingleInsertQuery, visitSingleInsertQuery>> or <<visitMultiInsertQuery, visitMultiInsertQuery>>

* link:InsertIntoDir.adoc[InsertIntoDir] for...FIXME

| withJoinRelations
a| [[withJoinRelations]] link:spark-sql-LogicalPlan-Join.adoc[Join] for a <<visitFromClause, FROM clause>> and <<visitRelation, relation>> alone.

The following join types are supported:

* `INNER` (default)
* `CROSS`
* `LEFT` (with optional `OUTER`)
* `LEFT SEMI`
* `RIGHT` (with optional `OUTER`)
* `FULL` (with optional `OUTER`)
* `ANTI` (optionally prefixed with `LEFT`)

The following join criteria are supported:

* `ON booleanExpression`
* `USING '(' identifier (',' identifier)* ')'`

Joins can be `NATURAL` (with no join criteria).

| withQueryResultClauses
| [[withQueryResultClauses]]

| withQuerySpecification
a| [[withQuerySpecification]] Adds a query specification to a logical operator.

For transform `SELECT` (with `TRANSFORM`, `MAP` or `REDUCE` qualifiers), `withQuerySpecification` does...FIXME

---

For regular `SELECT` (no `TRANSFORM`, `MAP` or `REDUCE` qualifiers), `withQuerySpecification` adds (in that order):

. <<withGenerate, Generate>> unary logical operators (if used in the parsed SQL text)

. `Filter` unary logical plan (if used in the parsed SQL text)

. <<withAggregation, GroupingSets or Aggregate>> unary logical operators (if used in the parsed SQL text)

. `Project` and/or `Filter` unary logical operators

. <<withWindows, WithWindowDefinition>> unary logical operator (if used in the parsed SQL text)

. <<withHints, UnresolvedHint>> unary logical operator (if used in the parsed SQL text)

| withPredicate
a| [[withPredicate]]
* `NOT? IN '(' query ')'` gives an link:spark-sql-Expression-In.adoc[In] predicate expression with a link:spark-sql-Expression-ListQuery.adoc[ListQuery] subquery expression

* `NOT? IN '(' expression (',' expression)* ')'` gives an link:spark-sql-Expression-In.adoc[In] predicate expression

| withWindows
a| [[withWindows]] link:spark-sql-LogicalPlan-WithWindowDefinition.adoc[WithWindowDefinition] for link:spark-sql-functions-windows.adoc[window aggregates] (given `WINDOW` definitions).

Used for <<withQueryResultClauses, withQueryResultClauses>> and <<withQuerySpecification, withQuerySpecification>> with `windows` definition.

```
WINDOW identifier AS windowSpec
  (',' identifier AS windowSpec)*
```

TIP: Consult `windows`, `namedWindow`, `windowSpec`, `windowFrame`, and `frameBound` (with `windowRef` and `windowDef`) ANTLR parsing rules for Spark SQL in link:++https://github.com/apache/spark/blob/master/sql/catalyst/src/main/antlr4/org/apache/spark/sql/catalyst/parser/SqlBase.g4#L629++[SqlBase.g4].
|===

NOTE: `AstBuilder` belongs to `org.apache.spark.sql.catalyst.parser` package.

=== [[function-examples]] Function Examples

The examples are handled by <<visitFunctionCall, visitFunctionCall>>.

[source, scala]
----
import spark.sessionState.sqlParser

scala> sqlParser.parseExpression("foo()")
res0: org.apache.spark.sql.catalyst.expressions.Expression = 'foo()

scala> sqlParser.parseExpression("foo() OVER windowSpecRef")
res1: org.apache.spark.sql.catalyst.expressions.Expression = unresolvedwindowexpression('foo(), WindowSpecReference(windowSpecRef))

scala> sqlParser.parseExpression("foo() OVER (CLUSTER BY field)")
res2: org.apache.spark.sql.catalyst.expressions.Expression = 'foo() windowspecdefinition('field, UnspecifiedFrame)
----

=== [[aliasPlan]] `aliasPlan` Internal Method

[source, scala]
----
aliasPlan(alias: ParserRuleContext, plan: LogicalPlan): LogicalPlan
----

`aliasPlan`...FIXME

NOTE: `aliasPlan` is used when...FIXME

=== [[mayApplyAliasPlan]] `mayApplyAliasPlan` Internal Method

[source, scala]
----
mayApplyAliasPlan(tableAlias: TableAliasContext, plan: LogicalPlan): LogicalPlan
----

`mayApplyAliasPlan`...FIXME

NOTE: `mayApplyAliasPlan` is used when...FIXME
