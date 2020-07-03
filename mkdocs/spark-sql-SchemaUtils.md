# SchemaUtils Helper Object

`SchemaUtils` is a Scala object that is used for the following:

* <<checkColumnNameDuplication, checkColumnNameDuplication>>

* <<checkSchemaColumnNameDuplication, checkSchemaColumnNameDuplication>>

=== [[checkColumnNameDuplication]] `checkColumnNameDuplication` Method

[source, scala]
----
checkColumnNameDuplication(
  columnNames: Seq[String],
  colType: String,
  resolver: Resolver): Unit  // <1>
checkColumnNameDuplication(
  columnNames: Seq[String],
  colType: String,
  caseSensitiveAnalysis: Boolean): Unit
----
<1> Uses the other `checkColumnNameDuplication` with `caseSensitiveAnalysis` flag per <<isCaseSensitiveAnalysis, isCaseSensitiveAnalysis>>

`checkColumnNameDuplication`...FIXME

NOTE: `checkColumnNameDuplication` is used when...FIXME

=== [[checkSchemaColumnNameDuplication]] `checkSchemaColumnNameDuplication` Method

[source, scala]
----
checkSchemaColumnNameDuplication(
  schema: StructType, colType: String, caseSensitiveAnalysis: Boolean = false): Unit
----

`checkSchemaColumnNameDuplication`...FIXME

NOTE: `checkSchemaColumnNameDuplication` is used when...FIXME

=== [[isCaseSensitiveAnalysis]] `isCaseSensitiveAnalysis` Internal Method

[source, scala]
----
isCaseSensitiveAnalysis(resolver: Resolver): Boolean
----

`isCaseSensitiveAnalysis`...FIXME

NOTE: `isCaseSensitiveAnalysis` is used when...FIXME
