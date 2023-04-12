====
MongoDBScript.md
====

1: Introduction

- NoSQL solution
- mongos, mongod (server), mongosh (mongo's shell)
- collections uses documents etc...

2: CRUD & the Basics

> sudo mongod --port 27018

# opening a server

> db;

# shows a current db

> show dbs

# shows dbs

> db.someDataName.insertOne({});

# inserting just one record

> db.someDataName.find().pretty();

# getting prettified whole prettified someDataName

CRUD commands:

- Create: insertOne(data, options), insertMany(data, options)

- Read: find(filter, options) - all that satisfy the filter, findOne(filter, options)

- Update: updateOne(filter, data, options), updateMany(filter, data, options), replaceOne(filter, data, options)

- Delete: deleteOne(filter, options), deleteMany(filter, options)

> db.someDataName().deleteMany();

# deleting many records

> db.flightData.updateOne({ distance: 12000 }, { $set: { marker: "delete" } });

# changes property if it exists or adds it if it is not there

> db.flightData.updateMany({}, { $set: { marker: "deleteThisOne" } });

# changes all the records

> db.flightData.find({ intercontinental: true });

# finding data

> db.flightData.find({ distance: { $gt: 10000 } });

# finding data that have distance g-reater t-han 10000 (use findOne to find the first one that satisfy the $gt)

> db.flightData.update({ \_id: ObjectId("6410b3242fcb1e5b81c6088d") }, { delayed: true })

# this will patch the object itself with { delayd: true }

> db.flightData.replaceOne({ delayed: true }, { newObject: "some value" });

# - db.passengers.find() <- will give you a cursor instead of the data (what if you accidentally got 1 billion records?)

> db.passengers.find().toArray();

# gives you an array, but it is not performant if you have many different elements

db.passengers.find().forEach((document) => {
printjson(document);

> });

# you can use forEach with syntax that resembles js

- .pretty() will only work on cursors

Projection (GraphQL like-a-method):

> db.passengers.find({}, { name: 1 });

# get only name parameter and set it to 1 to include it

> db.passengers.find({}, { name: 1, \_id: 0 })

# we can ommit \_id by typing this ^

What can I put into the database?

Embedded Documents:

- embedded levels
- Max of 16mb a document
- theoretically endless embedding is possible but is there an usecase for it?

Arrays:

- can hold any data
- can include documents

> db.passengers.findOne({ name: "Albert Twostone" }).hobbies;

# you can get his hobbies by using dot notation

> db.passengers.find({ hobbies: "sport" });

# find passengers that have sport as a hobby

> db.flightData.find({ "status.description": "on-time" });

# finding status.description that is "on-time"

3: Schemas & Relations

Data Types:

- Text: "Some text"
- Boolean: true or false
- Number: Integer (int32), NumberLong(int64) - however it uses float by default, NumberDecimal(highPrecisionNumber)
- ObjectId: ObjectId("uuid988AIUNf79A&SFNK")
- ISODate: ISODate("2018-09-09")
- Timestamp: Timestamp(8712433322) - this will be always unique
- Embedded Document: { "a": {...} }
- Array: { "a": [...] }

```javascript
insertOne({
  name: "Fresh Apples Inc",
  isStartup: true,
  employees: 33,
  funding: 7123947128349817234987,
  details: {
    ceo: "Mark Super",
    tags: [
      "super",
      "perfect",
      {
        title: "new Date()",
        insertedAt: "new Timestamp()",
      },
    ],
  },
});
```

^ this will result in that:
(funding will get stored will get notated scientifically - it used to cut numbers)

```json
  {
    _id: ObjectId("64120b6f6f5261f39f3caebb"),
    name: 'Fresh Apples Inc',
    isStartup: true,
    employees: 33,
    funding: 7.123947128349818e+21,
    details: {
      ceo: 'Mark Super',
      tags: [
        'super',
        'perfect',
        {
          title: ISODate("2023-03-15T18:16:15.112Z"),
          insertedAt: Timestamp({ t: 0, i: 0 })
        }
      ]
    }
}
```

> db.stats();

# getting stats of database

- inserting data with NumberInt() will make the number be stored as integer instead of default floating point (double)

In MongoDB you cannot use documents that take up more space than 16 mega bytes and you cannot use more than 100 levels of embedded documents
Text however, can be as up to 16 mega bytes

When planning data schemas you should:

- ask yourself which data does your app need or generate? (define the fields you'll need)
- where do I need my Data? (define your required collections + field groupings)
- which kind of data or information do I want do display? (defines queries)
- how often do I fetch my data? (define whether you should optimize for easy fetching)
- how often do I write or change? (deinfe whether you should optimize for easy writing)

Relations:

> var someId = db.databaseName.someProperty;

# getting something into a variable

> db.someDatabase.findOne({ \_id: someId });

# then find it like this

> db.cars.insertOne({ car: 23455, owner: ObjectId("641211f16f5261f39f3caebe") });

# inserting ObjectId directly inside insertOne method

In MongoDB you rather have ids in eg. "orders" so that the data will change everywhere rather than in just one place.

- $replace and $unset properties of update methods are used to replace and remove keys from the data.

Joining with $lookup:

> db.books.aggregate([ { $lookup: { from: "authors", localField: "authors", foreignField: "_id" as: "creators" } } ])

# this will create new key "creators" in a book that has an "authors" array with ObjectIds.

- from: authors collection
- localField: where the references can be found
- foreignField: "\_id": what field will get referenced from "from" key
- as: the new key that will be created

MongoDB will validate schema is the schema itself is defined, otherwise it will accept anything.

validationLevel:
strict => all inserts & updates
moderate => all inserts & updates to correct documents

validationAction
error => just stop the action
warn => proceed on some terms

adding validation can be done on creation of the collection:

```javascript
db.createCollection("posts", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["title, text, creator, comments"],
      properties: {
        title: {
          bsonType: "string",
          description: "must be a string and is required",
        },
        text: {
          bsonType: "string",
          description: "must be a string and is required",
        },
        creator: {
          bsonType: "objectId",
          description: "must be an objectId and is required",
        },
        comments: {
          bsonType: "array",
          description: "must be an array and is required",
          items: {
            bsonType: "object",
            properties: {
              text: {
                bsonType: "string",
                description: "must be a string and is required",
              },
              author: {
                bsonType: "objectId",
                description: "must be an objectId and is required",
              },
            },
          },
        },
      },
    },
  },
});
```

# inserting error might look like this

```json
{
  failingDocumentId: ObjectId("6412268c6f5261f39f3caec1"),
  details: {
      operatorName: '$jsonSchema',
      schemaRulesNotSatisfied: [
        {
          operatorName: 'required',
          specifiedAs: { required: [ 'title, text, creator, comments' ] },
          missingProperties: [ 'title, text, creator, comments' ]
        }
      ]
  }
}
```

# run this to add validator into a collection

> db.runCommand({ collMod: "posts", validator: { ...validation } });

4. The Shell & The Server

# Making two different folders for database data and logs.

mongod --dbpath /path/... --logpath /path/...

# Repairing database

mongod --repair ...

# Changing storageEngie

mongod --storageEngine

# Forking (only works on Mac and Linux) - it runs in the background

mongod --fork --logpath /path/...

# Running a MongoDB server in Windows operating system

net start mongodb

MongoDB Config File:

# Create a mongod.cfg whos structure resembles this:

<!--
storage:
  dbPath: "/your/path/to/the/db/folder"
systemLog:
  destination:  file
  path: "/your/path/to/the/logs.log"
-->

# ... then run your MongoDB server this way:

mongod -f mongod.cfg

# More on mongosh

... --nodb // run with no database connection

# Getting mongosh help

mongosh --help

# while connected run this to get help

db.help()

# help on collection

db.collectionName.help()

# copied

---

Helpful Articles/ Docs:

## More Details about Config Files: https://docs.mongodb.com/manual/reference/configuration-options/

## More Details about the Shell (mongo) Options: https://docs.mongodb.com/manual/reference/program/mongo/

## More Details about the Server (mongod) Options: https://docs.mongodb.com/manual/reference/program/mongod/

5. Create Operations

# Standard for inserting:

insertOne() or insertMany() or insert() (insert() is not recommended)

# dropping database

db.dropDatabase()

# inserting many documents

db.someDatabase.insertMany()

# specyfing additional information (if you overwrite the \_id mongo will not generate its own id)

db.someDatabase.insertMany([{_id: "sports"...}...], )

# iterables are indexed with 0, 1, 2, 3...

# MongoDB uses ordered insert. That means it will insert everything until an error.

# this configuration object will make insert and unordered one.

db.someDatabase.insertMany([...], { ordered: false })

# WriteConcern is an configuration option.

# j = journal, w = write

db.someDatabase.insertMany([...], { j: true or undefined, w: toHowManyInstancesThisWriteShouldBeAcknowledged, wtimeout: 200 })

# if you do not care about the data being pushed into the db you can use that

db.someDatabase.insertOne({ name: "Chris" }, { writeConcern: { w: 1, j: true, wtimeout: 1000} })

# Atomicity:

# transaction either successes as a whole or fails as a whole

# Importing data:

# json file can be imported, run this

mongoimport data.json -d saveToThisDatabase -c toThisCollection --jsonArray --drop

--drop = drop if existent
--jsonArray = data is an json array
-d db = self explanatory
-c collection = self explanatory

6. Reading Documents with Operators

# Finding:

db.collection.find({ age: 30 })
db.collection.find({ age: { $gt: 30 } })

# You get:

# Query Operators -> Locate Data

# Projection Operator -> Modify data presentation

# Update Operator -> Modify and add additional data

# Query Selectors: Comparison, Evaluation, Logical, Array, Element, Comments, Geospatial

# Examples:

db.c.findOne({}) <- first document

db.c.findOne({ property: 1 })

# Comparison operators:

db.c.find({ runtime: { $eq: 60 } })

# $eq = equal

# $ne = not equal

# $lt(e) = lower than (equal)

# $gt(e) = greater than (equal)

# Querying Embedded Documents (query not only the top level fields):

db.c.find({ "rating.average": { $gt: 7.0 } } )

db.c.find({ thisIsTheArray: "findThisElement" })

db.c.find({ thisIsTheArray: ["thisExactArray"] })

# someValue can be equal to 30 or 42

db.c.find({ someValue: { $in: [30, 42] } })

# or not like this.

db.c.find({ someValue: { $nin: [30, 42] } })

# $or = either this or that (an array of conditions)

# $nor = not this or that (inverse of the $or)

# $and = this AND that (an array of filters)

# $not = get items that do not satisfy a condition (takes filtering object)

# $exists = check if a property exists or not, example: thisProp: { $exists: true }

# $ne = not equal

# $type = someProperty: { $type: [ "string", "double" ] }

# $regex = { $regex: /music/ }

# use $volume field and then compare $volume > $target (those are numbers)

# $expr = { $expr: { $gt: ["$volume", "target"] } }

# $cond = condition

> $expr = { $expr: { $gt: [{ $cond: { if: { $gte: ["$volume", 190] }, then: { $subtract: ["$volume", 10] } } }] } }

db.cfind({ $or: [{ "rating.average": { $lt: 5 } }, { "rating.average": { $eq: 3 } }]})

db.c.find({ "rating.average": { $lt: 5 } }).count()

db.c.find({ prop: true, prop2: "someString" })

# Quering arrays:

# db.users.find({ "hobbies.title": "Sports" })

# $size = find object that property of has exactly 3 elements (eg. $size: 3)

# $all = unordered, exactly equal to eg. two different strings (eg. $all: ["itemName1", "itemName2"])

# $elemMatch = { hobbies: { $elemMatch: { title: "Sports", frequency: { $gte: { 3 } } } } }

# Cursors: (what if your database constist of 1000000000000 elements? shell gets you 20 documents at once)

db.someCollection.find().next()

# .next() method

const dataCursor = db.users.find()

# get the next one

> dataCursor.next()

# forEaching through all the documents

> dataCursor.forEach(() => { printjson() })

# self explanatory (wether this is exhausted or not)

> dataCursor.hasNext()

# Sorting cursor results: (-1: descending, 1: ascending) (also I added two different fields)

> db.c.find().sort({ "rating.average": -1, runtime: -1})

# Skipping results (skipping 2 results in this example)

> db.c.find().sort().skip(2)

# Limiting to return certain amount of documents (limit to 5 results in this case)

> db.c.find().limit(5)

# Shaping the data (using projection)

# in this case you can omit the \_id: 0, without that it is always included

> db.c.find({}, { name: 1, genres: 1, runtime: 1, rating: 1, \_id: 0 })

# or

> db.c.find({}, { name: 1, genres: 1, runtime: 1, "rating.subRatingCondition": 1, \_id: 0 })

# now working with arrays (case: return 1 element from the matching find condition)

> db.c.find({ genres: "Drama" }, { "genres.$": 1 })

# get only if "Drama" is there and project them with or without "Horror"

> db.c.find({ genres: "Drama" }, { "genres.$": { $elemMatch: "Horror" } })

# One last special projection operator:

> db.c.find({ "rating.average": { $gt: 9 } }, { genres: { $slice: 2 OR [1 (skip 1), 2 (slice out 2 elements)] } } )

7. Updating documents

# doing a complete overwrite

> db.users.updateOne({ \_id: ObjectId("someid9090") }, { $set: { hobbies: ["Sports"] } })

- $set = does not overwrite data

> db.c.updateMany({ "hobbies.title": "Sports" }, { $set: { isSporty: true } })

# incrementing and decrementing values (case: updating aging) (incrementing age by +2, set -1 to decrement)

# this would create a conflict if you use $inc and then increment the same field

> db.c.upateMany({ name: "Manuel" }, { $inc: { }, $set: { age: 2 } })
> db.c.upateMany({ name: "Manuel" }, { $inc: { }, $set: { isSporty: false } })

# setting if Chris's age to minimum of 36 (changes only if the value is less than that, using $max is an equivalent of <=)

> db.c.updateOne({ name: "Chris" }, { $min: { age: 36 } })

# $mul = multiply the age (case) by 3

> db.c.updateOne({ name: "Chris" }, { $mul: { age: 3 } })

# dropping a field with $unset

> db.c.updateMany({ isSporty: false }, { $unset: { phone: "" } })

# renaming a field (from "age" to "totalAge")

> db.c.updateMany({}, { $rename: { age: "totalAge" } })

# either updating or inserting (using 3rd argument of update function) (update + insert = upsert)

> db.c.updateOne({ name: "Maria" }, { $set: { age: 29 } }, { upsert: true })

# find one and the same document that match the $and

> db.c.updateMany({ hobbies: { $elemMatch: { title: "Sports", frequency: { $gt: 3 } } } })

# ... and updating (refering to the FOUND documents with .$)

> db.c.updateMany({ hobbies: { $elemMatch: { title: "Sports", frequency: { $gt: 3 } } } }, { $set: { "hobbies.$.highFrequency": true } } })

# changing all documents that match the condition

# that changes only the first element of the array

> db.c.updateMany({"hobbies.favorites": { $gt: 2 } }, { $set: { "hobbies.$.goodFrequency" } })

# ... this might help you (for each element update the frequency field)

> db.c.updateMany({ totalAge: { $gt: 30 } }, { $inc: { "hobbies.$[].frequency": -1 } })

# update only if the found documents have some array filter satisfied

> db.c.updateMany({ "hobbies.frequency": { $gt: 2 } }, { $set: { "hobbies.$[el].goodFrequency": true } }, { arrayFilters: [{ "el.frequency": { $gt: 2 } }], $sort: { frequency: -1 } })

# remove the field that has a "Good Wine" as a value

> db.c.updateMany({ name: "Maria" }, { $pull: { title: "Good Wine" } })

> db.c.updateOne({ name: "Chris" }, { $pop: { hobbies: 1 - for the last one -1 for the first one } })

# $push vs $addToSet - addToSet adds only unique values, push does for all of them

8. Deleting documents

> db.c.deleteOne({ name: "Chris" })

# remember that $exists will check for the existence only!

> db.c.deleteMany({ age: { $exists: true } })

# removing every item of a collection

> db.deleteMyItems.deleteMany({})

# deleting everything (rarely used)

> db.dropMeIAmACollection.drop()

9. Indexes (finding data efficiently)

# Collection scan (brute force scan = COLLSCAN) vs Index scan (pointers == IXSCAN)

# Indexes don't come for free -> with many inserts they might become not that efficient

# new version for counting documents:

> db.c.estimatedDocumentCount()

# .explain() method: (works with finding)

> db.c.explain().find({})

# here's the output

```json
{
  "explainVersion": "1",
  "queryPlanner": {
    "namespace": "contactData.contacts",
    "indexFilterSet": false,
    "parsedQuery": {},
    "queryHash": "17830885",
    "planCacheKey": "17830885",
    "maxIndexedOrSolutionsReached": false,
    "maxIndexedAndSolutionsReached": false,
    "maxScansToExplodeReached": false,
    "winningPlan": { "stage": "COLLSCAN", "direction": "forward" },
    "rejectedPlans": []
  },
  "command": { "find": "contacts", "filter": {}, "$db": "contactData" },
  "serverInfo": {
    "host": "MacBook-Pro-ukasz.local",
    "port": 27007,
    "version": "6.0.4",
    "gitVersion": "44ff59461c1353638a71e710f385a566bcd2f547"
  },
  "serverParameters": {
    "internalQueryFacetBufferSizeBytes": 104857600,
    "internalQueryFacetMaxOutputDocSizeBytes": 104857600,
    "internalLookupStageIntermediateDocumentMaxSizeBytes": 104857600,
    "internalDocumentSourceGroupMaxMemoryBytes": 104857600,
    "internalQueryMaxBlockingSortMemoryUsageBytes": 104857600,
    "internalQueryProhibitBlockingMergeOnMongoS": 0,
    "internalQueryMaxAddToSetBytes": 104857600,
    "internalDocumentSourceSetWindowFieldsMaxMemoryBytes": 104857600
  },
  "ok": 1
}
```

# adding an index (either ascending or descending) (look up time will get smallers)

> db.c.createIndex({ "dob.age": -1 })

# remember that index cannot be seen in a query result

# dropping an index: (if you get a big % of a collection using COLLSCAN might be faster, use COLLSCAN if these queries return not more than like 20% - 30%)

> db.c.dropIndex({ "drop.for.this": "field" })

# creating a indexed string value

> db.c.explain("executionStats").find({})

# Compound Index: (a combination of two indexes) (it will use either IXSCAN or COLLSCAN, from left to right)

> db.c.createIndex({ age: 1, gender: "male" })s

# using indexes for sorting (mongodb has 32 megabytes of memory, watch out when sorting -> indexes are already sorted)

> db.c.find({ ... }).sort()

# getIndexes()

> db.c.getIndexes()

# output ^:

```json
[{ "v": 2, "key": { "_id": 1 }, "name": "_id_", "ns": "contactData.contacts" }]
```

# another example (ascending order, unique: true) (this will show you wether you have duplicates or not)

> db.c.createIndex({ email: 1 }, { unique: true })

# note on indexes: they help you avoid duplicates and guarantee that you have a unique values in that collection

# Partial Filter: (mongo ensures that you don't loose any data instead of caring about the performance in some cases)

# mongo will treat no-exist indexes are existing ones

> db.c.createIndex({ "dob.age": 1 }, { partialFilterExpression: { "dob.age" } })

# avoid the clash with unique

> db.c.createIndex({ "dob.age": 1 }, { partialFilterExpression: { email: { $exists: true } } })

# TTL (Time To Leave - Index)

> db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 10000 })

