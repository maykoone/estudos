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
    // The basic data structures in MongoDB are documents.
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
