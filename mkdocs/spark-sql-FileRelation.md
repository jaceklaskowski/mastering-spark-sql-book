# FileRelation

`FileRelation` is the <<contract, contract>> of relations that are backed by files.

[[contract]]
[source, scala]
----
package org.apache.spark.sql.execution

trait FileRelation {
  def inputFiles: Array[String]
}
----

.FileRelation Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| [[inputFiles]] `inputFiles`
|
The list of files that will be read when scanning the relation.

Used exclusively when `Dataset` is requested to link:spark-sql-Dataset.adoc#inputFiles[inputFiles]
|===

[[implementations]]
.FileRelations
[cols="1,2",options="header",width="100%"]
|===
| FileRelation
| Description

| [[HadoopFsRelation]] link:spark-sql-BaseRelation-HadoopFsRelation.adoc[HadoopFsRelation]
|
|===