# Query Diagnosis & Query Planning (.explain(): queryPlanner, executionStats, allPlanExecution)

# Efficient Queries & Covered Queries

# ms Process Time

# IXSCAN typically beats COLLSCAN (# of keys examined, $ of documents examined that should be as close as possible to # of documents returned)

# Covered Query - where all the fields that are being accessed by the query are also included in the index.

# ----

# When mongo rejects a plan?

# Winning Plans?

# Approach 1: Who's the first to find 100 documents? (MongoDB caches queries, but will validate the cache once there were many inserts)

- Write Treshold: 1000

- Index is Rebuilt

- Other Indexes are Added or Removed

- MongoDB Server is Restarted

# Multi Key Index should be rather added to arrays. (for queries that target multi value arrays) (only possible with just one array)

# Text Indexes: ($regex is rather slow) (this will have to have a value of "text") (they are expensive)

> db.c.createIndex({ description: "text" })

# then find by this index: (they work kind of like regexes)

> db.c.find({ $text: { $search: "awesome" } })
> db.c.find({ $text: { $search: "\"look for this sentence in a text\"" } })

# We can find out how .find() scores the results (and sort by them)

> db.c.find({ $text: { $search: "thisAwesomeText" }, { score: { $meta: "textScore" } } }).sort({ score: { $meta: "textScore" } })

