# Aggregation Framework

## Aggregation Structure

```javascript
// simple first example
db.solarSystem.aggregate([{
  "$match": {
    "atmosphericComposition": { "$in": [/O2/] },
    "meanTemperature": { $gte: -40, "$lte": 40 }
  }
}, {
  "$project": {
    "_id": 0,
    "name": 1,
    "hasMoons": { "$gt": ["$numberOfMoons", 0] }
  }
}], { "allowDiskUse": true});

```

## `$match`

```javascript
// $match all celestial bodies, not equal to Star
db.solarSystem.aggregate([{
  "$match": { "type": { "$ne": "Star" } }
}]).pretty()

// same query using find command
db.solarSystem.find({ "type": { "$ne": "Star" } }).pretty();
```

 Lab: Find movies that meet the following criteria: imdb.rating is at least 7; genres does not contain "Crime" or "Horror"; rated is either "PG" or "G"; languages contains "English" and "Japanese"

```javascript
var pipeline = [ { $match: { "imdb.rating": { $gte: 7}, genres: { $nin: ["Crime", "Horror"] }, rated: { $in: ["PG", "G"] }, languages: { $all: ["English", "Japanese"] } } } ]
db.movies.aggregate(pipeline).itcount()
```

## `$project`

```javascript
// project ``name`` and remove ``_id``
db.solarSystem.aggregate([{ "$project": { "_id": 0, "name": 1 } }]);

// project ``name`` and ``gravity`` fields, including default ``_id``
db.solarSystem.aggregate([{ "$project": { "name": 1, "gravity": 1 } }]);

// using dot-notation to express the projection fields
db.solarSystem.aggregate([{ "$project": { "_id": 0, "name": 1, "gravity.value": 1 } }]);

// reassing ``gravity`` field with value from ``gravity.value`` embeded field
db.solarSystem.aggregate([{"$project": { "_id": 0, "name": 1, "gravity": "$gravity.value" }}]);

// creating a document new field ``surfaceGravity``
db.solarSystem.aggregate([{"$project": { "_id": 0, "name": 1, "surfaceGravity": "$gravity.value" }}]);

// creating a new field ``myWeight`` using expressions
db.solarSystem.aggregate([{"$project": { "_id": 0, "name": 1, "myWeight": { "$multiply": [ { "$divide": [ "$gravity.value", 9.8 ] }, 86 ] } }}]);
```

Lab: find a count of the number of movies that have a title composed of one word

```javascript
db.movies.aggregate([
  {
    $match: {
      title: {
        $type: "string"
      }
    }
  },
  {
    $project: {
      title: { $split: ["$title", " "] },
      _id: 0
    }
  },
  {
    $match: {
      title: { $size: 1 }
    }
  }
]).itcount()
```

Example: assure that writers is an array and is not empty, then project title, cast and writers in upper case

```javascript
db.movies.aggregate([
  { 
    $match: { 
      "writers": {
        $elemMatch: { $exists: true }
      }
    }
  }, 
  { 
    $project: {
      _id: 0,
      title: 1,
      cast: 1,
      writers: {
        $map: {
          input: "$writers",
          as: "writer",
          in: { $toUpper: "$$writer" }
        }
      }
    }
  }
]).pretty()
```

Example: find how many movies in our movies collection are a "labor of love", where the same person appears in cast, directors, and writers

```javascript
db.movies.aggregate([
  {
    $match: {
      cast: { $elemMatch: { $exists: true } },
      directors: { $elemMatch: { $exists: true } },
      writers: { $elemMatch: { $exists: true } }
    }
  },
  {
    $project: {
      _id: 0,
      cast: 1,
      directors: 1,
      writers: {
        $map: {
          input: "$writers",
          as: "writer",
          in: {
            $arrayElemAt: [
              {
                $split: ["$$writer", " ("]
              },
              0
            ]
          }
        }
      }
    }
  },
  {
    $project: {
      labor_of_love: {
        $gt: [
          { $size: { $setIntersection: ["$cast", "$directors", "$writers"] } },
          0
        ]
      }
    }
  },
  {
    $match: { labor_of_love: true }
  },
  {
    $count: "labors of love"
  }
])

```

## `$addFields`

