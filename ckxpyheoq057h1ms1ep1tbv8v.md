---
title: "Schema Validation in MongoDB"
datePublished: Tue Dec 28 2021 10:17:52 GMT+0000 (Coordinated Universal Time)
cuid: ckxpyheoq057h1ms1ep1tbv8v
slug: schema-validation-in-mongodb
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1639636859144/MnuE3muBa.png
tags: nosql, mongodb, databases

---

Though MongoDB offers no restriction on the schema of the documents, we may need some restrictions on it as per our use-cases. MongoDB provides validators for this specific purpose.

There are two validation levels:
1. **Strict (default):** It means to validate all the documents while inserting and updating.
2. **Moderate:** It means to validate all the documents while inserting; and to validate the documents that were valid before, during updates.

There are two validation actions that can be performed:
1. **Error (default):** Rejects the document to be inserted/updated if the validation fails.
2. **Warn:** Logs the validation error and allows the document to be inserted/updated.

Let's see an example:

Suppose we have an `employees` database. Let's create an `employeeDetails` collection with `name`, `email`, `phone_no` and `address` fields. The validation that needs to be applied is:

* All fields are required.
* `name` is a `string`.
* `email` is a `string`.
* `phone_no` is an `int`.
* `address` is a `string`.

Let's apply a validator for the above conditions and create a collection keeping the `validationLevel` as `moderate`:
```
employees> db.createCollection("employeeDetails", {validator: {
$jsonSchema: {
         bsonType: "object",
         required: ["name", "email", "phone_no", "address"],
         properties: {
             name: {
                 bsonType: "string",
                 description: "must be a string and is required."
             },
             email: {
                 bsonType: "string",
                 description: "must be a string and is required."
             },
             phone_no: {
                 bsonType: "int",
                 description:"must be a long and is required."
             },
             address: {
                 bsonType: "string",
                 description:"must be a string and is required."
             }
         }
     }
},
validationLevel: "moderate"})
output> { ok: 1 }
```
Now, let's insert employee details:
```
employees> db.employeeDetails.insertOne({
name: "Employee 1", 
email: "employee1@gmail.com", 
phone_no: 1234, 
address: "Address 1"})
output> {
  acknowledged: true,
  insertedId: ObjectId("61bae54e09662dcd9fc9d763")
}
```
Since these details are valid, they're inserted. Let's try one more time:
```
employees> db.employeeDetails.insertOne({
name: "Employee 2", 
email: "employee2@gmail.com", 
phone_no: "5678", 
address: "Address 2"})
output> Uncaught:
MongoServerError: Document failed validation
Additional information: {
  failingDocumentId: ObjectId("61bae56b09662dcd9fc9d764"),
  details: {
    operatorName: '$jsonSchema',
    schemaRulesNotSatisfied: [
      {
        operatorName: 'properties',
        propertiesNotSatisfied: [ { propertyName: 'phone_no', details: [ [Object] ] } ]
      }
    ]
  }
}
```
The validation failed this time because the `phone_no` provided was `string` but as per the validator, it should be `int`. Similarly, we can use different options with `validators`, `validationLevel` and `validationAction`. 

---
If you want to add a validator to an existing collection, we can do that with the help of `collMod` command with the `validator` option.

Check the  [official documentation](https://docs.mongodb.com/manual/core/schema-validation/)  for more details.

Thanks for reading!