# dropping by index name

> db.c.dropIndex("dropThisIndexWhichHasThisName")

# exluce something from $text index search

> db.c.find({ $text: { $search: "thisAwesomeText -doNotLookForThis" } })

# setting a default language (so some words will end up in the index) (case: title is worth 10 times less than description)

> db.c.createIndex({ title: "text" }, { default_language: "english", weights: { title: 1, description: 10 } })

# also try using $caseSensitive and $language

# Building Indexes

# Foreground mode: Collection is locked during index creation, faster

# Background mode: Collection is accessible during index creation, slower

# running .js script in mongo

> mongosh some_script.js

# and then... (you can use two background queries)

> db.c.createIndex({ age: 1 }, { background: true })

10. Geospatial Data

# GeoJSON (is available outside mongodb)

# start from picking any place from Google Maps

# eg. 52.1435167 (latitude), 21.0426796 (longitude)

# inserting using GeoJSON

> db.c.insertOne({ name: "Some church", location: { type: "Point", coordinates: [-122.23476234, 34.82933] } })

# quering locations

> db.c.find({ location: { $near: { $geometry: { type: "Point", coordinates: [-123.234, -23.9827389472] } } } })

# adding geospatial index

> db.c.createIndex({ location: "2dspheres" })

# quering with min and max distance

