# Datasets, DataFrames and RDDs

Many may have been asking yourself why they should be using Datasets rather than the foundation of all Spark - RDDs using case classes.

This document collects advantages of `Dataset` vs `RDD[CaseClass]` to answer https://twitter.com/danosipov/status/704421546203308033[the question Dan has asked on twitter]:

> "In #Spark, what is the advantage of a DataSet over an RDD[CaseClass]?"

## Saving to or Writing from Data Sources

With Dataset API, loading data from a data source or saving it to one is as simple as using <<spark-sql-SparkSession.adoc#read, SparkSession.read>> or <<spark-sql-dataset-operators.adoc#write, Dataset.write>> methods, appropriately.

## Accessing Fields / Columns

You `select` columns in a datasets without worrying about the positions of the columns.

In RDD, you have to do an additional hop over a case class and access fields by name.
