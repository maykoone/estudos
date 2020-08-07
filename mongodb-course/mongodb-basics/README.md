# Creating Documents
## insertOne()
`db.movieDetails.insertOne({"_id" : "tt0796366", "title" : "Star Trek II: The Wrath of Khan", year : 1982, type: "movie"})`

## insertMany()
### Ordered

```javascript
db.myMovies.insertMany(
  [
    {
      "_id" : "tt0084726",
      "title" : "Star Trek II: The Wrath of Khan",
      "year" : 1982,
      "type" : "movie"
    },
    {
      "_id" : "tt0796366",
      "title" : "Star Trek",
      "year" : 2009,
      "type" : "movie"
    },
    {
      "_id" : "tt0084726",
      "title" : "Star Trek II: The Wrath of Khan",
      "year" : 1982,
      "type" : "movie"
    },
    {
      "_id" : "tt1408101",
      "title" : "Star Trek Into Darkness",
      "year" : 2013,
      "type" : "movie"
    },
    {
      "_id" : "tt0117731",
      "title" : "Star Trek: First Contact",
      "year" : 1996,
      "type" : "movie"
    }
  ]
);

```

### Unordered

```javascript
db.myMovies.insertMany(
  [
    {
      "_id" : "tt0084726",
      "title" : "Star Trek II: The Wrath of Khan",
      "year" : 1982,
      "type" : "movie"
    },
    {
      "_id" : "tt0796366",
      "title" : "Star Trek",
      "year" : 2009,
      "type" : "movie"
    },
    {
      "_id" : "tt0084726",
      "title" : "Star Trek II: The Wrath of Khan",
      "year" : 1982,
      "type" : "movie"
    },
    {
      "_id" : "tt1408101",
      "title" : "Star Trek Into Darkness",
      "year" : 2013,
      "type" : "movie"
    },
    {
      "_id" : "tt0117731",
      "title" : "Star Trek: First Contact",
      "year" : 1996,
      "type" : "movie"
    }
  ],
  {
    ordered: false
  }
);

```

# Reading Documents: Scalar Fields
## How many movies in the movieDetails collection have exactly 2 award wins and 2 award nominations?
`db.movieDetails.find({"awards.wins": 2, "awards.nominations": 2}).count()`

## How many movies in the movieDetails collection are rated PG and have exactly 10 award nominations?
`db.movieDetails.find({"rated": "PG", "awards.nominations": 10}).count()`

# Reading Documents: Array Fields
## How many documents list just two writers: "Ethan Coen" and "Joel Coen", in that order?
`db.movieDetails.find({"writers": ["Ethan Coen", "Joel Coen"]}).count()`

## How many movies in the movieDetails collection list "Family" among its genres?
`db.movieDetails.find({"genres": "Family"}).count()`

## How many movies in the movieDetails collection list "Western" second among its genres?
`db.movieDetails.find({"genres.1": "Western"}).count()`

# Projections
## Show only movie titles
`db.movieDetails.find({}, {"title": 1})`

## Show only movie titles exclude _id
`db.movieDetails.find({}, {_id: 0, title: 1})`

## Show only movie titles, director and actors from 2010 movies
`db.movieDetails.find({"year": 2010}, {_id: 0, title: 1, director: 1, actors: 1})`

# Updating documents
## updateOne
### update movie poster of The Martian movie
`db.movieDetails.updateOne({"title": "The Martian"}, { $set: {"poster": "http://ia.media-imdb.com/images/M/MV5BMTU4NTczODkwM15BMl5BanBnXkFtZTcwMzEyMTIyMw@@._V1_SX300.jpg"}})`

### increment metacritic field of The Martian movie by 5
`db.movieDetails.updateOne({"title": "The Martian"}, { $inc: {"metacritic": 5}})`

### Add the Thriller genre to The Martian movie
`db.movieDetails.updateOne({"title": "The Martian"}, { $push: { "genres": "Thriller"}})`

## updateMany
### eliminate the rated field where its value is null
`db.movieDetails.updateMany({rated: null}, { $unset: { rated: ""}})`

## upsert
### replace movie with imdb.id = "tt0076727" or create if it didn't already exist
```javascript
db.movieDetails.updateOne({
    "imdb.id": "tt0076727"
  }, {
    $set: {
      "title" "The Martian",
      "year": 2015,
	...
 }, {
   upsert: true
 });
```

## replaceOne
```javascript
let filter = {title: "House, M.D., Season Four: New Beginnings"}

let doc = db.movieDetails.findOne(filter);

doc.poster;

doc.poster = "https://www.imdb.com/title/tt1329164/mediaviewer/rm2619416576";

doc.genres;

doc.genres.push("TV Series");

db.movieDetails.replaceOne(filter, doc);
```