> > db.c.find({ location: { $near: { $geometry: { type: "Point", coordinates: [-123.234, -23.9827389472] }, $maxDistance: 30, $minDistance: 10 } } })

# which points are inside of that? (an array of polygon arrays)

> db.c.find({ location: { $geoWithin: { $geometry: { type: "Polygon", coordinates: [[p1, p2, p3, p4]] } } } })

# checking wether the user in is in the area

> db.c.insertOne({ name: "Some area", area: { type: "Polygon", coordinates: [[p1, p2, p3, p4]] } })
> db.areas.find(area: { $geoIntersects: { $geometry: { type: "Point", coordinates: [-122.823, 38.28393] } } })

# finding places within a certain radius around the point (can be user)

> db.places.find({ location: { $geoWithin: { $centerSphere: [[-123.234, 83.822], 1 / 6378.1 (radians)] } } })

11. Aggregation Framework (running on server)

# pipeline of steps (a framework for find method)

> mongoimport persons.json -d analytics -c persons -jsonArray
> use analytics

# if you see Type "it" for more <- it is a cursor

```json
> db.c.aggregate([
  {
    $match: {
      gender: "female"
    }
  },
  {
    $group: {
      _id: {
        state: "$location.state"
      }
    },
    // group does accumulate data
    totalPersons: {
      $sum: 1
    }
  },
  {
    $sort: {
      totalPersons: -1
    }
  }
])
```

