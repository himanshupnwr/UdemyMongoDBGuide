# UdemyMongoDBGuide

connection string to connect using mongo shell 

`"mongodb://localhost:27017"`

connect to remote host

`"mongodb://mongodb0.example.com:28015"` or `mongosh --host mongodb0.example.com --port 28015`

using authentication

` mongosh "mongodb://mongodb0.example.com:28015" --username alice --authenticationDatabase admin`

type `exit()` to disconnect

Commands
-------------------
basic commands

`db.help()` `db.version()` `db.stats()` `show dbs` `use <dbname>` `show collections`

create collection

`create collection - db.createCollection("<dbname>")`

Inserting commands
--------------------
insertOne

`db.<dbname>.insertOne("jsonstringhere")` insertMany -> `db.<dbname>.insertMany("[jsonstringhere, josnstringhere]")`

Find documents
-----------------
- `db.dbname.find().pretty()` - `db.dbname.find().toArray()`

- `findone - db.getCollections('posts').findOne({postId:3015})` 

- `find - db.getCollections('posts').find({postId:3015})`

- `find - db.getCollections('posts').find({"author.name":"emily watson"})` - nested object query use . symbol

- `db.passengers.find({},{name:1}).pretty()` - gives us all documents with only name key and value for all top 20 documents as per cursor rule

.find gives us a cursor which is by default set to 20 documents

- `find().hasNext()` -> to check if cursor has any next element to show

- `.find().next` -> gives me the next document in line

type `it` for more

`dataCursor.forEach(doc => {printjson(doc)}` -> uses javascript foreach to display all documents in the datacursor document

Query operators
--------------------
- $or - `{$or: [{shared : "true"},{tags : "programming"}]}`

- $eq - 

- $lt less than - `{comments : {$lt:5}}`

- $and - `{$and: [{comments : {$gt:0}},{comments : {$lt:5}}]}`

- $ne

- $gt : greater than - `{comments : {$gt:0}}`

- $in - `{tags: {$in : ["programming", "coding"]}}`

- $nin

- $regex

and many other operators

Helper methods
---------------------
sort()

- `.sort({comments:1})`  limit() - `.limit(2) - limit to 2 results`  skip() - `.skip(2) - skip first 2 results`

update documents
-----------------
.update - takes the object and replace the existing object completly based on the filter  .updateOne()  .updateMany() <query>,<update>,<options>

update operators
----------------
$set 

- `db.posts.updateOne({postId:2618},{$set: {shared: true}})`

$set

- `db.posts.updateMany({},{$set: {shared: true}})` - to set this for all documents

$rename

$unset

- `db.posts.updateOne({tags[]}, {$unset:{tags:1}})`

$currentDate

$inc - increment

- `db.posts.updateOne({postId:8541},{$inc:{comments:1}}`

$addToSet

ReplaceOne()
------------

- `db.flightData.replaceOne({jsondocument})` //replace the document based on the matching id

Delete documents
--------------------
- `deleteOne({departureAirport: "TXL"})` 

- `deleteOne({_id: "testid"})` `deleteMany({}}` - to delete all `deleteMany({filtercondition}}`

- `db.dropDatabase()` to delete database `db.myCollection.Drop()` to delete a single collection

Aggregation
-------------
```javascript
[
  {
    $group: {
      _id: "$author.name",
    },
  },
]
```

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

NumberInt creates a int32 value

- `NumberInt(55)`

NumberLong creates a int64 value

- `NumberLong(7489729384792)`

