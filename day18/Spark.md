#### DataFrame API
- Spark Dataframes are like distributed in-memory table with named columns and schemas, where each column has a specific data type.
- Dataframes are immutable and Spark keeps a lineage of all transformations.
- A `schema` in Spark defines the column names and associated data types for a DataFrame.
- Schemas come into play when you are reading structured data from an external data source.

#### Common DataFrame Operations
- DataFrameReaders -> To read data into a DataFrame from myraid data sources in formats such as JSON, CSV, Parquet, Text, AVRO etc.
- DataFrameWriters -> To write DataFrame back to a data source
- A common data operation is to explore and transform your data, and then persist the DataFrame in Parquet format or save it as a SQL table.

#### Transformations and Actions

- Projection
- Renaming, adding and dropping columns
- Aggregations

#### Dataset API

- Conceptually, you can think of a DataFrame in Scala as an alias for a collection of generic objects, Dataset[Row], where Row is a generic untyped JVM object that may hold different types of fields.
- A dataset, by contrast, is a collection of `strongly typed` JVM objects in Scala or a class in Java.

##### Typed Objects, Untyped Objects, and Generic Rows
- Datasets make sense only in Java and Scala, whereas in Python and R only DataFrames make sense.

#### Spark SQL and the Underlying Engine

- Catalyst Optimizer -> takes a computational query and converts it into an execution plan.
- Project Tungsten -> generates efficient bytecode to run on each machine.