```javascript
// using ``$addFields`` to generate the new computed field values
db.solarSystem.aggregate([
{"$addFields":{
    "gravity": "$gravity.value",
    "mass": "$mass.value",
    "radius": "$radius.value",
    "sma": "$sma.value"}
}]);

// combining ``$project`` with ``$addFields``
db.solarSystem.aggregate([
{"$project": {
    "_id": 0,
    "name": 1,
    "gravity": 1,
    "mass": 1,
    "radius": 1,
    "sma": 1}
},
{"$addFields": {
    "gravity": "$gravity.value",
    "mass": "$mass.value",
    "radius": "$radius.value",
    "sma": "$sma.value"
}}]);

```

## `$geoNear`

```javascript
// using ``$geoNear`` stage
db.nycFacilities.aggregate([
  {
    "$geoNear": {
      "near": {
        "type": "Point",
        "coordinates": [-73.98769766092299, 40.757345233626594]
      },
      "distanceField": "distanceFromMongoDB",
      "spherical": true
    }
  }
]).pretty();

// include ``limit`` to results
db.nycFacilities.aggregate([
  {
    $geoNear: {
      near: {
        type: "Point",
        coordinates: [-73.98769766092299, 40.757345233626594]
      },
      distanceField: "distanceFromMongoDB",
      spherical: true,
      query: { type: "Hospital" },
      limit: 5
    }
  }
]).pretty()

```

## cursor like stages

Using the find method we can use cursor methods

```javascript
// count the number of documents
db.solarSystem.find({}, {"_id": 0, "name": 1, "numberOfMoons": 1}).count();

// skip documents
db.solarSystem.find({}, {"_id": 0, "name": 1, "numberOfMoons": 1}).skip(5).pretty();

// limit documents
db.solarSystem.find({}, {"_id": 0, "name": 1, "numberOfMoons": 1}).limit(5).pretty();

// sort documents
db.solarSystem.find({}, { "_id": 0, "name": 1, "numberOfMoons": 1 }).sort( {"numberOfMoons": -1 } ).pretty();
```

We can do the same using the aggregation framework

```javascript

// ``$limit`` stage
db.solarSystem.aggregate([{
  "$project": {
    "_id": 0,
    "name": 1,
    "numberOfMoons": 1
  }
},
{ "$limit": 5  }]).pretty();

// ``skip`` stage
db.solarSystem.aggregate([{
  "$project": {
    "_id": 0,
    "name": 1,
    "numberOfMoons": 1
  }
}, {
  "$skip": 1
}]).pretty()

// removing ``$project`` stage since it does not interfere with our count
db.solarSystem.aggregate([{
  "$match": {
    "type": "Terrestrial planet"
  }
}, {
  "$count": "terrestrial planets"
}]).pretty();

// sorting on more than one field
db.solarSystem.aggregate([{
  "$project": {
    "_id": 0,
    "name": 1,
    "hasMagneticField": 1,
    "numberOfMoons": 1
  }
}, {
  "$sort": { "hasMagneticField": -1, "numberOfMoons": -1 }
}]).pretty();
```

## `$sample`

Randomly selects the specified number of documents from its input.

```javascript
// sampling 200 documents of collection ``nycFacilities``
db.nycFacilities.aggregate([{"$sample": { "size": 200 }}]).pretty();
```

Lab: For movies released in the USA with a tomatoes.viewer.rating greater than or equal to 3, calculate a new field called num_favs that represets how many favorites appear in the cast field of the movie. Sort your results by num_favs, tomatoes.viewer.rating, and title, all in descending order.

What is the title of the 25th film in the aggregation result?

- My pipeline:

```javascript
var favorites = [
  "Sandra Bullock",
  "Tom Hanks",
  "Julia Roberts",
  "Kevin Spacey",
  "George Clooney"]
db.movies.aggregate([
  {
    $match: { 
      cast: { $elemMatch: { $exists: true } }, 
      countries: "USA", 
      "tomatoes.viewer.rating": { $gte: 3 } }
  },
  {
    $addFields: { 
      num_favs: { $size: { $setIntersection: ["$cast", favorites] } } 
    }
  },
  { 
    $sort: { num_favs: -1, "tomatoes.viewer.rating": -1, title: -1 } 
  },
  { 
    $skip: 24 
  },
  { 
    $limit: 1 
  }
]).pretty()
```

- course pipeline (using the `$in` operator to match, whereas i only checked for the existence of the field; using `$project` instead of `$addFields`)

