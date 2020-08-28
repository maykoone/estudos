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

## Basic Reads

- Find One / Find Single Document

    ```java
    // without any query
    MongoCursor cursor = moviesCollection.find(new Document()).limit(1).iterator();
    Document first = (Document) cursor.next();

    // using a query
    Document queryFilter = new Document("cast", "Salma Hayek");
    Document actual = moviesCollection.find(queryFilter).limit(1).iterator().next();

    /**
    * if we call next() on an iterator that returns no document we will get a NoSuchElementException.
    * To be safe, we should use the tryNext method. This will return null if nothing exists in the
    * iterator
    */
    Document actual =
        moviesCollection.find(new Document("title", "foobarbizzlebazzle")).iterator().tryNext();
    ```

- Find a list of documents

    ```java
    Document queryFilter = new Document("cast", "Salma Hayek");

    // we'll create an ArrayList to hold our results
    List<Document> results = new ArrayList<>();

    // now we issue the query, and send them directly into our container
    moviesCollection.find(queryFilter).into(results);
    ```

- Projections

    ```java
    // db.movies.find({cast: "Salma Hayek"}, { title: 1, year: 1, _id: 0 })
    Document queryFilter = new Document("cast", "Salma Hayek");
    Document result =
        moviesCollection
            .find(queryFilter)
            .limit(1)
             // unless we explicitly remove the _id key it will remain in the result or results.
            .projection(new Document("title", 1).append("year", 1).append("_id", 0)) 
            .iterator()
            .tryNext();
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
    import static com.mongodb.client.model.Projections.*;
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

## POJO with the MongoDB Java Driver

[POJO Docs](http://mongodb.github.io/mongo-java-driver/3.6/driver/getting-started/quick-start-pojo/), [Codecs Tutorial](http://mongodb.github.io/mongo-java-driver/3.2/bson/codecs/)

- Read

    ```java
    //our POJO
    import org.bson.codecs.pojo.annotations.BsonProperty;
    import org.bson.types.ObjectId;

    public class ActorBasic {
        @BsonProperty("_id")
        private ObjectId id;

        private String name;
        @BsonProperty("date_of_birth")
        private Date dateOfBirth;

        private List awards;
        @BsonProperty("num_movies")
        private int numMovies;

        // getters and setters omitted
    }
    /**
     * Here we are instantiating a codec registry and telling it to use
     * the default pojo provider, building it with an automatic setting,
     * which uses type introspection.
     * 
     * - Registry is a "factory" of Codecs
     * - Codecs determines how BSON data is converted and into what type.
     * - There are default and custom codecs
     */
    CodecRegistry pojoCodecRegistry =
        CodecRegistries.fromRegistries(
            MongoClientSettings.getDefaultCodecRegistry(),
            CodecRegistries.fromProviders(PojoCodecProvider.builder().automatic(true).build()));
    
    MongoCollection<ActorBasic> actors =
        testDb.getCollection("actors", ActorBasic.class).withCodecRegistry(pojoCodecRegistry);

    // create a  query to retrieve a document with a given actor id.
    Bson queryFilter = new Document("_id", actor1Id);

    // use our query to pipe our document into an ActorBasic object in one
    // quick line
    ActorBasic pojoActor = actors.find(queryFilter).iterator().tryNext();
    ```

- Read With Custom Codec
    ```java
    /**
     * There are scenarios where you have to write custom codecs.
     * Sometimes your app uses data types that are
     *  different from what is stored in the database.
     */
    // example of custom codec implementation
    public class ActorCodec implements CollectibleCodec<ActorWithStringId> {

        private final Codec<Document> documentCodec;

        public ActorCodec() {
            this.documentCodec = new DocumentCodec();
        }

        public void encode(
            BsonWriter bsonWriter, ActorWithStringId actor, EncoderContext encoderContext) {
            // logic to map actor to document
            documentCodec.encode(bsonWriter, actorDoc, encoderContext);
        }

        @SuppressWarnings("unchecked")
        @Override
        public ActorWithStringId decode(BsonReader bsonReader, DecoderContext decoderContext) {
            Document actorDoc = documentCodec.decode(bsonReader, decoderContext);
            //logic to map document to actor
            return actor;
        }

        @Override
        public Class<ActorWithStringId> getEncoderClass() {}

        @Override
        public ActorWithStringId generateIdIfAbsentFromDocument(ActorWithStringId actor) {}

        @Override
        public boolean documentHasId(ActorWithStringId actor) {}

        @Override
        public BsonString getDocumentId(ActorWithStringId actor) {}
    }

     // first we establish the use of our new custom codec
    ActorCodec actorCodec = new ActorCodec();
    // then create a codec registry with this codec
    CodecRegistry codecRegistry =
        CodecRegistries.fromRegistries(MongoClientSettings.getDefaultCodecRegistry(), CodecRegistries.fromCodecs(actorCodec));
    // we can now access the actors collection with the use of our custom
    // codec that is specifically tailored for the actor documents.
    Bson queryFilter = new Document("_id", actor1Id);
    MongoCollection<ActorWithStringId> customCodecActors =
        testDb.getCollection("actors", ActorWithStringId.class).withCodecRegistry(codecRegistry);
    // we retrieve the first actor document
    ActorWithStringId actor = customCodecActors.find(Filters.eq("_id", actor1Id)).first();
    ```

- Read using custom field codec

    ```java
    /**
    * use the default CodecRegistry while customizing the
    * fields that we know need special treatment
    */
    // Implementing Codec Interface
    public class StringObjectIdCodec implements Codec<String> {}
    // select a class that will be used as our POJO
    ClassModelBuilder<ActorWithStringId> classModelBuilder =
        ClassModel.builder(ActorWithStringId.class);
    // get the property that needs type conversion
    PropertyModelBuilder<String> idPropertyModelBuilder =
        (PropertyModelBuilder<String>) classModelBuilder.getProperty("id");
    // apply type conversion to the property of interest
    // StringObjectIdCodec describes specifically how to encode and decode
    // the ObjectId into a String and vice-versa.
    idPropertyModelBuilder.codec(new StringObjectIdCodec());
    // use the default CodecRegistry, with the changes implemented above
    // through registering the classModelBuilder with the PojoCodecProvider
    CodecRegistry stringIdCodecRegistry =
        fromRegistries(
            MongoClientSettings.getDefaultCodecRegistry(),
            fromProviders(
                PojoCodecProvider.builder()
                    .register(classModelBuilder.build())
                    .automatic(true)
                    .build()));
    // we're done! Lets test if it worked! let us get the actors collection
    MongoCollection<ActorWithStringId> actors =
        testDb
            .getCollection("actors", ActorWithStringId.class)
            .withCodecRegistry(stringIdCodecRegistry);
    ```

- Write using custom codec

    ```java
    // custom codec
    ActorCodec actorCodec = new ActorCodec();
    // we now create a codecRegistry with the custom Codec
    CodecRegistry codecRegistry =
        fromRegistries(MongoClientSettings.getDefaultCodecRegistry(), fromCodecs(actorCodec));
    // and get the "actors" collection using our new Registry
    MongoCollection<ActorWithStringId> customCodecActors =
        testDb.getCollection("actors", ActorWithStringId.class).withCodecRegistry(codecRegistry);
    // we can now create a new actor and insert it directly into our
    // collection with all required features present.
    ActorWithStringId actorNew = new ActorWithStringId();
    actorNew.setNumMovies(2);
    actorNew.setName("Norberto");
    customCodecActors.insertOne(actorNew);
    ```
