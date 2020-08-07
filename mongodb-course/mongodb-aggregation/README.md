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

 Lab 1: Find movies that meet the following criteria: imdb.rating is at least 7; genres does not contain "Crime" or "Horror"; rated is either "PG" or "G"; languages contains "English" and "Japanese"

```javascript
var pipeline = [ { $match: { "imdb.rating": { $gte: 7}, genres: { $nin: ["Crime", "Horror"] }, rated: { $in: ["PG", "G"] }, languages: { $all: ["English", "Japanese"] } } } ]
db.movies.aggregate(pipeline).itcount()
```