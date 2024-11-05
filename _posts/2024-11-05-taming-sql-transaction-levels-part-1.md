---
title: Taming SQL Isolation Levels (Part 1 - Read Anomalies & Fixes)
description: Dive into SQL transaction isolation levels with real-world examples. Learn why your database queries return inconsistent results and how to fix read anomalies in PostgreSQL and SQLite.
categories: SQL Databases C# .NET 
featured_image: /assets/img/taming-sql-transactions-part-1/thumbnail.svg
image: /assets/img/taming-sql-transactions-part-1/thumbnail.svg
featured_image_alt: an image showing illustration of a database 
comments: true
---
When talking about SQL database transactions, the first thing that pops up next is usually ACID (or ORM, if you are lazy). 

Transaction and ACID combined give us (developers) guarantees and save us a lot of time and trouble as they do a lot of heavy lifting, like allowing rollback or the abort a set of operations (Atomicity) if a problem is encountered, ensuring that no data is loss (durability) and making sure that values in the database respect certain constraints and have some consistency (C).

And as with every abstraction, we need to understand its working, its quirks, and most importantly the cost that comes with it.

In my view, isolation (I) is an interesting one, especially for data manipulation in concurrent environments. And it’s up to us as developers to set what isolation level we need, that’s why it’s important to understand it more than the other letters (no discrimination intended, without the other letters I wouldn’t be able to write this article). 

## Database Isolation Levels

Isolation guarantees apply to parallel-running transactions, especially those that manipulate the same resource in the database (table or row, we are talking about SQL here). Isolation dictates what’s visible and what’s not from other ongoing transactions. 

Contrary to Atomicity, Consistency, and Durability, Isolation is non-binary, hence, the levels of isolation. 

In the SQL Standard (**ISO/IEC 9075)** we find four isolation levels, ordered from loose to strict: 

- READ UNCOMMITTED
- READ COMMITTED
- REPEATABLE READ
- SERIALIZABLE

These levels are the standard ones, database vendors don’t implement all of them, for example in SQLite we only have READ UNCOMMITTED and SERIALIZABLE and in PostgreSQL READ UNCOMMITTED acts like READ COMMITTED, so to explore each level we need to use two different databases.

To grasp them all, let’s create problems and try to solve them after a brief analysis. 

## Scenario 1: Dirty Reads and Dirty Writes

To simulate the dirtiness, let’s imagine we have a room booking service, where we have 2 use cases: 

- Use case 1: A user can rent a room and be invoiced.
- Use case 2: Get an inventory of the rooms booked and the invoices issued.

The database schema is pretty simple:

![Database Schema](/assets/img/taming-sql-transactions-part-1/database_schema.svg)

Four tables, one to hold user data, one to hold rooms, one to hold Invoices, and the last to hold bookings. 

To book a room we need three information : 

- The `userId`
- The `roomId`
- The date, start time, and end time of the booking.

The booking process would be to : 

1. Start a transaction
2. Check if the user exists
3. Check if the room exists
4. book the room
5. Invoice the user
6. End Transaction

The interaction with the database is summarized in the following diagram (you can zoom, it’s SVG 😉):

![A diagram showing interaction to book a room](/assets/img/taming-sql-transactions-part-1/interaction.svg)

Now, to have the inventory, we need to get all Bookings and all Invoices and return a result. The interaction is the following : 

![A diagram showing interaction to get the inventory](/assets/img/taming-sql-transactions-part-1/inventory_interactin.svg)

The inventory is just a check on the current state of the business, by getting all the bookings and the invoices. 

For this first scenario, I will use the loosest isolation level, READ UNCOMMITTED, we will try to identify the problems that can occur and the solution to it. 

READ UNCOMMITTED transactions, as the name suggests, will read values from other transactions even if they didn’t commit yet, for example, in the following interaction the inventory transaction will read an intermediary state : 

![Diagram showing the issue create by read uncommitted](/assets/img/taming-sql-transactions-part-1/read_uncommitted.svg)

 In the scenario above, the inventory transaction will have, **a single invoice** and no **booking**. It shows the system in an inconsistent state, this phenomenon is called a “dirty read”.  

In some scenarios, this might be all right, but in others, it may cause troubles and false positives in the business, we invoiced a user without booking their room. 

Theoretically, There is also another issue that we might encounter with the read uncommitted is the  “dirty write”.

Similar to the dirty read, dirty writes happen when transactions step on each other and override each other values even before they commit. At the end of the transactions, we can’t know the state of the database.  

I say theoretically because some vendors prevent it even if you run the transaction with the read uncommitted isolation. 

### Running the code

To implement read uncommitted, I had to use SQLite and active it, using : 

```sql
PRAGMA read_uncommitted = ON;
```

In the response to the inventory, I added a boolean to say if inventory is consistent or not, the number of bookings is equal to the number of invoices. To know if we had a dirty read or not. 

