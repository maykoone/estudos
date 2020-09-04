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

## Relationships

### Relationships types and cardinality

* one-to-one, one-to-many, many-to-many are the usual cardinalities
* one-to-zillions is useful in the Big Data World
* even better, use "maximum" and "most likely" values using a tuple of the form: `[min, likely, max]`

### One-To-Many Relationship

* There are a lot of choices: embed or reference, and choose the side between "one" and "many"
    1. Embed
        - in the `one` side

            ```javascript
            //one item many reviews [0, 20]
            // items entity
            // * the documents from the many side are embedded
            // * simple applications with few documents to embed
            // * need to process main object and the N related documents together
            // * indexing is done on the array
            {
                _id: <int>,
                title: <string>,
                ...,
                reviews: [
                    {
                        user_name: <string>,
                        date: <date>
                    }
                ]
            }
            ```

        - in the `many` side

            ```javascript
            // one shipping address to many orders [10M]
            // orders entity
            // * less often used
            // * useful if "many" side is queried more often than the "one" side
            // * embedded object is duplicated
            {
                _id: 1,
                date: <date>,
                shipping_address: {
                    street: "100 Forest",
                    city: "Palo Alto"
                }
            },
            {
                _id: 2,
                date: <date>,
                shipping_address: {
                    street: "100 Forest",
                    city: "Palo Alto"
                }
            }
            ```
        - Usually, embedding in the entity most queried
    2. Reference
        - in the `one` side

            ```javascript
            // one zip many stores [0, 1, 5]
            // * array of references
            // * allows for large documents and a high count of these
            // * list of references available when retrieving the main object
            // zips entities
            {
                _id: new ObjectId(""),
                city: "",
                zip: "",
                stores: ["store1", "store2"]
            },

            //stores entities
            {
                _id: 1,
                storeId: "store1"
                ...
            }
            ```
        - in the `many` side

            ```javascript
            //one zip many stores [0,1,5]
            // * prefered representation using references
            // * allows for large documetns and a high count of these
            // * no need to manage the references on the "one" side
            //zip entities
            {
                _id: new ObjectId(""),
                city: "",
                zip: ""
            }

            //stores entities
            {
                _id: 1,
                storeId: "store1"
                ...,
                zip: <string>
            }

            ```
        - Usually, referencing in the `many` side
* Duplication may occur when emebedding on the "many" side. However, it may be OK, or even preferable
* Prefer embedding over referencing for simplicity, or when there is a small number of referenced documents as all related information is kept together
* Embed on the side of most queried collection
* Prefer referencing when the associated documents are not always needed with most often queried documents

### Many-to-Many Relationships

There are two many-to-many representations

1. Embed
    - array of subdocuments in the `many` side
    - array of subdocuments in the other `many` side
    - Usually, only the most queried side is considered

        ```javascript
        // many carts [10K] many items [0, 100K]
        // an item may appear in many carts and carts may have many items
        // * the documents from the less queried side are embedded
        // * results in duplication
        // * keep "source" for the embedded documents in another collectio
        // * indexing is done on the array
        // carts entities
        {
            _id: new ObjectId(),
            date: date,
            items: [
                {
                    _id: int,
                    title: string,
                    price: decimal
                }
            ]
        }

        // items entities
        {
            _id: new ObjectId(),
            title: string,
            price: decimal
        }
        ```
2. Reference
    - array of references in one `many` side

        ```javascript
        // many stores [1000] and many items [100K]
        // * array of references to the documents of the other collection
        // * references readily available upon first query on the "main" collection
        // items entities
        {
            _id: new ObjectId(),
            title: string,
            price: decimal,
            sold_at: ["store1", "store2", "store3"]
        }

        //stores entities
        {
            _id: new ObjectId(),
            storeId: string,
            name: string
        }
        ```
    - array of references in the other `many` side

        ```javascript
        // reference in the secondary side
        // * array of references to the documents of the other collection
        // * need a secondary query to get more information
        // items entities
        {
            _id: new ObjectId(),
            title: string,
            price: decimal
        }

        //stores entities
        {
            _id: new ObjectId(),
            storeId: string,
            name: string,
            items_sold: [
                "item1", "item2", "item3"
            ]
        }
        ```
    - It is possible to have arrays of references in both collections, however, it is not necessary and creates unnecessary overhead.

