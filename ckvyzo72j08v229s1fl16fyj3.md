---
title: "What is Caching?"
datePublished: Sun Nov 14 2021 08:41:39 GMT+0000 (Coordinated Universal Time)
cuid: ckvyzo72j08v229s1fl16fyj3
slug: what-is-caching
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1636878927800/KvP_9oyae.png
tags: software-development, databases, caching, software-engineering

---

Let's check out some fundamentals about caching!

### What is caching?

- Caching means storing some data in a fast-access memory which helps in reducing the load on the database and also prevents repeated computations.


### What are the types of caches available?

**1. Spatial Cache:**

- It is a locality-based cache.

- Fetches the data from the nearby blocks also, from where the data was fetched from the storage device.


**2. Temporal Cache:**

- It is a time-based cache.

- The least recently used data is thrown away.


**3. Distributed Cache:**

These are of two types:

-  *Write through cache:* any change to data is first made to cache and then to the disk/store.

-  *Write back cache:* Changes are written to the cache. Then cache writes the changes to the disk asynchronously.


### What are the problems with caching?

- Data Inconsistency is possible.

- Thrashing - Inserting and Deleting data from the cache without actually using it.

- Extra network calls in case of cache-hit.
---
Thanks for reading! 

Want to add something I missed? Reply in the comments!