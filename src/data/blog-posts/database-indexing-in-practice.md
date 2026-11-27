---
title: How Much Difference Can a Database Index Make?
slug: how-much-difference-can-a-database-index-make
publishDate: 26 November 2026
description: An experiment with PostgreSQL 16, two million synthetic transaction records, and BankPerf Lab to investigate the impact of composite B-tree indexing on query performance.
---

## The curiosity behind the experiment

I've always found database performance interesting, especially the things that happen beneath the surface when a database has to process large volumes of data. It's one thing to understand concepts such as indexing, query optimisation and execution plans theoretically; it's another to observe how they influence the behaviour of an actual database.

Recently, i decided to explore this more practically through a personal project i'm building called **BankPerf Lab**. The idea is to create a controlled environment where I can experiment with database engineering, performance testing, optimisation and other aspects of systems engineering.

For my first investigation, I wanted to start with something fundamental but important: **How much difference can a database index actually make?**

---

## The problem

Consider a banking application retrieving the most recent transactions associated with a customer's account. As transaction volumes grow, finding a relatively small number of records within a large transaction table can become expensive, especially when the database has to examine a substantial portion of that table before finding the records it needs.

I wanted to understand what happens when PostgreSQL encounters this kind of query and, more importantly, whether a suitably designed index could make the operation more efficient.

To investigate this, i created a PostgreSQL database with approximately 2.35 million synthetic records across six tables such as branches, customers, transaction channels, accounts, transactions and audit logs. The `transactions` table was the largest, with about two million records, which makes it a suitable starting point for the experiment.

My hypothesis was that a composite B-tree index on `(account_id, created_at DESC)` would significantly reduce the execution time of an account transaction-history query by helping PostgreSQL locate the relevant records while also satisfying the requested ordering.

---

## Inside BankPerf Lab

I built the environment locally on my Apple MacBook Pro M4, using Docker Compose to provision PostgreSQL 16.16. The dataset was designed around a simplified banking domain, which allows me to investigate transaction-related queries against a reasonably substantial volume of synthetic data without relying on actual banking records.

With the environment ready, i created the following composite index on the `transactions` table:

```sql
CREATE INDEX idx_transactions_account_created
ON transactions (account_id, created_at DESC);
```

The column arrangement was deliberate. The first column, `account_id`, supports filtering transactions by account, while the second, `created_at DESC`, aligns with the requirement to retrieve the most recent transactions first. In other words, i wanted one index to support both the filtering and ordering requirements of the query.

The next step was to find out whether PostgreSQL would actually take advantage of it.

---

## Putting the hypothesis to the test

I selected a query that retrieves the 20 most recent transactions for a particular account:

```sql
SELECT
    transaction_id,
    transaction_ref,
    transaction_type,
    amount,
    status,
    created_at
FROM transactions
WHERE account_id = 25025
ORDER BY created_at DESC
LIMIT 20;
```

Rather than relying solely on the time taken to execute the query, i used PostgreSQL's `EXPLAIN (ANALYZE, BUFFERS)` to examine the execution plan alongside runtime and buffer statistics. This would help me understand not just whether the query became faster, but what PostgreSQL was doing differently.

I tested the same query under two conditions. First, i executed it with the composite index available. I then temporarily removed the index and executed the query again. To avoid permanently changing the database structure, i performed the second test inside a database transaction and used `ROLLBACK` afterwards to restore the original state.

This gave me two execution conditions to compare: the same query, against the same dataset, with a different index-availability state.

---

## Then came the results...

![PostgreSQL indexing performance comparison](/personal-blog/images/indexing-performance.png)

*Figure 1: Observed execution time with and without the composite B-tree index.*

The difference was quite striking. The first execution with the index available completed in **0.150 milliseconds**, while the execution without the index took **581.503 milliseconds**. That's an observed execution-time ratio of approximately **3,877×**.

However, the execution time was only part of the story. When i examined the execution plans, i found that PostgreSQL had selected entirely different strategies depending on whether the index was available.

With the index in place, PostgreSQL used an `Index Scan` on `idx_transactions_account_created`. Because the index supported both the account filter and the requested descending chronological order, PostgreSQL did not need to introduce a separate sorting operation.

Without the index, however, PostgreSQL resorted to a `Parallel Sequential Scan`, followed by a `Sort` and `Gather Merge`. It launched two parallel workers and examined a substantial portion of the transaction table to identify the matching records. Interestingly, despite the query's `LIMIT 20`, only 13 transactions matched the specified account.

The buffer statistics reinforced the difference in the amount of database activity involved:

| Measurement | With index | Without index |
|---|---:|---:|
| Shared buffer hits | 19 | 3,883 |
| Shared buffer reads | 0 reported | 34,617 |

I was also curious about the physical structure supporting the indexed execution, so i went a step further and inspected the B-tree using PostgreSQL's `pgstatindex()` function. The index occupied approximately 60 MB and had three tree levels, 7,663 leaf pages and an average leaf density of 90.03%.

This was interesting because it allowed me to look beyond the execution plan and examine the actual structure PostgreSQL maintained to support the query.

---

## The trade-offs and a few lessons

Of course, indexes aren't free. While they can improve read performance, they also consume storage and introduce additional maintenance overhead whenever records are inserted, updated or deleted. Creating indexes indiscriminately can therefore increase storage requirements and affect write performance.

There's also an important qualification to my results. This was an initial experiment rather than a statistically controlled benchmark. i performed two indexed executions and one non-indexed execution, without standardising the buffer-cache state between the tests. The indexed executions reported no shared buffer reads, whereas the non-indexed execution reported 34,617.

Consequently, the approximately 3,877× difference should be treated as an observation from this particular experiment, not a universal performance guarantee. A more comprehensive investigation would involve repeated executions, controlled cache conditions, different transaction volumes and concurrent workloads.

Nevertheless, the experiment reinforced something important for me: **database optimisation should be evidence-driven**. Understanding indexes theoretically is useful, but examining execution plans, buffer activity and physical index statistics provides a much deeper understanding of why a query behaves the way it does. It also helps distinguish between an optimisation that looks good on paper and one that actually makes a measurable difference.

---

## What's next?

This is the first investigation in the BankPerf Lab series, and i intend to build on it by exploring more areas of database engineering and performance optimisation. Instead of stopping at what a database does, i want to understand the internal mechanisms that influence how it behaves and how those mechanisms affect real-world systems.

The next investigation will explore **B-tree versus BRIN indexing in PostgreSQL**, looking at their structural differences, performance characteristics and the circumstances in which each might be appropriate.

For now, this first experiment has been a useful reminder that sometimes the best way to understand a system is to build something, formulate a question, test an assumption and investigate the results.

And that's exactly what i intend to keep doing with this Lab.