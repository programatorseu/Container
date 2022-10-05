# Database

## Intro

place to store data  / save app state 

**query** - command we send to a database to get it to do something

If db is table - **schema** are columns --- shape of data

**types of databases**

- relational - RDBMS or SQL 
- document-based (NoSQL)
- graph 
- keey-value store 



Atomic ( if crashes -- can not be divided)

Consitent ( if master crushes then slavery must run)

Isolation (multi-threaded db)

Durability (f a server crashes that we can restore it to the state )

**transactions** - like an envelope (guarantees that no other query will happen between them)



### 2. NoSQL Databases

"marketing" term

we define schema on "fly"

With a relational database, you'll have to define the shape of your data upfront

when we put wrong name - it thinks we are trying to create new column 



### 2.1 MongoDB

```
docker run --name test-mongo -dit -p 27017:27017 --rm mongo:4.4.1
```

run a new MongoDB container  and call it `test-mongo` 

```
docker exec -it test-mongo mongo
```

run the command `mongo` inside of that `test-mongo` container so we can do things inside of the container.

`mongo` - official container put together by MongoDB Inc themselves)

```
show dbs; 
```

Database --> group of collection 

1 row -> document  = entry  = record

Table of rrows - collection 

```
use adoption; # automatically created
db.pets # name of collection
db.insertOne() # function that takes JS object and change into Record
db.pets.insertOne({name: "Bleki", type: "dog", breed: "Kundel", age: 16})
```



> { "acknowledged" : true, "insertedId" : ObjectId("632b08eb2a35516b591ef4de") }



```
db.pets.count() # 1
db.stats() # show us we have 1 collection
```

gHud9953!328



```bash
db.pets.findOne() 
```

>{
>	"_id" : ObjectId("633824d63db943b74401b8d7"),
>	"name" : "Bleki",
>	"type" : "dog",
>	"breed" : "Kundel",
>	"age" : 16
>}

first type that match particular type : 

```bash
db.pets.findOne({type: "cat"})  # null
db.pets.findOne({type: "dog"}) # {object with props} 
```



```bash
db.pets.find({type: "dog"}) # give iterator to iterate over all
```



`insertMany` to populate database with many thousands of records 

```js
db.pets.insertMany(
  Array.from({ length: 10000 }).map((_, index) => ({
    name: [
      "Luna",
      "Fido",
      "Fluffy",
      "Carina",
      "Spot",
      "Beethoven",
      "Baxter",
      "Dug",
      "Zero",
      "Santa's Little Helper",
      "Snoopy",
    ][index % 9],
    type: ["dog", "cat", "bird", "reptile"][index % 4],
    age: (index % 18) + 1,
    breed: [
      "Havanese",
      "Bichon Frise",
      "Beagle",
      "Cockatoo",
      "African Gray",
      "Tabby",
      "Iguana",
    ][index % 7],
    index: index,
  }))
);
```



```js
db.pets.findOne({index: 1337 })
db.pets.findOne({type: "dog", age: 9 })

db.pets.find({type: "dog", age: 9})
// returns 20 records we need to type it to iterate over
db.pets.count({type: "dog", age: 9})
db.pets.find({type: "dog"}).limit(5)
// skip limits 20 records -> show all from collection
db.pets.find({type: "dog"}).limit(40).toArray()

```

like SQL where --- we provide subquery to mongoDB 

```js
db.pets.count({type: "cat", age: {$gt:12}});
```

> gt - greater than
>
> $gte - greater than or equal to
>
> $lt - less than
>
> $lte - less than or equal to
>
> $eq - equals (usually not necessary)
>
> $ne - not equals
>
> $in - has the value in the array (MongoDB can store arrays and objects too!)
>
> $nin - does not have the value in the array

```js
db.pets.find({name: "Fido", type: {$ne: "dog"}})
```

### 2.2 Logical operators

and : array and pass queries

```js
db.pets.find({
    type: "bird",
    $and: [{age: {$gte: 4}}, {age: {$lte: 8}}],
}); 
```

**sorts **

here we have example of descending order : 

```js
db.pets.find({
    type: "dog"
}).sort({age: -1 })
```

### 2.3 Projections

limit which fields we return : 

```js
db.pets.find({ type: "dog" }, { name: 1, breed: 1 });
```

