# Mongodb for java developers

## MongoDB URI

```
mongodb://<username>:<password>@<cluster-node01-url:port><cluster-node02-url:port>,<cluster-node03-url:port>/<database>

or srv string (service record that holds the lists of hostnames of the cluster). we dont need to know each server in the cluster

mongodb+srv://<username>:<password>@<host>/database
```

## MongoDB Java driver

```xml
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongodb-driver-sync|mongodb-driver-async</artifactId>
    <version>...</version>
</dependency>
```

### Java driver base classes

- MongoClient

    ```java
    MongoClient mongoClient;
    // basic provide string uri to MongoClients builder
    mongoClient = MongoClients.create(uri);

    // it's also possible to extend the configuration using MongoClientSettings
    ConnectionString connectionString = new ConnectionString(uri);
        MongoClientSettings clientSettings =
            MongoClientSettings.builder()
                .applyConnectionString(connectionString)
                .applicationName("mflix")
                .applyToConnectionPoolSettings(
                    builder -> builder.maxWaitTime(1000, TimeUnit.MILLISECONDS))
                .build();

    mongoClient = MongoClients.create(clientSettings);
    ```

- MongoDatabase

    ```java
    // with mongoClient we can get a MongoDatabase object to access, create or drop our collections
    MongoDatabase database;
    database = mongoClient.getDatabase("mflix");
    ```

- MongoCollection

    ```java
    // MongoCollection instance is what is used to read and write to the documents
    MongoCollection collection;
    collection = database.getCollection("movies");
    MongoIterable<Document> cursor = collection.find().skip(10).limit(20);
    ```

- Document

    ```java
    // The basic data structures in MongoDB are documents. We use documents to define data objects, queries, update operations and configuration settings
    Document document;
    document = new Document("name", new Document("first", "Norberto").append("last", "Leite"));
    collection.insertOne(document);
    ```

- Bson

    ```java
    //The Document class implements the Bson interface, because Documents are BSON data structures.
    Bson bson;
    bson = new Document("name", new Document("first", "Norberto").append("last", "Leite"));
    ```

## Query Builders

- Filters Builder

    ```java
    import static com.mongodb.client.model.Filters.*;
    // given the query below
    // db.movies.find({cast: "Salma Hayek"}).limit(1)

    // in the old style we use Document to build the query
    Document onerousFilter = new Document("cast", "Salma Hayek");
    Document actual = moviesCollection.find(onerousFilter).limit(1).iterator().tryNext();

    // using query Filters Builder API we can compose our queries
    Bson queryFilter = eq("cast", "Salma Hayek");
    Document builderActual = moviesCollection.find(queryFilter).limit(1).iterator().tryNext();

    // db.movie.find({cast: { $all: ["Salma Hayek", "Johnny Depp"] }})
    //old style
    Document oldFilter =
        new Document("cast", new Document("$all", Arrays.asList("Salma Hayek", "Johnny Depp")));

    // Filters Builder
    Bson queryFilter = all("cast", "Salma Hayek", "Johnny Depp");

    // Multipe predicates
    // db.movies.find({
    //     cast: "Tom Hanks",
    //     year: { $gte: 1990, $lt: 2005 },
    //     metacritic: { $gte: 80 }
    // })
    Bson queryFilter =
        and(
            // matching tom hanks
            eq("cast", "Tom Hanks"),
            // released after 1990
            gte("year", 1990),
            // but before 2005
            lt("year", 2005),
            // with a minimum metacritic of 80
            gte("metacritic", 80));
    List<Document> results = new ArrayList<>();
    moviesCollection.find(queryFilter).into(results);
    ```

- Projections Builder

    ```java
    // given the following query where we want to project only title and year
    // db.movies.find({cast: "Salma Hayek"}, { title: 1, year: 1 })

    // using Document
    Document oldFilter = new Document("cast", "Salma Hayek");
    Document oldResult =
        moviesCollection
            .find(oldFilter)
            .limit(1)
            .projection(new Document("title", 1).append("year", 1))
            .iterator()
            .tryNext();

    // With Projections
    Bson queryFilter = and(eq("cast", "Salma Hayek"));
    Document result =
        moviesCollection
            .find(queryFilter)
            .limit(1)
            // this feels much more declarative
            .projection(fields(include("title", "year")))
            .iterator()
            .tryNext();

    // we can exclude _id field using excludeId() or exclude(field) to exclude an arbitrary field
    Document no_id =
        moviesCollection
            .find(queryFilter)
            .limit(1)
            .projection(fields(include("title", "year"), excludeId()))
            .iterator()
            .tryNext();
    ```
