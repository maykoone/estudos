# Data Modeling

## The Document Model in MongoDB

* MongoDB stores data as Documents
* Data is stored as BSON (binary representation of JSON) documents
* Document fields can be values, embedded documents, or arrays of values and documents
* Documents are kept within collections
* There are one or more collections in a database
* MongoDB is a Flexible Schema Database
* Documents in the same collection don't need to have the exact same list of fields
* The data type in any given field can vary across documents

## Constraints in Data Modeling

1. The nature of your dataset and hardware define the need to model your data
2. It is important to identify those exact constraints and their impact to create a better model
3. As your software and the technological landscape change, your model should be re-evaluated and updated
acoordingly


## The Data Modeling Methodology

1. Workload
    * data size, important reads and writes, quantify ops, qualify ops
2. Relationships
    * identify them, link or embed the related entities
3. Patterns
    * apply the ones for needed optimizations

Use the methodology in a flexible fashion. Select the step within a phase that makes sense for you project.

## Simplicity or Performance

The main trade that you will face is simplicity versus performance, prioritizing one over the other or trying to find a balance between the two.

* Modeling for simplicity
    * Small requirements in term of CPU, disk, I/O, memory
    * Apply the Workload phase to identify most important operations
    * In the Relationships phase you will try to keep the model simple by using embedded documents
    * As a result you likely see fewer collection in your design where each document contains more information
* Modeling for performance
    * Resources are likely to be used to the maximum
    * The system may require very fast read or writes operations or may have to support a ton of operations
    * Start by identifying the important operations, but also quantify in those in terms of metrics (Operation per second, required latency)
    * You will often see more collections in your design and also need to apply a series of schema design patterns to ensure the best usage of resources

## Identify the Workload

* Quantify and Qualify the queries as much as you can
* Few CRUD operations will drive the design
* For each CRUD operations try to identify:
    * Actor
    * Data in Operation 
    * Operation Type (write/read) 
    * Rate 
    * Data Durability 
    * Data Read/Written 
    * Data Life 
    * Response Latency 
    * Query Time 
    * Data Read 
    * Data Freshess