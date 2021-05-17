### Spark SQL

- SparkSession provides unified entry point for programming Spark with structured APIs. 
- You can use a SparkSession to access Spark functionality: just import the class and create an instance in your code.
- To issue any SQL query, use the sql() method on the SparkSession instance.
- All spark.sql queries executed in this way return a DataFrame which lets you perform further Spark operations you desire.
- Tables hold data. Associated with each table in Spark is its relavent metadata, which is information about the table and its data: the schema, description, table name, database name, column names, partitions, physical location where the actual data resides. `All of this is stored in a central metastore`.
- Instead of having a separate metastore for Spark tables, Spark by default uses the Apache Hive metastore, located at /user/hive/warehous, to persist all the metadata about your tables.
- Spark allows you to create two types of tables: managed and unmanaged.
- For managed table, spark manages both the metadata and the data in the file store. This could be a local filesystem, HDFS, or an object store such as Amazon S3 or Azure Blob.
- For an unmanaged table, Spark only manages the metadata, while you manage the data yourself in an external data source such as Cassandra.
- With a managed table, because Spark manages everything, a SQL command such as `DROP TABLE table_name` delets both the metadata and the data.
- With unmanaged table, the same command will delete only the metadata, not the actual table.
- Metadata is captured in the Catalog, a high-level abstraction in Spark SQL for storing metadata.
- Data Sources for DataFrames and SQL Tables
- DataFrameReader
- I need to figure out how to setup databricks notebook and play with examples in this chapter.