`name:1` - it works like true - that we want to include in our result

```js
db.pets.find(
    {type: "dog"},
    {name: 1}
).limit(5)
```

>{ "_id" : ObjectId("633928f9cb486deedb5a0bcd"), "name" : "Bleki" }

```js
db.pets.find(
    {type: "dog"},
    {name: 1,
    _id: 0}
).limit(1)
```

> { "name" : "Bleki" }

 

### 2.4 Updating MongoDB

```js
db.pets.updateOne(
    { type: "dog", name: "Luna", breed: "Havanese" },
    { $set: { owner: "Brian Holt" } }
  );
```

update many and increment age by 1 :

```js
db.pets.updateMany({ type: "dog" }, { $inc: { age: 1 } });
```

**upsert**

- update if you found it 
- insert if you have not found it 

```js
db.pets.updateOne(
  {
    type: "dog",
    name: "Sudo",
    breed: "Wheaten",
  },
  {
    $set: {
      type: "dog",
      name: "Sudo",
      breed: "Wheaten",
      age: 5,
      index: 10000,
      owner: "Sarah Drasner",
    },
  },
  {
    upsert: true,
  }
);
```



### 2.5 Deleting

```js
db.pets.deleteMany({ type: "reptile", breed: "Havanese" });
  
```

findAndDelete  - will return found element in the end

```js
db.pets.findOneAndDelete({name: "Fido", type: "reptile"});
```

**bulkwrite**

--> queue up few queries and run 

--> do bunch of things at once 



### 2.6 Indexes in MongoDB

 Indexes are a separate data structure that a database maintains so that it can find things quickly.

```js
db.pets.find({name: "Fido"}).explain("executionStats")
```

