# GenerateUnsafeProjection

`GenerateUnsafeProjection` is a link:spark-sql-CodeGenerator.adoc[CodeGenerator] that <<create, generates the bytecode for a UnsafeProjection for given expressions>> (i.e. `CodeGenerator[Seq[Expression], UnsafeProjection]`).

[source, scala]
----
GenerateUnsafeProjection: Seq[Expression] => UnsafeProjection
----

[TIP]
====
Enable `DEBUG` logging level for `org.apache.spark.sql.catalyst.expressions.codegen.GenerateUnsafeProjection` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.catalyst.expressions.codegen.GenerateUnsafeProjection=DEBUG
```

Refer to link:spark-logging.adoc[Logging].
====

=== [[generate]] Generating UnsafeProjection -- `generate` Method

[source, scala]
----
generate(
  expressions: Seq[Expression],
  subexpressionEliminationEnabled: Boolean): UnsafeProjection
----

`generate` <<canonicalize, canonicalize>> the input `expressions` followed by <<create, generating a JVM bytecode for a UnsafeProjection>> for the expressions.

[NOTE]
====
`generate` is used when:

* `UnsafeProjection` factory object is requested for a link:spark-sql-UnsafeProjection.adoc#create[UnsafeProjection]

* `ExpressionEncoder` is requested to link:spark-sql-ExpressionEncoder.adoc#extractProjection[initialize the internal UnsafeProjection]

* `FileFormat` is requested to link:spark-sql-FileFormat.adoc#buildReaderWithPartitionValues[build a data reader with partition column values appended]

* `OrcFileFormat` is requested to link:spark-sql-OrcFileFormat.adoc#buildReaderWithPartitionValues[build a data reader with partition column values appended]

* `ParquetFileFormat` is requested to link:spark-sql-ParquetFileFormat.adoc#buildReaderWithPartitionValues[build a data reader with partition column values appended]

* `GroupedIterator` is requested for `keyProjection`

* `ObjectOperator` is requested to `serializeObjectToRow`

* (Spark MLlib) `LibSVMFileFormat` is requested to `buildReader`

* (Spark Structured Streaming) `StateStoreRestoreExec`, `StateStoreSaveExec` and `StreamingDeduplicateExec` are requested to execute
====

=== [[canonicalize]] `canonicalize` Method

[source, scala]
----
canonicalize(in: Seq[Expression]): Seq[Expression]
----

`canonicalize` removes unnecessary `Alias` expressions.

Internally, `canonicalize` uses `ExpressionCanonicalizer` rule executor (that in turn uses just one `CleanExpressions` expression rule).

=== [[create]] Generating JVM Bytecode For UnsafeProjection For Given Expressions (With Optional Subexpression Elimination) -- `create` Method

[source, scala]
----
create(
  expressions: Seq[Expression],
  subexpressionEliminationEnabled: Boolean): UnsafeProjection
create(references: Seq[Expression]): UnsafeProjection // <1>
----
<1> Calls the former `create` with `subexpressionEliminationEnabled` flag off

`create` first creates a link:spark-sql-CodeGenerator.adoc#newCodeGenContext[CodegenContext] and an <<createCode, Java source code>> for the input `expressions`.

`create` creates a code body with `public java.lang.Object generate(Object[] references)` method that creates a `SpecificUnsafeProjection`.

[source, java]
----
public java.lang.Object generate(Object[] references) {
  return new SpecificUnsafeProjection(references);
}

class SpecificUnsafeProjection extends UnsafeProjection {
  ...
}
----

`create` creates a `CodeAndComment` with the code body and link:spark-sql-CodegenContext.adoc#placeHolderToComments[comment placeholders].

You should see the following DEBUG message in the logs:

```
DEBUG GenerateUnsafeProjection: code for [expressions]:
[code]
```

[TIP]
====
Enable `DEBUG` logging level for `org.apache.spark.sql.catalyst.expressions.codegen.CodeGenerator` logger to see the message above.

```
log4j.logger.org.apache.spark.sql.catalyst.expressions.codegen.CodeGenerator=DEBUG
```

See link:spark-sql-CodeGenerator.adoc#logging[CodeGenerator].
====

`create` requests `CodeGenerator` to link:spark-sql-CodeGenerator.adoc#compile[compile the Java source code to JVM bytecode (using Janino)].

`create` requests `CodegenContext` for link:spark-sql-CodegenContext.adoc#references[references] and requests the compiled class to create a `SpecificUnsafeProjection` for the input references that in the end is the final link:spark-sql-UnsafeProjection.adoc[UnsafeProjection].

NOTE: (Single-argument) `create` is part of link:spark-sql-CodeGenerator.adoc#create[CodeGenerator Contract].

=== [[createCode]] Creating ExprCode for Expressions (With Optional Subexpression Elimination) -- `createCode` Method

[source, scala]
----
createCode(
  ctx: CodegenContext,
  expressions: Seq[Expression],
  useSubexprElimination: Boolean = false): ExprCode
----

`createCode` requests the input `CodegenContext` to link:spark-sql-CodegenContext.adoc#generateExpressions[generate a Java source code for code-generated evaluation] of every expression in the input `expressions`.

`createCode`...FIXME

[source, scala]
----
import org.apache.spark.sql.catalyst.expressions.codegen.CodegenContext
val ctx = new CodegenContext

// Use Catalyst DSL
import org.apache.spark.sql.catalyst.dsl.expressions._
val expressions = "hello".expr.as("world") :: "hello".expr.as("world") :: Nil

import org.apache.spark.sql.catalyst.expressions.codegen.GenerateUnsafeProjection
val eval = GenerateUnsafeProjection.createCode(ctx, expressions, useSubexprElimination = true)

scala> println(eval.code)

        mutableStateArray1[0].reset();

        mutableStateArray2[0].write(0, ((UTF8String) references[0] /* literal */));


            mutableStateArray2[0].write(1, ((UTF8String) references[1] /* literal */));
        mutableStateArray[0].setTotalSize(mutableStateArray1[0].totalSize());


scala> println(eval.value)
mutableStateArray[0]
----

[NOTE]
====
`createCode` is used when:

* `CreateNamedStructUnsafe` is requested to link:spark-sql-Expression-CreateNamedStructUnsafe.adoc#doGenCode[generate a Java source code]

* `GenerateUnsafeProjection` is requested to <<create, create a UnsafeProjection>>

* `CodegenSupport` is requested to link:spark-sql-CodegenSupport.adoc#prepareRowVar[prepareRowVar] (to link:spark-sql-CodegenSupport.adoc#consume[generate a Java source code to consume generated columns or row from a physical operator])

* `HashAggregateExec` is requested to link:spark-sql-SparkPlan-HashAggregateExec.adoc#doProduceWithKeys[doProduceWithKeys] and link:spark-sql-SparkPlan-HashAggregateExec.adoc#doConsumeWithKeys[doConsumeWithKeys]

* `BroadcastHashJoinExec` is requested to link:spark-sql-SparkPlan-BroadcastHashJoinExec.adoc#genStreamSideJoinKey[genStreamSideJoinKey] (when generating the Java source code for joins)
====