# Deleting documents
## deleteOne
### delete the movie with exact id
`db.reviews.deleteOne({_id: ObjectId("595..")})`

## deleteMany
### delete all movies from a specific reviewer
`db.reviews.deleteMany({reviewer_id: 759723314})`

# Query language
## Comparison operators
### Greater than
`db.movieDetails.find({runtime: {$gt: 90}}, {_id: 0, title: 1, runtime: 1})`

### Greater than, Less than
`db.movieDetails.find({runtime: {$gt: 90, $lt: 120}}, {_id: 0, title: 1, runtime: 1})`

### Greater than or equal, less than or equal
`db.movieDetails.find({runtime: {$gte: 90, $lte: 120}}, {_id: 0, title: 1, runtime: 1})`

### Not equal
`db.movieDetails.find({rated: {$ne: "UNRATED"}}, {_id: 0, title: 1, rated: 1})`

### In
`db.movieDetails.find({rated: {$in: ["G", "PG"]}}, {_id: 0, title: 1, rated: 1})`

### Not in
`db.movieDetails.find({rated: {$nin: ["R", "PG-13"]}}, {_id: 0, title: 1, rated: 1}).pretty()`

## Element operators
### filter documents that has mpaaRating key field
`db.moviesDetails.find({mpaaRating: {$exists: true}})`

### When using null comparison, keep in mind that it will return documents that also has no mpaaRating field
`db.movieDetails.find({mpaaRating: null})`

### Filter documents that has viewerRating with type of int
`db.movies.find({viewerRating: {$type: "int"}}).pretty()`

## Logical operators
### `$or` Operator
```javascript
db.movieDetails.find({$or: [{"tomato.meter": {$gt: 95}},                               
                            {"metacritic": {$gt: 88}}]},
                     {_id: 0, title: 1, "tomato.meter": 1, "metacritic": 1})
```

### `$and` Operator
```javascript
db.movieDetails.find({$and: [{"tomato.meter": {$gt: 95}},                               
                            {"metacritic": {$gt: 88}}]},
                     {_id: 0, title: 1, "tomato.meter": 1, "metacritic": 1})
```
All clauses are implicit `$and` operator
```javascript
db.movieDetails.find({"tomato.meter": {$gt: 95},                               
                      "metacritic": {$gt: 88}},
                     {_id: 0, title: 1, "tomato.meter": 1, "metacritic": 1})
```

Use `$and` operator when need to compare on same field
```javascript
db.movieDetails.find({$and: [{"metacritic": {$ne: null}},
                             {"metacritic": {$exists: true}}]},
                          {_id: 0, title: 1, "metacritic": 1})
```

## Array operators
### `$all` operator (filter documents in which the array field matches all elements specified in the query)
```javascript
db.movieDetails.find({genres: {$all: ["Comedy", "Crime", "Drama"]}}, 
                     {_id: 0, title: 1, genres: 1}).pretty()

```

### `$size` operator (select documents if the array field is a specified size)
```javascript
db.movieDetails.find({countries: {$size: 1}}).pretty()
```

### `$elemMatch` operator (Selects documents if element in the array field matches all the specified `$elemMatch` conditions)
```javascript
martian = db.movieDetails.findOne({title: "The Martian"})
delete martian._id;
martian.boxOffice = [
    {"country": "USA", "revenue": 228.4},
    {"country": "Australia", "revenue": 19.6},
    {"country": "UK", "revenue": 33.9},
    {"country": "Germany", "revenue": 16.2},
    {"country": "France", "revenue": 19.8}
]
db.movieDetails.insertOne(martian);
db.movieDetails.find({boxOffice: {$elemMatch: {"country": "Germany", "revenue": {$gt: 17}}}})
db.movieDetails.find({boxOffice: {$elemMatch: {"country": "Germany", "revenue": {$gt: 16}}}})
```

another example of `$elemMatch`
```javascript
// surveys collection schema
{_id: ObjectId("5964e8e5f0df64e7bc2d7373"),
 results: [{product: "abc", score: 10}, {product: "xyz", score: 9}]}

// How many documents in the results.surveys collection contain a score of 7 for the product, "abc"?
db.surveys.find({"results": {$elemMatch: {"product": "abc", "score": 7}}}).count()
```

## `$regex` Operator
### Selects documents where values match a specified regular expression
```javascript
db.movieDetails.find({"awards.text": {$regex: /^Won.* /}}, {_id: 0, title: 1, "awards.text": 1}).pretty()
```



