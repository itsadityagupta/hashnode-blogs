---
title: "What are row/column-based databases?"
datePublished: Fri Oct 01 2021 04:09:35 GMT+0000 (Coordinated Universal Time)
cuid: cku7uku840cq7szs12cmxgm7f
slug: what-are-rowcolumn-based-databases
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1633060950085/_sYvJwcNq.png
tags: data, software-development, software-architecture, databases

---


### Row-based databases:

• These store data in a row-major fashion.

• For a given query, entire rows of records need to be read from the disk along with all the columns.

• Multiple rows along with matching columns are returned as a result of queries

• Hence, even if we require only a few columns, the entire record will be read.

• This results in heavy IO operations to disk and impacts performance.

### Column-based databases:

• These stores data in a column-major fashion.

• Every column is mapped to its corresponding data.

• Hence, only the required columns are read from the disk for a particular query.

• So multiple columns along with matching rows are returned. This is much more efficient as reads are done only for matched rows.

### Comparison:⚔️

• Since column-based databases map columns to other fields, any deletions or updations will have to go through every column and update the state there. Which results in unnecessary IO operations.

• If the query demands all the columns, then the column-based format is of no use as it will have to read from every column with separate I/O operations.

• Since every column is now mapped to other fields, the size of the database increases.

• Column-based databases use compression for duplicate values of the same columns which further enhances performance.

• Hence, the column-based format is used in Online Analytical Processing where analysis is done on a few of thousands of columns and for read-heavy operations.

• For row-based databases, they don't use additional storage.

• They don't have any compression.

• They have an optimal speed for both reading and write operations.

• They're easy to understand.

• Hence, these are used in Online Transaction Processing.

• Examples of column-based databases: Redshift, BigQuery, SnowFlake

• Examples of row-based databases: Postgres, MySQL


> Now use whatever suits your use case. I won't do that for you!😂

### References:

- https://dataschool.com/data-modeling-101/row-vs-column-oriented-databases


- https://www.youtube.com/watch?v=Vw1fCeD06YI


Did I miss anything? Add it in the comments!✍️

Thanks for reading!

