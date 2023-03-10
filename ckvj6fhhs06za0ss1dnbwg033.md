---
title: "Projection in MongoDB"
datePublished: Wed Nov 03 2021 07:06:31 GMT+0000 (Coordinated Universal Time)
cuid: ckvj6fhhs06za0ss1dnbwg033
slug: projection-in-mongodb
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1635856214455/9itfA12_4.png
tags: nosql, mongodb, databases

---

Projections in MongoDB are a way to transfer only the required data from a MongoDB server, rather than fetching the entire data and then filtering it.

This helps in reducing the load to the network and the backend applications, along with a faster data transfer.

For example, we have the below database:
```
hospital> db.patients.find()
output> [
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
  },
  {
    _id: ObjectId("6181317f0802aea7b1828646"),
    firstname: 'Mark',
    lastname: 'Manson',
    age: 50,
    history: [ { disease: 'fever', treatment: 45 } ]
  }
]
```
Suppose, we want the names of people older than 30. Then, we can do:
```
hospital> db.patients.find({age: {$gt: 30}}, {firstname: 1})
output> [
  { _id: ObjectId("61780b10d9a6a5af8bddb299"), firstname: 'Robin' },
  { _id: ObjectId("6181317f0802aea7b1828646"), firstname: 'Mark' }
]
```
As you can notice, in the second parameter `firstname: 1` means to include only the firstname in the result. But `_id` is also there.

This is because `_id` is by default present in the output of a projection and needs to be explicitly excluded.
```
hospital> db.patients.find({age: {$gt: 30}}, {firstname: 1, _id: 0})
output> [ { firstname: 'Robin' }, { firstname: 'Mark' } ]
```
Another example, if we need patients having `cold` as one of the diseases:
```
hospital> db.patients.find({"history.disease": "cold"}, {firstname: 1, _id: 0})
output> [ { firstname: 'Aditya' } ]
```

The `find` function first filters the data and then filters the columns ***on the server itself*** before sending the data to the backend application through the network.

The above examples are just a brief introduction to the capabilities of projections. To dive deeper into the topic, refer to the  [official documentation](https://docs.mongodb.com/manual/reference/operator/projection/positional/).

Thanks for reading.