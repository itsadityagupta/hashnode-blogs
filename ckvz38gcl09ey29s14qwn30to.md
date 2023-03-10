---
title: "Relations in MongoDB"
datePublished: Sun Nov 14 2021 10:21:23 GMT+0000 (Coordinated Universal Time)
cuid: ckvz38gcl09ey29s14qwn30to
slug: relations-in-mongodb
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1636883707313/i8dhgbOE6.png
tags: nosql, mongodb, databases

---

Relations are an important part of any database. We encounter situations all the time when we have data items related to each other. This relationship needs to be depicted inside the MongoDB database somehow.

The relationship scenarios that can happen are the following:
1. One-to-One
2. One-to-Many
3. Many-to-Many

We can represent the relationships in two ways:

1. Using embedded documents: We can embed the document inside our main document if they're related.
2. Using references: We can have data stored in different collections and store the reference(id) wherever the relationship needs to be specified.

Which method to use depends highly on a specific use case and must be explored individually. However, you can ask the following questions to get more clarity on the way to proceed forward:
1. How do you fetch your data?
2. How often do you change it?
3. If you change it, does it needs to be changed everywhere?
4. Are duplicates fine?

These are only some of the questions. You can explore more as per your use case. But generally speaking, *one-to-one and one-to-many relationships may preferably use embedded documents and many-to-many relationships may preferably use references.*

---

Now, enough of theory. Let's see this with an example. Suppose we have a `bookRegistry` database with the details of all the books and their authors. Since a book can have multiple authors and an author can have multiple books, this is a `many-to-many` relationship.

If we want to use the embedded documents approach in this case, it is evident that for every book, we need to add author details inside the book document itself. This is not at all a problem. But the problem comes when some details regarding an author have to be modified. In such cases, we'll have to make changes in all the books that the author has written, which is not convenient.

Hence, we'll use the reference method. We'll create separate collections, `books` and `authors`, to store books and authors separately and indicate their relationship with a reference to an `id`. 

Let's start with some queries now!

---

- Create a `bookRegistry` database.
```
test> use bookRegistry
switched to db bookRegistry
```
- Create `authors` collection and insert few author details.
```
bookRegistry> db.authors.insertMany([{name: "Cal Newport", age: 45}, {name: "Mark Manson", age: 50}, {name: "Robin Sharma", age: 55}])
{
  acknowledged: true,
  insertedIds: {
    '0': ObjectId("6190b538a540378a7d997600"),
    '1': ObjectId("6190b538a540378a7d997601"),
    '2': ObjectId("6190b538a540378a7d997602")
  }
}
```
- Review the `authors` collection.
```
bookRegistry> db.authors.find()
[
  {
    _id: ObjectId("6190b538a540378a7d997600"),
    name: 'Cal Newport',
    age: 45
  },
  {
    _id: ObjectId("6190b538a540378a7d997601"),
    name: 'Mark Manson',
    age: 50
  },
  {
    _id: ObjectId("6190b538a540378a7d997602"),
    name: 'Robin Sharma',
    age: 55
  }
]
```
- Create a `books` collection and insert a few books. The `authors` field will contain the `_id` of the author in the `authors` collection
```
bookRegistry> db.books.insertMany([{title: "Deep Work", authors: [ ObjectId("6190b538a540378a7d997600")]}, {title: "Subtle Art of Not giving a F*ck", authors: [ ObjectId("6190b538a540378a7d997601")]}, {title: "The monk who sold his ferrari", authors: [ObjectId("6190b538a540378a7d997602")]}])
{
  acknowledged: true,
  insertedIds: {
    '0': ObjectId("6190b5e3a540378a7d997603"),
    '1': ObjectId("6190b5e3a540378a7d997604"),
    '2': ObjectId("6190b5e3a540378a7d997605")
  }
}
```
- Review the `books` collection.
```
bookRegistry> db.books.find()
[
  {
    _id: ObjectId("6190b5e3a540378a7d997603"),
    title: 'Deep Work',
    authors: [ ObjectId("6190b538a540378a7d997600") ]
  },
  {
    _id: ObjectId("6190b5e3a540378a7d997604"),
    title: 'Subtle Art of Not giving a F*ck',
    authors: [ ObjectId("6190b538a540378a7d997601") ]
  },
  {
    _id: ObjectId("6190b5e3a540378a7d997605"),
    title: 'The monk who sold his ferrari',
    authors: [ ObjectId("6190b538a540378a7d997602") ]
  }
]
```
- Render complete details of the book along with author details.

Since we have the details in two separate collections, we'll need to aggregate those into a single document to render complete details. This is done by the `aggregation` framework using `aggregate` function.

`Aggregate` function takes a series of commands, which can be learned from the  [official documentation](https://docs.mongodb.com/manual/aggregation/). But for our use case, `lookup` is the command. 
This takes a left-outer join to the given collection and renders the result. The syntax for this command is:
```
{
   $lookup:
     {
       from: <collection to join>,
       localField: <field from the input documents, in which we have the reference>,
       foreignField: <field from the documents of the "from" collection, which contains the reference>,
       as: <name of the field in which to have the output>
     }
}
```
Now, let's write the query.
```
bookRegistry> db.books.aggregate([{$lookup: {from: "authors", localField: "authors", foreignField: "_id", as: "authorDetails"}}])
[
  {
    _id: ObjectId("6190b5e3a540378a7d997603"),
    title: 'Deep Work',
    authors: [ ObjectId("6190b538a540378a7d997600") ],
    authorDetails: [
      {
        _id: ObjectId("6190b538a540378a7d997600"),
        name: 'Cal Newport',
        age: 45
      }
    ]
  },
  {
    _id: ObjectId("6190b5e3a540378a7d997604"),
    title: 'Subtle Art of Not giving a F*ck',
    authors: [ ObjectId("6190b538a540378a7d997601") ],
    authorDetails: [
      {
        _id: ObjectId("6190b538a540378a7d997601"),
        name: 'Mark Manson',
        age: 50
      }
    ]
  },
  {
    _id: ObjectId("6190b5e3a540378a7d997605"),
    title: 'The monk who sold his ferrari',
    authors: [ ObjectId("6190b538a540378a7d997602") ],
    authorDetails: [
      {
        _id: ObjectId("6190b538a540378a7d997602"),
        name: 'Robin Sharma',
        age: 55
      }
    ]
  }
]
```
As you can see, we have the entire details of the books along with the authors rendered in the output. The details are mentioned in `authorDetails` field without changing any other field.

---

Thanks for reading the article! If I've missed anything or you have something to add, do add a comment to this post!