>"winningPlan" : {
>		"stage" : "COLLSCAN",

it is going trough each column 

create simple index on name column : 

```js
db.pets.createIndex({name: 1});
```

it is really expensive -> on production server - it will take down entire app !! 

```
db.pets.find({name: "Fido"}).explain("executionStats")
```



>"executionStages" : {
>			"stage" : "FETCH",

to check what are indexes : 

```js
db.pets.getIndexes()
```

> [
> 	{
> 		"v" : 2,
> 		"key" : {
> 			"_id" : 1
> 		},
> 		"name" : "_id_"
> 	},
> 	{
> 		"v" : 2,
> 		"key" : {
> 			"name" : 1
> 		},
> 		"name" : "name_1"
> 	}
> ]

```js
db.pets.createIndex({name: 1}, {unique: true});
```

### 2.7 Text Search Indexes

-  create index again  - it will create new index containing all 3 fields : 

```js
db.pets.createIndex({type: "text", breed: "text", name: "text"});

```

$text - special operator 

$search -   



```js
db.pets.find({$text: {$search: "dog Havenes Luna" }});
```

> { "_id" : ObjectId("63392913cb486deedb5a32a2"), "name" : "Spot", "type" : "dog", "age" : 5, "breed" : "Havanese", "index" : 9940 }
> { "_id" : ObjectId("63392913cb486deedb5a329e"), "name" : "Luna", "type" : "dog", "age" : 1, "breed" : "Cockatoo", "index" : 9936 }

want the thing that matches most closely with your search terms. We can do that, it just doesn't do that by default.

```js
db.pets
  .find({ $text: { $search: "dog Havanese Luna" } })
  .sort({ score: { $meta: "textScore" } });
```

### 2.8 Aggregation

- aggregation pipeline 
- Map Reduce

 

take data and sliced it up to different buckets 

we can do additions / combine them / unwind them 

aggregation goes in stages : 

1. do this  ( bucketing )

   break pet collection down into buckets by age 

   	- group by age 
   	- add boundaries of age (0-2. 3-8, 9-14)
   	- add default - for anything that does not match boundary goes with that bucket

    -  define what kind of ouput we want to put in there 
       - we want to `count` {total sum - increase count by 1 }



With the output you're defining what you want to pass to the next step.  In this case we just want to sum them up by adding 1 to the count each  time we see a pet that matches a bucket.



```js
db.pets.aggregate([
  {
    $bucket: {
      groupBy: "$age",
      boundaries: [0, 3, 9, 15],
      default: "very senior",
      output: {
        count: { $sum: 1 },
      },
    },
  },
]);
```

`$match` stage of aggregtion  -- exclude every pet that isn't a dog

`$sort` - here we are using decreasing 

```js
db.pets.aggregate([
    {
        $match: {
            type: "dog"
        },
    },
    {
        $bucket: {
            groupBy: "$age",
            boundaries: [0,3,9,15],
            default: "very senior",
            output: {
                count: {$sum: 1},
            },
        },
    },
    {
        $sort: {
            count: -1,
        }
    }
    
]);
```

## 2.9 Node app

```bash
npm init -y
npm i express mongodb@3.6.2 express@4.17.1
mkdir static
touch static/index.html server.js
```



## 3. SQL - Relational Database 

- like a spreadsheet 

big table with many columns and rows.

Each row reprents one entry in the database and each column represents one field in the database.

there is pre-made schema 

- can have multiple databases that all relate to each other.



in MongoDB we send JSON query objects  / , here we'll send query strings that the database will read and execute for us



- Mysql / MariaDB  ( acquired by oracle

- microsoft SQL / Oracle / DB2
- SQLITe
- PostgreSQL



```bash
docker run --name my-postgres -e POSTGRES_PASSWORD=mysecretpassword -p 5432:5432 -d --rm postgres:13.0
docker exec -it -u postgres my-postgres psql
```



**CREATE DATABASE**

```sql
CREATE DATABASE message_boards;
\c message_boards  # switch to this db 
-- see all databases
\l

-- see all tables in this database, probably won't see anything
\d
\? -- see all commands
\h -- see queries
-- run a shell command
\! ls && echo "hi from shell!"
```



**Create TABLE**

```sql
CREATE TABLE users (
  user_id INTEGER PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
  username VARCHAR ( 25 ) UNIQUE NOT NULL,
  email VARCHAR ( 50 ) UNIQUE NOT NULL,
  full_name VARCHAR ( 100 ) NOT NULL,
  last_login TIMESTAMP,
  created_on TIMESTAMP NOT NULL
);
```

`GENERATED ALWAYS AS IDENTITY` means. It's autoincrementing. 

`VARCHAR ( 50 ) UNIQUE NOT NULL,` string (50 limit), unique, not omitted

```sql
INSERT INTO users (username, email, full_name, created_on) VALUES ('btholt', 'lol@example.com', 'Brian Holt', NOW());

SELECT * FROM users
```

This `\` notation is how you give admin commands to PostgreSQL through its `psql` CLI 



## 3.1 Querying PostgreqSQL

```sql
SELECT * FROM users;
SELECT * FROM users LIMIT 10;
SELECT username, email, user_id FROM users WHERE user_id=150;
SELECT username, email, user_id FROM users WHERE last_login IS NULL LIMIT 10;
```

math - to see users had not logged in and were created more than six months ago

```sql
SELECT username, email, user_id, created_on FROM users WHERE last_login IS NULL AND created_on < NOW() - interval '6 months'  LIMIT 10;
```

```sql
UPDATE users SET full_name= 'Brian Holt', email = 'lol@example.com' WHERE user_id = 2 RETURNING *;
```

- You just comma separate to do multiple sets.
- Make sure you use single quotes. Double quotes cause errors.
- RETURNING * is optional. This is basically saying "do the update and return to me the records after they've been updated.



## 3.2 complex query

```sql
CREATE TABLE boards (
  board_id INTEGER PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
  board_name VARCHAR ( 50 ) UNIQUE NOT NULL,
  board_description TEXT NOT NULL
);
CREATE TABLE comments (
  comment_id INTEGER PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
  user_id INT REFERENCES users(user_id) ON DELETE CASCADE,
  board_id INT REFERENCES boards(board_id) ON DELETE CASCADE,
  comment TEXT NOT NULL,
  time TIMESTAMP
);
```



`user_id INT REFERENCES users(user_id)` - make foreign key ( reference primary key of another table ! )

```sql
SELECT comment_id, user_id, LEFT(comment, 20) AS preview FROM comments WHERE board_id = 39;
```

The `LEFT` function will return the first X characters of a string

The `AS` keyword lets you rename how the string is projected. 

```text
 comment_id | user_id |       preview
------------+---------+----------------------
         63 |     858 | Maecenas tristique,
```