# projecting

```json
> db.c.aggregate({
  {
    $project: {
      _id: 0,
      gender: 1,
      fullName: {
        $concat: ['$name.first', ' ', '$name.last']
      }
    }
  }
})
```

```json
> db.c.aggregate({
  {
    $project: {
      _id: 0,
      gender: 1,
      fullName: {
        $concat: [
          { $toUpper: '$name.first' }, ' ', { $toUpper: '$name.first' }
        ]
      }
    }
  }
})
```

```json
> db.c.aggregate({
  {
    $project: {
      _id: 0,
      gender: 1,
      fullName: {
        $concat: [
          { $toUpper: { $substrCP: ['$name.first', 0, 1] } } },
          { $substrCP: ['$name.first', 1, { $subtract: [ { $strLenCP: "$name.first" } ] }] },
          ' ',
          { $toUpper: { $substrCP: ['$name.last', 0, 1] } } },
          { $substrCP: ['$name.last', 1, { $subtract: [ { $strLenCP: "$name.last" } ] }] },
        ]
      }
    }
  }
})
```

```json
> db.c.aggregate([
  {
    $project: {
      _id: 0,
      name: 1,
      email: 1,
      location: {
        type: "Point",
        coordinates: [
          { $convert: { input: "$location.coordinates.longitude", to: "double", onError: 0.0, onNull: 0.0 } }
          "$location.coordinates.longitude",
          "$location.coordinates.latitude"
        ]
      }
    }
  },
  {
    $project: {
      _id: 0,
      gender: 1,
      location: 1,
      fullName: {
        $concat: [
          { $toUpper: { $substrCP: ['$name.first', 0, 1] } } },
          { $substrCP: ['$name.first', 1, { $subtract: [ { $strLenCP: "$name.first" } ] }] },
          ' ',
          { $toUpper: { $substrCP: ['$name.last', 0, 1] } } },
          { $substrCP: ['$name.last', 1, { $subtract: [ { $strLenCP: "$name.last" } ] }] },
        ]
      }
    }
  }
])
```

