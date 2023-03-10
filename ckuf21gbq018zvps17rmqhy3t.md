---
title: "What is Database Indexing?"
datePublished: Wed Oct 06 2021 05:12:51 GMT+0000 (Coordinated Universal Time)
cuid: ckuf21gbq018zvps17rmqhy3t
slug: what-is-database-indexing
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1633496897583/U4oCsasea.png
tags: data, software-development, software-architecture, databases, software-engineering

---

#### 1. What is a database index?
• An index is a data structure, stored alongside a table, that helps in faster access of records from a table.

• It is created against a column and stores the location of the row for that column value.

#### 2. Why is an index required?
• An index helps in the faster access of data from the table. 

• As it stores the pointer to the table, the I/O operations required are reduced drastically, resulting in higher performance.

#### 3. What are some key features of indexes?
• It's used for read-heavy operations.

• When updating a row/column, an index associated with that needs to get updated as well, slowing the performance.

• The most popular data structures used for creating indexes are B-Trees and Hash Tables. (I won't go into specifics as these are well-known data structures)

• B-Trees are generally preferred due to their logarithmic time complexity of CRUD operations.

• Indexes are only used when querying an indexed column with a specific value. They can't be used with commands such as "like".

• An index for the primary key is created by default.

• Since indexes are stored alongside the table, they require additional storage. So, we need to create indexes carefully.

• An index for the primary key is created by default.

• Creating an index for a column that is read rarely or is updated frequently is a terrible idea.

#### References and further reads

• https://stackoverflow.com/questions/1108/how-does-database-indexing-work

• https://www.youtube.com/watch?v=T9n_-_oLrbM&list=PLQnljOFTspQXjD0HOzN7P2tgzu7scWpl2&index=8


Did I miss anything? Add it in the comments!