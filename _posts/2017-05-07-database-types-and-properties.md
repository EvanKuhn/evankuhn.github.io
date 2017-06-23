---
layout: post
title: "Database Types and Properties"
author: Evan Kuhn
date: 2017-05-07 00:19:00 -0700
categories: databases
description: A short intro to database types, ACID vs BASE, and data normalization.
---

## Relational Database

Relational databases are based on the relational model of data, first proposed in 1970 by E. F. Codd [1].  They have the following properties:

- A relational database (RDBMS) consists of structured data stored in tables, much like a spreadsheet.  Data in different tables can be related to each other via key fields.

- RDBMS are "column-narrow" in that they typically store relatively few columns and many many rows [2].

- RDBMS also provide ACID guarantees (see below), which puts strict promises on queries with an emphasis on **consistency.**  However, this comes at a cost of complexity and ultimately price.

- RDBMS were not built for availability.  You'd need to architect that yourself.

- RDBMS systems were **built to scale up, not out.** They were originally intended to be run on large, monolithic mainframes, unlike the collections of commodity hardware we see today.  To scale out, you can do a few things [3]:

  1. Add **read slaves** to reduce load on the master.
  1. Add a **caching** layer, such as memcached, to further reduce read load.
  1. **Shard** your data by placing different subsets of data on different hosts. This must be handled at the application layer, which imposes a high development cost. It does, however, allow you to increase your storage space and read and write capacity.

Common relational databases: MySQL, SQL Server, Oracle.

## NoSQL / Object-Oriented Database

NoSQL databases have a number of properties that are different from relational databases:

- They do not store structured data in row-column form, but rather store **unstructured** or semi-structured data.  This data can take the form of scalar values, arrays, hashes, etc.

- NoSQL databases were built for availability **over consistency.**  This takes the form of BASE guarantees, rather than the ACID guarantees of relational databases.

- NoSQL databases are **eventually consistent.**  While they do not provide the consistency guarantees of relational DBs, they will eventually be consistent when input stops for some period of time.

- NoSQL databases are **built to scale out,** such that you can add a new node and the DB will automatically rebalance the data to the new nodes.

- NoSQL databases achieve scalability by relaxing the strict requirements of relational DBs.

NoSQL provides a few types of storage:

- **Key-value store**
  - These are simple and can be extremely performant, especially when stored in RAM.
  - Examples: Aerospike, Berkeley DB, MemchacheDB, Redis, Riak
- **Document database**
  - A document database stores documents that are parsed by the application.  A document may be XML, JSON, YAML, binary BSON, etc.
  - A document is retrieved by key.
  - A document database also may provide an API to search for documents by contents.
  - Examples: Couchbase, CouchDB, DocumentDB, MongoDB, RethinkDB
- **Graph database**
  - Designed for data whose relations are well represented as a graph consisting of elements interconnected with a finite number of relations between them. The type of data could be social relations, public transport links, road maps or network topologies. [4]

Common NoSQL databases: MongoDB, Cassandra, Redis.

## In-Memory Caches & Databases

Some databases (NoSQL in particular, maintain data in RAM), thus allowing for extremely high performance.  Some may store data fully in memory (with no disk backing), while others do persist data to disk.

## Transaction Guarantees: Acid vs. Base

### ACID

ACID is a set of promises made by relational databases that emphasize **consistency.**  Think of storing bank transactions: the bank would rather risk the database becoming unavailable rather than a incorrect data being stored or returned.  ACID stands for (from [5]):

- **Atomicity:** Either the task (or all tasks) within a transaction are performed or none of them are. This is the all-or-none principle. If one element of a transaction fails the entire transaction fails.

- **Consistency:** The transaction must meet all protocols or rules defined by the system at all times. The transaction does not violate those protocols and the database must remain in a consistent state at the beginning and end of a transaction; there are never any half-completed transactions.

- **Isolation:** No transaction has access to any other transaction that is in an intermediate or unfinished state. Thus, each transaction is independent unto itself. This is required for both performance and consistency of transactions within a database.