# transforming the Birthdate (remember about taking new fields to another stage)

```json
> db.c.aggregate([
  {
    $project: {
      birthdate: {
        $convert: {
          input: "$dob.date",
          to: "date"
        }
      },
      age: "$dob.age"
    }
  }
])
```

# onError, onNull safety

```json
> db.c.aggregate([
  {
    $project: {
      birthdate: {
        $toDate: "$dob.date"
      },
      age: "$dob.age"
    }
  },
  {
    $group: {
      _id: {
        birthYear: {
          $isoWeekYear: "$birthDay"
        }
      }
    }
  },
  {
    $sort: {
      numPersons: -1
    }
  }
])
```

# $group (1:n) (sum, count, average, build array) vs $project (1:1) (include/exclude fields, transform fields (within a single document))

# using $unwind to flatten arrays

```json
> db.persons.aggregate([
  {
    $unwind: "$flattenThisArray"
  },
  {
    $group: {
      _id: {
        age: "$age"
      },
      allHobbies: {
        $push | $addToSet (you get only unique values to this array): "$hobbies"
      }
    }
  }
])
```

# more projections ($slice for array slicing)

```json
db.c.aggregate({
  {
    $project: {
      _id: 0,
      examScore: {
        $slice: ["$examScore", 1]
      }
    }
  }
})
```

# finding how many exams friend took

```json
db.friends.aggregate({
  {
    $project: {
      _id: 0,
      thisValueNameWillGetStored: {
        $size: "$examScores"
      }
    }
  }
})
```

# seeing records only higher that 60

```json
db.friends.aggregate({
  {
    $project: {
      _id: 0,
      scores: {
        $filter: {
          input: "$examScores",
          as: "scoreNewNames",
          cond ("condition"): {
            $gt ("array of values it should compare"): ["$$scoreNewName.score", "60"]
            // one dollar sign would mean a "field"
          }
        }
      }
    }
  }
})
```

```json
db.friends.aggregate({
  { $unwind: "examScores" },
  { $project: { _id: 0, name: 1, age: 1, score: "$examScores.scores" } },
  { $sort: { score: -1 } },
  { $group: { _id: "$_id", name: { $first: "$name" } , maxScore: { $max: "$score" } } },
  { $sort: { maxScore: -1 } }
})
```

# understanding $bucket stage

# getting buckets for ages that are listed in the array and display this peoples' names

```json
db.persons.aggregate([
  {
    $bucket: {
      groupBy: "$dob.age",
      boundaries: [0, 18, 30, 50, 80, 120],
        output: {
        names: {
          averageAge: { $avg: "$dob.age" },
          $push: "$name.first"
        }
      }
    }
  }
])
```

# additional stages

# finding 10 people with oldest birth day

```json
db.persons.aggregate([
  {
    $project: {
      _id: 0,
      name: {
        $concat: [
          "$name.first", " ", "$name.last"
        ]
      },
      birthdate: {
        $toDate: "$dob.date"
      }
    }
  },
  {
    $sort: {
      birthdate: 1
    }
  },
  {
    $limit: 10
    // straight-forward stage - just a limiter
  },
  {
    $skip: 10
    // please understand that order matters in aggregation framework in mongodb
  }
])
```

# MongoDB actually tries its best to optimize your Aggregation Pipelines without interfering with your logic.

# Learn more about the default optimizations MongoDB performs in this article: https://docs.mongodb.com/manual/core/aggregation-pipeline-optimization/

# ----

# writing result of a pipeline into a collection

```json
db.c.aggregate([
  // ...
  {
    $out: "theNewCollectionNameThatContainsTheResults"
  }
])
```

# working with $geoNear

```json
db.c.aggregate([
  {
    $geoNear: {
      // this stage has a direct access to a collection
      // other stages do not have it
      near: {
        type: "Point",
        coordinates: [-18.2, -42.2]
      },
      maxDistance: 1000,
      num: 10,
      query: {
        age: {
          $gt: 18
        }
      },
      // where should be the the distance stored
      distanceField: "newDistanceFieldName"
    }
  }
])
```

### Summary:

# Stages & Operators

# There are plenty of available stages and operators you can choose from

# Stages define the different steps your data is funneled through

# Each stage receives the output of the last stage as input

# Operators can be used inside of stages to transform, limit or re-calculate data

### Important Stages:

# #match, $group, $project, $sort and $unwind

# Whilst there are some common behaviors between find() filters + projection and $match + $project, the aggregation -

# stages generally are more flexible

12. Working with Numeric Data

# mongo will save numbers as 64bit doubles by default, it depends on the client using the database

- int32 (standard for more usages)

- int64 (usage for laaaarge integers)

- 64bit double (floats)

- high precision doubles 128bit (34 digits) (used most likely in scientific calculations)

# there is an imprecision behind the scenes (29.00000000000002 - something like this) consider using this: (to make size of document smaller)

