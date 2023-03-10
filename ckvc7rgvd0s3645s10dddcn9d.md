---
title: "401 Unauthorized error when using CosmosDB REST APIs"
datePublished: Fri Oct 29 2021 10:09:27 GMT+0000 (Coordinated Universal Time)
cuid: ckvc7rgvd0s3645s10dddcn9d
slug: 401-unauthorized-error
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1635502041473/oeX7vWFinT.png
tags: authentication, nosql, azure, databases

---

Since Cosmos DB provides REST APIs to query their data, the process needs to be handled carefully while generating the authentication token. 

Though I won't go into the depth of creating an authentication token, if you're having this 401 Unauthorized error, there's a high chance that something is wrong with the string used to create the authentication token.

Let's look at the possible scenarios.

* **Create a new Document**: As you can see from the  [documentation](https://docs.microsoft.com/en-us/rest/api/cosmos-db/create-a-document), we don't need any `doc-id` or `doc-name` for this. So, the string used to generate auth token should be `dbs/{db-name}/colls/{coll-name}/docs`. And then it's simply a POST request.

* In all other cases when you require a `doc-id` or `doc-name`, the string will be different. In case of `doc-id`, the string will be `dbs/{db-name}/colls/{coll-name}/docs/{doc-id}` whereas in the case of `doc-name`, the string will be `doc-name` only.

**Note:** For APIs that require `doc-name` as mentioned in the documentation, you can also use `doc-id`.

A more detailed solution can be found in  [this stack overflow answer](https://stackoverflow.com/a/39068225/10308665).

I hope that it helps.



