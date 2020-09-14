# MongoDB Performance

## Hardware considerations

* In MongoDB deployments, memory is a quintessencial resource
* MongoDB has storage engines that are very dependent on RAM
* A signigicant number of operations rely heavily in RAM
    * Aggregation pipeline operations
    * Index traversing
    * Write operations (Writes are first performed in RAM allocate pages)
    * Query engine
    * Connections
* More RAM you have available, the more performance your department of MongoDB will tend to be
* CPU is generally more attached with two main factors
    * Storage engines
    * Concurrency level
* By default, MongoDB will try to use all available CPU cores to responde to incoming requests
* MongoDB will perform better the more CPU resources we have available
* There are certain operations that will require the availability of CPU cycles
    * Page compression
    * data calculation
    * Aggregation framework operations
    * Map Reduc
* For persisting data, MongoDB will use disks
* The faster we can write and read data, the faster you persistency layer will respond to database and application requests
* The types of disks will greatly affect the overall performance of your MongoDB deployment
* MongoDB will benefit from some but not all RAID architectures
* The recommended RAID architecture for MongoDB deployments is RAID 10
* RAID 5 and RAID 6 are higly discourage for MongoDB deployments
* MongoDB deployments also rely on network hardware
    * The faster and the larger the bandwidth is for you network, the better performance you will experience

## How Data is Stored on Disk?

* `/data/db` is the default MongoDB data path
* We can change the default data path

    `mongod --dbpath /data/db --fork --logpath /data/db/mongodb.log`

* For each collection, or index, the storage engine will write an individual file.
* By specifying `--directoryperdb` we'll get slightly different organization in the way that we are going to have a folder for each single database

    `mongod --dbpath /data/db --fork --logpath /data/db/mongodb.log --directoryperdb`
* We can go a little bit step forward in terms of organization of our data, especially on WiredTiger

    `mongod --dbpath /data/db --fork --logpath /data/db/mongodb.log --directoryperdb --wiredTigerDirectoryForIndexes`

* If we look into our database folder, our dbpath, we're still creating one single folder for each database and we're going to have a single directory for collections and one for all index files. If we have several disks in our server, this will enable a great deal of I/O paralyzation. We can create symbolic links to mount points on different physical drives, one for collections and other for the indexes.
    * Paralyzation of I/O  can improve the overall throughput of our persistency layer.



## Indexes

* What problem do indexes try to solve?
    * Slow Queries
* If we don't use an index when we query our collection, then the database will have to look at every single document.
* Indexes reduce the number of documents MongoDB needs to examine to satisfy a query.
* The MongoDB index keeps a reference to every document in our collection
* When we create an index, we have to specify which fields on the documents in our collection we want to index on.
* `_id` field is automatically indexed on all collections
* It's possible to have many indexes on the same collection
* MongoDB uses a data structure called a b-tree to store its indexes
* Index overhead
    * With each additional index, we decrease our write speed for a collection
    * Every time there's a new document inserted, updated or deleted, a collections indexes need to be updated
    * We don't want to have too many unnecessary indexes in a collection because there would then be an unnecessary loss in insert, update and delete performance.

### Single Field Indexes

* Simplest indexes in MongoDB
* `db.<collection>.createIndex({ <field> : <direction> })`
* Key features
    * Keys from only one field
    * Can find a single value for the indexed field
    * Can find a range of values
    * Can use dot notation to index fields in subdocuments

      ```javascript
      db.collection.createIndex({ "subDoc.indexedField": 1 })
      ```
    * Can be used to find several distinct values in a single query

```shell
# query without index, Collection scan, look at every document to return one document
> db.people.find({ "ssn" : "720-38-5636" }).explain("executionStats")
{
  "queryPlanner" : {
    ...
    "winningPlan" : {
      "stage" : "COLLSCAN",
      ...
    },
    "rejectedPlans" : [ ]
  },
  "executionStats" : {
    "executionSuccess" : true,
    "nReturned" : 1,
    "executionTimeMillis" : 63,
    "totalKeysExamined" : 0,
    "totalDocsExamined" : 50474,
    ...
  },
  "serverInfo" : {
    ...
  },
  "ok" : 1,
  ...
}

# create index on the ssn field
> db.people.createIndex( { ssn: 1 } )

# issue our query again, now using the index scan
> db.people.find({ "ssn" : "720-38-5636" }).explain("executionStats")
{
  "queryPlanner" : {
    ...
    "winningPlan" : {
      "stage" : "FETCH",
      "inputStage" : {
        "stage" : "IXSCAN",
        ...
      }
    },
    "rejectedPlans" : [ ]
  },
  "executionStats" : {
    "executionSuccess" : true,
    "nReturned" : 1,
    "executionTimeMillis" : 0,
    "totalKeysExamined" : 1,
    "totalDocsExamined" : 1,
    ...
  },
  "serverInfo" : {
    ...
  },
  "ok" : 1,
  ...
}
```