# save some space

> db.c.insertOne({ age: NumberInt("29") })

> db.c.deleteMany({})

# Trying to store values that exceeds 32 bit integer range

# it will get stored as a totally different number

# if you exceed it like this 213487845234 and add just one it will go negative (case without using NumberInt())

# using NumberLong()

> db.c.insertOne({ valuation: NumberLong(9812739487) })

# storing biggest possible value (using no quotation marks it will get limited with javascript)

> db.c.insertOne({ valuation: NumberLong("1827634781268364") })

# adding strings to numeric values will not work

# you cannot increment Longs with "standard" integer without any constructor

# if you want to make it precise you should use high precision doubles

> db.c.aggregate([ { $project: { result: { $subtract: [ "$a", "$b" ] } } } ])

> db.c.insertOne({ a: NumberDecimal("0.3"), b: NumberDecimal("0.1") })

# Summary

# Useful Articles/ Docs:

# Float vs Double vs Decimal - A Discussion on Precision: https://stackoverflow.com/questions/618535/difference-between-decimal-float-and-double-in-net

# Number Ranges: https://social.msdn.microsoft.com/Forums/vstudio/en-US/d2f723c7-f00a-4600-945a-72da23cbc53d/can-anyone-explain-clearly-about-float-vs-decimal-vs-double-?forum=csharpgeneral

# Modelling Number/ Monetary Data in MongoDB: https://docs.mongodb.com/manual/tutorial/model-monetary-data/

13. Security & User Authentication

- Authenatication & Authorization
- Transport Encryption
- Encryption at Rest

- Auditing
- Server & Network Config and Setup
- Backups & Software Updates

# role based access control

# authenaticated users can access the db but only authorized ones can write to it.

# mongo is role based access control

# privilege: resource & action

# why roles? there are different types of users (eg. admin or standard user)

# creating & editing users

> sudo mongod --auth (authentication is obligatory)

> db.auth('max', 'somepassword')

> mongo -u "someUsername" -p "somePassword"

# localhost exception (you are allowed to add one user)

> db.createUser({ user: "Lucas", pwd: "somepassword123", roles: ["userAdminAnyDatabse"] })

> db.auth("Lucas", "somepassword123")

# Built-In Roles

- Database User: read & readWrite
- Database Admin: dbAdmin, userAdmin, dbOwner
- All Database Roles: readAnyDatabase, readyWriteAnyDatabase, userAdminAnyDatabase, dbAdminAnyDatabase (cross database access)
- Cluster Admin: clusterManager, clusterMonitor, hostManager, clusterAdmin,
- Backup/Restore: backup, restore
- Superuser: dbOwner (admin), userAdmin (admin), userAdminAnyDatabase (root)

# create another user

> mongo -u Lucas -p somePassword --authenticationDatabase adminDatabaseName (specifying database to which to authenticate)

> use differentDB

> db.createUser({ user: 'appdev', pwd: 'dev', roles: ["readWrite"] })

# to logout

> db.logout()

# roles are scoped to the database you've used while creating the user

# updating user (roles will get replaced)

> db.updateUser("someUsername", { roles: [ "readWrite", { roles: "readWrite", db: "blog" } ] })

# Transport Encryption: (mongodb ssl)

# https://www.mongodb.com/docs/manual/tutorial/configure-ssl/

# windows openssl https://wiki.openssl.org/index.php/Binaries (allows to run the exact same linux command on windows)

# you will be asked couple of questions

# important is: Common Name: "localhost" on localhost (address of a web server)

> cat mongodb-cert.key mongodb-cert.crt > mongodb.pem

# in that ^ folder start "mongo" and user --sslMode

> mongo --sslMode requireSSL --sslPEMKeyFile mongodb.pem

> db.shutdownServer()

> mongo --ssl --sslCAFile mongodb.pem

# Encrytion at REST

# means that Storage is also encrypted, this is built in enterprise (also remember about hashing data and password)

# Wrap up:

# Users & Roles (role based access approach in mongodb)

# Helpful Articles/ Docs:

# Official "Encryption at Rest" Docs: https://docs.mongodb.com/manual/core/security-encryption-at-rest/

# Official Security Checklist: https://docs.mongodb.com/manual/administration/security-checklist/

# What is SSL/ TLS? => https://www.acunetix.com/blog/articles/tls-security-what-is-tls-ssl-part-1/

# Official MongoDB SSL Setup Docs: https://docs.mongodb.com/manual/tutorial/configure-ssl/

# Official MongoDB Users & Auth Docs: https://docs.mongodb.com/manual/core/authentication/

# Official Built-in Roles Docs: https://docs.mongodb.com/manual/core/security-built-in-roles/

# Official Custom Roles Docs: https://docs.mongodb.com/manual/core/security-user-defined-roles/

14. Performance, Fault Tolerance & Deployment

- What influences Performance?
- Capped Collections
- Replica Sets
- Sharding
- MongoDB Server Deployment

# influence on performance

- efficient queries / operations
- indexes
- fitting data schema

# capped collections (store where data is deleted when new one is inserted? - high throughput (not a usecase for posts, users... etc))

