---
title: "Taming SQL Isolation Levels (Part 1: Read Anomalies & Fixes)"
seo_title: "Understanding SQL Transaction Isolation Levels: A Guide to Read Anomalies | Part 1"
description: "Learn how to fix inconsistent database reads with SQL isolation levels. See real examples of dirty reads and non-repeatable reads with PostgreSQL and SQLite, plus performance benchmarks."
categories: [SQL, Databases, C#, .NET, Performance]
keywords: [sql isolation levels, database transactions, dirty reads, non-repeatable reads, postgresql, sqlite, database performance, acid properties, transaction isolation]
featured_image: /assets/img/taming-sql-transactions-part-1/thumbnail.svg
image: /assets/img/taming-sql-transactions-part-1/thumbnail.svg
featured_image_alt: "SQL Transaction Isolation Levels - database consistency illustration"
comments: true
---
When discussing SQL database transactions, ACID properties are typically the first concept that comes to mind (or ORM, for those seeking abstraction). 

Transactions and ACID properties together provide developers with guarantees and handle much of the complexity, such as enabling rollbacks for sets of operations (Atomicity), ensuring data persistence (Durability), and maintaining database constraints (Consistency).

And as with every abstraction, we need to understand its working, its quirks, and most importantly the cost that comes with it.

In my view, isolation (I) is an interesting one, especially for data manipulation in concurrent environments. And it's up to us as developers to set what isolation level we need, that's why it's important to understand it more than the other letters (no discrimination intended, without the other letters I wouldn't be able to write this article). 

## What Are Database Isolation Levels? A Quick Guide

Isolation guarantees apply to concurrent transactions, particularly those that manipulate the same database resources (tables or rows in SQL databases). Isolation dictates what's visible and what's not from other ongoing transactions. 

Contrary to Atomicity, Consistency, and Durability, Isolation is non-binary, hence, the levels of isolation. 

In the SQL Standard (ISO/IEC 9075) we find four isolation levels, ordered from loose to strict: 

- READ UNCOMMITTED
- READ COMMITTED
- REPEATABLE READ
- SERIALIZABLE

These levels are the standard ones, database vendors don't implement all of them, for example in SQLite we only have READ UNCOMMITTED and SERIALIZABLE and in PostgreSQL READ UNCOMMITTED acts like READ COMMITTED, so to explore each level we need to use two different databases.

## Real-World Example: Building a Room Booking System

To demonstrate isolation issues, let's consider a room booking service with two primary use cases:

- Use case 1: A user can rent a room and be invoiced.
- Use case 2: Get an inventory of the rooms booked and the invoices issued.

### The Database Schema

The database schema is pretty simple:

![Database Schema](/assets/img/taming-sql-transactions-part-1/database_schema.svg)

Four tables, one to hold user data, one to hold rooms, one to hold Invoices, and the last to hold bookings. 

### How Bookings Work

To book a room, we need three pieces of information:

- The `userId`
- The `roomId`
- The date, start time, and end time of the booking.

The booking process would be:

1. Start a transaction
2. Check if the user exists
3. Check if the room exists
4. Book the room
5. Invoice the user
6. End Transaction

The interaction with the database is summarized in the following diagram:

![A diagram showing interaction to book a room](/assets/img/taming-sql-transactions-part-1/interaction.svg)

### The Inventory Problem

Now, to have the inventory, we need to get all Bookings and all Invoices and return a result. The interaction is the following:

![A diagram showing interaction to get the inventory](/assets/img/taming-sql-transactions-part-1/inventory_interactin.svg)

The inventory is just a check on the current state of the business, by getting all the bookings and the invoices.

## Investigating Isolation Issues

### Problem 1: Dirty Reads with READ UNCOMMITTED

For this first scenario, I will use the loosest isolation level, READ UNCOMMITTED. We will try to identify the problems that can occur and their solutions.

READ UNCOMMITTED transactions, as the name suggests, will read values from other transactions even if they didn't commit yet. For example, in the following interaction, the inventory transaction will read an intermediary state:

![Diagram showing the issue create by read uncommitted](/assets/img/taming-sql-transactions-part-1/read_uncommitted.svg)

In the scenario above, the inventory transaction will have **a single invoice** and no **booking**. It shows the system in an inconsistent state, this phenomenon is called a "dirty read".

In some scenarios, this might be acceptable, but in others, it may cause troubles and false positives in the business - we invoiced a user without booking their room.

Theoretically, there is also another issue that we might encounter with READ UNCOMMITTED called "dirty write". Similar to the dirty read, dirty writes happen when transactions step on each other and override each other's values even before they commit. At the end of the transactions, we can't know the state of the database.

I say theoretically because some vendors prevent it even if you run the transaction with READ UNCOMMITTED isolation.

### Testing READ UNCOMMITTED in Action

To implement READ UNCOMMITTED, I had to use SQLite and activate it using:

```sql
PRAGMA read_uncommitted = ON;
```

In the response to the inventory, I added a boolean to indicate if the inventory is consistent (number of bookings equals number of invoices) to detect dirty reads.