* Ensure it is a "many-to-many" relationship that should not be simplified
* A "many-to-many" relationship can be replaced by two "one-to-many" relationships but does not have to with the document model
* Prefer embedding on the most queried side
* Prefer embedding for information that is primarily static over time and may profit from duplication
* Prefer referencing over embedding to avoid managing duplication

### One-to-One Relationships

1. Embed
    - fields at same level

        ```javascript
        // one use one shipping address
        // users entities
        {
            _id: new ObjectId(),
            name: string,
            shipping_street: string,
            shipping_city: string,
            shipping_zip: string
        }
        ```
    - grouping in sub-documents

        ```javascript
        // * preferred representation
        // * preserves simplicity
        // * documents are clearer
        // users entities
        {
            _id: new ObejectId(),
            name: string,
            shipping_address: {
                street: string,
                city: string,
                zip: string
            }
        }
        ```
2. Reference
    - same identifier in both documents
        - in the main `one` side
        - in the secondary `one` side

        ```javascript
        //one store one store details
        // * add complexity
        // * possible performance improvements with:
        //      * smaller disk access
        //      * smaller amount of RAM needed
        // * ensure that same identifier are unique in both collections
        // stores entities
        {
            _id: ObjectId(),
            storeId: string,
            name: string
        }
        // store details
        {
            _id: ObjectId(),
            storeId: string,
            description: string,
            staff: [
                {
                    name: string,
                    address: string
                }
            ]
        }
        ```

* Prefer embedding over referencing for simplicity
* Use subdocuments to organize the fields
* Use a reference for optimization purposes

### One-to-Zillions

* It is a particular case of the one-to-many relationships
* The only available representation is to reference the document on the `one` side of the relationship from the `zillion` side
* Pay extra attention to queries and code that handle `zillions` of documents.

## Patterns