> db.createCollection("capped", { capped: true, size: 100 (in bytes), max: 3 (measured in documents)})

> db.c.find().sort({ $natural: -1 }) (this has to do something with sorting)

# Replica Sets:

# Client -> MongoDB Server -> Primary Node (and then it talks to Secondary Nodes, if primary goes offline we can talk to different one)

# Replica Set(Secondary Node, Secondary Node, Secondary Node,...)

# helps with fault tolerancy

# Sharding: (Horizontal Scaling)

# These servers work together and data is distributed acorss these servers

# queries are run across all shards

[mongod (set of nodes - replica set)] - [mongod (set of nodes - replica set)] - [mongod (set of nodes - replica set)] - [mongod (set of nodes - replica set)]

# Client -> mongos (Router) -> (mongod, mongod, mongod [Shard Key - field added to documents for router to understand the "stuff"])

# Queries & Sharding

# find() -> mongos -> broadcast across Shards (maybe it found the right shard key, so it reads from Shard 2)

# Deploying a MongoDB Server (use MongoDB Atlas)

# You can set Shard, replica sets, RAM, disc space etc... without you caring about best practices

# Go for backups, alerts

# Useful Articles/ Docs:

# Official Docs on Replica Sets: https://docs.mongodb.com/manual/replication/

# Official Docs on Sharding: https://docs.mongodb.com/manual/sharding/

15. Transactions (it works kind of like a git pull requests)

We want to remove posts of some user but without any failure when performing some action.
Those things either fail or succeed together.
If the session is broken, no query is run.

# usage of transactions

> db.users.deleteOne({id: 1234})

# w need a session

> const session = db.getMongo().startSession()
> session.startTransaction()

# ^ making a "remembered" session of commands

> const usersCollection = session.getDatabase("blog").users
> const postsCollection = session.getDatabase("blog").posts

> usersCollection.deleteOne({ \_id: 9817023490 })

# at the end you must

> session.commitTransaction()

# so that it is saved into the "original" database

# Useful Articles/ Docs:

# Official Docs on Transactions: https://docs.mongodb.com/manual/core/transactions/

16. From Mongo Shell to Drivers (Shell commands vs Driver commands)

Shell:

- Configure Database
- Create Collections
- Create Indexes

Driver:

- CRUD Operations
- Aggregation Pipelines

# storing as 128 Decimal (using Decimal128 constructor)

> const Decimal128 = mongo.Decimal128

> Decimal128.fromString(request.body.price)

# remember about closing the client

> client.close()

# using a cursor object in a nodejs driver

> db().collection().find().forEach(element => { console.log(element) })

# adding indexes should rather happen in mongo's shell

# making sure that emails are unique in a database

> db.users.createIndex({ email: 1 }, { unique: true })

# Wrap up:

# Helpful Articles/ Docs:

# Learn how to build a full RESTful API with Node.js: https://academind.com/learn/node-js/building-a-restful-api-with/

17. MongoDB Stitch (removing REST API)

# MongoDB Stitch is now labeled "MongoDB Realm".

# The web console also looks a bit different but it works as shown and the code we write also will work as shown - make sure to use the same package as we're installing in the coming lecture (mongodb-stitch-browser-sdk).

# One thing that is a bit hidden now => The "Clients" tab which you'll see in the next lectures can now be found under "Clusters" => "Working with your Cluster".

# What is Stitch?

# Servless Platform for Building Applications

# Build around Atlas (Cloud Database)

# Authentication

# User will get temporary credentials

# React to Events

# Execute Code / Functionality in the Cloud

# Call from client or something else

# Checkout MongoDB Mobile (storing data offline)

# Stitch Triggers

# Stitch Functions

# Stitch Services

# Dive in:

- Stitch Apps -> Create application
- it can manage authentication for you?

> npm install mongodb-stitch-browser-sdk (most likely named 'realm' now)

> .initializeDefaultAppClient('key98172347')

> const mongodb = Stitch.getAppClient().getServiceClient(RemoteMongoClient.factory, 'mongodb-atlas')

# and then:

> mongo.db('shop').collection('products')

# You have to possess users in your database if you would like to use Stitch (Realm)

# Adding authentication

# import AnonymousAuthentication

# set Rules for collection and then set their permissions

# remember about parsing mongo specific objects like Decimal128

> npm install --save bson

> BSON.ObjectId(productId)

> find().asArray()

# 403 = unauthorized

# authentication

# build Email Confirmation URL. Something like "https://service123.com/confirm-new-password"

> import { UserPasswordAuthProviderClient.factory }

> const credential = new UserPasswordCredential({ authData.email, authData.password })

# Functions & Triggers

# whenever the app loads

> this.client.callFunction('Greet', ['Lucas'])

# just use the javascript editor inside mongo Realm

# consider using "Logs"

# Wrap up:

# Helpful Articles/ Docs:

# Official Stitch Docs: https://docs.mongodb.com/stitch/

# Complete Stitch Username + Password Auth Flow: https://docs.mongodb.com/stitch/authentication/userpass/

# Stitch Services (e.g. AWS S3): https://docs.mongodb.com/stitch/reference/partner-services/amazon-service/
