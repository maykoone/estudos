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