If you just use a number (e.g. insertOne({a: 1}), this will get added as a normal double into the database. The reason for this is that the shell is based on JS which only knows float/ double values and doesn't differ between integers and floats.

NumberDecimal creates a high-precision double `value => NumberDecimal("12.99")` => This can be helpful for cases where you need (many) exact decimal places for calculations.

When not working with the shell but a MongoDB driver for your app programming language (e.g. PHP, .NET, Node.js, ...), you can use the driver to create these specific numbers.

Relations in MongoDB
--------------------
- One to One Embedded
- One to Many Using References
- One to Many Embedded
- One to Many References
- Many to Many Embedded
- Many to Many References

Merging Reference Relations using $lookup

`db.books.aggregate([{$lookup:{from:"authors", localField:"authorsField", foreignField:"_id", as:"creators"}}])`

this is slow as compared to embedded documents so better use embedded relations as compared to this.

Schema Validation
-------------------
we can vaidate the incoming data based on the schema of the collection. If the schema is valid then it is accepted otherwise it is rejected.

Validation Level

strict -> all insert and updates

moderate -> All insert and updates to correct documents

we can throw and error and block the data update or log a warning and proceed with data update.

We can add the validation when we create a collection

```javascript
db.createCollection('posts', {
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['title', 'text', 'creator', 'comments'],
      properties: {
        title: {
          bsonType: 'string',
          description: 'must be a string and is required'
        },
        text: {
          bsonType: 'string',
          description: 'must be a string and is required'
        },
        creator: {
          bsonType: 'objectId',
          description: 'must be an objectid and is required'
        },
        comments: {
          bsonType: 'array',
          description: 'must be an array and is required',
          items: {
            bsonType: 'object',
            required: ['text', 'author'],
            properties: {
              text: {
                bsonType: 'string',
                description: 'must be a string and is required'
              },
              author: {
                bsonType: 'objectId',
                description: 'must be an objectid and is required'
              }
            }
          }
        }
      }
    }
  }
});
```
callMod - collectionModifier when collection is alreay created and we have to modify it

```javascript
db.runCommand({
  collMod: 'posts',
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['title', 'text', 'creator', 'comments'],
      properties: {
        title: {
          bsonType: 'string',
          description: 'must be a string and is required'
        },
        text: {
          bsonType: 'string',
          description: 'must be a string and is required'
        },
        creator: {
          bsonType: 'objectId',
          description: 'must be an objectid and is required'
        },
        comments: {
          bsonType: 'array',
          description: 'must be an array and is required',
          items: {
            bsonType: 'object',
            required: ['text', 'author'],
            properties: {
              text: {
                bsonType: 'string',
                description: 'must be a string and is required'
              },
              author: {
                bsonType: 'objectId',
                description: 'must be an objectid and is required'
              }
            }
          }
        }
      }
    }
  },
  validationAction: 'warn'
});
```
MongoD
-----------
Copy the path to Environment variables path to run mongod command

C:\Program Files\MongoDB\Server\6.0\bin

After this open the command prompt and type `mongod --help`

To set the path for file directory in which we want to have the mongo databses 

`mongod --dbpath C:/Mongodatabase/development`

To set the path for file directory in which we want to have the mongo logs

`mongod --dbpath C:/Mongodatabase/logs/log`

To shut down Mongo DB Server from command prompt use

`db.shutdownServer()`

`net stop MongoDB`

We can modify the mongodb config file in bin folder

```
storage:
  dbPath: "/your/path/to/the/db/folder"
systemLog:
  destination:  file
  path: "/your/path/to/the/logs.log"
```

to use an exisiting mongodb config file for new mongodb instance on a system use, copy the config file in bin

-> `mongod -f <bin path to config.cfg>`

Create Operations
-------------------
Ordered insert -> if any document fails to insert in the insertmany operation the subsequent documents are also not added

we can configure this by using ordered command configuration as a second arguement

Ordered arguement tell if the insert should be ordered which is default or unordered which changes the default behaviour

`db.hobbies.insertMany([{_id:"yoga", name:"Yoga"}, {_id:"cooking", name:"Cooking"}], {ordered: false})`

WriteConcern
-------------

We have a client and we have our mongodb server, the client would be the shell or your application using a mongodb driver, the mongodb server is what you start with the mongod executable.

Now let's say you want to insert one document in there and it would also be true for insert many and for update and so on, so for all the write operations because it's the write concern option we're talking about.

So we're trying to insert one document, now as you learned in the first module of this course, on the mongodb server, we have a so-called storage engine, this is the engine being responsible for really writing our data onto the disk and also for managing it in memory.

So your write might first end up in memory here and there, it manages the data which it needs to access with high frequency because that is faster than working with the disk. Of course your write is also scheduled to then end up on the disk, that is not the thing, so it will also eventually store your data on the disk.

Now you can configure a so-called write concern on all the write operations like insert one with an additional argument, the write concern argument which is in turn a document where you can set settings

`{w:1, j:undefined}`

now what does this mean? The w simply means write and the number here indicates to how many instances, in case you're using multiple instances on one server,
so on how many instances you want this write to be acknowledged. With W1 which is the default, you're basically saying hey my mongodb server should have accepted that write, so the storage engine is aware of it and will eventually write it to the disk. 

The J stands for the journal, the journal is an additional file which the storage engine manages which is like a To-Do file. The journal can be kept to then for example save operations that the storage engine needs to-do that have not been completed yet, like the write. Now it is aware of the write and that it needs to store that data on disk just by the write being acknowledged and being there in memory, it doesn't need to keep a journal for that.

The idea of that journal file which is a real file on the disk is just that it is aware of this and if the server should go down for some reason or anything else happens, that file is still there and if you restart the server or if it recovers basically, it can look into that file and see what it needs to-do and that is of course a nice back up because the memory might have been wiped by then.

So your write could be lost if it's not written to the journal, if it hasn't been written to the real data files yet, that is the idea of the journal, it's like a back up to-do list.

Now the question is why do we write it in the journal and not directly into the database files? Because writing into the database files simply is more performance heavy, the journal is like a single line which describes the write operation, writing into the database files is of course a more complex task because there you need to find the right position to insert it, if you have indexes, you also need to update these, so it simply takes longer, adding a to-do in a journal is faster.

Still that also takes longer than not using the journal and the default is that the journal is not getting used with j undefined and that does simply mean that the storage engine will eventually handle this write and also write it to the journal but you don't have that information yet, you don't know if it has been stored in the journal yet, if the write succeeded yet, if the write has been done on the disk, you don't know any of that, you just know that the server is aware of your write. So if the server should go down in that exact moment, it might indeed not have done the write because it hasn't been added to the journal yet, it hasn't been saved to the database files yet.

`{w:1, j:true}`
`db.Patient.insertOne({name:"Alia", age:45}, {writeConcern:{w:1, j:true}})`

Now you can set a different option and set J to true, what you're now saying is hey please only report a success for this write to me after it has been both acknowledged and been saved to the journal, so now I have a greater security that this will happen and succeed even if the server should face issues right now.

`{w:1, wtimeout:200, j:true}`

Now there also is a third option not directly related to the journal but it's a W timeout option, now this simply means which timeframe do you give your server to report a success for this write before you cancel it.

So if you have some issues with the network connection or anything like that, you might simply timeout here, obviously setting this too low might timeout even though it would have perfectly succeeded normally and therefore you should know what you do when you set this timeout value because if you set it to a very low number, you might fail a lot even though there is no actual problem, just some small latency.

So this is the write concern and how you can control this, obviously enabling the journal confirmation means that your writes will take longer because you dont just tell the server about them but you also need to wait for the server to store that write operation in the journal but you get higher security that the write also succeeded. Again this is a decision you have to make depending on your application needs, what you need.

-> Mongo DB CRUD Operation are atomic on the document level(inluding embedded documents)

Importing Data
---------------
cd <navigate to json file which has the data>

--drop -> if collection exists then drop the existing one and add the new one

`mongoimport file.json -d movieData -c movies --jsonArray --drop`

cd path to mongoimport.exe, copy json file to same folder as mongoimport.exe

`.\mongoimport.exe persons.json -d Indexes -c persons --jsonArray`

Read Operations
---------------

- $lte - lower than or equal to

- $eq

- $neq

- $in - takes an array and check for values in the object

- `db.movies.find({runtime:{$in:[30,42]}}).pretty()` runtime either 30 or 42

$nin is not inside condition

not equals to -> {$not:{$eq:60}}

$gte -> greater than equal to

.count() - to get the count of the documents that passed the operator conditions

$or -> `{$or:[{"rating.average":{$lt:5}, {"rating.average":{$gt:9.3}}}]}`

$nor is the opposite of $or operator

Element Operators
------------------

$exists tells if the object is inside the document 

- `db.users.find({age:{$exists:true, $gt:30}}).pretty()`

$type -> check for the type given in the condition

- `db.Users.find({phone:{$type:"double"}}).pretty()`

- `db.Users.find({phone:{$type:["double","string"]}}).pretty()` - checking multiple types

$regex -> finding text snippets using regex - but they are not very performant

- `db.movies.find({summary:{$regex:/musical/}}).pretty()` - returns all documents where in the summary this word can be found

$expr -> when we want to compare two objects inside a document and use this comparison to find other similar documents

- `db.sales.find({$expr: {$gt: ["$volume", "$target"]}}).pretty()`

$cond, $subtract, $if, $else operator

- `db.sales.find({$expr: {$gt: [{$cond: {if: {$gte: ["$volume", 190]}, then: {$subtract: ["$volume", 30]}, else:"$volume"}}, "$target"]}}).pretty()`

$size - check the size of elements inside the array - `db.users.find({hobbies:{$size:3}}).pretty()`

- $all - `db.moviestarts.find({genre: {$all: ["action", "thriller"]}}).pretty()`

$elemMatch - if we want the conditions to work in the same element

- `db.users.find({hobbies: {$elemMatch: {title:"sports", frequency: {$gte:3}}}}).pretty()`

Sorting
----------

we use the number 1 and -1 for passing the type of sorting we need. 1 means ascending and -1 meand desending

- `db.movies.find().sort({"rating.average": -1}).pretty()` minus 1 for descending

- `db.movies.find().sort({"rating.average": 1, runtime: 1}).pretty()` - sorting by multiple criteria

- `db.movies.find().sort({"rating.average": 1, runtime: 1}).skip(10).pretty()` - results with skipped 10 documents

limits - `db.movies.find().sort({"rating.average": 1, runtime: 1}).skip(10).limit(10).pretty()`

Projections
-------------

we can use projection to shape the structure of our results. if we want to include inly few elements of out document

- `db.movies.find({}, {name: 1, genres: 1, runtime: 1, rating: 1, _id: 0}).pretty()`

if we want to exculde id field as it is not excluded by default we have to pass 0 for it

projections also works on the embedded data

- `db.movies.find({}. {name:1, genres: 1, runtime: 1, "rating.average": 1, "schedule.time": 1, _id: 0}).pretty()`

In Arrays

- `db.movies.find({genres: "Drama"}, {"genres.$": 1}).pretty()` - give only one item from array genres drama

- `db.movies.find({genres: "Drama"}, {genres: {$elemMatch: {$eq: "Horror"}}}).pretty()`

$slice

- `db.movies.find({"rating.average": {$gt:9}}, {genres:{$slice:2}, name: 1}).pretty()`

slice is related to arrays. only the first 2 elements returned for the array because we used slice 2

- `db.movies.find({"rating.average": {$gt:9}}, {genres:{$slice: [1, 2]}, name: 1}).pretty()`

skip the first item and give next 2 items

Assignment 4
-------------
find all movies with exactly two genres

- `{genre: {$size: 2}}`

find all movies that aired in 2018

- `{"meta.aired":2018}`

find all movies with ratings greater than 8 and lower than 10

- `{ratings: {$elemMatch: {$gt:8, $lt: 10}}}`

Update Operations
------------------

document updating operator - update, updating fields, updating arrays 

update query based on the unique id of the document

`db.users.updateOne({_id: ObjectId("6415be06a17bfffcea22594f")}, {$set: {hobbies: [{title: "Sports", frequency: 5}, {title: "Cooking", frequency: 3}, {title: "Hiking", frequency: 7}]}})`

override the exisiting fields if there is a match and there is any change in data, if data does not exist then add it.

- `db.users.updateMany({"hobbies.title": "Sports"}, {$set: {isSporty: true}})`

Updating multiple fields with $set

- `db.users.updateOne({_id: ObjectId("6415be06a17bfffcea225952")}, {$set: {age:40, phone: 1234567890}})`

Incrementing and decrementing values

- `db.users.updateOne({name: "Manuel"}, {$inc: {age: 2}})`

Set along with update

- `db.users.updateOne({name: "Manuel"}, {$inc: {age: 2}, $set: {isSporty: false}})`

Use $min, $max, $mul

$min only changes the value if the existing value is lower than the existing value

- `db.users.updateOne({name: "Chris"}, {$min: {age: 35}})`

$max will only update the value if the updated value is higher than the existing value

$mul will multiply the value matched with the one given in the query

- `db.users.updateOne({name: "Chris"}, {$mul: {age: 1.1}})`

drop a field from a document or documents

- `db.users.updateMany({isSporty:true}, {$set: {phone:null}})`

$unset allows us to unset or you can say drop a field from a collection document

- `db.users.updateMany({isSporty:true}, {$unset: {phone:""}})`

$rename - renaming fields operator

- `db.users.updateMany({}, {$rename: {age: "totalAge"}})`

upsert - command to insert if update does not match

- `db.users.updateOne({name: "Maria"}, {$set: {age: 39, hobbies: [{title: "Good Food", frequency: 5}], isSporty: true}}, {upsert: true})`

Update Assignment
-------------------

upsert two new documents in a collection
  
- `db.teams.updateOne({}, {$set: {title: "Football, requireteam: true}, {upsert: true})`
  
- `db.teams.updateOne({title: "Running"}, {$set: {requireteam: false}}, {upsert: true})`

update all documents that requires a team by adding a field with minimum amount of players required

- `db.teams.updateOne({requireteam: true}, {$set: {minimumplayers: 11}})`

update all documents that requires a team by increasing the number of required players by 10
  
- `db.teams.updateOne({requireteam: true}, {$inc: {minimumplayers: 10}})`

Update Arrays
---------------

find and elemMatch in arrays
  
- `db.users.find({hobbies: {$elemMatch: {title: "Sports", frequency: {$gte: 3}}}}).pretty()`
  
- `db.users.find({hobbies: {$elemMatch: {title: "Sports", frequency: {$gte: 3}}}}).count()`
  
- `db.users.updateMany({hobbies: {$elemMatch: {title: "Sports", frequency: {$gte: 3}}}}, 
 {$set: {"hobbies.$.highFrequency": true}})`
  
$ sign placeholder is a lot helper when we want to update a specific element in an array
  
- `db.users.updateMany({totalAge: {$gt: 30}}, {$inc: {"hobbies.$[].frequency": -1}})`
  
$[] means for each element inside the hobbies array as without [] ot updates only the first element
  
- `db.users.updateMany({totalAge: {$gt: 30}}, {$inc: {"hobbies.$[].frequency": -1}})`
  
using array filters

```javascript
db.users.updateMany({totalAge: {$gt: 30}}, {$set: {"hobbies.$[el].goodFrequency": true}},
  {arrayFilters: [{"el.frequency": {$gt: 2}}]})
```
  
$push - adding elements to arrays
  
- `db.users.updateOne({name: "Maria"}, {$push: {hobbies: {title: "sports", frequency: 2}}})`
  
```javascript
db.users.updateOne({name: "Maria"}, {$push: {hobbies: 
  {$each: [{title: "Good Wine", frequency: 1}, {title: "Hockey", frequency: 2}], 
  $sort: {frequency: -1}}}})
```
  
$pull - used to remove data from array
  
- `db.users.updteOne({name: "Maria"}, {$pull: {hobbies: {title: "Hiking"}}})`
  
$addToSet - used to add the data to the existing array set
  
- `db.users.updateOne({name: "Maria"}, {$addToSet: {hobbies: {title: "Hiking", frequency: 2}}})`
  
Delete Documents
-------------------
  
- `db.users.deleteOne({name: "Chris"})`
  
- `db.users.deleteMany({age: {$gt: 30}, isSporty: true})`
  
- `db.users.deleteMany({age: {$exists: false}, isSporty: true})`

Indexes
---------

indexes are used to make the search within a collection faster. The conventional method to search is that when we find a document in a collection we search the complete collection. But if we create indexes then in the sorted index we can search for the information much more efficiently. But we also cannot create indexes based each item of the document because then we will have to update the indexes created on all insert and updates.

use explain() to explain the query

```
db.persons.explain().find({"dob.age": {$gt:60}})
```

mongodb thinks in so-called plans and plans are simply alternatives it considers for executing that query and in the end it will find a winning plan and that winning plan is essentially what it did to get our results

to explain the statistics for the query use the below command

```
db.persons.explain("executionStats").find({"dob.age": {$gt:60}})
```

Whilst we can't really see the index, you can think of the index as a simple list of values + pointers to the original document.

Something like this (for the "age" field):

`(29, "address in memory/ collection a1")`

`(30, "address in memory/ collection a2")`

`(33, "address in memory/ collection a3")`

The documents in the collection would be at the "addresses" a1, a2 and a3. The order does not have to match the order in the index (and most likely, it indeed won't).

The important thing is that the index items are ordered (ascending or descending - depending on how you created the index). `createIndex({age: 1})` creates an index with ascending sorting, `createIndex({age: -1})` creates one with descending sorting.

MongoDB is now able to quickly find a fitting document when you filter for its age as it has a sorted list. Sorted lists are way quicker to search because you can skip entire ranges (and don't have to look at every single document).

Additionally, sorting (via sort(...)) will also be sped up because you already have a sorted list. 

Indexes are only beneficial if we are searching to get data that is only a small fraction of the entire collection. If the data returned is huge and is a big part of the collection then indexes slow you down becuase then there is this extra overhead of going to the index and then going to most of the documents anyways.

creating an index

`db.persons.createIndex({gender: 1})`

dropping an index

`db.persons.dropIndex({gender: 1})`

compound index - makes and index bases on multiple keys and their order does not matters

`db.persons.createIndex({"dob.age: 1, gender: 1"})`

but if we have a compound index we cannot search based on only on of the indexed objects. Then indexes will only run on the find combination of both the indexes objects. This is the limitation of the indexes.

but we can use sort to use this situation

`db.persons.explain().find({"dob.age": 35}).sort({gender: 1})`

now the query will use indexes although we have given just one of the indexed elements in the query because mongo db knows that it already has a sorted list of the documents based on the age and gender.

Mongo DB has a memory restriction of 32MB for sorting. Mongodb fetch all our documents in memory and then do the sorting and for large collection this can be too much to sort. So we need indexes to sort in a manner so that it does not exceed the memory threshold of 32mb. So if we already have a sorted index then it helps with the sorting.

to get the indexes we have created

`db.persons.getIndexes()`

default index is based on _id and another will be what we have created. The _id index which we get by default is an unique index by default.

Configuraing indexes for unique field

- `db.Contacts.createIndex({email: 1}, {unique: true})`

Partial Filters
----------------

partial filters are used to filter the indexes based on the filter expression

- `db.Contacts.createsIndex({"dob.age" : 1}, {partialFilterExpression: {gender: "male"}})`

This will create an index based on age but only for gender where value is male.

but after this whenever we search based on the index we also have to give the filter expression condition also other it will do collection scan and not index scan

The benefit is that our index is smaller if we have index based on filtered data.

if we have created an index where empty value should be considered as unique so that when we enter empty value or same value the we do not get error from mongodb

- `db.users.createIndex({email:1}, {unique: true, partialfilterExpression: {email: {$exists: true}}})`

TTL (Time to Live) Indexes
-----------------------------

add current data with doucments in mongodb - createdAt: new Date()

to remove every element after a certain amount of time

```
db.sessions.createIndex({createdAt: 1}, {expireAfterSeconds:10})
```

This will delete all the items that are a part of this index after 10 seconds

Covered query means that what we are searchig using a query is covered by an index, that make our query faster to execute as compared to query that is not covered by indexes. So make sure that the queries that we are running most frequently is covered by an index.

How mongoDB reject a plan or creates a winning plan
-----------------------------------------------------

Mongo DB caches the winning plan for the query which is the best plan for executing the data. But it also clears the cache after sometine when indexes are rebuilt or when we restart the server.

Using explain and statistics about the queries we can always find out how our query is performing and optimize our queries.

Multi-Key Indexes
---------------------

If we create an index on an array then mongodb pulls out all the values in the array and stores them as separate element in an index. Multi key indexes are bigger than single field indexes. They are bigger so they takes more space so make them only if we query that array frequently. 

creates index on hobby array - `db.contacts.createIndex({hobbies:1})`

we can create index on an element inside an array and it will also be a multikey index.

we can also create an index with array and single object together

`db.contacts.createIndex({name: 1, hobbies: 1})`

but we cannot create an index with an array and an element inside of that array together.

Understanding Text Indexes
------------------------------

`db.products.createIndex({description: "text"})`
  
This will create a text index where it will create an array of all the keyword that are in the description.
  
`db.products.find({$text: {search: "awesome"}}).pretty()`
  
`db.products.find({$text: {search: "\"red book\""}}).pretty()` //looking for keywords in text
  
mongo db assigns a score to a result for text based indexes
  
`db.products.find({$text: {search: "awesome Tshirt"}}, {score: {$meta: "textscore"}}).pretty()`
  
to sort based on this `$meta` score
 
```javascript
db.products.find({$text: {search: "awesome Tshirt"}}, {score: {$meta: "textscore"}}).sort({score: {$meta: "textscore"}}).pretty()
```

we can have only one text index but we can merge the text of multiple fields to create one index
  
`db.products.createIndex({title: "text", description: "text"})`
  
The above query will create an index for both the text and description field in the same index
  
Using text indexes to exclude words
-------------------------------------
  
if we excucively wants to exculde any word from index search we can use the - sign
  
`db.products.find({$text: {search: "awesome -Tshirt"}}).pretty()`
  
this will give all the results based on the results such that the index should not contain the tshirt word
  
when we create an index the database stops insertion as it might conflict with the indexes so to add the indexes in the background
  
`db.ratings.createIndex({age: 1}, {background: true})`
  
Working with Geospacial data
-----------------------------

to add geospatial data we have to add it as an embedded document and the add coordiantes in an array

```javascript
db.places.insertOne({name: "California", location: {type: "Point", coordinates:[-122.4724356, 37.7672544]}})
```

to find a point that is close to our point we also needs a geospatial index to use the $near query

creating a geospatial index - `db.places.createIndex({location: "2dsphere"})`

to create an index on an area - `db.places.createIndex({area: "2dsphere"})`

use $near query

```javascript
db.places.find({location: {$near: {$geometry: {type: "Point", coordinates:[-122.471114, 37.771104]}}}})
```

to find a place with a polygon we can write query to find places within a given polygon using $geowithin and $geometry

```javascript
db.places.find({location: {$geoWithin: {$geometry:{type: "Polygon", coordinates: [[cordinates1, cordinates2, cordinates3, cordinates4]]}}}}).pretty()
```

To find out if a user is inside a geospatial area - use $geoIntersect

```javascript
db.areas.find({area: {$geoIntersects: {$geometry: {type: "Point", coordinates}}}})
```

Find places within certain radius

```javascript
db.places.find({location: {$geowithin: {$centreSphere: [[-122.46203, 37.77286], 1/6378.1]}}}).pretty()
```

Assignment for Geospatial Queries

Pick 3 points on googlemaps and store them in a collection

```javascript
db.places.insertOne({name: "Beergarden", location: {type: "Point", coordinates: [11.59228, 48.15203]}})
```

```javascript
db.places.insertOne({name: "Octoberfest", location: {type: "Point", coordinates: [11.54965, 48.13203]}})
```

```javascript
db.places.insertOne({name: "MyOldPlace", location: {type: "Point", coordinates: [11.56934, 48.15105]}})
```

Pick a point and find the nearest place within a min and max distance

```javascript
const nearLocation = [11.59475, 48.14235]

db.places.createsIndex({location: "2dsphere"})

db.places.find({location: {$near: {geometry: {type: "Point", coordinates: nearLocation}, 
  $minDistance: 1000, $maxDistance: 2000
  }}}).pretty()
```

Pick an area and see which points (that are stored in your collection) it contains

```javascript

const p1 = [11.6097, 48.14522]
const p2 = [11.57142, 48.15416]
const p3 = [11.6, 48.15954]

const polygonArea = [[p1, p2, p3, p1]]

const polygonObject = {type: "Polygon", coordinates: polygonArea}

db.places.find({location: {$geoWithin: {$geometry: polygonObject}}}).pretty()
```

Store at least one area in a different collection

```javascript
db.areas.insertOne({name: secondCollectionPlaces, area: polygonObject})

db.areas.find({area: {$goeIntersects: {$geometry: {type: "Point", coordinates: [11.59228, 48.15203]}}}}).pretty()

db.areas.createIndex({area: "2dsphere"})
```

Pick a point and find out which areas in your collection contain that point

```javascript
db.areas.find({area: {$geoIntersects: {$geometry: {type: "Point", coordinates: [11.61779, 48.15122]}}}}).pretty()
```

Aggregation Framework
-----------------------

Aggregation framework is about building a pipleine of steps that runs on the data that is retreived from your collection and then gives you the output in the form you need. 

example of an aggregation query

- match stage is taking a filter as we define it as an arguement to the find method.

`db.persons.aggregate([{$match: {gender: "female"}}]).pretty()`

- group stage

group allows us to group data by a certain field or by multiple fields. 
the _id in group defines which fields we want to group. The value of _id here is a document. The $symbol tell mongodb here that i am referring to a field of this document and then use . to point to a nested field. 

Group accumulates data. Thsi means that we will have multiple documents with the same state and group will only output one. So multiple documents will be merged into one because we are aggregating. 

query with match and group stage

```javascript
db.persons.aggregate([
  {$match: {gender: 'female'}},
  {$group: {_id: {state: "$location.state"}, totalPersons: {$sum: 1}}}
]).pretty()
```

query with the sort stage

```javascript
db.persons.aggregate([
  {$match: {gender: 'female'}},
  {$group: {_id: {state: "$location.state"}, totalPersons: {$sum: 1}}},
  {$sort: {totalPersons: -1}}
]).pretty()
```

- Assignment

match the people with age greater than 50, Group by gender and find out the average age and order the output by total persons

```javascript
db.persons.aggregate([
    { $match: { 'dob.age': { $gt: 50 } } },
    { $group: {
        _id: { gender: '$gender' },
        numPersons: { $sum: 1 },
        avgAge: { $avg: '$dob.age' }
      }},
    { $sort: { numPersons: -1 } }
  ]).pretty();
```

- $Project stage

This stage allows us to transform each document instead of grouping multiple together. Project stage is a more powerful version of projections.

```javascript
db.persons.aggregate([
  {$project: {_id:0, gender: 1, fullName: {$concat: ["$name.first", " ", "$name.last"]}}}
])
```

To convert only the first letter to upper case for first and last name

```javascript
db.persons.aggregate([
    { $project: {_id: 0, gender: 1, fullName: {
          $concat: [
            { $toUpper: { $substrCP: ['$name.first', 0, 1] } }, //0 is start and 1 is total characters to change to upper
            { $substrCP: [ '$name.first', 1, { $subtract: [{ $strLenCP: '$name.first' }, 1] }]},
            ' ',
            { $toUpper: { $substrCP: ['$name.last', 0, 1] } },
            { $substrCP: [ '$name.last', 1, { $subtract: [{ $strLenCP: '$name.last' }, 1] }]}
          ]}}}
  ]).pretty();
```
  
Convert location data to geojson object
  
```javascript
  db.persons.aggregate([
  {$project: {_id:0, name: 1, email: 1, 
    location: {type: 'Point', 
    coordiantes: [ 
      { $convert: { input: '$location.coordinates.longitude', to: "double", onError: 0.0, onNull: 0.0}}, 
      { $convert: { input: '$location.coordinates.latitude', to: "double", onError: 0.0, onNull: 0.0}}
    ]}}}
])
```

Converting birtdate - `birthdate: { $convert: { input: '$dob.date', to: 'date' } },` or `birthdate: { $toDate: '$dob.date' },` by using $toDate shortcut

There are other shortcuts available in mongoDB for conversion

Group by birthyear after conversion and group and sort them, we can group on results of project

```javascript
{ $group: { _id: { birthYear: { $isoWeekYear: "$birthdate" } }, numPersons: { $sum: 1 } } },
    { $sort: { numPersons: -1 } }
```

Pushing elementa in newly created arrays
-----------------------------------------

$push allows us to push an array into every incoming document

```javascript
db.persons.aggregate([
  {$group: {_id: {age: "$age"}, allHobbies: {$push: "hobbies"}}}
]).pretty()
```

To pull out the elements of an array use unwind. Unlike groups $unwind takes one document and pulls out multiple douments out of it

```javascript
db.friends.aggregate([
  { $unwind: "$hobbies"},
  { $group: {_id: {age: "$age"}, allHobbies: {$push: "$hobbies"}}}
])
```

Now to remove duplicate values from the results of above query we can use $addToSet instead of $push

```javascript
db.friends.aggregate([
  { $unwind: "$hobbies"},
  { $group: {_id: {age: "$age"}, allHobbies: {$addToSet: "$hobbies"}}}
])
```

Using Projections with Arrays - $slice
--------------------------------

```javascript
db.friends.aggregate([
  {$project: { _id: 0, examScore: { $slice: ["$examScores", 1]}}}
])
```

Get the length of an array - $size
-----------------------------

```javascript
db.friends.aggregate([
  {$project: { _id: 0, numScore: { $size: "$examScores" }}}
])
```

Filter our certain elements in an array - $filter
----------------------------------------

To return array with score value greater than 60

```javascript
db.friends.aggregate([
  {$project: { _id: 0, examScores: { $filter: { input: "$examScores", as: "sc", cond: { $gt: ["$$sc.score, 60"] }}}}}
])
```