- **Durability:** Once the transaction is complete, it will persist as complete and cannot be undone; it will survive system failure, power loss and other types of system breakdowns.

In other words, tasks in an ACID database either run fully to completion in one logically atomic operation and are fully persisted, or they fail and have zero effect.

### BASE

While ACID databases emphasize consistency, BASE databases emphasize **availability.**  This follows from the CAP theorem: if we want our database to be highly available, we must choose partition tolerance (P), since all networks can fail.  That leaves a choice between availability (A) and consistency (C), so consistency is sacrificed.

BASE stands for (from [5]):

- **Basically Available:** This constraint states that the system does guarantee the availability of the data as regards CAP Theorem; there will be a response to any request. But, that response could still be ‘failure’ to obtain the requested data or the data may be in an inconsistent or changing state, much like waiting for a check to clear in your bank account.

- **Soft state:** The state of the system could change over time, so even during times without input there may be changes going on due to ‘eventual consistency,’ thus the state of the system is always ‘soft.’

- **Eventual consistency:** The system will eventually become consistent once it stops receiving input. The data will propagate to everywhere it should sooner or later, but the system will continue to receive input and is not checking the consistency of every transaction before it moves onto the next one. Werner Vogel’s article “Eventually Consistent – Revisited” covers this topic is much greater detail.

## Relational Database Normalization

Database normalization is process used to organize a database into tables and columns in order to:

- Minimize data duplication
- Maximize data integrity
- Avoid data modification issues
- Simplify queries

The following anomalies can occur on non-normalized databases:

Assume we have a table of employee + office data, with the employee id as the primary key:

![Sample employee database](http://www.essentialsql.com/wp-content/uploads/2014/06/Intro-Insert-Anomaly.png)

**Insert Anomaly:** occurs when we cannot insert some valid information because we do not have all info for a row.  For example, say we want to record a new office location.  We cannot insert that info until we have an employee at that office, since the primary key is the employee id.

**Update Anomaly:** occurs when the same data item is recorded in multiple rows.  In this case, updating that item in one row would require updating it in all rows, for consistency.

**Deletion Anomaly:** occurs when deleting one row causes multiple sets of data to disappear.  For example, if we delete our last New York employee from the database, we also lose the info for the NY office!

There are three (actually more) **normal forms** that databases adhere to when considering data normalization:

**First normal form (1NF)** sets the very basic rules for an organized database:
- Eliminate duplicative columns from the same table.
- Create separate tables for each group of related data and identify each row with a unique column or set of columns (the primary key).

**Second normal form (2NF)** further addresses the concept of removing duplicative data:
- Meet all the requirements of the first normal form.
- Remove subsets of data that apply to multiple rows of a table and place them in separate tables.
- Create relationships between these new tables and their predecessors through the use of foreign keys.

**Third normal form (3NF)** goes one large step further:
- Meet all the requirements of the second normal form.
- Remove columns that are not dependent upon the primary key.

## References

[1] ["Relational database."](https://en.wikipedia.org/wiki/Relational_database) Wikipedia.
[2] ["What are the different types of databases?"](https://www.quora.com/What-are-the-different-types-of-databases) Quora. <br/>
[3] ["Scaling your Website with Memcached, discussion with Northscale cofounder."](https://www.youtube.com/watch?v=uMxZ4RI6sCQ) YouTube. <br/>
[4] ["NoSQL."](https://en.wikipedia.org/wiki/NoSQL) Wikipedia. <br/>
[5] Roe, Charles. ["ACID vs. BASE: The Shifting pH of Database Transaction Processing."](http://www.dataversity.net/acid-vs-base-the-shifting-ph-of-database-transaction-processing/) <br/>
[6] ["Database Normalization."](https://en.wikipedia.org/wiki/Database_normalization) Wikipedia. <br/>
[7] Wenzel, Kris. ["DB Normalization and Design Explained in Simple English."](https://www.essentialsql.com/get-ready-to-learn-sql-database-normalization-explained-in-simple-english/) <br/>
[8] Chapple, Mike. ["Database Normalization Basics."](https://www.thoughtco.com/database-normalization-basics-1019735) <br/>