- Many patterns lead to some situations that would require some additional actions.
    * Duplicating data across documents.
        * Resolve with bulk updates
    * Accepting staleness in some pieces of data
        * Resolve with updates based on [change streams](https://docs.mongodb.com/manual/changeStreams/)
    * Write extra application side logic to ensure referencial integrity
        * Resolve or prevent the inconsistencies with change streams or transactions

### Attribute Pattern

- Problem
    * Lots of similar field
    * Want to search across many fields at once
    * Fields present in only a small subset of documents
- Solution
    * Break the field/value into sub-document with:
        * fieldA:field
    * Example:

        ```javascript
        // this
        {"color": "blue", "size": "large"}
        // becomes
        {
            [
                {"k": "color", "v": "blue"},
                {"k": "size", "v": "large"}
            ]
        }
        ```
- Benefits and Trade-Offs
    * Easier to index
    * Allow for non-deterministic field names
    * Ability to qualify the relationship of the original field an value
- Use Cases Examples
    * Characteristics of a product

        ```javascript
        // products collections
        // Collection with polimorphic documents
        // We have commons fields across a majority of documents (brand, price, etc..).
        // Then we have commons fields across many documents (color, size), 
        // these fields may have different meanings for the different products.
        // Then we have a set list of fields that are not going to exist in all products. 
        {
            description: "Cherry Coke 6-pack",
            manufacturer: "Coca-Cola",
            brand: "Coke",
            price: 5.99,
            ...
            color: "red",
            size: "12 ounces",
            ...
            container: "can",
            sweetener: "sugar"
        },
        {
            description: "Evian 500ml",
            manufacturer: "Danone",
            brand: "Evian",
            price: 1.99,
            ...
            size: "500 ml",
            ...
            container: "plastic bottle"
        },
        {
            description: "Charger",
            manufacturer: "China",
            brand: "MongoDB",
            price: 0,
            ...
            color: "black",
            size: "100 x 70 x 10 mm",
            ...
            input: "5V/1300 mA",
            output: "5V/1A",
            capacity: "5000 mAh"
        }

        /*
        To search effectively on one of those field that are not going to exist in all products, we'll
        an index. If you have tons of fields, you may have a lot of indexes.
        */
        db.products.find({"capacity": {$gt: 4000}}) // index on capacity
        db.products.find({"output": "5V"}) // index on output

        /*
        For this case you want to use attribute pattern.
        First we identify the list of fields to tranpose (in our case, "input", "output", "capacity"),
        Then for each field in associated value we create that pair. (The name of the keys for those pairs do not matter, let's use K for key and V for value)
        */
        {
            description: "Charger",
            manufacturer: "China",
            brand: "MongoDB",
            price: 0,
            ...
            color: "black",
            size: "100 x 70 x 10 mm",
            ...
            add_specs: [
                {k: "input", v: "5V/1300 mA"},
                {k: "output", v: "5V/1A"},
                {k: "capacity", v: "5000 mAh"}
            ]
        }

        // The last thing to do is to create an index
        db.products.createIndex({"add_specs.k":1, "add_specs.v":1})
        db.products.find({"add_specs": {$elemMatch: {k: "capacity", v: "5000 mAh"}}})
        ```
    * Set of fields all having same value type

        ```javascript
        // movies collections
        {
            title: "Dunkirk",
            ...
            release_USA: "2017/07/23",
            release_Mexico: "2017/08/01",
            release_France: "2017/08/01
        }

        // What if we want to find all the movies released between two dates across all countries?
        // Spoiler: a very complex query!!

        // Applying Attribute pattern
        {
            title: "Durkirk",
            ...
            releases: [
                {k: "release_USA", v: "2017/07/23"},
                {k: "release_Mexico", v: "2017/08/01"},
                {k: "release_France", v: "2017/08/01"},
            ]
        }

        //now issue the following simple query (using the date as string to illustrate the example)
        db.movies.find({"releases.v":{$gte:"2017/07",$lt:"2017/08"}})
        ```

> With the release of the [Wildcard Index](https://docs.mongodb.com/manual/core/index-wildcard/) functionality in MongoDB 4.2, some use cases of the Attribute Pattern can be replaced by this new index type.

### Extended Reference Pattern

- Problem
    * Too many repetitive joins
- Solution
    * Avoid a Join (`$lookup`) by embedding the joined table
    * Identify fields on the lookup side
    * Bring those fields into the main object
- Benefits and Trade-Offs
    * Faster reads
    * Reduce number of joins and lookups
    * May introduce lots of duplication if extended reference contais fields that mutate a lot
- Use cases
    * Catalog
    * Mobile Applications
    * Real-Time Analytics
    * Many-to-One relationships

        ```javascript
        // One customer can have many orders, and one order belongs to one customer
        // custormers collection
        {
            _id: "<objectId>",
            customer_id: "<string>"
            street: "<string>",
            city: "<string>",
            country: "<string>"
        }
        // orders collection
        {
            _id: "<objectId>",
            order_id: "<string>",
            customer_id: "<string>"
        }
        

        /* Let's say that our application focus is on order management and fullfilment.
        And we will query for specific orders way more often than query all the orders
        for a given custormer. For a given order we will need the shipping address information
        from the one side of the relationship, in this case, the customers, and we will
        need to perform a join most of the time.

        The prefered way to do this is only to copy the fields you need to access frequently,
        leaving the rest of the information in the source collection. We built an extended
        reference, meaning the reference is rich enough, that we will not need to perform
        the join most of the time 
        */
        // custormers collection
        {
            _id: "<objectId>",
            customer_id: "<string>"
            street: "<string>",
            city: "<string>",
            country: "<string>"
        }
        // orders collection
        {
            _id: "<objectId>",
            order_id: "<string>",
            customer_id: "<string>",
            shipping_address: {
                street: "<string>",
                city: "<string>",
                country: "<string>"
            }
        }
        ```

> The extended reference pattern will work best if you select fields that do not change often.

### Subset Pattern

- Problem
    * Working set is too big to fit in memory
    * Lot of pages are evicted from memory
    * A large part of documents is rarely needed
- Solution
    * Split the collection in 2 collections
        * Most used part of documents
        * Less used part of documents
    * Duplicate part of a 1-N or N-N relationship that is often used in the most used side
- Benefits and Trade-Offs
    * Smaller working set, as often used documents are smaller
    * Shorter disk access for bringing in additional documents from the most used collection
    * more round trips to the server
    * A little more space used on disk
- Use Cases Examples
    * List of reviews for a product
    * List of comments on an article
    * list of actors in a movie

        ```javascript
        /*
        Let's say we have a system that keeps a lot of movies in memory, and each of theses
        movies is taking a fair amount of memory.
        */
        // movies collection
        {
            _id: "<objectId>",
            title: "<string>",
            ...,
            complete_script: "<string>"
            cast: [
                ..., // [0, 1000]
                {
                    name: "<string>",
                    role: "<string>",
                }
            ]
        }

        /* Maybe the are some information that we don't need to use that often in those documents.
        For example, most of the time users want to see the top actors rather than all of them.
        We could keep only 20 of the cast members. The rest of the information can go into a separate collection.
        */
        // movies collection
        {
          _id: "<objectId>",
          title: "<string>",
          ...,
          complete_script: "<string>"
          cast: [
            ..., // [0, 20]
            {
              name: "<string>",
              role: "<string>"
            }
          ]
        }
        // cast collections [0,1000]
        {
          name: "<string>",
          role: "<string>"
        }

        /*
          A field that has a one-to-one relationship, for example, the full script
          could be moved to a new collection. And you could access this information
          through the $lookup operator.
        */
        // movies_extra_info collection
        {
          movie_title: "<string>",
          complete_script: "<string>",
          ...
        }
        ```

### Computed Pattern

- Problem
  * Costly computation or manipulation of data
  * Executed frequently on the same date, producing the same result
- Solution
  * Performe the operation and store the result in the appropriate document and collection
  * If need to redo the operations, keep the source of them
- Benefits and Trade-Offs
  * Read queries are faster
  * Saving on resources like CPU and Disk
  * May be difficult to identify the need
  * Avoid applying or overusing it unless needed
- Use Cases Examples
  * Internet of Things
  * Event sourcing
  * Time series Data
  * Frequent Aggregation Framework queries

    ```javascript
    /* Let's say we have a write operation
    that comes in. This piece of data is added as a document to a given
    collection. */

    // screenings
    {
      _id: "<objectId>",
      theather: "Cinema 1",
      movie_title: "Matrix",
      num_viewers: "123"
      revenue: "5300"
    },
    {
      _id: "<objectId>",
      theather: "Cinema 2",
      movie_title: "Matrix",
      num_viewers: "203"
      revenue: "7300"
    },
    {
      _id: "<objectId>",
      theather: "Cinema 3",
      movie_title: "Matrix",
      num_viewers: "65"
      revenue: "1300"
    }

    /*Another part of the application reads this collection and does a sum on the numbers.
    If we are doing 1000 times more reads than writes, the sum operation we do with those reads
    is identifical and very often does the exact same calculation.
    */

    // sum operation
    {
      movie_title: "Matrix",
      num_viewers: "391"
      revenue: "13900"
    },

    /*By calculating the results when get a new piece of data, we read the other elements for
    the sum and store the result in another collection with documents more appropriate to keep
    the sum for that element, maybe in a background job at regular intervals.
    This results in much fewer computation in the system
    and also reduce the amount of data being read.
    */
    ```

### Bucket Pattern

- Problem
  * Avoiding too many documents, or too big documents
  * A 1-to-Many relantionships that can't be embedded
- Solution
  * Define the optimal amount of information to group together
  * Create arrays to store the information in the main object
  * It is basically an embedded 1-to-Many relationship, where you get N documents,
  each having an average of Many sub documents
- Benefits and Trade-Offs
  * Good balance between number of data access and size of data returned
  * Makes data more manageable
  * Easy to prune data
  * Can lead to poor query results in not designed correctly
  * Less friendly to BI tools
- Use Cases Examples
  * Internet of Things

    ```javascript
    /* Let's say we have 10 million temperature sensors, 
    and each sensor is sending us a piece of data every minute.

    Trying to store each piece in a document may not work as we get
    too many documents to manage. On the other end, if we keep
    one document per device, each document is likely to reach
    the maximum size of 16 MB after a while.

    One suggestion is to have one document per device, per day.
    Each document would only carry information for a single device on 
    a single day.
    */

    // One document per device per day
    {
      device_id: 000123456,
      type: "2A",
      date: ISODate("2018-03-02"),
      temp: [ //one array per hour
        [20.0, 20.1, 20.2, ...],
        [22.0, 22.1, 22.2, ...]
      ]
    },
    {
      device_id: 000123456,
      type: "2A",
      date: ISODate("2018-03-03"), // another day
      temp: [ //one array per hour
        [20.0, 20.1, 20.2, ...],
        [22.0, 22.1, 22.2, ...]
      ]
    }

    .// Another option is to have one document per device, per hour.
    {
      device_id: 000123456,
      type: "2A",
      date: ISODate("2018-03-02T13"),
      temp: { 1: 20.0, 2: 20.1, 3: 20.2, ...}
    }
    {
      device_id: 000123456,
      type: "2A",
      date: ISODate("2018-03-02T14"), //another hour
      temp: { 1: 20.0, 2: 20.1, 3: 20.2, ...}
    }
    ```
  * Data Warehouse
  * Lots of information associated to one object

### Schema Versioning Pattern

- Problem
  * Avoid downtime while doing schema updagrades
  * Upgrading all documents can take hours, days or even weeks when dealing with big data
  * Don't want to update all documents
- Solution
  * Each document gets as `schema_version` field
  * Application can handle all versions
  * Choose your strategy to migrate the documents
- Benefits and Trande-Offs
  * No downtime needed
  * Feel in control of the migration
  * Less future technical debt
  * May need 2 indexes for same field while in migration periodo
- Use Cases Examples
  * Every application that use a database, deployed in production and heavily used.
  * System with a lot of legacy data

  ```javascript
  /* Let's say we have a peoples collections like this:
  */
  {
    _id: "<objectId>",
    firstname: "<string>",
    lastname: "<string>",
    cellphone: "<string>",
    workphone: "<string>",
    skype: "<string>",
    email:  "<string>"
  }

  /* So instead of growing the documents with unpredictable contact methods,
  the attribute pattern is applied.

  New documents get writter n the latest shape. For existing documents we will
  need to make an update to change the shape and the two shapes may coexist during the migration.
  To help the application identify the shape we will have a version
  field.
  */

  {
    _id: "<objectId>",
    schema_version: 2,
    firstname: "<string>",
    lastname: "<string>",
    contacts: [
      { methods: "<string>", value: "<string>"}
    ]
  }
  ```

### Tree Patterns

- Problem
  * Representation of hierarchical structured data
  * Different access patterns to navigate the tree
  * Provide optimized model for common operations
- Solution
  * Child Reference
  * Parent Reference
  * Array of Ancestors
  * Materialized Paths
- Benefits and Trade-Offs
  * Child Reference
    * Easy to navigate to children nodes or thee descending access patterns
  * Parent Reference
    * Immediate parent node discovery and tree updates
  * Array of Ancestors
    * Navigate upwards on the ancestors path
  * Materialized Path
    * Makes use of regular expressions to find nodes in the tree
- Use Cases Examples
  * Org Charts
  * Product Categories

    ```javascript
    // product categories
    // Parent Reference
    {
      ...,
      name: "Office",
      parent: "Swag",
      ...
    }

    // Child reference
    {
      ...,
      name: "Office",
      children: ["Books", "Eletronics", "Stickers"]
    }

    // Array of Ancestors
    {
      name: "Books",
      ancestors: ["Swag", "Office"]
    }

    // Materialized Paths
    {
      name: "Books"
      ancestors: ".Swag.Office"
    }

    // Ancestors Array + Parent Reference
    {
      name: "Umbrellas",
      parent: "Fashion",
      ancestors: ["Swag", "Fashion"]
    }
    ```

> * `$graphLookup` may be useful to query the tree structure
> * Focus on the most common queries and operations to select the most effective tree pattern

### Polymorphic Pattern

- Problem
  * Objects more similar than different
  * Want to keep objects in same collection
- Solution
  * Field tracks the type of document or sub-document
  * Application has different code paths per document type or has subclasses
- Benefits and Trade-Offs
  * Easier to implement
  * Allow to query across a single collection
- Use Cases Examples
  * Single View
  * Product Catalog

    ```javascript
    // Products
    {
      product_type: "shirt", //type of the document
      color: "blue",
      size: "large",
      price: "100.00"
    }

    {
      product_type: "book",
      title: "Le Petit Price",
      size: "20cm x 15cm x 1cm",
      price: "10.00"
    }
    ```
  * Content Management

### Approximation Pattern

- Problem
  * Data is expensive to calculate
  * It does not matter if the number is not precise
- Solution
  * Fewer, but writes with higher payload
- Benefits and Trade-Offs
  * Less writes
  * Less contention on documents
  * Statistically valid numbers
  * Not exact numbers
  * Must be implemented in the application
- Use Cases examples
  * Web page counters
  * Any counters with tolerance to imprecision
  * Metrics statistics

### Outlier Patterns

- Problem
  * Few documents would drive the solution
  * Impact would be negative on the majority of queries
- Solution
  * Implementation that works for majority
  * Field identifies outliers as exception
  * Outliers are handle differently on the application side
- Benefits and Trade-Offs
  * Optimized solution for most use cases
  * Differences handled application side
  * Difficult for aggregation or ad hoc queries
- Use Cases Examples
  * Social Networks
  * Popularity