```javascript
var favorites = [
  "Sandra Bullock",
  "Tom Hanks",
  "Julia Roberts",
  "Kevin Spacey",
  "George Clooney"]

db.movies.aggregate([
  {
    $match: {
      "tomatoes.viewer.rating": { $gte: 3 },
      countries: "USA",
      cast: {
        $in: favorites
      }
    }
  },
  {
    $project: {
      _id: 0,
      title: 1,
      "tomatoes.viewer.rating": 1,
      num_favs: {
        $size: {
          $setIntersection: [
            "$cast",
            favorites
          ]
        }
      }
    }
  },
  {
    $sort: { num_favs: -1, "tomatoes.viewer.rating": -1, title: -1 }
  },
  {
    $skip: 24
  },
  {
    $limit: 1
  }
])

```

Lab: Calculate an average rating for each movie in our collection where English is an available language, the minimum imdb.rating is at least 1, the minimum imdb.votes is at least 1, and it was released in 1990 or after. What film has the lowest normalized_rating?

- My pipeline

```javascript
db.movies.aggregate([
  { 
    $match: { 
      languages: { $in: ["English"] }, 
      "imdb.rating": { $gte: 1 }, 
      "imdb.votes": { $gte: 1 }, 
      year: { $gte: 1990 } } 
  },
  { 
    $project: { 
      _id: 0, 
      title: 1, 
      year: 1, 
      languages: 1,
      imdb: 1 } 
  },
  { 
    $addFields: { 
      scaled_votes: { 
        $add: [
          1, 
          { 
            $multiply: [
            9, { 
              $divide: [
                { $subtract: ["$imdb.votes", 5] },
                { $subtract: [1521105, 5] }
              ] 
            }] 
          }]
      } 
    } 
  },
  { 
    $addFields: { 
      normalized_rating: { $avg: ["$scaled_votes", "$imdb.rating"] } 
    } 
  },
  { 
    $sort: { normalized_rating: 1 } 
  },
  { 
    $limit: 1 
  }
])
```

- course pipeline (it uses only `$project` stage instead of `$addFields`. I break the scale and normalization in two `$addFields` stages)

```javascript
db.movies.aggregate([
  {
    $match: {
      year: { $gte: 1990 },
      languages: { $in: ["English"] },
      "imdb.votes": { $gte: 1 },
      "imdb.rating": { $gte: 1 }
    }
  },
  {
    $project: {
      _id: 0,
      title: 1,
      "imdb.rating": 1,
      "imdb.votes": 1,
      normalized_rating: {
        $avg: [
          "$imdb.rating",
          {
            $add: [
              1,
              {
                $multiply: [
                  9,
                  {
                    $divide: [
                      { $subtract: ["$imdb.votes", 5] },
                      { $subtract: [1521105, 5] }
                    ]
                  }
                ]
              }
            ]
          }
        ]
      }
    }
  },
  { $sort: { normalized_rating: 1 } },
  { $limit: 1 }
])
```

## `$group`

```javascript
// grouping by year and getting a count per year using the { $sum: 1 } pattern
db.movies.aggregate([
  {
    "$group": {
      "_id": "$year",
      "numFilmsThisYear": { "$sum": 1 }
    }
  }
])

// grouping on the number of directors a film has, demonstrating that we have to
// validate types to protect some expressions
db.movies.aggregate([
  {
    "$group": {
      "_id": {
        "numDirectors": {
          "$cond": [{ "$isArray": "$directors" }, { "$size": "$directors" }, 0]
        }
      },
      "numFilms": { "$sum": 1 },
      "averageMetacritic": { "$avg": "$metacritic" }
    }
  },
  {
    "$sort": { "_id.numDirectors": -1 }
  }
])

// showing how to group all documents together. By convention, we use null or an
// empty string, ""
db.movies.aggregate([
  {
    "$group": {
      "_id": null,
      "count": { "$sum": 1 }
    }
  }
])

```

## accumulator operators `$min`, `$max`, `$sum`, `$avg`, `$stdDevPop`

```javascript
db.icecream_data.aggregate([
  { "$project": { "_id": 0, "max_high": { "$max": "$trends.avg_high_tmp" } } }
])

db.icecream_data.aggregate([
  { "$project": { "_id": 0, "min_low": { "$min": "$trends.avg_low_tmp" } } }
])

// getting the average and standard deviations of the consumer price index
db.icecream_data.aggregate([
  {
    "$project": {
      "_id": 0,
      "average_cpi": { "$avg": "$trends.icecream_cpi" },
      "cpi_deviation": { "$stdDevPop": "$trends.icecream_cpi" }
    }
  }
])

// using the $sum expression to get total yearly sales
db.icecream_data.aggregate([
  {
    "$project": {
      "_id": 0,
      "yearly_sales (millions)": { "$sum": "$trends.icecream_sales_in_millions" }
    }
  }
])

```

