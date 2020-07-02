title: ParseToDate

# ParseToDate Expression

`ParseToDate` is a link:spark-sql-Expression-RuntimeReplaceable.adoc[RuntimeReplaceable] expression that <<creating-instance, represents>> the link:spark-sql-functions-datetime.adoc#to_date[to_date] function (in logical query plans).

[source, scala]
----
// DEMO to_date(e: Column): Column
// DEMO to_date(e: Column, fmt: String): Column
----

As a `RuntimeReplaceable` expression, `ParseToDate` is replaced by link:spark-sql-Optimizer.adoc#ReplaceExpressions[Catalyst Optimizer] with the <<child, child>> expression:

* `Cast(left, DateType)` for `to_date(e: Column): Column` function

* `Cast(Cast(UnixTimestamp(left, format), TimestampType), DateType)` for `to_date(e: Column, fmt: String): Column` function

[source, scala]
----
// FIXME DEMO Conversion to `Cast(left, DateType)`
// FIXME DEMO Conversion to `Cast(Cast(UnixTimestamp(left, format), TimestampType), DateType)`
----

=== [[creating-instance]] Creating ParseToDate Instance

`ParseToDate` takes the following when created:

* [[left]] Left link:spark-sql-Expression.adoc[expression]
* [[format]] `format` link:spark-sql-Expression.adoc[expression]
* [[child]] Child link:spark-sql-Expression.adoc[expression]
