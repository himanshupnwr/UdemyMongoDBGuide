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

.find gives us a cursor which is by default set to 20 documents

`find().hasNext()` -> to check if cursor has any next element to show
`.find().next` -> gives me the next document in line

type it for more

`dataCursor.forEach(doc => {printjson(doc)}` -> uses javascript foreach to display all documents in the datacursor document

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
```
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

```
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
Example -> `mongod --dbpath C:/Mongodatabase/development`

To set the path for file directory in which we want to have the mongo logs
Example -> `mongod --dbpath C:/Mongodatabase/logs/log`

To shut down Mongo DB Server from command prompt use
-> `db.shutdownServer()`
-> `net stop MongoDB`

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

-> `{w:1, j:true}`
`db.Patient.insertOne({name:"Alia", age:45}, {writeConcern:{w:1, j:true}})`

Now you can set a different option and set J to true, what you're now saying is hey please only report a success for this write to me after it has been both acknowledged and been saved to the journal, so now I have a greater security that this will happen and succeed even if the server should face issues right now.

`{w:1, wtimeout:200, j:true}`

Now there also is a third option not directly related to the journal but it's a W timeout option, now this simply means which timeframe do you give your server to report a success for this write before you cancel it.

So if you have some issues with the network connection or anything like that, you might simply timeout here, obviously setting this too low might timeout even though it would have perfectly succeeded normally and therefore you should know what you do when you set this timeout value because if you set it to a very low number, you might fail a lot even though there is no actual problem, just some small latency.

So this is the write concern and how you can control this, obviously enabling the journal confirmation means that your writes will take longer because you don't just tell the server about them but you also need to wait for the server to store that write operation in the journal but you get higher security that the write also succeeded. Again this is a decision you have to make depending on your application needs, what you need.

-> Mongo DB CRUD Operation are atomic on the document level(inluding embedded documents)

Importing Data
---------------
cd <navigate to json file which has the data>
--drop -> if collection exists then drop the existing one and add the new one

`mongoimport file.json -d movieData -c movies --jsonArray --drop`

Read Operations
---------------

$lte - lower than or equal to
$eq
$neq
$in - takes an array and check for values in the object `db.movies.find({runtime:{$in:[30,42]}}).pretty()` runtime either 30 or 42
$nin is not inside condition
not equals to -> {$not:{$eq:60}}
$gte -> greater than equal to
.count() - to get the count of the documents that passed the operator conditions

$or -> `{$or:[{"rating.average":{$lt:5}, {"rating.average":{$gt:9.3}}}]}`
$nor is the opposite of $or operator

Element Operators
------------------

$exists tells if the object is inside the document `db.users.find({age:{$exists:true, $gt:30}}).pretty()`

$type -> check for the type given in the condition - `db.Users.find({phone:{$type:"double"}}).pretty()`

`db.Users.find({phone:{$type:["double","string"]}}).pretty()` - checking multiple types

$regex -> finding text snippets using regex - but they are not very performant
`db.movies.find({summary:{$regex:/musical/}}).pretty()` - returns all documents where in the summary this word can be found

$expr -> when we want to compare two objects inside a document and use this comparison to find other similar documents
`db.sales.find({$expr: {$gt: ["$volume", "$target"]}}).pretty()`

$cond, $subtract, $if, $else operator
`db.sales.find({$expr: {$gt: [{$cond: {if: {$gte: ["$volume", 190]}, then: {$subtract: ["$volume", 30]}, else:"$volume"}}, "$target"]}}).pretty()`

$size - check the size of elements inside the array - `db.users.find({hobbies:{$size:3}}).pretty()`

$all - `db.moviestarts.find({genre: {$all: ["action", "thriller"]}}).pretty()`

$elemMatch - if we want the conditions to work in the same element
`db.users.find({hobbies: {$elemMatch: {title:"sports", frequency: {$gte:3}}}}).pretty()`

Sorting
----------

we use the number 1 and -1 for passing the type of sorting we need. 1 means ascending and -1 meand desending

`db.movies.find().sort({"rating.average": -1}).pretty()` minus 1 for descending

`db.movies.find().sort({"rating.average": 1, runtime: 1}).pretty()` - sorting by multiple criteria

`db.movies.find().sort({"rating.average": 1, runtime: 1}).skip(10).pretty()` - results with skipped 10 documents

limits - `db.movies.find().sort({"rating.average": 1, runtime: 1}).skip(10).limit(10).pretty()`

Projections
-------------

we can use projection to shape the structure of our results. if we want to include inly few elements of out document

`db.movies.find({}, {name: 1, genres: 1, runtime: 1, rating: 1, _id: 0}).pretty()`

if we want to exculde id field as it is not excluded by default we have to pass 0 for it

projections also works on the embedded data

`db.movies.find({}. {name:1, genres: 1, runtime: 1, "rating.average": 1, "schedule.time": 1, _id: 0}).pretty()`

In Arrays - `db.movies.find({genres: "Drama"}, {"genres.$": 1}).pretty()` - give only one item from array genres drama

`db.movies.find({genres: "Drama"}, {genres: {$elemMatch: {$eq: "Horror"}}}).pretty()`

$slice -> `db.movies.find({"rating.average": {$gt:9}}, {genres:{$slice:2}, name: 1}).pretty()`
slice is related to arrays. only the first 2 elements returned for the array because we used slice 2

`db.movies.find({"rating.average": {$gt:9}}, {genres:{$slice: [1, 2]}, name: 1}).pretty()`
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
