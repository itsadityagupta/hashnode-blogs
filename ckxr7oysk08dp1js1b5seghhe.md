---
title: "Creating and Importing Documents in MongoDB"
datePublished: Wed Dec 29 2021 07:23:27 GMT+0000 (Coordinated Universal Time)
cuid: ckxr7oysk08dp1js1b5seghhe
slug: creating-and-importing-documents-in-mongodb
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1639640448155/O7b_GT8CD.png
tags: nosql, mongodb, databases

---

I've already written about the basic CRUD operations in MongoDB in a [previous blog post](https://adityagupta.hashnode.dev/mongodb-crud-operations). In this blog post, I'll discuss some important aspects of insert commands in MongoDB.

---

## 1. Introduction to insert commands

`insertOne` and `insertMany` commands are used to insert one and multiple documents respectively. Apart from these, we also have the `insert` command. This can insert multiple or one documents itself. But the other two commands are preferred to increase better code readability. 

Also, the `insert` command doesn't give back the `_id` of the documents inserted, which may be required as per the use case.

---

## 2. Working of `insertMany`

`insertMany` does an *ordered* insert by default. This means that it'll start from the first document given and start inserting them one by one. If any of them have a duplicate ID that is already present in the database, it will throw an exception and cancel the insert operation for further documents.

*But the ones already inserted will not be rolled back*. This can be the desired behaviour but if not, we can do an **unordered insert** which will be performed for every document given, even if the previous document throws any exception. This can be done by passing a second argument to `insertMany` like: **{ordered: false}**.

```
employees> db.employeeDetails.insertMany([ 
{
    name: "Employee 4", 
    email: "employee4@gmail.com", 
    phone_no: "56743", 
    address: "Address 4"
},
{
    name: "Employee 3", 
    email: "employee3@gmail.com", 
    phone_no: 19876, 
    address: "address 3"
}
], {ordered: false}
)
output> Uncaught:
MongoBulkWriteError: Document failed validation
Result: BulkWriteResult {
  result: {
    ok: 1,
    writeErrors: [
      WriteError {
        err: {
          index: 0,
          code: 121,
          errmsg: 'Document failed validation',
          errInfo: {
            failingDocumentId: ObjectId("61bafcedde146d867cab4e65"),
            details: {
              operatorName: '$jsonSchema',
              schemaRulesNotSatisfied: [Array]
            }
          },
          op: {
            name: 'Employee 4',
            email: 'employee4@gmail.com',
            phone_no: '56743',
            address: 'Address 4',
            _id: ObjectId("61bafcedde146d867cab4e65")
          }
        }
      }
    ],
    writeConcernErrors: [],
    insertedIds: [
      { index: 0, _id: ObjectId("61bafcedde146d867cab4e64") },
      { index: 1, _id: ObjectId("61bafcedde146d867cab4e65") }
    ],
    nInserted: 1,
    nUpserted: 0,
    nMatched: 0,
    nModified: 0,
    nRemoved: 0,
    upserted: []
  }
}
```
In the above code, employee details are added. As per the validators applied, all the fields are required and `phone_no` must be of type `int`. The first employee details have a phone number as a `string`. As a result, the validation failed. But since it was unordered, the second one was inserted successfully into the collection.

---

## 3. WriteConcern

During the insert operation, there can be multiple instances of the database running and that we can have huge commands waiting to be processed by the database. In such cases of high load, if the database goes down, all the queries waiting for execution will be lost.

This can be prevented by writing the queries to a journal first and then moving them to the storage engine for execution. This however will slightly impact the performance since the queries need to be written to a journal and then executed.

It can be used just like `ordered` i.e. `{writeConcern: {w: 1, j: true, timeout: 200}}`

Here `w` signifies the number of database instances running, `j` signifies whether the journal is to be used (j: true) or not (j: undefined) and `wtimeout` is the maximum time until which the write operation must be successful.

---

## 4. Importing data from a JSON file

To import data from a JSON file that contains an array of documents, we can use the `mongoimport` utility.

```
C:\>mongoimport <path-to-file> -d <db-name> -c <collection-name> --jsonArray --drop
2021-12-16T15:56:31.300+0530    connected to: mongodb://localhost/
2021-12-16T15:56:31.304+0530    dropping: moviesData.movies
2021-12-16T15:56:31.358+0530    240 document(s) imported successfully. 0 document(s) failed to import.
```
Here `--jsonArray` signifies that the file will contain an array of JSON objects and `--drop` indicates that the given collection be dropped first and then created again if it exists.

There are many more options that can be referred to from [the official documentation](https://docs.mongodb.com/database-tools/mongoimport/).

---

## 5. Atomicity

MongoDB provides atomicity on a `per-document` level. This means if any error occurs during the insertion of a document, the changes will be rolled back. But if the error occurs after the insertion, then the changes will not be rolled back (as in the case of `insertMany`).

---

This was all about creating documents and importing documents from a JSON file.

Thanks for reading!