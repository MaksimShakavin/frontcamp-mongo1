### 3.1 How many “Chinese” (cuisine) restaurants are in “Queens” (borough
#### Query:
```javascript
 db.restaurants.find({borough:'Queens',cuisine:'Chinese'}).count()
```
#### Result:
```
728
```

### 3.2 What is the _id of the restaurant which has the grade with the highest ever score?
#### Query:
```javascript
db.restaurants.aggregate([
     {$unwind:"$grades"},
     {$sort:{'grades.score':-1}},
     {$limit:1},
     {$project:{"grades.score":1}}
])
```
#### Result:
```
{ "_id" : ObjectId("5a5e4db2b6bb7999b2fb6687"), "grades" : { "score" : 131 } }
```
### 3.3 Add a grade { grade: "A", score: 7, date: ISODate() } to every restaurant in “Manhattan” (borough)
#### Query:
```javascript 
db.restaurants.update(
 {borough:'Manhattan'},
 {$push:
    {grades:{
        grade:"A",
        score: 7,
        date: ISODate()
        }}},
  {multi: true}
 )
```
#### Result:
```
WriteResult({ "nMatched" : 10259, "nUpserted" : 0, "nModified" : 10259 })
```
### 3.4 What are the names of the restaurants which have a grade at index 8 with score less then 7? Use projection to include only names without _id.
#### Query:
```javascript
db.restaurants.find({'grades.8.score': {$lt: 7}}, {_id: 0, name: 1})
```
#### Result:
```
{ "name" : "Silver Krust West Indian Restaurant" }
{ "name" : "Pure Food" }
```
### 3.5 What are _id and borough of “Seafood” (cuisine) restaurants  which received at least one “B” grade in period from 2014-02-01 to 2014-03-01? Use projection to include only _id and borough.
#### Query
```javascript
db.restaurants.find({
 cuisine:'Seafood',
 grades:{
   $elemMatch: {
	grade:'B',
	date: {$gt: ISODate('2014-02-01'),$lt:ISODate('2014-03-01')}
   }
 }
},{borough: 1})
```
#### Result:
```
{ "_id" : ObjectId("5a5e4db2b6bb7999b2fb9a93"), "borough" : "Bronx" }
{ "_id" : ObjectId("5a5e4db2b6bb7999b2fb9d0a"), "borough" : "Manhattan" }
```
### 4.1 What are the names of the restaurants which have a grade at index 8 with score less then 7? Use projection to include only names without _id.
#### Query:
```javascript
db.restaurants.createIndex({name: 1})
```
#### Proof:
```
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "frontcamp.restaurants",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "name" : {
                                "$eq" : "Glorious Food"
                        }
                },
                "winningPlan" : {
                        "stage" : "FETCH",
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "keyPattern" : {
                                        "name" : 1
                                },
                                "indexName" : "name_1",
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                        "name" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : false,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "name" : [
                                                "[\"Glorious Food\", \"Glorious Food\"]"
                                        ]
                                }
                        }
                },
                "rejectedPlans" : [ ]
        },
        "serverInfo" : {
                "host" : "drozdovase",
                "port" : 27017,
                "version" : "3.6.2",
                "gitVersion" : "489d177dbd0f0420a8ca04d39fd78d0a2c539420"
        },
        "ok" : 1
}
```
### 4.2. Drop index from task 4.1
#### Query
```javascript
db.restaurants.dropIndex({name: 1})
```
#### Result
```
{ "nIndexesWas" : 2, "ok" : 1 }
```
### 4.3. Create an index to make this query covered and provide proof (from explain() or Compass UI) that it is indeed covered:
```javascript
db.restaurants.find({ restaurant_id: "41098650" }, { _id: 0, borough: 1 })
```
#### Query:
```
db.restaurants.createIndex({restaurant_id:1, borough:1})
```
#### Results: 
```json
{
  "createdCollectionAutomatically" : false,
  "numIndexesBefore" : 1,
  "numIndexesAfter" : 2,
  "ok" : 1
}
```
#### Proof: 
```
	Results : {
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "frontcamp.restaurants",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "restaurant_id" : {
                                "$eq" : "41098650"
                        }
                },
                "winningPlan" : {
                        "stage" : "PROJECTION",
                        "transformBy" : {
                                "_id" : 0,
                                "borough" : 1
                        },
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "keyPattern" : {
                                        "restaurant_id" : 1,
                                        "borough" : 1
                                },
                                "indexName" : "restaurant_id_1_borough_1",
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                        "restaurant_id" : [ ],
                                        "borough" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : false,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "restaurant_id" : [
                                                "[\"41098650\", \"41098650\"]"
                                        ],
                                        "borough" : [
                                                "[MinKey, MaxKey]"
                                        ]
                                }
                        }
                },
                "rejectedPlans" : [ ]
        },
        "serverInfo" : {
                "host" : "drozdovase",
                "port" : 27017,
                "version" : "3.6.2",
                "gitVersion" : "489d177dbd0f0420a8ca04d39fd78d0a2c539420"
        },
        "ok" : 1
}
```

