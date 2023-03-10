---
title: "MongoDB CRUD Operations"
datePublished: Fri Oct 29 2021 12:25:53 GMT+0000 (Coordinated Universal Time)
cuid: ckvccmxle00h5cms1469fd6gw
slug: mongodb-crud-operations
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1635481915463/hiPIdjNIRc.png
tags: nosql, mongodb, databases, crud

---

Let's have a basic understanding of MongoDB CRUD operations with the help of an example.

First, let's create a database to store the data:

```
test> use hospital
output> switched to db hospital

``` 

Now we have a Hospital database. Let's create a "patients" collection and add some data to it:

```
hospital> db.patients.insertMany([{
firstname: "Aditya", 
lastname: "Gupta", 
age: 22, 
history: [{disease: "cold", treatment: 1}]
}, 
{
firstname: "Mark", 
lastname: "Manson", 
age: 50, 
history: [{disease: "fever", treatment: 2}]
}, 
{
firstname: "Robin", 
lastname: "Sharma", 
age: 55, 
history: [{disease: "sneeze", treatment: 4}]
}])
output> {
  acknowledged: true,
  insertedIds: {
    '0': ObjectId("61780b10d9a6a5af8bddb297"),
    '1': ObjectId("61780b10d9a6a5af8bddb298"),
    '2': ObjectId("61780b10d9a6a5af8bddb299")
  }
}
```
As you can see, there's no need to explicitly create a collection. Just try inserting data into a collection and if it doesn't exist, MongoDB will create it. 

Now, we have the data ready with 3 patients. Let's play!

#### 1. Fetch all the documents of patients collection.

```
hospital> db.patients.find()
output>[
  {
    _id: ObjectId("61780b10d9a6a5af8bddb297"),
    firstname: 'Aditya',
    lastname: 'Gupta',
    age: 22,
    history: [ { disease: 'cold', treatment: 1 } ]
  },
  {
    _id: ObjectId("61780b10d9a6a5af8bddb298"),
    firstname: 'Mark',
    lastname: 'Manson',
    age: 50,
    history: [ { disease: 'fever', treatment: 2 } ]
  },
  {
    _id: ObjectId("61780b10d9a6a5af8bddb299"),
    firstname: 'Robin',
    lastname: 'Sharma',
    age: 55,
    history: [ { disease: 'sneeze', treatment: 4 } ]
  }
]
```

#### 2. Find the patients having ages greater than 25.

```
hospital> db.patients.find({age: {$gt: 25}})
output>[
  {
    _id: ObjectId("61780b10d9a6a5af8bddb298"),
    firstname: 'Mark',
    lastname: 'Manson',
    age: 50,
    history: [ { disease: 'fever', treatment: 2 } ]
  },
  {
    _id: ObjectId("61780b10d9a6a5af8bddb299"),
    firstname: 'Robin',
    lastname: 'Sharma',
    age: 55,
    history: [ { disease: 'sneeze', treatment: 4 } ]
  }
]
```
Here, the `$gt` refers to `greater than` and `$` signifies that it is a special keyword for MongoDB. MongoDB has many such keywords. Look at the documentation to learn more about them.

3 Update `Aditya Gupta`'s age from **22** to **25**.
```
hospital> db.patients.updateOne({firstname: "Aditya", lastname: "Gupta"}, {$set: {age: 25}})
output>{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
```
The `updateOne` updates the first document that matches the matching criteria(first parameter), with the options given in the second parameter.
Now, check if it's updated.
```
hospital> db.patients.find()
output>[
  {
    _id: ObjectId("61780b10d9a6a5af8bddb297"),
    firstname: 'Aditya',
    lastname: 'Gupta',
    age: 25, <- updated successfully
    history: [ { disease: 'cold', treatment: 1 } ]
  },
  {
    _id: ObjectId("61780b10d9a6a5af8bddb298"),
    firstname: 'Mark',
    lastname: 'Manson',
    age: 50,
    history: [ { disease: 'fever', treatment: 2 } ]
  },
  {
    _id: ObjectId("61780b10d9a6a5af8bddb299"),
    firstname: 'Robin',
    lastname: 'Sharma',
    age: 55,
    history: [ { disease: 'sneeze', treatment: 4 } ]
  }
]
```
Similarly, there is `updateMany` which, as the name suggests, updates all the documents that match the matching criteria.
#### 3. Add a disease to the `history` array to the patient `Aditya Gupta`.
```
hospital> db.patients.updateOne({firstname: "Aditya"}, {$push: {history: {disease: "xyz", treatment: 45}}})
output>{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
```
Let's see the updated database:
```
hospital> db.patients.find()
output>[
  {
    _id: ObjectId("61780b10d9a6a5af8bddb297"),
    firstname: 'Aditya',
    lastname: 'Gupta',
    age: 25,
    history: [
      { disease: 'cold', treatment: 1 },
      { disease: 'xyz', treatment: 45 }
    ]
  },
  {
    _id: ObjectId("61780b10d9a6a5af8bddb298"),
    firstname: 'Mark',
    lastname: 'Manson',
    age: 50,
    history: [ { disease: 'fever', treatment: 2 } ]
  },
  {
    _id: ObjectId("61780b10d9a6a5af8bddb299"),
    firstname: 'Robin',
    lastname: 'Sharma',
    age: 55,
    history: [ { disease: 'sneeze', treatment: 4 } ]
  }
]
```
The `$push` command is used to add elements to an array.
#### 4. Delete the patients having `fever` as one of the diseases.
```
hospital> db.patients.deleteOne({"history.disease": "fever"})
output>{ acknowledged: true, deletedCount: 1 }
```
Let's look at the database now:
```
hospital> db.patients.find()
[
  {
    _id: ObjectId("61780b10d9a6a5af8bddb297"),
    firstname: 'Aditya',
    lastname: 'Gupta',
    age: 25,
    history: [
      { disease: 'cold', treatment: 1 },
      { disease: 'xyz', treatment: 45 }
    ]
  },
  {
    _id: ObjectId("61780b10d9a6a5af8bddb299"),
    firstname: 'Robin',
    lastname: 'Sharma',
    age: 55,
    history: [ { disease: 'sneeze', treatment: 4 } ]
  }
]
```
As you can see, the patient `Mark Manson` is deleted from the database as he had `fever` as a value in one of the items in the `history` array.
_____
These were some of the basic CRUD operations in MongoDB. Of course, there are a lot of variations of the above-mentioned functions which can be learned from the  [official documentation](https://docs.mongodb.com/).

Thanks for reading!