Lab: for all films that won at least 1 Oscar, calculate the standard deviation, highest, lowest, and average imdb.rating

```javascript
db.movies.aggregate([
  { 
    $match: { 
      awards: /Won \d{1,2} Oscars?/
    }
  }, 
  { 
    $group: {
      _id: null,
      highest_rating: { $max: "$imdb.rating"},
      lowest_rating: {$min: "$imdb.rating"},
      average_rating: {$avg: "$imdb.rating"},
      deviation: {$stdDevSamp: "$imdb.rating"}
    }
  }
])

```

## `$unwind`

```javascript
// sample schema
{ "year" : 2012, "imdb" : { "rating" : 7.2, "votes" : 127, "id" : 2244376 }, "genres" : [ "Drama" ] }
{ "year" : 2013, "imdb" : { "rating" : 6.5, "votes" : 2338, "id" : 2992146 }, "genres" : [ "Action", "Adventure", "Drama" ] }
{ "year" : 2014, "imdb" : { "rating" : 6.2, "votes" : 446, "id" : 3038664 }, "genres" : [ "Drama" ] }
{ "year" : 2015, "imdb" : { "rating" : 6.1, "votes" : 209, "id" : 4629032 }, "genres" : [ "Drama", "War" ] }
{ "year" : 2013, "imdb" : { "rating" : 5.8, "votes" : 4355, "id" : 2988272 }, "genres" : [ "Comedy", "Drama", "Romance" ] }
{ "year" : 2015, "imdb" : { "rating" : 7.5, "votes" : 63, "id" : 4620316 }, "genres" : [ "Drama" ] }
{ "year" : 2014, "imdb" : { "rating" : 8.1, "votes" : 337941, "id" : 2084970 }, "genres" : [ "Biography", "Drama", "Thriller" ] }
{ "year" : 2010, "imdb" : { "rating" : 6.9, "votes" : 3347, "id" : 1315350 }, "genres" : [ "Comedy", "Drama" ] }
{ "year" : 2014, "imdb" : { "rating" : 2.1, "votes" : 567, "id" : 2547942 }, "genres" : [ "Action", "Sci-Fi" ] }
{ "year" : 2015, "imdb" : { "rating" : 6.2, "votes" : 442, "id" : 3132632 }, "genres" : [ "Documentary", "Comedy" ] }

// finding the top rated genres per year from 2010 to 2015...
db.movies.aggregate([
  {
    "$match": {
      "imdb.rating": { "$gt": 0 },
      "year": { "$gte": 2010, "$lte": 2015 },
      "runtime": { "$gte": 90 }
    }
  },
  {
    "$unwind": "$genres"
  },
  {
    "$group": {
      "_id": {
        "year": "$year",
        "genre": "$genres"
      },
      "average_rating": { "$avg": "$imdb.rating" }
    }
  },
  {
    "$sort": { "_id.year": -1, "average_rating": -1 }
  },
  {
    "$group": {
      "_id": "$_id.year",
      "genre": { "$first": "$_id.genre" },
      "average_rating": { "$first": "$average_rating" }
    }
  },
  {
    "$sort": { "_id": -1 }
  }
])

// output
{ "_id" : 2015, "genre" : "Biography", "average_rating" : 7.423404255319149 }
{ "_id" : 2014, "genre" : "Documentary", "average_rating" : 7.212587412587413 }
{ "_id" : 2013, "genre" : "Documentary", "average_rating" : 7.158196721311475 }
{ "_id" : 2012, "genre" : "Talk-Show", "average_rating" : 8.2 }
{ "_id" : 2011, "genre" : "Documentary", "average_rating" : 7.262857142857143 }
{ "_id" : 2010, "genre" : "News", "average_rating" : 7.65 }
```

Lab: We'd like to calculate how many movies every cast member has been in and get an average imdb.rating for each cast member

```javascript
db.movies.aggregate([
  { 
    $match: {
      "cast": {$elemMatch: { $exists: true }},
      languages: { $in: ["English"]}
    }
  },
  {
    $unwind: "$cast" 
  },
  {
    $group: {
      _id: "$cast",
      numFilms: {$sum: 1},
      average: { $avg: "$imdb.rating" }
    } 
  },
  {
    $sort: {numFilms: -1}
  } 
])
```
