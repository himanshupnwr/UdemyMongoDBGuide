# UdemyMongoDBGuide

connection string to connect using mongo shell - "mongodb://localhost:27017"
connect to remote host - "mongodb://mongodb0.example.com:28015" or mongosh --host mongodb0.example.com --port 28015
using authentication - mongosh "mongodb://mongodb0.example.com:28015" --username alice --authenticationDatabase admin

type exit() to disconnect

commands
----------
db.help()
db.version()
db.stats()
show dbs
use <dbname>
show collections

create collection - db.createCollection("<dbname>")

inserting commands
- db.<dbname>.insertOne("jsonstringhere")
- db.<dbname>.insertMany("[jsonstringhere, josnstringhere]")

find documents
- db.dbname.find().pretty()
- db.dbname.find().toArray()
- findone - db.getCollections('posts').findOne({postId:3015})
- find - db.getCollections('posts').find({postId:3015})
- find - db.getCollections('posts').find({"author.name":"emily watson"}) -nested object query use . symbol
- db.passengers.find({},{name:1}).pretty() - gives us all documents with only name key and value for all top 20 documents as per cursor rule

Query operators
--------------------
$or - {$or: [{shared : "true"},{tags : "programming"}]}
$eq - 
$lt less than - {comments : {$lt:5}}
$and - {$and: [{comments : {$gt:0}},{comments : {$lt:5}}]}
$ne
$gt : greater than - {comments : {$gt:0}}
$in - {tags: {$in : ["programming", "coding"]}}
$nin
$regex

and many other operators

Helper methods
---------------------
sort() - .sort({comments:1})
limit() - .limit(2) - limit to 2 results
skip() - .skip(2) - skip first 2 results

update documents
-----------------
.update - takes the object and replace the existing object completly based on the filter
.updateOne()
.updateMany()
<query>,<update>,<options>

update operators
----------------
$set - db.posts.updateOne({postId:2618},{$set: {shared: true}})
$set - db.posts.updateMany({},{$set: {shared: true}}) - to set this for all documents
$rename
$unset - db.posts.updateOne({tags[]}, {$unset:{tags:1}})
$currentDate
$inc - increment - db.posts.updateOne({postId:8541},{$inc:{comments:1}}
$addToSet

ReplaceOne()
------------
db.flightData.replaceOne({jsondocument}) //replace the document based on the matching id

delete documents
--------------------
deleteOne({departureAirport: "TXL"})
deleteOne({_id: "testid"})
deleteMany({}} - to delete all
deleteMany({filtercondition}}

db.dropDatabase() to delete database
db.myCollection.Drop() to delete a single collection

Aggregation
-------------
[
  {
    $group: {
      _id: "$author.name",
    },
  },
]

this will parse the existing collection documents and based on this condition creates new documents

mongo Utilities
-----------------
mongoexport
mongoimport
mongodump
mongorestore

export to txt file
-----------------------
mongoexport -d forum -c posts -o posts.txt

The complete mongo db course
------------------------------
using foreach for iterating through all documents of a collection
db.passengers.find().forEach((passengerData) => printjson(passengerData)})