### Explain method

* Using explain on a query is the best way to analyze what happens when the query is executed.
* Explain can also be used to tell us what would happen without the query being executed
* Important things returned by explain
  * Execution time
  * Number of keys read, documents read and documents returned
  * Plans selected and rejected

```javascript
// Running explain is as easy as appending the explain method to the end of our query
db.people.find({"address.city":"Lake Meaganton"}).explain()

// If you need to rerun several queries, it's suggested that we create an explainable object
exp = db.people.explain()
exp.find({"address.city":"Lake Meaganton"})
exp.find({"address.city":"Lake Brenda"})

// The default mode of operation for explain is to return what would happen without actually executing the query
// is the exact same thing as passing the paramenter queryPlanner to explain
exp = db.people.explain("queryPlanner")

// We can also pass executionStats as an argument to explain. This will execute the query and return different statistics
exp = db.people.explain("executionStats")

// The most verbose way that we can get explain output is by using allPlansExecution
exp = db.people.explain("allPlansExecution")

```

### Sorting with Single Field Indexes

* Documents can be sorted in memory or by using and index.
* Sorting a large amount of documents in memory might be an expensive operation.
* The server is going to abort sorting in memory when 32 MB of memory is being used.
* In an index, the keys are ordered according to the field specified during index creation. The server can tak advantage of this via sort.
* If a query is using an index scan, the order of the documents returned is guaranteed to be sorted by the index keys.
* It's important to point out that the documents are only going to be ordered according to the fields that make up the index.
* We can sort the documents either ascending or descending, regardless of the physical ordering of the index keys.

### Compound Indexes

* A compound index is an index on two or more fields

  ```javascript
  db.people.createIndex({ last_name: 1, first_name: 1})
  ```
* The field or fields that come first are in some ways more useful than the fields that come later

### Compound Indexes: Prefixes

* An index prefix is a continuous subset of our compound index that starts to the left-hand side an it has to be continuous
* An index prefix can be used just like a regular index
* The query planner will ignore the other parts of the index and will use the prefix to find your documents
* So let's consider the following compound index

  ```javascript
  db.people.createIndex({ job: 1, last_name: 1, first_name: 1})
  ```

* It has the following index prefixes:
  * `{ job: 1 }`
  * `{ job: 1, last_name: 1}`
  * `{ job: 1, last_name: 1, first_name: 1}`
* if we have a compound index, it can serve as query use for both the compound and any of its prefixes, but it won't use an index when we're not querying on a prefix.
* There's no point in building two indexes when you can have everything with just one index

### Sorting with Compound Indexes

* We can use compound indexes for sorting by using index key pattern as our sort predicate
* So let's consider our previous `job_last_name_first_name` index. It can support the following sort operations
  * `db.people.find({}).sort({ job: 1 })`
  * `db.people.find({}).sort({ job: -1 })`
  * `db.people.find({}).sort({ job: 1, last_name: 1 })`
  * `db.people.find({}).sort({ job: -1, last_name: -1 })`
  * `db.people.find({}).sort({ job: 1, last_name: 1, first_name: 1 })`
  * `db.people.find({}).sort({ job: -1, last_name: -1, first_name: -1 })`
* However it cannot support the following sort operations
  * `db.people.find({}).sort({ job: 1, last_name: -1 })`
  * `db.people.find({}).sort({ job: -1, last_name: 1, first_name: -1 })`
* In order to walk the index backwards all we need to do is invert each key
* We can filter and sort our queries by splitting up our index prefix between the query and sort predicates

  ```javascript
  db.people.find({job:"Financial adviser"}).sort({job:1, last_name: 1})
  ```

### Multikey indexes

* When we index on a field that is an array, this is what we call a multikey index.
* Each entry in the array, the server will create a separate index key
* We can also index on nested documents
* We want to be careful with creating multikey indexes because we want to make sure that our arrays don't grow too large
* MongoDB only recognizes that an index is multikey when a document is inserted where that field is in an array
* If we try to create an index where both fields are arrays, then this should fail
* We can still create compound multikey index when only one field is an array

### Partial Indexes