### 4.4 Create a partial index on cuisine field which will be used only when filtering on borough equal to �Staten Island�:
```
    db.restaurants.find({ borough: "Staten Island", cuisine: "American" }) � uses index
    db.restaurants.find({ borough: "Staten Island", name: "Bagel Land" }) � does not use index
    db.restaurants.find({ borough: "Queens", cuisine: "Pizza" }) � does not use index
```
#### Query:
```
db.restaurants.createIndex({borough: 1, cuisine: 1},{partialFilterExpression: {borough:'Staten Island'}})
```
#### Result:
```
{
 "createdCollectionAutomatically" : false,
 "numIndexesBefore" : 2,
 "numIndexesAfter" : 3,
 "ok" : 1
}
```
### 4.5. Create an index to make query from task 3.4 covered and provide proof (from explain() or Compass UI) that it is indeed covered
#### Query: 
```javascript
db.restaurants.createIndex({borough: 1 },{partialFilterExpression: {cuisine:'Seafood',grades:{grade:"B", date: {$gt: ISODate('2014-02-01'),$lt:ISODate('2014-03-01')}}}})
```
#### Result:
```json
{
 "createdCollectionAutomatically" : false,
 "numIndexesBefore" : 3,
 "numIndexesAfter" : 4,
 "ok" : 1
}
```
#### Proof: 
```
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "frontcamp.restaurants",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "$and" : [
                                {
                                        "grades" : {
                                                "$elemMatch" : {
                                                        "$and" : [
                                                                {
                                                                        "grade" : {
                                                                                "$eq" : "B"
                                                                        }
                                                                },
                                                                {
                                                                        "date" : {
                                                                                "$lt" : ISODate("2014-03-01T00:00:00Z")
                                                                        }
                                                                },
                                                                {
                                                                        "date" : {
                                                                                "$gt" : ISODate("2014-02-01T00:00:00Z")
                                                                        }
                                                                }
                                                        ]
                                                }
                                        }
                                },
                                {
                                        "cuisine" : {
                                                "$eq" : "Seafood"
                                        }
                                }
                        ]
                },
                "winningPlan" : {
                        "stage" : "PROJECTION",
                        "transformBy" : {
                                "borough" : 1
                        },
                        "inputStage" : {
                                "stage" : "COLLSCAN",
                                "filter" : {
                                        "$and" : [
                                                {
                                                        "grades" : {
                                                                "$elemMatch" : {
                                                                        "$and" : [
                                                                                {
                                                                                        "grade" : {
                                                                                                "$eq" : "B"
                                                                                        }
                                                                                },
                                                                                {
                                                                                        "date" : {
                                                                                                "$lt" : ISODate("2014-03-01T00:00:00Z")
                                                                                        }
                                                                                },
                                                                                {
                                                                                        "date" : {
                                                                                                "$gt" : ISODate("2014-02-01T00:00:00Z")
                                                                                        }
                                                                                }
                                                                        ]
                                                                }
                                                        }
                                                },
                                                {
                                                        "cuisine" : {
                                                                "$eq" : "Seafood"
                                                        }
                                                }
                                        ]
                                },
                                "direction" : "forward"
                        }
                },
                "rejectedPlans" : [ ]
        },
        "serverInfo" : {
                "host" : "drozdovase",
                "port" : 27017,
                "version" : "3.6.2",
                "gitVersion" : "489d177dbd0f0420a8ca04d39fd78d0a2c539420"
        },
        "ok" : 1
}
```