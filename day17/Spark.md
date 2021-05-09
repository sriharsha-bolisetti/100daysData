
SparkSession - An object that provides a point of entry to interact with underlying Spark functionality and allows programming Spark with its APIs. 
- In an interactive Spark shell, the Spark driver instantiates a SparkSession for you, while in a Spark application, you create a SparkSession object youself.
- Job -> A parallel computation consisting of multiple tasks that gets spawned in response to a Spark action.
- Stage 
- Task

Actions and Transformations contribute to a Spark Query Plan. Nothing in a query plan is executed untill an action is invoked. 

- A huge advantage of the lazy evaluation scheme is that Spark can inspect your computational query and ascertain how it can optimize it.
- This optimization can be done by either joining or pipelining some operations and assigning them to a stage, or breaking them into stages by determining which operations require a shuffle or exchange of data across clusters.