* We can index only a portion of the documents in a collection

  ```javascript
  db.collection.createIndex(
    { field: <1 or -1>, },
    { partialFilterExpression: <expression>})

  // example
  db.restaurants.createIndex(
   { cuisine: 1, name: 1 },
   { partialFilterExpression: { rating: { $gt: 5 } } }
  )
  ```
* When we create a partial index, we're effectively reducing the number of index keys that we need to store.
* Partial indexes can also be useful multikey indexes
* sparse indexes are a special case of partial indexes. With a sparse index, we only index documents where the field exists, rathen than creating an index key with a null value

  ```javascript
  db.restaurants.createIndex(
    { stars: 1 },
    { sparse: true }
  )

  //same as
  db.restaurants.createIndex(
    { stars: 1 },
    { partialFilterExpression: { stars: { $exists: true } } }
  )
  ```
* In order to use a partial index, the query must be guaranteed to match a subset of the documents, specfied by the filter expression.

  ```javascript
  db.restaurants.createIndex(
    { stars: 1},
    { partialFilterExpression: { stars: { $gt: 3.5 }}}
  )

  db.restaurants.find({'address.city': 'New York', stars: { $gt: 4.0 }})
  ```
* `_id` indexes cannot be partial indexes
* Shard key indexes cannot be partial indexes

### Text Indexes

* For certain use cases, it can be useful to search for documents based on the words that are a part of those text fields
* We can leverage MongoDB's full text search capabilities, while avoiding collections scans.
* Rather than specifying that we want our indexes to be ascending or descending, rather we pass a special text keyword to `createIndex`

  ```javascript
  db.collection.createIndex({ field: "text" })
  ```
* MongoDB is going to process the text field and create an index key for every unique word in the string
* By default, text indexes are case insensitive
* We want to be aware that the bigger our text fields are, the more index keys per document we'll be producing. We'll want to watch very closely how big our index is getting to make sure that it fits entirely in RAM
* Text indexes it's going to take longer than normal to build and we'll also see a more significant decrease in write perfomance, than with a typical index.
* One strategy for reducing the number of index keys that need to be examined, would be to create a compound text index.

  ```javascript
  db.products.createIndex({ category: 1, productName: "text" })
  ```
* We use `$text` and `$search` operators to search using text indexes

  ```javascript
  db.products.find({
    category: "Clothing",
    $text: { $search: "t-shirt" }
  })
  ```
* The `$text` operator assigns a score to each document, based on the relevance of that document for a given search. We can project the special `textScore` value to our returned results

  ```javascript
  db.products.find({
    category: "Clothing",
    $text: { $search: "t-shirt" }
  },
  {
    score: { $meta: "textScore" }
  })
  ```

### Collations

* Collations allow users to specify language specific rules for string comparison
* There are multiple different settings that collations allow
* Collations can be defined at several different levels.

  ```javascript
  // we can define a collation for a collection
  db.createCollection("my_collection", { collation: { locale: "pt" }})

  // we can use collations for specific requests
  db.my_collection.find({_id: {$exists: true}}).collation({locale: 'it'})
  db.my_collection.aggregate([ {$match: {_id: {$exists: true}}}], {collation: {locale: 'es'}})

  // we can specify different collations for our indexes
  db.my_collection.createIndex( {name: 1 }, {collation: {locale: 'it' }})
  ```
* To use an index with a specific collation, the query must match the collation of the index

  ```javascript
  db.people.createIndex( {name: 1}, {collation: {locale: 'it' }})

  //we're only able to use that index when specify the same collation in the query
  db.people.find({name: 'Leonardo'}).collation({locale: 'it'})
  ```

### Wildcard Indexes

* Wildcard indexes allow you to dynamically create indexes on all fields or a selected subset of fields for each document in a collection
* Some workloads have unpredictable access patterns. This can make it very difficult to plan an effective indexing strategy
* We need able to index on multiple fields without the overhead of maintaining multiple indexes
* If we use a wildcard index, we can assume that certain indexes exists without the need to manually create them
* Wildcard indexes are not a replacement for traditional indexes
* We can index all fields in a collection or use dot notation and wildcard projections to index a subset of fields in each document
* Syntax

  ```javascript
  // index everything
  db.data.createIndex({'$**': 1})

  // index 'a.b' and all subpaths
  db.data.createIndex({'a.b.$**': 1})

  // index 'a' and all subpaths
  db.data.createIndex({'$**': 1}, {wildcardProjection: {a: 1}})

  // index everything but 'a'
  db.data.createIndex({'$**': 1}, {wildcardProjection: {a: 0}})
  ```