To simulate parallel requests, I used [k6](https://k6.io/) and used the [checks](https://grafana.com/docs/k6/latest/using-k6/checks/) to tell if the inventory was consistent or not.

The script to book rooms will simulate 100 users booking rooms for 70 seconds : 

```sql
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

The script to get the inventory will simulate a single user getting the inventory for 70s : 

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

Although this benchmark isn’t a real benchmark as it lacks many benchmarking practices, we capture the results anyway. 

1. The booking load test result:

| Number of requests | Failed Requests | Avg request duration | p(90) request duration | median duration | number of users  | total test duration |
| --- | --- | --- | --- | --- | --- | --- |
| 23288 | 0 | 217.2ms | 590.29ms | 48.55ms | 100 | 1min 10s |
1. The inventory load test result:

| Number of requests | Failed Requests | Avg request duration | p(90) request duration | median duration | number of users  | total test duration |
| --- | --- | --- | --- | --- | --- | --- |
| 71 | 0 | 983.28ms | 2.5s | 681.24ms | 1 | 1min 10s |

And in the 71 requests we got 33% of them with valid inventory (24 right and 47 wrong)

### Solutions:

From a business perspective, sometimes it’s all right to have intermediary non-valid results of such transactions, or we can run the inventory outside of business hours to avoid having parallel writing requests. 

In case we need to fix this technically, we can upgrade our isolation level to READ COMMITTED. 

### Applying READ COMMITTED isolation level

Unfortunately, SQLite doesn’t have the level READ COMMITTED, to fix that, I’ll use PostgreSQL. This means we can’t compare the results with previous run as the environment has changed (especially the database)

Switching the isolation level and running the same scenario. 

1. The booking load test result:

| Number of requests | Failed Requests | Avg request duration | p(90) request duration | median duration | number of users  | total test duration |
| --- | --- | --- | --- | --- | --- | --- |
| 36902 | 0 | 135.54ms | 223.51ms | 130.23ms | 100 | 1min 10s |
1. The inventory load test result:

| Number of requests | Failed Requests | Avg request duration | p(90) request duration | median duration | number of users  | total test duration |
| --- | --- | --- | --- | --- | --- | --- |
| 200 | 0 | 340.98ms | 661.54ms | 267.27ms | 1 | 1min 10s |

And of the 200 requests, we got 5% of them with valid inventory (10 right and 190 wrong) 🤕🤕🤕 

I promised you to fix the problem, but I made it worse (I was as surprised as you the first time I encountered the numbers)

What happened?

Well, the read committed doesn’t protect us from another concurrent reads’ anomaly, called the non-repeatable reads, in our case, it happens when the inventory transaction reads one part before a booking transaction ends, so it reads the current committed value and reads the other part after the booking transaction ends, so it reads the new committed value. It’s clearer with an illustration. 

Let’s suppose that the database has 1 invoice and 1 booking before the transaction in the figure happens.

![Diagram showing the issue when we have read committed](/assets/img/taming-sql-transactions-part-1/non_repeatable_reads.svg)

So at the unfortunate moments 1 and 2, the committed values are different, at (1) in the invoices we only have 1 invoice, and in (2) we have 2 bookings, that’s a discrepancy. 

And the more concurrent transactions on the same resources we have, the more we are susceptible to falling into this inconsistency.

This phenomenon is called read skew or “non-repeatable read” because if you read again the database just after point 2, you will get the correct state of the database. 

This can be not an issue, for instance, if the user gets a non-repeatable read they can refresh and they will get the correct read but when this happen in backups or when for example we are creating reports this can be problematic. 

The solution is another level up of the isolation level to the next one, which is REPEATABLE READ or SNAPSHOT Isolation (it’s called differently from one vendor to another). 

### Using The REPEATABLE READ / SNAPSHOT Isolation

With the snapshot isolation, the database will take a snapshot (you can see it as a picture) of the state of the tables when the transaction starts, and all reads from that transaction will read from that picture. Rather than locking tables, [PostgreSQL implements a called named Multi-Version Concurrency Control](https://www.postgresql.org/docs/current/mvcc-intro.html) (MVCC) this advantageous since reading never blocks writing and writing never blocks reading.

By using snapshot isolation we get : 

1. The booking load test result:

| Number of requests | Failed Requests | Avg request duration | p(90) request duration | median duration | number of users  | total test duration |
| --- | --- | --- | --- | --- | --- | --- |
| 37241 | 0 | 134.28ms | 249.13ms | 122.83ms | 100 | 1min 10s |
1. The inventory load test result:

| Number of requests | Failed Requests | Avg request duration | p(90) request duration | median duration | number of users  | total test duration |
| --- | --- | --- | --- | --- | --- | --- |
| 193 | 0 | 354.25ms | 645.43ms | 332.06ms | 1 | 1min 10s |

And of the 193 requests, we got 100% of them with valid inventory. 

We are happy, aren’t we? 

So, using the snapshot isolation level we have technically resolved read skews, at least for our booking inventory scenario. 

If you were an attentive reader, you would have noticed that our booking system doesn’t check for overlapping bookings and in a real life scenario we need to implement such verification but with it other problem. 

The kind of problems we solved here are problems that rose from write transactions getting in the way of read transactions, the next one that we will look at are ‘write’ transactions getting in the way of each others. 

Let’s sum up. 

## Summary

In this article, we went through the different issues that might arise when we have concurrent access to the same database resources, in particular read transactions getting inconcsistent results because of parallel running write transaction. We analyzed their occurence frequency and the potential impact on the business. Finally, we explored the isolation levels provided by SQLite and PostgreSQL that could help us solve each problem. 

In the next part we will discuss write/write transaction concurrency issue and we will also see how they impact business results and how to solve them using yet another isolation level.