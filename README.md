# UdemyMongoDBGuide

connection string to connect using mongo shell -> `"mongodb://localhost:27017"`

connect to remote host -> `"mongodb://mongodb0.example.com:28015"` or `mongosh --host mongodb0.example.com --port 28015`

using authentication ->` mongosh "mongodb://mongodb0.example.com:28015" --username alice --authenticationDatabase admin`

type `exit()` to disconnect

Commands
-------------------
basic commands -> `db.help()` `db.version()` `db.stats()` `show dbs` `use <dbname>` `show collections`

create collection -> `create collection - db.createCollection("<dbname>")`

Inserting commands
--------------------
insertOne -> `db.<dbname>.insertOne("jsonstringhere")` insertMany -> `db.<dbname>.insertMany("[jsonstringhere, josnstringhere]")`

Find documents
-----------------
- `db.dbname.find().pretty()` - `db.dbname.find().toArray()`

- `findone - db.getCollections('posts').findOne({postId:3015})` - `find - db.getCollections('posts').find({postId:3015})`

- `find - db.getCollections('posts').find({"author.name":"emily watson"})` - nested object query use . symbol

- `db.passengers.find({},{name:1}).pretty()` - gives us all documents with only name key and value for all top 20 documents as per cursor rule

Query operators
--------------------
$or - `{$or: [{shared : "true"},{tags : "programming"}]}`

$eq - 

$lt less than - `{comments : {$lt:5}}`

$and - `{$and: [{comments : {$gt:0}},{comments : {$lt:5}}]}`

$ne

$gt : greater than - `{comments : {$gt:0}}`

$in - `{tags: {$in : ["programming", "coding"]}}`

$nin

$regex

and many other operators

Helper methods
---------------------
sort() - `.sort({comments:1})`  limit() - `.limit(2) - limit to 2 results`  skip() - `.skip(2) - skip first 2 results`

update documents
-----------------
.update - takes the object and replace the existing object completly based on the filter  .updateOne()  .updateMany() <query>,<update>,<options>

update operators
----------------
$set - `db.posts.updateOne({postId:2618},{$set: {shared: true}})`

$set - `db.posts.updateMany({},{$set: {shared: true}})` - to set this for all documents

$rename

$unset - `db.posts.updateOne({tags[]}, {$unset:{tags:1}})`

$currentDate

$inc - increment - `db.posts.updateOne({postId:8541},{$inc:{comments:1}}`

$addToSet

ReplaceOne()
------------
`db.flightData.replaceOne({jsondocument})` //replace the document based on the matching id

Delete documents
--------------------
`deleteOne({departureAirport: "TXL"})` `deleteOne({_id: "testid"})` `deleteMany({}}` - to delete all `deleteMany({filtercondition}}`

`db.dropDatabase()` to delete database `db.myCollection.Drop()` to delete a single collection

Aggregation
-------------
`[
  {
    $group: {
      _id: "$author.name",
    },
  },
]`

this will parse the existing collection documents and based on this condition creates new documents

Mongo Utilities
-----------------
`mongoexport` `mongoimport` `mongodump` `mongorestore`

Export to txt file
-----------------------
mongoexport -d forum -c posts -o posts.txt

Foreach
------------------------------
using foreach for iterating through all documents of a collection

`db.passengers.find().forEach((passengerData) => printjson(passengerData)})`

Data Types & Limits
----------------------
MongoDB has a couple of hard limits - most importantly, a single document in a collection (including all embedded documents it might have) must be <= 16mb. Additionally, you may only have 100 levels of embedded documents.

Important data type limits are:

Normal integers (int32) can hold a maximum value of +-2,147,483,647

Long integers (int64) can hold a maximum value of +-9,223,372,036,854,775,807

Text can be as long as you want - the limit is the 16mb restriction for the overall document

It is also important to understand the difference between int32 (NumberInt), int64 (NumberLong) and a normal number as you can enter it in the shell. The same goes for a normal double and NumberDecimal.

NumberInt creates a int32 value => `NumberInt(55)`

NumberLong creates a int64 value => `NumberLong(7489729384792)`

If you just use a number (e.g. insertOne({a: 1}), this will get added as a normal double into the database. The reason for this is that the shell is based on JS which only knows float/ double values and doesn't differ between integers and floats.

NumberDecimal creates a high-precision double value => NumberDecimal("12.99") => This can be helpful for cases where you need (many) exact decimal places for calculations.

When not working with the shell but a MongoDB driver for your app programming language (e.g. PHP, .NET, Node.js, ...), you can use the driver to create these specific numbers.