To simulate parallel requests, I used [k6](https://k6.io/) and its [checks](https://grafana.com/docs/k6/latest/using-k6/checks/) feature to verify inventory consistency.

The script to book rooms simulates 100 users booking rooms for 70 seconds:

```js
import http from 'k6/http';

//  k6 run .\book_lt.js

export const options = {
   stages: [
       {duration: '20s', target: 100},
       {duration: '30s', target: 100},
       {duration: '20s', target: 0},
   ],
};

export default function () {
   let url = 'http://localhost:5130/v1/rooms/bookings';
   let body = JSON.stringify({
       "userId": "9A841211-47E1-4DE1-8628-9EFB9E811162",
       "roomId": "CEE3BBCD-BD83-4175-B30B-8233D26FFDDF",
       "date": "2024-10-26",
       "start": "08:00:00",
       "end": "10:00:00"
   });
   const params = {
       headers: {
           'Content-Type': 'application/json',
       },
   };
   let response = http.post(url, body, params);
}
```

The script to get the inventory simulates a single user querying the inventory for 70 seconds:

```sql
import http from 'k6/http';
import {check } from 'k6';

// k6 run --address "localhost:3000" .\inventory_lt.js

export const options = {
   stages: [
       { duration: '1s', target: 1 },
       { duration: '69s', target: 1 },
   ],
};

export default function () {
   let res = http.get('https://localhost:7191/v1/inventory');
   
   check(res, {
       'is valid inventory': (r) => r.body.includes('true')
   })
}
```

While this benchmark doesn't follow all standard benchmarking practices, the results provide valuable insights:

1. The booking load test results:

| Number of requests | Failed Requests | Avg request duration | p(90) request duration | median duration | number of users  | total test duration |
| --- | --- | --- | --- | --- | --- | --- |
| 23288 | 0 | 217.2ms | 590.29ms | 48.55ms | 100 | 1min 10s |

2. The inventory load test results:

| Number of requests | Failed Requests | Avg request duration | p(90) request duration | median duration | number of users  | total test duration |
| --- | --- | --- | --- | --- | --- | --- |
| 71 | 0 | 983.28ms | 2.5s | 681.24ms | 1 | 1min 10s |

Of the 71 inventory requests, only 33% showed valid inventory (24 correct vs 47 incorrect).

## Solving Isolation Problems

### Solution 1: Moving to READ COMMITTED

From a business perspective, intermediate inconsistent results may be acceptable in some cases. Alternatively, inventory operations can be scheduled outside business hours to minimize concurrent write operations.

For technical solutions, we can upgrade our isolation level to READ COMMITTED.

### Why READ COMMITTED Isn't Enough

Since SQLite doesn't support READ COMMITTED, we'll switch to PostgreSQL. Note that this means we cannot directly compare results with the previous run due to the different environment.

After switching the isolation level and running the same scenario:

1. The booking load test results:

| Number of requests | Failed Requests | Avg request duration | p(90) request duration | median duration | number of users  | total test duration |
| --- | --- | --- | --- | --- | --- | --- |
| 36902 | 0 | 135.54ms | 223.51ms | 130.23ms | 100 | 1min 10s |

2. The inventory load test results:

| Number of requests | Failed Requests | Avg request duration | p(90) request duration | median duration | number of users  | total test duration |
| --- | --- | --- | --- | --- | --- | --- |
| 200 | 0 | 340.98ms | 661.54ms | 267.27ms | 1 | 1min 10s |

Surprisingly, of the 200 requests, only 5% showed valid inventory (10 correct vs 190 incorrect).

What happened?

READ COMMITTED doesn't protect us from another concurrent read anomaly called non-repeatable reads. In our case, this occurs when the inventory transaction reads one part before a booking transaction ends (reading the current committed value) and reads the other part after the booking transaction ends (reading the new committed value).

Let's suppose the database has 1 invoice and 1 booking before the transactions in the figure occur:

![Diagram showing the issue when we have read committed](/assets/img/taming-sql-transactions-part-1/non_repeatable_reads.svg)

At moments 1 and 2, the committed values differ. At point (1), we have only 1 invoice, while at point (2), we have 2 bookings - creating a discrepancy.

The more concurrent transactions on the same resources, the more likely we are to encounter this inconsistency.

This phenomenon is called read skew or "non-repeatable read" because reading the database again just after point 2 would show the correct state.

While this might not be problematic for user interfaces where a refresh can resolve the inconsistency, it can cause serious issues for operations like backups or report generation.

### Solution 2: Using REPEATABLE READ for Consistency

The solution is to upgrade to the next isolation level: REPEATABLE READ or SNAPSHOT isolation (terminology varies by vendor).

With snapshot isolation, the database captures a snapshot of the table states when the transaction begins, and all reads within that transaction use that snapshot. Rather than locking tables, [PostgreSQL implements Multi-Version Concurrency Control](https://www.postgresql.org/docs/current/mvcc-intro.html) (MVCC). This approach is advantageous since reading operations never block writing operations and vice versa.

Using snapshot isolation, we get:

1. The booking load test results:

| Number of requests | Failed Requests | Avg request duration | p(90) request duration | median duration | number of users  | total test duration |
| --- | --- | --- | --- | --- | --- | --- |
| 37241 | 0 | 134.28ms | 249.13ms | 122.83ms | 100 | 1min 10s |

2. The inventory load test results:

| Number of requests | Failed Requests | Avg request duration | p(90) request duration | median duration | number of users  | total test duration |
| --- | --- | --- | --- | --- | --- | --- |
| 193 | 0 | 354.25ms | 645.43ms | 332.06ms | 1 | 1min 10s |

Of the 193 requests, 100% showed valid inventory.

## Next Steps: Write Conflicts and SERIALIZABLE Isolation

While this resolves our immediate concerns, it's worth noting that our booking system doesn't check for overlapping bookings. In a real-world scenario, we would need to implement such verification, which introduces additional challenges.

The problems we've solved here arise from write transactions interfering with read transactions. In the next article, we'll examine issues that occur when write transactions interfere with each other.

## Resources and Further Reading

We've explored various issues that can arise from concurrent access to database resources, particularly how read transactions can receive inconsistent results due to parallel write transactions. We analyzed their frequency of occurrence and potential business impact. Finally, we examined how different isolation levels in SQLite and PostgreSQL can help solve these problems.

In the next part, we will discuss write/write transaction concurrency issues and explore solutions using additional isolation levels.