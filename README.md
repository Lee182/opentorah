# opentorah
An express app (in very early stages at the moment) which renders the hebrew bible and mechanical transation to your browser from a mongo database.
<br/>
The data structure is a flat level structure where a single word is an single entry
## Install
### Dependencies
- mongodb (see here for instalation)
- nodejs

### Initilizing
```
$ git clone /respitory/
$ cd /respitory/

$ mongoimport --db=reader -c torah --jsonArray torah.json
$ npm install
```
## Start-up

Navigate to the opentorah folder and run
```
$ node app.js
// the default express server is at http://localhost:5775
```
In the url you can type localhost:5775/bookname/1?v=5
<br />
## Express App Functionality (...So far)
With the app 
```
localhost:5775/genesis                        // renders genesis 1
localhost:5775/leviticus/19                   // renders leviticus chapter 19
localhost:5775/deuteronomy/6?v=4              // uses a verse query renders only deuteronomy 6:4
```
Note the only books avalible atm is this array``["genesis", "exodus", "leviticus", "numbers", "deuteronomy"]``
## db queries (examples)
### Schema
The data structure is a flat level structure where a single word is an single entry
``` 
$ mongo reader                    // Connect to reader database
> db.torah.findOne({})            // Query the torah collection with the db
{
	"_id" : 1,                       // This is an words unique identifyer
	"book" : "genesis",
	"c" : 1,                         // Chapter
	"v" : 1,                         // Hebrew verse number
	"w" : 1,                         // Word number within that verse
	"heb" : "בְּרֵאשִׁית",                // Heberew
	"eng" : "in~SUMMIT"              // English Mechanical translation from Jeff Benner
}
```
How many words are there in the torah (or this data set)?
```
db.torah.count()
```
How many times does moses appear?
```
db.torah.find({eng:{$regex: "Mosheh"}}).count()           // 647
```
### Find a given value
#### Simple finds
for more about .find() you can read [in the docs](http://docs.mongodb.org/manual/reference/method/db.collection.find/) or an [example](http://docs.mongodb.org/manual/core/read-operations-introduction/)
<br/>
and query operators (those they start with a dollar sign $or) [here](http://docs.mongodb.org/manual/reference/operator/query/)
```
// Finds any word with the same value as hashamayim and sorts in ascending order 
> db.torah.find({heb:"הַשָּׁמיִם"}).sort({_id: 1})

// RegEx searches are used to find pattern in text, the .limit(5) 'limits' the top 5 results
> db.torah.find({eng:{$regex: "SUMMIT"}}).limit(5)
```
#### The Aggregation Framework
Again its probably good to dive in and read the [introduction](http://docs.mongodb.org/manual/core/aggregation-introduction/) and concepts on aggregation
<br/>
Here are some things you can do
Find the numbers of words without an translation eng: '[x]'
```
db.torah.aggregate([
  // Filter by book or other fields using $match
  { $match: {eng: '[X]', $or: [{book: "exodus"}, {book: "numbers"}]} },
  
  // Group the results by book to make an report
  { $group: {_id: "$book", count: {$sum: 1}, words: {$addToSet: "$_id"}} },
  
  // Hide words array and cleanup the format
  { $project: {_id:0, book: "$_id", n:"$count"} }
])
```
Find the top 20 words in the torah
```
db.torah.aggregate([
  { $group: {_id: "$heb", count: {$sum: 1}, eng: {$addToSet: "$eng"}} },
  { $sort: {count: -1}},
  { $limit: 20 }
])
```
Find the average number of words per chapter in an book
```
db.torah.aggregate([
  { $group: { _id: {book: "$book", c:"$c"}, count: {$sum: 1} } },
  { $group: {_id: "$_id.book", average: {$avg: "$count"}} }
])
```
## About 0.1
Notice this project is 0.0.1 version, this is because I have some __big aims__
- A mulit-layed computerised translation with dictionaries, lexicon and concordances
- Develop a plugable translation concept
  - Where users can modify the database,
    - to improve translation, 
    - create a translation, 
    - or change words at preference    
  - This data can be exported and someone else can import the data like plugand play

