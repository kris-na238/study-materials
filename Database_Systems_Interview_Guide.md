# Database Systems Interview Guide (MySQL, PostgreSQL, MongoDB)

## How to use this doc
This is a self-contained interview/revision reference — you shouldn't need to search
anything else while prepping. It covers the 25 most frequently asked Database Systems
interview topics across **MySQL**, **PostgreSQL**, and **MongoDB**, with 10 Q&A per topic
written at a senior-engineer depth. Each answer is written the way you'd actually say it
out loud in an interview. Skim the Rapid-fire Recap at the end the night before an interview;
read topic-by-topic when you have more time.

## Table of Contents
1. [ACID Transactions & Relational Fundamentals](#1-acid-transactions--relational-fundamentals)
2. [Indexing (B-Tree, Hash, Composite, Covering)](#2-indexing-b-tree-hash-composite-covering)
3. [Normalization & Denormalization](#3-normalization--denormalization)
4. [Isolation Levels & MVCC](#4-isolation-levels--mvcc)
5. [Locking & Concurrency Control](#5-locking--concurrency-control)
6. [Query Optimization & EXPLAIN Plans](#6-query-optimization--explain-plans)
7. [Joins — Types & Performance](#7-joins--types--performance)
8. [MySQL Storage Engines (InnoDB Internals)](#8-mysql-storage-engines-innodb-internals)
9. [PostgreSQL Architecture (WAL, VACUUM, MVCC)](#9-postgresql-architecture-wal-vacuum-mvcc)
10. [MongoDB Document Model & Schema Design](#10-mongodb-document-model--schema-design)
11. [MongoDB Aggregation Framework & Indexing](#11-mongodb-aggregation-framework--indexing)
12. [Replication (MySQL / PostgreSQL / MongoDB)](#12-replication-mysql--postgresql--mongodb)
13. [Sharding & Partitioning](#13-sharding--partitioning)
14. [CAP Theorem & Consistency Models](#14-cap-theorem--consistency-models)
15. [SQL vs NoSQL — Choosing the Right Database](#15-sql-vs-nosql--choosing-the-right-database)
16. [Scaling Databases (Read Replicas, Pooling, Vertical/Horizontal)](#16-scaling-databases-read-replicas-pooling-verticalhorizontal)
17. [Caching Strategies with Databases](#17-caching-strategies-with-databases)
18. [Schema Migrations & Evolution](#18-schema-migrations--evolution)
19. [Backup, Recovery & High Availability](#19-backup-recovery--high-availability)
20. [Deadlocks & Performance Troubleshooting](#20-deadlocks--performance-troubleshooting)
21. [Stored Procedures, Triggers & Views](#21-stored-procedures-triggers--views)
22. [JSON & Full-Text Search in Relational DBs](#22-json--full-text-search-in-relational-dbs)
23. [Database Security (SQL Injection, Roles, Encryption)](#23-database-security-sql-injection-roles-encryption)
24. [Distributed Transactions (2PC, Saga, Outbox)](#24-distributed-transactions-2pc-saga-outbox)
25. [Data Modeling: Embedding vs Referencing (Mongo) & Relational Design](#25-data-modeling-embedding-vs-referencing-mongo--relational-design)

---

## 1. ACID Transactions & Relational Fundamentals

**Primer:** ACID (**A**tomicity, **C**onsistency, **I**solation, **D**urability) is the
contract a relational DB gives you for a transaction — a group of operations that must
succeed or fail together. It's the foundation everything else (locking, isolation levels,
replication guarantees) is built on top of.

### Q1. What does each letter in ACID actually guarantee?
- **Atomicity**: all statements in a transaction commit or none do (`BEGIN...COMMIT/ROLLBACK`). A crash mid-transaction leaves no partial writes.
- **Consistency**: the DB moves from one valid state to another — constraints, foreign keys, triggers all still hold after commit. This is more "the app + DB together enforce invariants" than a DB-only guarantee.
- **Isolation**: concurrent transactions don't see each other's uncommitted changes (governed by isolation level, see Topic 4).
- **Durability**: once committed, data survives a crash — achieved via write-ahead logging (WAL in Postgres, redo log in InnoDB) flushed to disk before acknowledging commit.
- Interviewer may follow up with: "which of these is hardest to guarantee at scale?" → Consistency + Isolation, because they trade off directly against throughput.

### Q2. Why do we say "consistency" is partly an application responsibility, not just the DB's?
The DB enforces schema-level consistency (NOT NULL, foreign keys, unique constraints, check constraints). But business-level consistency — e.g. "account balance can never go negative" — is only guaranteed if your application wraps the right statements in a transaction and adds the right constraint. The DB can't infer business rules you didn't declare. Example: transferring money between two rows needs both the debit and credit in one transaction *and* a `CHECK (balance >= 0)` constraint — the transaction alone isn't enough.

### Q3. What's the difference between a statement and a transaction in terms of atomicity guarantees?
A single SQL statement is always atomic by default (even without `BEGIN`) — MySQL/Postgres auto-wrap it. A transaction is a group of statements you explicitly bound with `BEGIN...COMMIT` so the *group* becomes the atomic unit. Example:
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```
If the process dies after the first UPDATE, the transaction log lets the DB roll back on recovery — the second UPDATE never becomes visible.

### Q4. How is durability actually implemented under the hood?
Via **write-ahead logging (WAL)**: before a data page is modified on disk, the change is first written to a sequential append-only log and fsynced. On commit, the DB only needs the WAL record durable, not the actual data page (that can be flushed later, lazily, by a background checkpointer). On crash, recovery replays the WAL from the last checkpoint. This is why WAL/redo log tuning (`innodb_flush_log_at_trx_commit`, Postgres `synchronous_commit`) is the main durability-vs-throughput knob.

### Q5. What happens if you don't use transactions at all — what actually breaks?
Multi-statement operations become non-atomic: a crash or concurrent read between statements can observe or persist a half-done state (e.g., money debited but not credited). You also lose isolation guarantees — without a transaction boundary, each statement commits and becomes visible to other connections immediately, so a concurrent read can see intermediate states across your "logical" operation even if each individual statement is atomic.

### Q6. What is a "dirty write" and why is it the one anomaly no isolation level allows?
A dirty write is one transaction overwriting another transaction's *uncommitted* change. Every SQL isolation level, even the weakest (Read Uncommitted), prevents this — because if allowed, a rollback of one transaction would have to also silently revert data another transaction believes it legitimately wrote, making rollback semantics undefined. Contrast with a **dirty read** (reading uncommitted data), which Read Uncommitted *does* allow.

### Q7. Explain atomicity in the context of a multi-row `UPDATE` — is it all-or-nothing per row or per statement?
Per statement (and further, per transaction if wrapped). `UPDATE orders SET status='shipped' WHERE warehouse_id=3` either updates all matching rows or none — if row 500 out of 10,000 violates a constraint, the whole statement rolls back, not just row 500 onward. This surprises people who assume it processes row-by-row and stops.

### Q8. What's a savepoint and when would you use one?
A savepoint is a named point inside a transaction you can roll back to without aborting the whole transaction.
```sql
BEGIN;
INSERT INTO orders ...;
SAVEPOINT before_payment;
-- payment call fails
ROLLBACK TO SAVEPOINT before_payment;
-- order insert still stands, retry payment logic
COMMIT;
```
Useful for handling partial failures inside a larger unit of work (e.g., batch imports where one bad row shouldn't abort the whole batch).

### Q9. Does MongoDB support ACID transactions?
Yes, since MongoDB 4.0 (single replica set) and 4.2 (sharded clusters), multi-document ACID transactions are supported via `session.startTransaction()`. Before that, only single-document operations were atomic (which covers most use cases due to MongoDB's embedding-friendly document model — see Topic 25). Multi-document transactions in Mongo are more expensive than single-document ops, so idiomatic Mongo schema design tries to minimize the need for them by embedding related data in one document.

### Q10. In an interview, how do you decide out loud whether an operation needs a transaction?
Ask: "does this operation touch more than one row/document/table where an observer or a crash midway could see/leave an inconsistent state?" If yes → wrap in a transaction. Example: incrementing an inventory count and inserting an order row must be atomic together, or you can oversell. A single counter increment doesn't need one. Also weigh cost — long-running transactions hold locks and block other writers, so scope them as tightly as possible around just the statements that need atomicity.

**Red flags to avoid saying:**
- "Transactions make everything thread-safe" — they don't prevent race conditions across separate transactions without the right isolation level/locking.
- Confusing "consistency" in ACID with "consistency" in CAP theorem — they're different concepts (see Topic 14).
- Saying MongoDB "doesn't support transactions" — outdated as of 4.0+.

---

## 2. Indexing (B-Tree, Hash, Composite, Covering)

**Primer:** An index is a separate on-disk data structure that lets the DB find rows
without scanning the whole table. Almost every "why is this query slow" interview question
loops back to indexing — it's the single highest-leverage topic in this doc.

### Q1. How does a B-Tree index actually work, conceptually?
It's a balanced, sorted tree structure where each node holds keys and pointers to children, and leaf nodes hold pointers to the actual rows (or the row itself, in a clustered index). Because it's sorted and balanced, lookups, range scans, and ordered scans are all O(log n) to reach the starting leaf, then sequential from there. That's why B-Trees are good for equality (`=`), range (`>`, `<`, `BETWEEN`), and `ORDER BY`/`GROUP BY` on the indexed column — the sort order is already baked in.

### Q2. When would you use a Hash index instead of a B-Tree?
Hash indexes give O(1) average lookup for exact-match equality (`=`) but **can't** do range queries, sorting, or prefix matching, because hashing destroys ordering. Postgres supports explicit hash indexes (`CREATE INDEX ... USING HASH`) — rarely used in practice because B-Tree equality lookups are already fast and B-Tree also covers ranges, so you'd need a very specific high-throughput equality-only workload to justify it. MySQL's Memory engine uses hash indexes by default; InnoDB has an internal **adaptive hash index** it builds automatically for hot pages.

### Q3. What is a composite (multi-column) index and how does column order matter?
An index on `(a, b, c)` is sorted first by `a`, then `b` within each `a`, then `c` within each `b` — like a phone book sorted by (last name, first name). It can serve queries filtering on `a`, or `a AND b`, or `a AND b AND c`, but **not** efficiently on `b` alone or `c` alone (leftmost-prefix rule). Example:
```sql
CREATE INDEX idx_orders ON orders (customer_id, status, created_at);
-- uses index: WHERE customer_id=5 AND status='paid'
-- does NOT use index efficiently: WHERE status='paid' (no customer_id)
```
Interviewer may follow up with: "how do you decide column order?" → most selective / most commonly filtered-alone column first, generally equality columns before range columns.

### Q4. What is a covering index and why does it matter for performance?
A covering index contains *all* the columns a query needs (in the index itself), so the DB never has to do a second lookup into the actual table (a "bookmark lookup"/heap fetch) to fetch remaining columns. Example: `CREATE INDEX idx_cover ON orders (customer_id, status) INCLUDE (total)` (Postgres) lets `SELECT status, total FROM orders WHERE customer_id=5` be answered entirely from the index. This is often a 2-10x speedup on read-heavy hot paths because it avoids random I/O to the heap/clustered table.

### Q5. What's the difference between a clustered and a non-clustered index?
A **clustered index** determines the physical row order on disk — there can be only one per table (InnoDB always clusters on the primary key; if you don't define one, InnoDB creates a hidden row ID). A **non-clustered/secondary index** is a separate structure that maps key → primary key value, requiring an extra lookup ("bookmark lookup") into the clustered index to get the full row. This is why InnoDB primary key choice matters a lot (see Topic 8) — secondary indexes are bigger because they store the PK value per entry.

### Q6. Why can adding an index make writes slower, and how do you reason about the tradeoff?
Every `INSERT`/`UPDATE`/`DELETE` has to update every index on that table, not just the table data — so more indexes = more write amplification and more pages touched (worse cache locality too). The tradeoff is read-frequency vs write-frequency of that table: an append-heavy logging table might intentionally have very few indexes; a read-heavy `users` table can afford more. In an interview, always mention you'd check `pg_stat_user_indexes` / MySQL's `sys.schema_unused_indexes` to find and drop unused indexes rather than indexing preemptively.

### Q7. What is index selectivity and why does low selectivity make an index useless?
Selectivity = distinct values / total rows. A `gender` column with 2 values across 10M rows has terrible selectivity — an index scan would still return ~5M matching rows, so the optimizer will rightly ignore the index and do a full table scan (a full scan has better sequential I/O than reading millions of scattered index entries + heap lookups). Good index candidates: high-cardinality columns like `email`, `order_id`, `user_id` in a join.

### Q8. How do MongoDB indexes compare to relational indexes?
MongoDB indexes are also B-Trees under the hood (default index type), with the same composite/leftmost-prefix and covering-index rules applying. Extras MongoDB has: **multikey indexes** (auto-created when indexing an array field — one entry per array element), **text indexes** for basic full-text search, **TTL indexes** for auto-expiring documents (e.g., sessions), and **compound + partial indexes** (index only a subset of documents matching a filter, saving space). Example:
```js
db.orders.createIndex({ customerId: 1, status: 1 });
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 });
```

### Q9. How would you find and diagnose a missing-index problem in production?
- MySQL: enable/check the **slow query log**, run `EXPLAIN` on the offending query, look for `type: ALL` (full scan) or high `rows examined`.
- Postgres: `EXPLAIN (ANALYZE, BUFFERS)`, look for `Seq Scan` on a large table where a `Filter` is discarding most rows, check `pg_stat_statements` for total time by query.
- MongoDB: `.explain("executionStats")`, look at `totalDocsExamined` vs `nReturned` — a huge gap means a collection scan (`COLLSCAN`) instead of an index scan (`IXSCAN`).
- In all three, the tell is: rows/docs *examined* far exceeds rows *returned*.

### Q10. Candidate says "I'll just add an index on every column to be safe" — what's wrong with that answer and what would you say instead?
It shows they don't understand the write-amplification and storage tradeoff (Q6), and that the optimizer only uses one (or a merge of a few) indexes per query anyway, so most would be dead weight. Better answer: index based on actual query patterns (`WHERE`, `JOIN`, `ORDER BY` columns), validate with `EXPLAIN`, monitor for unused indexes, and prefer a few well-chosen composite/covering indexes over many single-column ones.

**Red flags to avoid saying:**
- "Indexes always speed up queries" — ignores write cost and low-selectivity cases where the optimizer skips them anyway.
- Confusing clustered index with "an index that's just fast."
- Not knowing the leftmost-prefix rule for composite indexes.

---

## 3. Normalization & Denormalization

**Primer:** Normalization organizes relational schema to eliminate redundancy and update
anomalies, via **normal forms (1NF-3NF, BCNF)**. Denormalization intentionally reintroduces
redundancy for read performance. Senior interviews care less about reciting normal-form
definitions and more about *when you'd break them on purpose*.

### Q1. Explain 1NF, 2NF, 3NF in plain terms with an example.
- **1NF**: every column holds atomic values, no repeating groups (no comma-separated lists in a cell). Fix: split into a child table.
- **2NF**: 1NF + every non-key column depends on the *whole* primary key, not part of it (only matters for composite keys). E.g., in `(order_id, product_id) -> product_name`, `product_name` depends only on `product_id`, not the full key — violates 2NF, move it to a `products` table.
- **3NF**: 2NF + no transitive dependencies (non-key column depending on another non-key column). E.g., `orders(customer_id, customer_city)` — `customer_city` depends on `customer_id`, not the order — move it to `customers`.

### Q2. What's BCNF and how is it stricter than 3NF?
**Boyce-Codd Normal Form**: every determinant (a column or set of columns that functionally determines another column) must be a candidate key. It handles edge cases 3NF misses when a table has multiple overlapping candidate keys. In practice, most schemas that satisfy 3NF also satisfy BCNF; the distinction rarely comes up outside academic contexts, but naming it shows depth.

### Q3. Give a concrete example of an update anomaly that normalization prevents.
Unnormalized: `orders(order_id, customer_name, customer_email)`. If a customer changes their email, you must update it in every order row they've ever placed — miss one and you have inconsistent data (an **update anomaly**). Normalized: `customers(customer_id, name, email)` + `orders(order_id, customer_id)` — email lives in exactly one place.

### Q4. Why and when would you deliberately denormalize a schema?
When read performance/latency matters more than write simplicity and storage, and the redundant data changes rarely. Classic example: storing `product_name` and `unit_price` directly on an `order_items` row (redundant with `products`) so historical orders don't need a join and, more importantly, don't change if the product's price changes later — that's actually *correctness*, not just performance, since an order should reflect price-at-purchase-time.

### Q5. What's the tradeoff you're accepting when you denormalize?
You take on responsibility for consistency yourself — the app (or triggers) must keep duplicated fields in sync, and you accept the article can go stale if you forget an update path. You're trading write-complexity/redundancy for read-speed and reduced join cost. Always name this tradeoff explicitly in an interview — it shows you're not "avoiding joins because I don't know how to write them."

### Q6. How does MongoDB's document model relate to normalization?
MongoDB documents are often intentionally denormalized/embedded by design (see Topic 25) — because there's no cheap join (pre-4.0, none at all; now `$lookup` exists but is comparatively expensive), the idiomatic pattern is to embed data that's read together, accepting redundancy. This is the opposite default philosophy from relational modeling, and interviewers want to hear that you know it's a deliberate tradeoff, not "NoSQL doesn't need normalization rules."

### Q7. What's a materialized view and how does it relate to denormalization?
A materialized view precomputes and stores the result of a query (unlike a regular view, which is just a saved query re-run each time). It's denormalization managed by the DB rather than the app.
```sql
CREATE MATERIALIZED VIEW daily_sales AS
SELECT date_trunc('day', created_at) AS day, SUM(total) AS revenue
FROM orders GROUP BY 1;
REFRESH MATERIALIZED VIEW daily_sales;
```
Tradeoff: data is stale until refreshed (manually, on schedule, or `CONCURRENTLY` in Postgres to avoid locking readers).

### Q8. How do you decide, in an interview whiteboard scenario, whether to normalize or denormalize a given table?
State the access pattern first: read/write ratio, whether joined data changes frequently, and query latency requirements. Rule of thumb I'd say out loud: "start normalized for correctness and flexibility; denormalize specific hot paths only after you've measured a real join-cost problem, and keep the source of truth normalized somewhere even if you maintain a denormalized read-optimized copy (e.g., via a cache or a materialized view)."

### Q9. What's a common denormalization pitfall candidates miss?
Forgetting to update *all* the redundant copies on write — e.g., denormalizing `follower_count` onto a `users` row but only updating it from one of three code paths that can create/remove a follow relationship, causing drift over time. Mention idempotent recomputation jobs or triggers as a safety net for this class of bug.

### Q10. Is a wide, denormalized MongoDB collection with deeply embedded arrays always the "NoSQL way to do it"?
No — over-embedding causes its own problems: documents can hit the 16MB size limit, unbounded arrays (e.g., embedding every "comment" ever on a "post") cause write amplification (whole document rewritten) and bad performance. The real answer is "embed what's bounded and read together, reference what's unbounded or independently accessed" — same tradeoff as relational denormalization, just phrased in document terms.

**Red flags to avoid saying:**
- "Normalization is always better" or "NoSQL means you never normalize" — both show shallow understanding.
- Reciting 1NF/2NF/3NF definitions without being able to give a concrete anomaly example.
- Not mentioning that denormalization shifts consistency burden to the application.

---

## 4. Isolation Levels & MVCC

**Primer:** Isolation levels define how much one transaction can "see" of another
transaction's concurrent, uncommitted-or-just-committed changes. **MVCC**
(Multi-Version Concurrency Control) is the mechanism Postgres, InnoDB, and MongoDB
(WiredTiger) use to implement isolation *without* readers blocking writers.

### Q1. List the four SQL standard isolation levels and the anomaly each one stops.
| Level | Prevents | Still allows |
|---|---|---|
| Read Uncommitted | nothing extra | dirty reads |
| Read Committed | dirty reads | non-repeatable reads, phantom reads |
| Repeatable Read | dirty + non-repeatable reads | phantom reads (mostly, see Q7) |
| Serializable | everything | — (transactions behave as if run one-at-a-time) |

Higher isolation = more correctness guarantees but more blocking/aborts and lower concurrency.

### Q2. Define dirty read, non-repeatable read, and phantom read with one-line examples each.
- **Dirty read**: T1 reads a row T2 wrote but hasn't committed yet; T2 rolls back — T1 read data that "never existed."
- **Non-repeatable read**: T1 reads a row twice in the same transaction and gets different values because T2 committed an update in between.
- **Phantom read**: T1 runs the same range query (`WHERE age > 30`) twice and gets a different *set of rows* because T2 inserted a new matching row in between.

### Q3. What isolation level does MySQL (InnoDB) use by default, and Postgres?
InnoDB defaults to **Repeatable Read** (and largely prevents phantom reads too, via **next-key locking**, going beyond the SQL standard's minimum for that level). PostgreSQL defaults to **Read Committed**. This is a genuinely common interview gotcha — candidates often assume both default the same way.

### Q4. Explain MVCC conceptually — how does it let readers avoid blocking writers?
Instead of readers taking locks on rows, every row version is tagged with the transaction ID that created it. A read transaction sees a **consistent snapshot** — only row versions committed before its snapshot was taken — regardless of what writers do concurrently afterward. Writers create new row versions rather than overwriting in place (Postgres) or maintain undo logs to reconstruct old versions (InnoDB). Result: `SELECT` never blocks on `UPDATE`/`INSERT`, and vice versa.

### Q5. If Postgres uses MVCC and creates a new row version on every UPDATE, what cleans up the old versions?
**VACUUM** (see Topic 9) — old ("dead") tuple versions that no transaction can still see are reclaimed by the vacuum process, and the space is marked reusable. If vacuum falls behind (e.g., a long-running transaction holds an old snapshot open), dead tuples pile up, causing table bloat and slower scans — a very common real-world Postgres incident.

### Q6. How does InnoDB implement MVCC differently from Postgres?
InnoDB keeps the current row version in the clustered index (in place) plus an **undo log** of previous versions in a separate rollback segment. A reader needing an old snapshot reconstructs it by applying undo records backward. Postgres instead keeps *all* versions directly in the table's heap (old and new tuples side by side) until vacuumed. This is why InnoDB doesn't need Postgres-style vacuuming for MVCC cleanup (though it has its own purge thread for the undo log) but Postgres table bloat is the more commonly discussed operational issue.

### Q7. Does Repeatable Read fully prevent phantom reads? What's the nuance interviewers probe for here?
In the SQL standard, Repeatable Read technically still allows phantoms. But **InnoDB's Repeatable Read specifically prevents phantoms too** via next-key locks (index-range locks) on write paths — meaning InnoDB's RR is stricter than the standard requires. Postgres's Repeatable Read (which it implements as **Snapshot Isolation**) also prevents phantoms for reads within the transaction's snapshot, but can still suffer **write skew** (a Serializable-only anomaly) — mentioning write skew here is a strong signal of depth.

### Q8. What is write skew and why does even Snapshot Isolation/Repeatable Read allow it?
Two transactions each read overlapping data, and each makes a decision based on what they read, then both commit changes that are individually valid but jointly violate an invariant neither transaction saw the other making. Classic example: an on-call system requires ≥1 doctor on call; two doctors each independently check "is someone else on call?", both see "yes," both remove themselves — now zero are on call. Neither transaction's *write* conflicted with the other's *write* (they wrote different rows), so snapshot isolation doesn't catch it. Only true Serializable isolation (or an explicit constraint/lock) prevents this.

### Q9. What does MongoDB use for isolation, and what's the default read/write concern relevant here?
WiredTiger (MongoDB's default storage engine) uses MVCC internally, similar in spirit to Postgres snapshot isolation. Multi-document transactions in MongoDB default to **snapshot** read concern, giving repeatable-read-like guarantees within the transaction. Outside of explicit transactions, single-document operations are always atomic and isolated (a reader never sees a partially-applied document write) — this is why MongoDB's embedding-heavy modeling (Topic 25) sidesteps needing multi-doc transactions for a lot of use cases.

### Q10. In an interview, how do you decide which isolation level to use for a given feature?
Say the tradeoff out loud: "I'd default to the DB's default (Read Committed for Postgres, Repeatable Read for InnoDB) unless I have a specific correctness concern." Bump to Serializable (or add an explicit `SELECT ... FOR UPDATE` row lock) only for operations vulnerable to write skew or requiring strict correctness (e.g., seat booking, inventory decrement) — and mention that Serializable has real throughput cost / can cause serialization-failure retries, so you'd scope it to the smallest transaction possible and have retry logic for serialization errors.

**Red flags to avoid saying:**
- Assuming MySQL and Postgres share the same default isolation level.
- Not knowing MVCC means readers don't block writers — the most-tested concept in this topic.
- Never having heard of write skew when discussing Repeatable Read/Snapshot Isolation limits.

---

## 5. Locking & Concurrency Control

**Primer:** Locking is how a database serializes conflicting concurrent writes (MVCC
handles reader/writer conflicts, but writer/writer conflicts still need locks). This topic
is where "why is my app hanging" production incidents usually trace back to.

### Q1. What's the difference between a shared (read) lock and an exclusive (write) lock?
A **shared lock (S)** lets multiple transactions read the same row concurrently but blocks any exclusive lock on it. An **exclusive lock (X)** is held for writes and blocks both shared and exclusive locks from anyone else on that row. Compatibility: S-S is fine (many readers), S-X and X-X both block.

### Q2. What does `SELECT ... FOR UPDATE` do and when would you use it?
It takes an exclusive row lock on the selected rows within the current transaction, blocking other transactions from updating (or also selecting `FOR UPDATE`) those rows until commit/rollback. Classic use: read-then-write logic that needs to be safe from concurrent modification, e.g.:
```sql
BEGIN;
SELECT quantity FROM inventory WHERE product_id = 42 FOR UPDATE;
-- app checks quantity > 0 in code
UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 42;
COMMIT;
```
Without `FOR UPDATE`, two concurrent transactions could both read quantity=1 and both decrement, overselling.

### Q3. What's the difference between pessimistic and optimistic concurrency control?
**Pessimistic** (locking, `FOR UPDATE`) assumes conflicts are likely — blocks other transactions upfront. **Optimistic** assumes conflicts are rare — proceeds without locking, then checks at commit time (typically via a `version`/`updated_at` column) whether the row changed since it was read, and fails/retries if so:
```sql
UPDATE inventory SET quantity = quantity - 1, version = version + 1
WHERE product_id = 42 AND version = 7;
-- if 0 rows affected, someone else updated first -> retry
```
Optimistic wins under low contention (no lock-wait overhead); pessimistic wins under high contention (avoids repeated retries).

### Q4. What is row-level vs table-level locking, and why does InnoDB prefer row-level?
Table-level locks block all rows in a table (simple, but kills concurrency). Row-level locks only block the specific rows touched, allowing unrelated rows to be written concurrently. InnoDB uses row-level locking as its default for DML, which is why it scales far better than MyISAM (table-level locking only) under concurrent write workloads — a major reason InnoDB replaced MyISAM as MySQL's default engine.

### Q5. Explain gap locks and next-key locks in InnoDB — why do they exist?
A **gap lock** locks the *space between* index values (not an existing row) to prevent other transactions from inserting a new row into that gap. A **next-key lock** is a gap lock + a lock on the existing record combined. They exist specifically to prevent phantom reads under Repeatable Read — e.g., locking the gap between `id=10` and `id=20` stops a concurrent insert of `id=15` that would otherwise appear as a phantom on re-query. This is InnoDB-specific behavior candidates rarely know unless they've hit a confusing deadlock caused by gap locks.

### Q6. What causes a deadlock and how does the DB resolve it?
Two transactions each hold a lock the other needs — e.g., T1 locks row A then wants row B, T2 locks row B then wants row A — circular wait. The DB runs a deadlock detector (InnoDB) or relies on a lock-wait timeout (Postgres, which detects cycles too) and picks a **victim transaction to abort** (typically the one with the least work done / cheapest to roll back), returning an error to that transaction's client so the app can retry.

### Q7. How do you prevent deadlocks at the application level?
Always acquire locks/update rows **in a consistent order** across all code paths (e.g., always lock the lower `id` first in a transfer between two accounts). Keep transactions short — the longer a transaction holds locks, the bigger the collision window. Avoid interactive/user-input-dependent transactions (never wait on a network call or user action while holding a DB transaction open).

### Q8. What's the difference between a lock wait timeout and a deadlock error, and how should app code handle each?
A **lock wait timeout** (`innodb_lock_wait_timeout`, Postgres `statement_timeout`) fires when a transaction waits too long for a lock that *isn't* necessarily part of a cycle — could just be genuine contention. A **deadlock** is detected immediately (no waiting for a timeout) because the DB proactively finds the cycle. Both should be handled the same way in app code: catch the specific error code, roll back, and retry with backoff — never assume "timeout" means something is broken versus just busy.

### Q9. Does MongoDB use locks, and at what granularity?
Yes — WiredTiger uses MVCC for most reads, but writes to the *same document* are serialized. Historically MongoDB had database-level then collection-level locking; modern WiredTiger does **document-level locking** for most operations, so concurrent writes to different documents in the same collection don't block each other. Intent locks are taken at higher granularities (collection/database) to detect conflicts efficiently without scanning every document lock.

### Q10. How would you debug a production incident where queries are suddenly hanging/timing out?
Say the concrete steps: check for **long-running/idle-in-transaction sessions** holding locks (`pg_stat_activity` in Postgres, `SHOW ENGINE INNODB STATUS` / `sys.innodb_lock_waits` in MySQL) — a forgotten open transaction from a bug or a connection pool leak is the #1 real-world cause. Identify the blocking session and either wait it out, or if it's genuinely stuck, kill it (`pg_terminate_backend`, `KILL` in MySQL) and fix the root cause (missing commit/rollback, connection leak).

**Red flags to avoid saying:**
- "MVCC means there's no locking at all" — writer-writer conflicts still need locks even under MVCC.
- Not being able to explain the difference between a deadlock and normal lock contention.
- Suggesting "just retry forever" without mentioning backoff or root-causing long-held locks.

---

## 6. Query Optimization & EXPLAIN Plans

**Primer:** The query planner/optimizer decides *how* to execute a SQL query — which
indexes to use, join order, join algorithm. Reading `EXPLAIN` output fluently is one of
the most practically tested senior-engineer skills, often via a live "here's a slow query,
fix it" exercise.

### Q1. What's the difference between `EXPLAIN` and `EXPLAIN ANALYZE`?
`EXPLAIN` shows the planner's *estimated* execution plan without running the query (cost estimates based on table statistics). `EXPLAIN ANALYZE` actually **executes** the query and shows real timing, real row counts, and actual vs estimated row mismatches — critical for spotting stale statistics. In Postgres, always add `BUFFERS` too: `EXPLAIN (ANALYZE, BUFFERS)` to see cache hits vs disk reads. Caution: `EXPLAIN ANALYZE` on a slow `UPDATE/DELETE` actually executes the write — wrap in a transaction and roll back if just profiling.

### Q2. In a Postgres plan, what's the difference between `Seq Scan`, `Index Scan`, and `Index Only Scan`?
- **Seq Scan**: reads the whole table sequentially — fine for small tables or when most rows match anyway.
- **Index Scan**: uses an index to find matching rows, then fetches each from the heap (extra I/O per row).
- **Index Only Scan**: the index alone contains every column needed — no heap fetch at all (fastest), possible when the query is covered by the index and the visibility map confirms rows are visible without checking the heap.

### Q3. In a MySQL `EXPLAIN` output, what do the `type`, `rows`, and `Extra` columns tell you?
- `type`: access method, roughly best-to-worst: `const/eq_ref` (single row via PK/unique) → `ref` (index equality) → `range` → `index` (full index scan) → `ALL` (full table scan, red flag on large tables).
- `rows`: estimated rows examined — compare against actual returned rows to spot inefficiency.
- `Extra`: look for `Using filesort` (expensive in-memory/disk sort not satisfied by an index — often fixable by adding an index matching `ORDER BY`) and `Using temporary` (temp table needed, common with `GROUP BY`/`DISTINCT` on unindexed columns) — both are common red flags to fix.

### Q4. Why might the query planner ignore an index that clearly exists on the filtered column?
Most common reasons: (1) low selectivity — planner estimates a full scan is cheaper (Topic 2 Q7); (2) stale statistics — run `ANALYZE` (Postgres) / `ANALYZE TABLE` (MySQL) to refresh row/value distribution estimates; (3) a function or type cast on the indexed column in the `WHERE` clause (`WHERE LOWER(email) = ...` won't use a plain index on `email` — needs a functional/expression index); (4) implicit type conversion (comparing a string column to a numeric literal without quotes).

### Q5. What is the N+1 query problem and how do you spot/fix it?
Running one query to get a list, then one additional query *per row* to fetch related data (e.g., fetch 50 orders, then loop and query `customer` for each — 51 queries total instead of 2). Spot it via query logs showing repeated near-identical queries, or an ORM's query counter in dev. Fix: batch-fetch with a single `WHERE id IN (...)` query, or use a `JOIN`/ORM eager-loading (`.includes`, `JOIN FETCH`, etc.). This is one of the most common real-world perf bugs and a very common interview question.

### Q6. How does the query planner decide join order and join algorithm, and why does it matter?
The optimizer estimates the cost of different join orders/algorithms using table statistics (row counts, selectivity) and picks the cheapest plan — for 3+ tables, join order changes the size of intermediate result sets dramatically. Algorithms: **nested loop** (good for small outer table + indexed inner lookup), **hash join** (good for large unsorted sets, builds a hash table on the smaller side), **merge join** (good when both sides are already sorted on the join key, e.g., via an index). You generally don't hand-pick these — you fix bad choices by ensuring good statistics and indexes exist so the optimizer's cost model is accurate.

### Q7. Query using `SELECT *` shows up in every review as a code smell — explain the actual performance reasons why.
It defeats covering indexes (forces a heap fetch even if the filter columns were covered), pulls unnecessary columns over the network/into app memory (especially costly for large `TEXT`/`BLOB` columns), and is fragile to schema changes (adding a column silently changes what the app receives). Always select only the columns you need — both for query plan quality and for API/contract stability.

### Q8. How would you optimize a slow `GROUP BY`/aggregation query?
Ensure an index exists on the `GROUP BY` columns (lets the planner do an index-ordered scan instead of `Using temporary; Using filesort`); pre-filter with `WHERE` before grouping rather than `HAVING` where possible (HAVING filters *after* aggregation, wasting work); consider a materialized view (Topic 3 Q7) if the aggregation is run frequently on largely-unchanged historical data; for very large tables consider pre-aggregating incrementally (rollup table updated on write) rather than aggregating on every read.

### Q9. How do you profile and optimize a slow MongoDB aggregation pipeline?
Use `.explain("executionStats")` on the pipeline, check for `COLLSCAN` stages early in the pipeline. Push `$match` and `$sort` stages as early as possible so they can use an index (an early `$match` can use `IXSCAN`; the same `$match` after a `$group` cannot). Use `$project` early to shrink documents flowing through later, expensive stages. Check `executionStats.executionStages` for stages with disproportionate `executionTimeMillisEstimate`.

### Q10. Interviewer gives you a query that's slow only in production, not in staging — what do you check?
Data volume/skew differences (staging often has far less/uniform data — index selectivity assumptions that hold in staging may not in prod); stale statistics on the prod table if `ANALYZE`/auto-vacuum-analyze hasn't run recently after a large data change; connection pool exhaustion or lock contention from concurrent load that doesn't exist in staging; and actual hardware/cache differences (prod's working set may not fit in buffer pool/shared_buffers the way staging's does). Say you'd reproduce with `EXPLAIN ANALYZE` directly against prod (read replica) rather than guessing from staging.

**Red flags to avoid saying:**
- Treating `EXPLAIN` (estimate) and `EXPLAIN ANALYZE` (actual execution) as interchangeable.
- Not knowing what `Using filesort`/`Using temporary` (MySQL) or `Seq Scan` (Postgres) mean.
- Suggesting "just add an index" without checking if a function/type-cast on the column is preventing its use.

---

## 7. Joins — Types & Performance

**Primer:** Joins combine rows from multiple tables based on a related column. Interviewers
test both *semantic* correctness (do you get the right rows) and *performance* reasoning
(cost, indexing implications).

### Q1. Explain INNER JOIN vs LEFT JOIN vs RIGHT JOIN vs FULL OUTER JOIN with a one-line mental model each.
- **INNER JOIN**: only rows with a match on both sides.
- **LEFT JOIN**: all rows from the left table, matched columns from the right (NULL if no match).
- **RIGHT JOIN**: mirror of LEFT — all rows from the right table (rarely used; people rewrite as LEFT JOIN with tables swapped for consistency).
- **FULL OUTER JOIN**: all rows from both sides, NULLs where no match on either side (not supported directly in MySQL — emulated via `LEFT JOIN UNION RIGHT JOIN`).

### Q2. What's a self-join and give a real example of when you'd use one?
A table joined to itself, usually to compare rows or represent hierarchical data.
```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```
Used for org charts, category trees, or finding duplicate/related rows within the same table.

### Q3. What's a CROSS JOIN and when is it (rarely) actually useful?
Produces the Cartesian product — every row from table A paired with every row from table B, with no `ON` condition. Rarely intentional in production queries (usually a missing join condition = bug), but legitimately used to generate combinations, e.g., cross-joining a `date_series` with a `store_id` list to build a complete report grid with zero-filled gaps for dates/stores with no sales.

### Q4. Why can a LEFT JOIN silently turn into an effective INNER JOIN, and how do you avoid the bug?
If you put a filter on the right table's column in the `WHERE` clause instead of the `ON` clause, rows with no match (NULL on the right) get filtered out by `WHERE`, because `NULL = value` is never true — collapsing your LEFT JOIN back to INNER JOIN behavior.
```sql
-- BUG: turns into inner join
SELECT * FROM customers c LEFT JOIN orders o ON c.id=o.customer_id WHERE o.status='paid';
-- FIX: filter belongs in ON
SELECT * FROM customers c LEFT JOIN orders o ON c.id=o.customer_id AND o.status='paid';
```

### Q5. How does join performance relate to indexing, specifically?
The join column on the "inner"/lookup side should be indexed (ideally the FK column), or every join becomes an expensive nested-loop full scan per outer row. Rule of thumb stated in interviews: "index your foreign keys" — Postgres does NOT auto-index FK columns (only the referenced PK side is auto-indexed), a very common real-world oversight; MySQL/InnoDB *does* auto-create an index for FK constraints.

### Q6. What does MongoDB's `$lookup` do and how does its performance compare to a SQL join?
`$lookup` is an aggregation stage that performs a left-outer-join-style operation against another collection, matching a local field to a foreign field.
```js
db.orders.aggregate([
  { $lookup: { from: "customers", localField: "customerId", foreignField: "_id", as: "customer" } }
]);
```
It's generally more expensive than a relational join because MongoDB wasn't architected around join optimization the way relational engines are — no join reordering/hash-join optimizer sophistication. It benefits heavily from an index on `foreignField`. This is a core reason MongoDB schema design favors embedding over referencing for data accessed together (Topic 25).

### Q7. What's the difference between a hash join and a nested loop join, performance-wise?
**Nested loop**: for each row in the outer table, scan/index-lookup the inner table — good when the outer set is small and the inner join column is indexed (cost ≈ outer_rows × index_lookup_cost). **Hash join**: build an in-memory hash table on the smaller input's join key, then probe it with the larger input — good for large, unindexed joins where a full scan of both sides is unavoidable anyway (cost ≈ linear in both table sizes, no per-row index lookups). The optimizer picks based on estimated table sizes and available indexes.

### Q8. How would you join two very large tables (both in the hundreds of millions of rows) efficiently?
Ensure the join column is indexed on at least one side; make sure statistics are fresh so the optimizer picks hash join over a mistaken nested loop; consider whether you actually need to join at query time versus precomputing/denormalizing a materialized view for the hot-path query (Topic 3); for genuinely huge analytical joins, consider partitioning both tables on the join key so partition-wise joins can run in parallel, or moving the analytical workload to a column-store/OLAP system rather than the OLTP primary DB.

### Q9. What's the difference between `UNION` and `UNION ALL`, and which is faster?
`UNION` deduplicates the combined result set (implicitly sorts/hashes to remove duplicates — extra cost). `UNION ALL` just concatenates results with no dedup — faster, and the correct choice whenever you already know there's no overlap or duplicates are acceptable/desired.

### Q10. Explain the difference between `WHERE` and `HAVING` in a query with a `JOIN` and `GROUP BY`.
`WHERE` filters rows *before* grouping/aggregation (cheaper — reduces rows early, can use indexes on filtered columns). `HAVING` filters *after* aggregation, on the aggregated result (e.g., `HAVING COUNT(*) > 5`) — necessary specifically because you can't reference an aggregate function's result in `WHERE`. Always push filters to `WHERE` unless the filter depends on an aggregate value.

**Red flags to avoid saying:**
- Not knowing that a WHERE-clause filter on the right table silently converts a LEFT JOIN to an INNER JOIN.
- Claiming Postgres auto-indexes foreign key columns (it doesn't — only MySQL/InnoDB does for FK constraints).
- Defaulting to `$lookup`-heavy MongoDB schemas without acknowledging the embed-vs-reference tradeoff.

---

## 8. MySQL Storage Engines (InnoDB Internals)

**Primer:** MySQL's pluggable storage engine architecture means the same SQL layer can
sit on top of different engines with very different guarantees. In practice, "MySQL" in a
modern interview almost always means **InnoDB** — knowing why it won over MyISAM shows
real depth.

### Q1. What's the core difference between InnoDB and MyISAM (MySQL's two classic engines)?
InnoDB: row-level locking, full ACID transaction support, foreign key constraints, crash recovery via redo/undo logs, MVCC. MyISAM: table-level locking only, no transactions, no foreign keys, but historically faster for pure read-heavy/full-table-scan workloads and had built-in full-text search before InnoDB gained it. InnoDB has been the default engine since MySQL 5.5 (2010) precisely because table-level locking made MyISAM unusable for concurrent-write OLTP workloads.

### Q2. Why does InnoDB's primary key choice matter so much for performance?
InnoDB is **index-organized** — the table data itself IS the clustered index, physically sorted by primary key. Every secondary index stores the primary key value (not a physical row pointer) to look up the full row. Consequences: (1) a random-value PK (like a UUID v4) causes random-order inserts, page splits, and fragmentation; (2) a large PK data type bloats every secondary index, since the PK value is duplicated into each one. This is why an auto-incrementing integer (or UUIDv7/ULID, which are time-sortable) is generally preferred over a random UUID as a primary key.

### Q3. What is the InnoDB buffer pool and why is its size the single most important tuning parameter?
It's InnoDB's in-memory cache of data pages and indexes. Reads/writes that hit the buffer pool avoid disk I/O entirely. `innodb_buffer_pool_size` is typically tuned to ~70-80% of available RAM on a dedicated DB server — too small and you thrash to disk on every query; the single biggest lever for MySQL performance tuning in almost every real-world incident.

### Q4. What's the redo log and how does it relate to durability and crash recovery?
The redo log is InnoDB's WAL (Topic 1 Q4) — physical, page-level changes are written here before being applied to the actual data pages, and fsynced according to `innodb_flush_log_at_trx_commit` (1 = fsync every commit, fully durable but slower; 0 or 2 = trade some durability for throughput). On crash, InnoDB replays the redo log from the last checkpoint to restore data pages to a consistent state.

### Q5. What's the undo log and what two things does it enable?
The undo log stores the previous version of a row before modification. It enables (1) **rollback** — undoing a transaction's changes if it aborts, and (2) **MVCC snapshot reads** — reconstructing an older row version for a transaction that started before the current change committed (Topic 4 Q6). Long-running transactions keep undo log entries from being purged, which can bloat the undo tablespace — a real operational gotcha.

### Q6. What is the InnoDB adaptive hash index and how is it different from a normal index?
InnoDB automatically builds an in-memory hash index on top of frequently-accessed pages of an existing B-Tree index, to speed up equality lookups on hot data — it's self-managed, not something you create explicitly (unlike a real hash index). Can occasionally be disabled (`innodb_adaptive_hash_index=OFF`) if profiling shows contention on its internal latch under very high concurrency, but it's on by default and generally beneficial.

### Q7. Why does InnoDB recommend avoiding very large transactions/batch operations without chunking?
Large transactions hold row locks and accumulate undo log entries for their entire duration, blocking other writers and preventing old undo entries from being purged (bloating the undo tablespace / causing "history list length" growth). Best practice: chunk large batch updates/deletes into smaller transactions (e.g., `LIMIT 1000` loops), committing between chunks, to bound lock hold time and undo growth.

### Q8. What's the difference between `AUTO_INCREMENT` locking behavior and why might it matter in high-concurrency inserts?
InnoDB uses a lightweight, short-duration lock (not held for the full transaction) to serialize allocation of auto-increment values under the default `innodb_autoinc_lock_mode` in modern MySQL, allowing concurrent inserts to proceed without becoming a bottleneck. In older/legacy modes, or for `INSERT ... SELECT` bulk inserts, a table-level auto-inc lock could be held for the whole statement, serializing concurrent inserts — worth knowing exists as a historical gotcha even if modern defaults mostly avoid it.

### Q9. What storage engine considerations are specific to MongoDB, for comparison (WiredTiger)?
MongoDB's default engine, **WiredTiger**, similarly does document-level locking, MVCC-based snapshots, and has its own in-memory cache (`wiredTigerCacheSizeGB`, defaulting to ~50% of RAM minus 1GB) analogous to InnoDB's buffer pool. It also compresses data on disk by default (snappy/zstd), which InnoDB does not do by default — worth mentioning as a genuine architectural difference when asked to compare storage layers across the two databases.

### Q10. In an interview, how do you explain "why InnoDB over MyISAM" concisely without just reciting a feature list?
Frame it around the actual failure mode: "MyISAM's table-level locking means a single write blocks all reads and writes on that table — fine for read-only/reporting workloads, but it falls over under any real concurrent OLTP write load. InnoDB's row-level locking plus MVCC lets reads and writes to different rows proceed concurrently, plus you get crash-safe recovery and referential integrity via foreign keys — which is why it's been the sane default for over a decade."

**Red flags to avoid saying:**
- Recommending MyISAM for a transactional/write-heavy workload.
- Not knowing InnoDB tables are clustered by primary key (index-organized) — this affects PK choice, a common follow-up.
- Confusing the redo log (crash recovery) with the undo log (rollback + MVCC) — they serve different purposes.

---

## 9. PostgreSQL Architecture (WAL, VACUUM, MVCC)

**Primer:** Postgres has a distinctive architecture — process-per-connection, heap-based
MVCC with in-place-append row versions, and a vacuum subsystem to reclaim space. Senior
Postgres interviews probe operational understanding, not just SQL syntax.

### Q1. Why does Postgres need VACUUM at all — what problem does it solve?
Because Postgres MVCC creates a new row version (tuple) on every `UPDATE` and marks old ones as dead rather than overwriting in place (Topic 4 Q6), dead tuples accumulate in the heap. VACUUM scans and reclaims the space used by dead tuples that no active transaction can still see, making it reusable for future inserts/updates. Without it, tables and indexes bloat indefinitely and scans get slower (more pages to read, most holding dead rows).

### Q2. What's the difference between `VACUUM`, `VACUUM FULL`, and autovacuum?
- **VACUUM**: reclaims dead tuple space for reuse *within* the table's existing files — doesn't shrink the file on disk, non-blocking (concurrent reads/writes allowed).
- **VACUUM FULL**: rewrites the entire table into a new, compact file, actually shrinking disk usage — but takes an exclusive lock, blocking all access for its duration. Rarely run on production tables without a maintenance window.
- **autovacuum**: a background process that runs VACUUM (and `ANALYZE`) automatically based on thresholds (`autovacuum_vacuum_scale_factor` etc.) — the default, "set and mostly forget" mechanism; tuning its aggressiveness is a common senior-DBA task.

### Q3. What is transaction ID wraparound and why is it a serious operational risk?
Postgres transaction IDs (XIDs) are 32-bit and compared with wraparound-aware logic; if the XID counter wraps around without vacuum "freezing" old tuples first, old data could appear to be from the future, causing catastrophic visibility corruption. Postgres protects against this by making autovacuum run aggressively (ignoring normal thresholds) as the oldest unfrozen XID approaches the wraparound danger zone, and in the extreme case will refuse new writes to force a vacuum rather than risk corruption. Real-world incident pattern: autovacuum disabled or starved on a huge table → wraparound emergency.

### Q4. Explain what the WAL (Write-Ahead Log) is used for beyond crash recovery.
Beyond durability/crash recovery (Topic 1 Q4), WAL is also the mechanism for **replication** — a replica applies the same WAL stream to stay in sync with the primary (Topic 12), and for **point-in-time recovery (PITR)** — replaying WAL up to a specific timestamp to restore from a base backup plus WAL archive (Topic 19). It's genuinely the backbone of most of Postgres's operational features, not just a durability detail.

### Q5. What's a checkpoint and why does checkpoint frequency matter for performance?
A checkpoint flushes all dirty (modified) in-memory pages to disk and records a point WAL replay could start from on crash recovery, letting old WAL segments be recycled. Frequent checkpoints mean shorter crash recovery time but more I/O overhead/write amplification during normal operation; infrequent checkpoints mean less overhead day-to-day but longer recovery time and larger WAL retention. Tuned via `checkpoint_timeout` and `max_wal_size`.

### Q6. What is table/index bloat, how do you detect it, and how do you fix it?
Bloat = dead tuples/pages not yet reclaimed, inflating table/index size beyond the "live" data's actual footprint — usually from vacuum falling behind (long-running transactions holding old snapshots open, or autovacuum misconfigured/too slow for the write rate). Detect via extensions like `pgstattuple` or bloat-estimate queries against `pg_stat_user_tables`. Fix: tune autovacuum to be more aggressive on hot tables, kill long-idle-in-transaction sessions, and as a last resort run `VACUUM FULL` or `pg_repack` (which rebuilds without a long exclusive lock) during a maintenance window.

### Q7. What's the difference between a hot standby and a streaming replica in Postgres, and what is "replication lag"?
A **streaming replica** continuously receives and applies WAL from the primary over a network connection (as opposed to periodically shipping WAL files). A **hot standby** is a replica that can also serve *read-only* queries while replaying WAL. **Replication lag** is the delay between a write committing on the primary and becoming visible on the replica — matters because reading from a lagging replica can return stale data, a common source of "I just wrote this and can't read it back" bugs when read/write are split across primary/replica.

### Q8. What's `pg_stat_activity` and how would you use it to diagnose a live production issue?
A system view showing every current connection/session: its query, state (`active`, `idle`, `idle in transaction`), wait event, and duration. Used to find long-running queries, sessions stuck `idle in transaction` (holding locks/old snapshots and blocking vacuum — a very common real incident), or to identify what's blocking a lock wait (joined with `pg_locks`). This is the single most-used diagnostic view in Postgres production incidents, worth naming explicitly in an interview.

### Q9. How does Postgres handle schema changes (e.g., `ALTER TABLE ADD COLUMN`) with minimal downtime?
Adding a column with no default (or a constant default, since Postgres 11+ stores it as metadata rather than rewriting the table) is fast/near-instant. Adding a `NOT NULL` column with a computed default, changing a column's type, or adding certain constraints historically required a full table rewrite (`ACCESS EXCLUSIVE` lock for the duration) — for large tables this needs an online migration strategy: add nullable column → backfill in batches → add `NOT NULL` via `NOT VALID` + `VALIDATE CONSTRAINT` (which takes a much shorter lock) rather than one blocking `ALTER TABLE`.

### Q10. Interviewer asks: "Postgres CPU usage spiked and queries are slow — walk me through your diagnosis." What do you say?
"First `pg_stat_activity` to see what's actually running and for how long, and check for lock waits via `pg_locks`. Check if autovacuum is running heavily (visible in `pg_stat_activity` as `autovacuum: VACUUM ...`) — often the culprit after a large bulk write. Check `pg_stat_statements` for which query pattern's total execution time jumped. Rule out a bad plan from stale statistics after a data shape change (`ANALYZE`). Only after isolating the query/session would I look at `EXPLAIN ANALYZE` on the specific offending query." This shows a methodical, tool-driven process rather than guessing.

**Red flags to avoid saying:**
- Thinking VACUUM's only job is disk cleanup — missing the wraparound-prevention and replication/PITR angles.
- Not knowing `idle in transaction` sessions are a common, real cause of vacuum/lock problems.
- Suggesting `VACUUM FULL` as a routine maintenance task without mentioning its exclusive lock cost.

---

## 10. MongoDB Document Model & Schema Design

**Primer:** MongoDB stores data as BSON documents (binary JSON) in collections, with no
enforced schema by default. Interviewers here are really testing whether you can design a
schema around *access patterns* rather than just relational habits copy-pasted into JSON.

### Q1. What is BSON and how is it different from JSON?
**BSON (Binary JSON)** is MongoDB's binary-encoded superset of JSON — adds types JSON lacks natively (e.g., `Date`, `ObjectId`, `Binary`, distinct `Int32`/`Int64`/`Double`), and is designed for fast encode/decode and traversal (each element is length-prefixed, so the parser can skip fields without scanning byte-by-byte). Documents are limited to 16MB.

### Q2. What is `_id` and `ObjectId`, and what information is encoded in an ObjectId?
Every document requires a unique `_id` field (auto-indexed as the primary key equivalent). If you don't supply one, MongoDB generates an `ObjectId` — a 12-byte value encoding: a 4-byte timestamp (seconds since epoch), a 5-byte random value (unique per process), and a 3-byte incrementing counter. This means ObjectIds are roughly time-sortable (useful for "created recently" queries without a separate timestamp field) but not strictly monotonic across processes.

### Q3. Does MongoDB enforce a schema? What is schema validation and when would you use it?
Not by default — a collection can hold documents with wildly different shapes ("schemaless"), but that's a spectrum, not a binary — real applications almost always have an implicit schema enforced at the application/ODM layer (e.g., Mongoose). MongoDB also supports optional **JSON Schema validation** at the collection level (`db.createCollection` with `validator`) to enforce required fields/types server-side as a safety net, without going fully rigid like a relational schema.

### Q4. What's the core schema design philosophy difference between MongoDB and a relational DB?
Relational: normalize, model entities and relationships, join at query time. MongoDB: **model around your application's access patterns** — data that is read together should generally live together (embedded) to avoid expensive joins/multiple round-trips, since MongoDB's join (`$lookup`) is less optimized than a relational join (Topic 7 Q6). The guiding question is "how will I query this?" more than "what's the theoretically normalized shape?" — see Topic 25 for the embed-vs-reference decision framework in detail.

### Q5. What's the 16MB document size limit and what schema design problems does it prevent you from ignoring?
Every BSON document is capped at 16MB (a deliberate limit to keep documents efficiently transferable and to discourage unbounded growth patterns). It directly rules out naive designs like "embed every comment ever posted on this blog post as an array inside the post document" — an unbounded array will eventually blow the limit and also cause write amplification (Topic 3 Q10) long before hitting it. Forces you to recognize unbounded relationships and model them with references or a bucket pattern instead.

### Q6. What is the "bucket pattern" and what problem does it solve?
Groups many small, related documents (e.g., IoT sensor readings taken every second) into fewer, larger documents bucketed by a time window (e.g., one document per sensor per hour, with an array of readings inside), instead of one document per reading. Solves two problems: reduces the total document/index-entry count (fewer index B-Tree entries to maintain), and improves read locality for "give me this sensor's readings for this hour" queries — one document fetch instead of thousands.

### Q7. Explain schema versioning in MongoDB — how do you evolve a document shape over time without downtime?
Since there's no enforced schema, you can deploy new application code that writes the new shape immediately, while old documents keep their old shape until they're next written or explicitly backfilled. Common pattern: add a `schemaVersion` field to documents, and write application read-paths that handle both old and new shapes (migrate-on-read), combined with a background job to backfill old documents — avoids a blocking, all-at-once migration on a huge collection.

### Q8. What is a polymorphic schema pattern in MongoDB and when is it useful?
Storing different "kinds" of related entities in the same collection with a `type` discriminator field and differing sets of other fields — e.g., a `products` collection where `type: "book"` documents have `author`/`isbn` and `type: "electronics"` documents have `warrantyMonths`. Useful when the entities are queried together often (e.g., a single product listing page) despite differing attributes — avoids needing a separate collection (and a `$lookup`-heavy union) per subtype.

### Q9. How do you model a many-to-many relationship in MongoDB — e.g., students and courses?
Two main options: (1) **two-way referencing** — each `student` document has an array of `courseIds`, each `course` document has an array of `studentIds`; good when you query from either side frequently, but you must keep both arrays in sync on write. (2) A **junction/linking collection** (`enrollments: { studentId, courseId, enrolledAt }`), closer to a relational join table; better when the relationship itself has attributes (like `enrolledAt`, `grade`) or when either side's array could grow unbounded (e.g., a popular course with 100k students shouldn't have a 100k-element array embedded anywhere).

### Q10. A candidate designs a MongoDB schema by directly translating their relational tables 1:1 into collections with `$lookup` joins everywhere — what's wrong with this, and what would you say in an interview?
It throws away MongoDB's actual strength (fast reads of related data via embedding) and inherits its weakest area (join performance), effectively building a slower, less-consistent relational database. I'd say: "I'd first map out the application's actual read/write patterns and frequency, then decide per-relationship whether to embed (bounded, read-together, doesn't need independent querying) or reference (unbounded, or needs independent access/updates) — not default to mirroring the relational schema."

**Red flags to avoid saying:**
- "MongoDB has no schema" as an absolute statement — application-level and optional JSON Schema validation both impose structure in practice.
- Designing 1:1 relational-to-document translations with heavy `$lookup` use everywhere.
- Not knowing about the 16MB document limit when proposing an embedding-heavy design.

---

## 11. MongoDB Aggregation Framework & Indexing

**Primer:** The aggregation framework is MongoDB's pipeline-based query/transformation
engine — the equivalent of SQL's `GROUP BY`/`JOIN`/window functions, expressed as an
ordered array of stages each transforming the document stream.

### Q1. What is the aggregation pipeline, conceptually, and name the most commonly used stages.
An ordered sequence of stages, each taking the previous stage's output documents as input and producing a new stream of documents. Most common stages: `$match` (filter, like `WHERE`), `$group` (aggregate, like `GROUP BY`), `$project` (reshape/select fields), `$sort`, `$limit`, `$lookup` (join), `$unwind` (flatten an array field into one document per element).

### Q2. Write a pipeline to get total order value per customer, only for customers with more than 5 orders.
```js
db.orders.aggregate([
  { $group: { _id: "$customerId", total: { $sum: "$amount" }, count: { $sum: 1 } } },
  { $match: { count: { $gt: 5 } } },
  { $sort: { total: -1 } }
]);
```
Note the second `$match` (post-aggregation filter) is the aggregation-pipeline equivalent of SQL's `HAVING`, distinct from a `$match` used pre-`$group` which behaves like `WHERE`.

### Q3. What does `$unwind` do and what's a common performance pitfall with it?
`$unwind` deconstructs an array field, outputting one document per array element (so a document with a 5-element array becomes 5 documents downstream). Pitfall: unwinding a large array multiplies document count dramatically before any filtering, causing memory/performance blowups — always `$match`/filter as much as possible *before* an `$unwind` if you only need a subset of array elements, or use `$filter`/`$elemMatch` projection to shrink the array first instead of unwinding the whole thing.

### Q4. Why should `$match` and `$sort` be placed as early as possible in a pipeline?
Because stages later in the pipeline (after a `$group` or `$project` that reshapes documents) can no longer use collection indexes — only `$match`/`$sort` stages at the *very start* of the pipeline can be satisfied by an index scan (`IXSCAN`) instead of a collection scan (`COLLSCAN`). This mirrors the general query-optimization principle of filtering as early as possible (Topic 6 Q6), just expressed as pipeline stage order instead of planner-chosen join order.

### Q5. What is a multikey index and what's the catch when a document has array fields on more than one indexed path?
A multikey index is automatically created when you index a field that holds an array — MongoDB indexes each array element separately. The catch: **you cannot create a compound multikey index on two array fields at once** (e.g., can't compound-index both `tags` and `categories` if both are arrays) — it would require indexing the Cartesian product of both arrays per document, which MongoDB disallows to avoid index explosion.

### Q6. What is a partial index and when would you use one?
An index that only includes documents matching a specified filter expression, reducing index size and write overhead for fields that are usually null/absent/irrelevant.
```js
db.orders.createIndex(
  { status: 1 },
  { partialFilterExpression: { status: "pending" } }
);
```
Useful when you frequently query "give me all pending orders" out of a mostly-completed/mostly-archived collection — indexing only the small "pending" subset instead of every historical order.

### Q7. What's the difference between `$facet` and running multiple separate aggregation queries?
`$facet` runs multiple independent sub-pipelines within a *single* aggregation call over the same input documents, returning all results together in one round-trip — e.g., simultaneously computing a paginated result set and a total count and a set of category facets/counts for a search UI, in one query instead of three. Tradeoff: all sub-pipelines run against the same initial document set (post any preceding `$match`), so it's efficient when you already need to scan/filter that set once regardless.

### Q8. How do you handle pagination efficiently in MongoDB for a large collection — what's wrong with `.skip()`?
`.skip(n)` still has to walk and discard the first `n` documents internally — for deep pagination (page 10,000) this gets progressively slower (O(n) cost per page). Better: **range/cursor-based pagination** — keep the sort key (e.g., `_id` or `createdAt`) of the last document seen, and query `{ createdAt: { $lt: lastSeenValue } }` with a `$sort` + `$limit`, which can use an index for O(log n) seek regardless of how deep you paginate.

### Q9. How does `explain()` on an aggregation pipeline help you optimize it?
`db.collection.aggregate([...]).explain("executionStats")` shows per-stage execution info, including whether early `$match`/`$sort` stages used `IXSCAN` vs fell back to `COLLSCAN`, `totalDocsExamined` vs `nReturned` per stage, and time spent per stage — lets you pinpoint exactly which stage is the bottleneck (e.g., a `$lookup` doing a full collection scan on the foreign collection because `foreignField` isn't indexed) rather than guessing at the whole pipeline.

### Q10. When would you reach for the aggregation framework instead of a simple `find()` query?
Whenever you need computation across documents (`$group`/`$sum`/`$avg`), reshaping output beyond simple field selection, joining data (`$lookup`), or multi-stage transforms that a flat filter+projection can't express — e.g., "top 5 products by revenue this month" needs grouping and sorting server-side; a plain `find()` can only filter/project/sort a single collection's documents without computed aggregates.

**Red flags to avoid saying:**
- Putting `$match`/`$sort` late in a pipeline and expecting index usage anyway.
- Not knowing `.skip()` degrades on deep pagination — a very common real-world Mongo perf bug.
- Confusing a pre-`$group` `$match` (acts like WHERE) with a post-`$group` `$match` (acts like HAVING).

---

## 12. Replication (MySQL / PostgreSQL / MongoDB)

**Primer:** Replication copies data from a primary node to one or more secondary nodes for
high availability, read scaling, and disaster recovery. Interviewers focus on *consistency
tradeoffs* (sync vs async) and *failover mechanics*.

### Q1. What's the difference between synchronous and asynchronous replication?
**Asynchronous**: the primary commits and acknowledges the client immediately, replicating to secondaries in the background — fast writes, but a primary crash can lose the last few unreplicated transactions. **Synchronous**: the primary waits for at least one secondary to confirm it has received (and possibly applied) the write before acknowledging the client — no data loss on failover, but every write pays the network round-trip latency to the replica. Most production setups default to async for throughput and accept a small data-loss window, or use semi-sync as a middle ground.

### Q2. What is semi-synchronous replication (MySQL) and what problem does it solve?
The primary waits for **at least one** replica to acknowledge it has *received* (not necessarily applied) the transaction's binlog event before acknowledging the client — a middle ground between full async (fast, riskier) and full sync (safe, slow). Solves the specific problem of async replication's failover data-loss window, without paying the cost of waiting for full apply on every replica.

### Q3. Explain how MySQL replication works at a mechanical level (binlog-based).
The primary writes every data-changing statement/row-change to the **binary log (binlog)**. Each replica's I/O thread connects to the primary and streams binlog events into its own **relay log**, then a separate SQL thread applies those events to the replica's data. Modern MySQL supports both statement-based (log the SQL) and row-based (log the actual row changes) binlog formats — row-based is now the default because it avoids replication drift from non-deterministic statements (e.g., `NOW()`, `UUID()`).

### Q4. How does PostgreSQL streaming replication differ mechanically from MySQL's binlog replication?
Postgres streams the **WAL (Write-Ahead Log)** itself (Topic 9 Q4) rather than a separate logical binlog — replicas replay the exact same physical WAL records the primary generated for crash recovery, making it inherently a byte-for-byte physical replica. Postgres also supports **logical replication** (decode WAL into row-level changes and replicate selectively, e.g., only specific tables, or into a different schema/version) as a separate, newer mechanism on top of the same WAL foundation.

### Q5. What is a MongoDB replica set and how does primary election work?
A group of `mongod` instances holding the same data set — one **primary** accepts all writes, secondaries replicate via the primary's **oplog** (operation log, conceptually similar to a binlog) and can serve reads. If the primary becomes unreachable, the remaining members hold an election (based on the Raft-derived consensus protocol) and promote the secondary with the most up-to-date oplog (among eligible/highest-priority members) to primary — typically completing within seconds.

### Q6. What is replication lag, what causes it, and how do you monitor it?
The delay between a write applying on the primary and being applied on a replica. Causes: replica hardware/I/O being slower than the primary's write rate, a single-threaded replication apply path struggling to keep up with highly parallel primary writes (historically a real MySQL limitation before multi-threaded replication), or network issues. Monitor via `SHOW REPLICA STATUS` (`Seconds_Behind_Source`) in MySQL, `pg_stat_replication` (`replay_lag`) in Postgres, `rs.printSecondaryReplicationInfo()` in MongoDB. Matters directly for read-replica correctness (Topic 9 Q7).

### Q7. What is read-your-writes consistency and how do you achieve it when reads are split across replicas?
The guarantee that a client sees its own just-written data on a subsequent read, even in a replicated, eventually-consistent-for-replicas setup. Techniques: route the read-after-write back to the primary for a short window; track a session's last-written log position/timestamp and only route it to a replica that has caught up past that point (MongoDB's **causal consistency sessions** do this natively via `session.startTransaction`/read concern "majority" + causal tokens); or simply always read critical-path data from the primary and only offload non-critical/analytics reads to replicas.

### Q8. What's the difference between a hot standby, a warm standby, and a cold standby?
**Hot standby**: continuously replicating and can take over near-instantly (and often serves reads meanwhile) — e.g., a Postgres streaming replica. **Warm standby**: replicating but not immediately promotable/serving traffic, needs a short activation step. **Cold standby**: infrastructure exists but isn't actively replicating — restored from periodic backups only when needed, with the largest recovery time and data-loss window (largest RTO/RPO, see Topic 19).

### Q9. What's the difference between replication and clustering/multi-primary setups, and what's the tradeoff of allowing writes on multiple nodes?
Standard replication is single-primary — one node accepts writes, others are read-only copies, avoiding write conflicts by construction. **Multi-primary/multi-master** setups (e.g., Galera Cluster for MySQL, or MongoDB doesn't natively support multi-primary within one replica set) allow writes on multiple nodes for lower write latency across regions, but must resolve write conflicts when the same row/document is modified concurrently on two primaries — adds real complexity (conflict resolution strategy, potential lost updates) that most teams should avoid unless they have a specific, well-understood need for multi-region write availability.

### Q10. A candidate says "we use a read replica so reads never affect write performance" — what's the flaw in that framing, and what would you correct?
It ignores that the replica still has to apply every write from the primary — under heavy replica read load, the replication-apply thread can fall behind (Q6), which delays how fresh replica reads are, and in synchronous/semi-sync setups can even slow down primary commits waiting on replica acknowledgment. I'd correct it to: "reads on a replica don't compete with writes on the primary for the same resources, but the replica still has a hard dependency on keeping up with write-apply, and lag needs active monitoring."

**Red flags to avoid saying:**
- Assuming replication is always synchronous by default (most production defaults are async).
- Not knowing what replication lag is or how it's monitored per database.
- Claiming read replicas have zero relationship to write load.

---

## 13. Sharding & Partitioning

**Primer:** Partitioning splits a large table/collection into smaller pieces *within one
node*; sharding splits data *across multiple nodes*. Both solve the same root problem — a
single node/table too big or too hot to serve efficiently — but at different scopes.

### Q1. What's the difference between partitioning and sharding?
**Partitioning**: dividing one logical table into multiple physical pieces on the *same* database instance (e.g., Postgres declarative partitioning, MySQL `PARTITION BY`) — improves query performance and maintenance (e.g., dropping an old partition instead of a slow `DELETE`) but doesn't add capacity beyond one node. **Sharding**: distributing data across *multiple* database instances/nodes, each holding a subset — adds horizontal write/storage capacity beyond a single machine's limits. You can combine both — shard across nodes, and partition within each shard.

### Q2. What are the common partitioning strategies and give a use case for each?
- **Range partitioning**: split by a value range, e.g., `orders` partitioned by month — great for time-series data where old partitions can be archived/dropped wholesale.
- **List partitioning**: split by discrete values, e.g., partition by `region` (`US`, `EU`, `APAC`).
- **Hash partitioning**: split by `hash(key) % n` — spreads load evenly when there's no natural range/list grouping, e.g., partition `users` by `hash(user_id)` to avoid hotspotting.

### Q3. What is a shard key and why is choosing it the single most important sharding decision?
The field(s) used to determine which shard a document/row lives on. It's the most important decision because it's expensive/operationally painful to change later (requires redistributing data), and a bad choice causes either **hotspotting** (all traffic/writes hitting one shard because the key isn't well-distributed — e.g., sharding on a monotonically increasing timestamp puts all *new* writes on the same shard) or **scatter-gather queries** (queries that don't include the shard key must fan out to every shard, losing the performance benefit of sharding).

### Q4. Explain how MongoDB sharding works at a high level (shard key, chunks, mongos, config servers).
Data is split into **chunks** (contiguous ranges of shard-key values), distributed across **shards** (each a replica set). A **mongos** router process sits in front, routing each query to the relevant shard(s) based on the shard key (or scatter-gathering to all shards if the query doesn't include it), consulting **config servers** which store the chunk-to-shard mapping. MongoDB automatically rebalances chunks across shards as data grows unevenly (the "balancer").

### Q5. What's the difference between range-based and hash-based shard keys in MongoDB, and the tradeoff?
**Ranged**: chunks are contiguous ranges of the shard key's value — enables efficient range queries on the shard key (query hits only relevant shards) but risks hotspotting if writes cluster at one end of the range (e.g., an ever-increasing `_id`/timestamp key means all new inserts go to the highest chunk/shard). **Hashed**: MongoDB hashes the shard key value to determine placement — spreads writes evenly, eliminating hotspotting, but a range query on the original field now has to scatter-gather across all shards since the hash destroys ordering.

### Q6. How would you shard a MySQL/Postgres database, given neither has fully automatic native sharding like MongoDB?
Typically done at the **application layer** (or via a proxy like Vitess for MySQL, or Citus for Postgres): pick a shard key (e.g., `tenant_id` for a multi-tenant SaaS), maintain a mapping of key-range/hash → physical database instance, and route each query to the correct instance in application code or via the proxy. Cross-shard joins/transactions become the app's problem to avoid or explicitly handle (Topic 24) — this operational complexity is exactly why teams delay sharding until vertical scaling and read replicas are truly exhausted.

### Q7. What's resharding and why is it painful in production?
Changing the shard key or redistributing data across a different number of shards (e.g., growing from 4 shards to 8) — requires moving large volumes of data between nodes while the system stays live, careful chunk-migration coordination (MongoDB automates this via its balancer, but it's still I/O-intensive and can impact live traffic if not rate-limited), and updating the routing layer. This is the concrete reason shard-key choice (Q3) is treated as a near-irreversible decision.

### Q8. Give a concrete example of a bad shard key choice and explain exactly why it fails.
Sharding a `messages` collection by `conversationId` seems reasonable until one viral/group conversation gets a disproportionate share of all messages — that single shard becomes a hotspot for both reads and writes (a "jumbo chunk" in MongoDB terms that can't be split further because it's one logical key value). Better: a **compound shard key** like `{ conversationId: 1, _id: 1 }`, or hash the key, to spread even a single hot conversation's writes/data more evenly, if the access pattern allows it.

### Q9. What is partition pruning and why does it matter for query performance?
When a query's `WHERE` clause includes the partition key, the planner can skip scanning partitions that can't contain matching rows entirely (e.g., `WHERE order_date >= '2026-01-01'` on a table range-partitioned by month only scans this year's partitions). This is the main performance win of partitioning for large time-series-style tables — without a partition-key filter, the query must still scan every partition, gaining nothing (and sometimes costing more due to partition-management overhead).

### Q10. When would you say "we don't need sharding yet" in an interview, and what would you do instead?
"I'd exhaust vertical scaling (bigger instance), read replicas for read-heavy load, caching (Topic 17), query/index optimization, and single-node partitioning first — sharding adds significant operational complexity (cross-shard queries, resharding risk, more moving parts to operate) and should be justified by an actual measured bottleneck — write throughput or storage exceeding what a single primary can handle — not adopted preemptively 'for scale' before it's needed."

**Red flags to avoid saying:**
- Using "sharding" and "partitioning" interchangeably without acknowledging they're different scopes (single-node vs multi-node).
- Suggesting sharding as a default/early architecture decision rather than a last resort after simpler scaling options.
- Not being able to explain why shard key choice is hard to change later.

---

## 14. CAP Theorem & Consistency Models

**Primer:** CAP theorem describes a fundamental tradeoff *during a network partition* —
not an always-on tradeoff, which is the single most common misunderstanding candidates
have. Getting this nuance right is a strong signal in senior interviews.

### Q1. State CAP theorem precisely — what does it actually claim?
During a **network partition** (P — nodes can't communicate), a distributed system must choose between **Consistency** (every read sees the latest write, or an error) and **Availability** (every request gets a non-error response, possibly with stale data). It does NOT claim you must always sacrifice one of the three in normal operation — partition tolerance is essentially mandatory for any distributed system (networks *will* partition), so CAP is really "C vs A, specifically when P is happening."

### Q2. Why is "partition tolerance" not really a choice in practice?
Because network partitions happen regardless of what you want (switch failures, packet loss, GC pauses making a node unreachable) — a distributed system that isn't partition-tolerant simply stops working correctly the moment a partition occurs. So real systems design for CP or AP behavior *during* a partition; they don't get to opt out of partition tolerance by choice.

### Q3. Classify MongoDB, and explain how its configurability affects a simple "CP or AP" label.
MongoDB defaults toward **CP** behavior — a replica set with no reachable primary (e.g., isolated in a network partition minority) refuses writes rather than accepting them inconsistently, and read concern "majority" only returns data acknowledged by a majority of replicas. But it's tunable: you can set write concern `w:1` (faster, less safe) or configure reads to hit any secondary regardless of staleness, shifting the practical behavior more toward availability at the cost of consistency guarantees.

### Q4. Classify a typical single-primary Postgres/MySQL setup under CAP.
Effectively **CP**-leaning: if the primary is unreachable, writes stop (or a failover promotes a new primary after detecting the partition, briefly unavailable during the transition) rather than accepting writes on multiple partitioned nodes and risking conflicting data. A multi-region, multi-primary setup would shift this tradeoff and require explicit conflict resolution (Topic 12 Q9) — the CAP classification depends on the actual replication topology, not the DB product name alone.

### Q5. What's the difference between strong consistency and eventual consistency, with a concrete example of each?
**Strong consistency**: any read after a write is guaranteed to return that write's value (or newer) — e.g., reading immediately from a single primary. **Eventual consistency**: after a write, reads *may* return stale data for some period, but will converge to the latest value once replication catches up — e.g., reading from a lagging replica, or an eventually-consistent system like early DynamoDB/Cassandra defaults. Example: posting a comment and refreshing immediately might not show it yet on an eventually consistent read replica, but will a few hundred ms later.

### Q6. What is "read your own writes" consistency and how is it different from strong consistency in general?
A specific, weaker guarantee than full strong consistency: a *given client/session* is guaranteed to see its own writes on subsequent reads, but makes no promise about what *other* clients see or when. It's a practical middle ground — cheaper to provide than global strong consistency, and covers the most common user-facing complaint ("I just posted this, why can't I see it?"). Implemented via sticky sessions to the primary, or causal-consistency tokens (Topic 12 Q7).

### Q7. What does PACELC add on top of CAP, and why is it considered a more complete/practical model?
PACELC extends CAP: "if **P**artitioned, choose **A** or **C**; **E**lse (normal operation, no partition), choose **L**atency or **C**onsistency." It captures that even *without* a partition, there's still a real tradeoff — synchronous replication for strong consistency adds latency to every write, even when everything is healthy. CAP alone only describes partition-time behavior; PACELC is considered more useful because it acknowledges the everyday latency/consistency tradeoff systems make constantly, not just during rare partition events.

### Q8. What does "quorum" mean and how does it relate to consistency tuning (e.g., in MongoDB's write concern or Cassandra-style systems)?
A quorum is "more than half" of the replica set — requiring a write (or read) to be acknowledged by a quorum guarantees any subsequent quorum read overlaps with at least one node that has the latest write, mathematically guaranteeing strong-enough consistency without needing *all* nodes to respond (which would sacrifice availability if even one node is slow/down). MongoDB's `w: "majority"` write concern is a quorum write; reading with read concern `"majority"` is a quorum read.

### Q9. How would you explain, in an interview, the real-world engineering decision behind choosing eventual consistency for a "like count" but strong consistency for an "account balance"?
"The cost of being wrong differs wildly: a like count showing 4,982 instead of 4,983 for a second causes zero harm and self-corrects, so I'd take the availability/latency win of eventual consistency there. An account balance being briefly wrong could let someone double-spend or see an incorrect balance they act on — the cost of inconsistency is real financial/trust damage, so that data path needs strong consistency (or at minimum a transactionally safe read) even at some latency cost." This shows you make the tradeoff per-feature, not as a single database-wide policy.

### Q10. Candidate says "we use MongoDB so we're eventually consistent" as a blanket statement — what's wrong, and how would you correct it?
It conflates "MongoDB is capable of eventual-consistency behavior in some configurations" with "MongoDB is always eventually consistent," which is wrong — a majority-write, majority-read configuration on the primary gives strong, read-your-writes consistency by default for that data path. I'd correct it to: "consistency is a per-operation configuration choice (write/read concern) in MongoDB, not an inherent, fixed property of the database — same underlying nuance as PACELC."

**Red flags to avoid saying:**
- "You must always sacrifice consistency OR availability" as a blanket, always-on rule (the tradeoff is specifically partition-time).
- Labeling a database "CP" or "AP" as an absolute, ignoring that write/read concern configuration changes the actual behavior.
- Not knowing PACELC exists when discussing the latency/consistency tradeoff outside of partitions.

---

## 15. SQL vs NoSQL — Choosing the Right Database

**Primer:** This topic tests judgment, not trivia — interviewers want to see a structured
decision framework, not "SQL for structured data, NoSQL for unstructured data" (an
oversimplification that undersells both).

### Q1. Give a decision framework you'd actually use to choose between a relational DB and MongoDB for a new service.
Ask, in order: (1) Does the data have strong relational integrity needs (foreign keys, multi-entity transactions, complex joins across many entities)? → leans relational. (2) Is the access pattern dominated by fetching one aggregate/entity at a time with its related data (e.g., a user profile with settings)? → leans document/MongoDB (embed-friendly). (3) What's the write pattern — flexible/evolving schema with frequent new optional fields (leans document) vs strict, well-understood schema (leans relational)? (4) What does the team already operate well? Operational familiarity is a legitimate, often decisive factor.

### Q2. What are the real advantages of a relational DB that don't get mentioned often enough?
Mature, powerful ad-hoc query capability (arbitrary joins/aggregations without pre-planning access patterns), strong multi-row/multi-table transactional guarantees by default, decades of tooling/ORMs/operational knowledge, and enforced referential integrity that catches bugs at write time instead of producing silent orphaned data. The underrated one: **flexibility for query patterns you didn't anticipate at design time** — a normalized schema can answer new questions later; a document schema optimized for one access pattern often can't without a migration.

### Q3. What are the real advantages of MongoDB that go beyond "it's schemaless"?
Horizontal write scaling via native sharding is genuinely easier to operate than sharding most relational databases; the document model maps naturally to nested, hierarchical application objects (avoiding ORM impedance mismatch for that shape of data); fast reads for aggregate-oriented access patterns since related data is co-located in one document (no join needed); and flexible/rapid schema iteration during early-stage development when the data shape is still evolving.

### Q4. Is "NoSQL is faster than SQL" a valid statement? Push back on it.
No — it's not generically true. A well-indexed relational query and a well-indexed MongoDB query for an equivalent single-entity lookup perform comparably; MongoDB's speed advantage shows specifically when it avoids a join a relational schema would require (data is embedded instead), and disappears or reverses when a MongoDB schema needs `$lookup`-heavy joins the relational engine would do better. Speed is a function of *schema and query design matching the engine's strengths*, not the category of database.

### Q5. Describe a scenario where you'd deliberately use both a relational DB and MongoDB in the same system (polyglot persistence).
E.g., an e-commerce platform: relational (Postgres) for orders, payments, inventory — strict transactional integrity, well-defined relationships, financial correctness matters. MongoDB for the product catalog — highly variable attributes per product category (a shirt has size/color, a laptop has RAM/CPU), frequently changing schema, read-heavy with data naturally read as one aggregate per product page. Each store is used where its model fits best rather than forcing one database to do both jobs adequately.

### Q6. What's the tradeoff of running polyglot persistence (multiple database types) that interviewers want you to acknowledge?
Operational complexity roughly doubles (two backup/monitoring/scaling/on-call playbooks, two sets of operational expertise needed on the team), and you lose the ability to do a single cross-store transaction or a single query joining data across both stores — any consistency between them becomes eventual and application-managed (e.g., via events/an outbox pattern, Topic 24). Only justified when the access-pattern mismatch between the two workloads is real and significant, not for its own sake.

### Q7. How would you evaluate whether a "we need NoSQL for scale" claim from a team is actually justified?
Ask what specific bottleneck they've measured — write throughput exceeding a single relational primary's capacity, storage exceeding practical vertical scaling, or query latency from joins that indexing/denormalization/caching can't fix? Relational databases (with read replicas, partitioning, connection pooling, and tools like Citus/Vitess) scale much further than commonly assumed before a genuine architecture change is warranted — "we need NoSQL for scale" is frequently said prematurely, before cheaper scaling levers (Topic 16) have been tried.

### Q8. Give an example of a schema-flexibility need that's genuinely better served by MongoDB than by a relational "EAV" (entity-attribute-value) workaround.
A product catalog with wildly different attribute sets per category could be forced into a relational EAV pattern (a generic `attributes(entity_id, key, value)` table) to fake flexible schema — but EAV loses type safety, makes queries on specific attributes awkward (`WHERE` clauses become joins against the EAV table per attribute), and kills index efficiency. A MongoDB document naturally holds `{ category: "shirt", size: "M", color: "blue" }` vs `{ category: "laptop", ram: "16GB" }` with real per-document indexing on whichever fields matter, without the EAV query pain.

### Q9. What's a common interview trap when discussing "NoSQL doesn't support transactions" — how do you avoid falling into it?
Stating it as an absolute, unqualified fact — outdated for MongoDB (4.0+ supports multi-document ACID transactions, Topic 1 Q9) and inaccurate across "NoSQL" broadly, since the category includes very different systems (document, key-value, wide-column, graph) with different guarantees. Avoid the trap by being specific: name the actual database and version, and describe the actual guarantee rather than a category-wide generalization.

### Q10. How would you answer "so which one do YOU prefer" in an interview without sounding like you're dodging?
Give a real opinion anchored to tradeoffs, not a non-answer: "For most new services I reach for relational by default — the transactional guarantees and flexible querying save time later when requirements shift, and requirements always shift. I'd choose a document DB specifically when I can name the access pattern that benefits from embedding, or when schema volatility during early development is a real, current pain — not preemptively." Shows judgment and a default, while making clear it's a reasoned choice, not dogma.

**Red flags to avoid saying:**
- "SQL is for structured data, NoSQL is for unstructured data" — both engines can model both today; the real distinction is access-pattern and consistency tradeoffs.
- Blanket claims like "NoSQL is faster" or "NoSQL doesn't do transactions" without qualification.
- Recommending a database switch to solve a problem that indexing/caching/query-tuning could fix.

---

## 16. Scaling Databases (Read Replicas, Pooling, Vertical/Horizontal)

**Primer:** Scaling a database is a *sequence of levers*, roughly in order of operational
cost — interviewers want to see you reach for the cheap ones first, not jump straight to
sharding.

### Q1. What's the difference between vertical and horizontal scaling for a database?
**Vertical scaling**: bigger machine — more CPU/RAM/faster disk on the same single instance. Simple (no code/architecture change), but has a hard ceiling (biggest instance a cloud provider offers) and a single point of failure. **Horizontal scaling**: more machines — read replicas, then sharding for writes. Effectively unlimited ceiling, but adds real operational and architectural complexity (Topics 12, 13).

### Q2. What is connection pooling and why is it critical even before you think about scaling hardware?
Each DB connection has real memory/CPU overhead (Postgres: a full OS process per connection by default) — an application opening a new raw connection per request exhausts the DB's max-connection limit under load and wastes resources on connection setup/teardown. A **connection pool** (app-side, like a pool in your DB driver/ORM, or a dedicated pooler like **PgBouncer**/**ProxySQL**) maintains a fixed set of reusable connections that requests borrow and return — often the single highest-leverage fix for "database falls over under load" before any hardware/architecture change is needed.

### Q3. What's the difference between PgBouncer's "session," "transaction," and "statement" pooling modes?
- **Session**: a client holds a real DB connection for its entire session — safest (supports everything, e.g., prepared statements, `LISTEN/NOTIFY`), least connection-multiplexing benefit.
- **Transaction**: the pooled connection is only bound to the client for the duration of one transaction, then returned to the pool — much higher multiplexing (fewer real connections needed), but breaks session-level features (can't hold a `SET` or prepared statement across transactions on a specific connection).
- **Statement**: connection returned after every single statement — highest multiplexing, but breaks multi-statement transactions entirely; rarely used.
Most production setups use transaction mode as the throughput/compatibility sweet spot.

### Q4. How do read replicas actually help scaling, and what workloads do they NOT help?
They add read capacity by letting `SELECT`-heavy traffic (dashboards, reporting, read-heavy API endpoints) hit replicas instead of the primary, freeing the primary's resources for writes. They do NOT help write throughput at all (every replica still has to apply every write, Topic 12 Q10) — a write-bottlenecked system gains nothing from adding read replicas; that requires vertical scaling of the primary or eventual sharding.

### Q5. What's the role of caching in a database scaling strategy, and where does it sit relative to read replicas?
An in-memory cache (Redis/Memcached, Topic 17) in front of the database absorbs read traffic *before* it ever reaches the DB at all — cheaper and faster than even a read replica, since it skips the DB entirely for cache hits. Order of levers for read-heavy scaling, roughly: query/index optimization → caching hot reads → read replicas for the remaining DB-bound read traffic → then sharding only if writes themselves are the bottleneck.

### Q6. What is query/statement timeout and why is it a scaling-adjacent safety mechanism?
A server-side limit (`statement_timeout` in Postgres, `MAX_EXECUTION_TIME` in MySQL) that kills a query running longer than the threshold. Without it, a single runaway/unindexed query under load can hold connections and locks indefinitely, cascading into a full outage as the connection pool fills up with stuck requests. It's a blast-radius control, not a performance optimization itself — worth naming as part of a "how do you keep the DB resilient under load" answer.

### Q7. How would you scale write throughput for a single relational primary before resorting to sharding?
Batch writes instead of many small individual `INSERT`s (fewer round trips + fewer individual commits/fsyncs); tune commit durability tradeoffs where acceptable (`synchronous_commit=off` for non-critical data in Postgres, being explicit this trades some durability); move non-critical writes off the critical path via an async queue (e.g., write an event, process/aggregate it into the DB in the background rather than synchronously per request); ensure indexes aren't over-provisioned (Topic 2 Q6, since every extra index taxes every write). Only after these would sharding (Topic 13) be the next lever.

### Q8. What's the difference between scaling reads and scaling writes, and why is the latter fundamentally harder?
Reads scale horizontally almost for free — replicas are just copies, and you can add as many as you want (network/replication overhead aside) since reads don't need to be serialized against each other. Writes must ultimately be serialized somewhere to maintain consistency — a single primary is the natural serialization point; distributing writes (sharding, multi-primary) means giving up either a single consistent ordering (needing conflict resolution) or accepting cross-shard coordination cost (Topic 24) — there's no equivalent "just add more" free lunch for writes.

### Q9. What role does a load balancer/proxy (like ProxySQL, PgBouncer, or MongoDB's mongos) play in a scaled database architecture?
Beyond connection pooling, these can do **read/write splitting** (route `SELECT`s to replicas, writes to primary, transparently to the app), health-check-based failover routing, and query-level load balancing across replicas — centralizing scaling logic in the infra layer rather than every application service reimplementing "which connection string do I use for a read vs write."

### Q10. Interviewer asks: "Traffic 10x'd overnight and the database is falling over — what do you check and do, in order, in the first hour?"
"First, check connection pool saturation and slow query logs/`pg_stat_activity` for what's actually consuming resources — often one bad query pattern, not genuine capacity exhaustion. Add/verify a statement timeout to stop pile-ups. If it's genuinely read-bound, temporarily point non-critical read traffic at a replica or add caching in front of the hottest endpoints. If it's connection exhaustion, verify pooling is configured correctly before assuming I need more hardware. Only after ruling out these would I do an emergency vertical scale-up of the primary as a stopgap while a real fix (indexing, caching, architecture) is planned." Shows triage instinct over panic-scaling.

**Red flags to avoid saying:**
- Jumping straight to "add a read replica" or "shard it" without mentioning connection pooling/caching/query tuning first.
- Not knowing read replicas don't help write throughput at all.
- Treating vertical and horizontal scaling as equally complex to implement (they're very different in operational cost).

---

## 17. Caching Strategies with Databases

**Primer:** Caching sits between the application and the database to absorb read load and
cut latency. The interview-relevant part is less "what's Redis" and more "which caching
pattern, and what happens on a cache/DB inconsistency."

### Q1. Explain cache-aside (lazy loading) and its main failure mode.
App checks cache first; on a miss, reads from DB, then writes the value into the cache for next time.
```
GET cache[key] -> miss -> GET db[key] -> SET cache[key] = value -> return value
```
Main failure mode: **stale data** — if the DB row changes and nothing invalidates/updates the cache entry, reads keep returning the old value until it naturally expires (TTL) or is explicitly invalidated on write.

### Q2. Explain write-through caching and how it differs from cache-aside on the write path.
Every write goes to the cache *and* the DB together (synchronously, as part of the same write operation), so the cache is never stale relative to the DB for that key. Tradeoff: every write pays the extra cache-write latency, and if the DB write succeeds but the cache write fails (or vice versa), the two can still diverge — needs a consistent ordering (write DB first, then cache, and treat cache-write failure as non-fatal since cache-aside's next read will repopulate it anyway).

### Q3. Explain write-behind (write-back) caching and its risk.
Writes go to the cache immediately (fast) and are asynchronously flushed to the DB later in batches. Fast writes, reduced DB write load (can coalesce multiple updates to the same key into one DB write). Risk: **data loss** if the cache node crashes before the async flush completes — only acceptable for data where some loss is tolerable, or when the cache layer itself is made durable/replicated.

### Q4. How do you invalidate a cache entry correctly when the underlying DB row is updated?
Either (a) delete the cache key on write and let the next read repopulate it (cache-aside's natural invalidation — generally preferred, simpler to reason about than updating the cache value directly, avoids races where the cache update and DB update use different logic), or (b) update the cache value directly (write-through). The dangerous anti-pattern: updating the DB but forgetting to touch the cache at all on one of several write code paths — exactly the same "keep denormalized copies in sync" risk as Topic 3 Q9, just at the cache layer.

### Q5. What is the "thundering herd" / cache stampede problem and how do you prevent it?
When a popular cache key expires, many concurrent requests all miss simultaneously and all hammer the DB to repopulate it at once — can spike DB load hard enough to cause an outage from what should've been a routine cache expiry. Prevent via: a short-lived **lock/mutex per key** so only one request repopulates while others wait briefly for the fresh value; **stale-while-revalidate** (serve the slightly-stale value while one request refreshes in the background); or staggering TTLs with jitter so many keys don't expire at the exact same instant.

### Q6. How would you decide a cache TTL for a given piece of data?
Balance staleness tolerance against DB load: data that changes rarely and where staleness is low-cost (e.g., a product category list) gets a long TTL (hours); data that changes often or where staleness is costly (e.g., inventory count near checkout) gets a short TTL or event-driven invalidation instead of relying on TTL alone. State the tradeoff explicitly in an interview rather than picking an arbitrary number: "TTL is a knob between cache-hit-rate/DB-load and staleness — I'd tune it per data type, not globally."

### Q7. What's the difference between caching at the application layer (Redis) vs the database's own internal caching (buffer pool/WiredTiger cache)?
The DB's internal cache (Topic 8 Q3) caches raw pages/indexes in memory but still pays query-execution cost (planning, locking, scanning) on every request. An external cache (Redis) typically stores the *final computed result* (e.g., a rendered API response or aggregated value), skipping query execution entirely on a hit — much cheaper per request, but now you own a second system's consistency and operational burden. They're complementary, not redundant: the DB cache helps every query including cache-miss fallthroughs; the app cache avoids hitting the DB at all for hot keys.

### Q8. How would you cache a paginated or frequently-changing list (e.g., "latest 20 orders") without serving badly stale data?
Cache with a short TTL rather than long-lived, since the "top N" list changes on every new write to the underlying set (naive key-based invalidation would mean invalidating the cache on every single new order, which is nearly as often as just not caching). Alternatively cache stable *sub-parts* (e.g., cache each individual order by ID, generate the list by fetching IDs from a short-lived/uncached "latest IDs" query then batch-hydrating from the per-ID cache) — reduces DB load for the expensive part (fetching full order data) while keeping the ordering fresh.

### Q9. What's the risk of caching database query results keyed by the raw SQL query string, and what's a better key strategy?
Any minor query variation (different whitespace, different parameter values, different column order) produces a different cache key even when semantically overlapping — leads to poor cache-hit rates and duplicated cached data. Better: key by a normalized, explicit identifier of *what* is being fetched (e.g., `user:1234:profile`) decoupled from the literal SQL, so the cache key's meaning is stable even if the underlying query implementation changes.

### Q10. Interviewer says "we added Redis caching but our database load didn't go down" — what would you check?
Check actual cache hit rate first — a low hit rate (wrong TTL, cache key too specific/fragmented, or caching data that's rarely re-requested) means most requests still fall through to the DB regardless of the cache existing. Check whether the cache is actually in the request path for the hot endpoints causing load, versus caching a different, less-hot query. Check for a stampede pattern (Q5) periodically wiping the cache's benefit. Most "caching didn't help" incidents trace back to caching the wrong thing or a low hit rate, not caching being ineffective in principle.

**Red flags to avoid saying:**
- Describing caching without mentioning invalidation strategy — writing the cache-read path without a coherent write-side invalidation plan is the most common real bug.
- Not knowing what a cache stampede is when discussing TTL-based expiry at scale.
- Assuming "add Redis" automatically reduces DB load without checking actual hit rate.

---

## 18. Schema Migrations & Evolution

**Primer:** Migrations are how schema changes get applied safely to a live production
database. Senior interviews care most about **zero-downtime migration technique** — the
gap between "I know the SQL" and "I've actually done this on a table with 500M rows."

### Q1. Why can a seemingly simple `ALTER TABLE ADD COLUMN` cause an outage on a large table?
Historically (and still in some engines/cases), adding a column required rewriting every existing row/page to include the new column, holding a table-level lock for the full duration — on a huge table this could take minutes, during which all reads/writes to that table block, causing an outage. Modern Postgres (11+) avoids this for a column with a constant or no default (metadata-only change), but MySQL's behavior varies by version/algorithm (`ALGORITHM=INSTANT` in MySQL 8 handles simple adds instantly; other changes still require `INPLACE` or full table copy) — always the right answer to say "it depends on the specific change and engine version, and I'd verify with `EXPLAIN`/docs before running it on a large prod table."

### Q2. What's the "expand-contract" (parallel change) pattern for migrations, and why is it preferred for breaking changes?
A three-phase pattern for changes that would otherwise require simultaneous code+schema deployment (impossible to do atomically with rolling deploys): **Expand** — add the new column/table alongside the old one, deploy code that writes to both; **Migrate** — backfill historical data into the new shape, deploy code that reads from the new shape; **Contract** — once fully cut over and verified, drop the old column/table. Preferred because at every step, both old and new application code versions (which coexist during a rolling deploy) work correctly against the current schema.

### Q3. Walk through, concretely, how you'd rename a column on a large, high-traffic table with zero downtime.
Never a direct `RENAME COLUMN` cutover on a live system with rolling deploys. Steps: (1) add the new column; (2) deploy code that dual-writes to both old and new columns; (3) backfill the new column from the old one in small batches (to avoid long locks/replication lag spikes); (4) deploy code that reads from the new column (still dual-writing as a safety net); (5) verify correctness/monitor; (6) deploy code that stops writing to the old column; (7) drop the old column in a later migration once confident. This is the expand-contract pattern applied concretely.

### Q4. What's the risk of adding a `NOT NULL` constraint to an existing column on a large table, and how do you do it safely in Postgres?
A plain `ALTER TABLE ... ALTER COLUMN ... SET NOT NULL` requires scanning the whole table to verify no existing NULLs, holding a lock for that duration. Safer path in Postgres: add a `CHECK` constraint with `NOT VALID` first (takes only a brief lock to add, doesn't validate existing rows immediately), then `VALIDATE CONSTRAINT` separately (scans the table but only takes a lesser lock, `SHARE UPDATE EXCLUSIVE`, that doesn't block reads/writes) — achieving the same guarantee with far less blocking than the naive one-step approach.

### Q5. Why should large data backfills be done in small batches rather than one big `UPDATE`?
A single massive `UPDATE` touching millions of rows holds locks on all of them for the statement's entire duration (blocking other writers to those rows), generates a huge amount of WAL/binlog/undo-log at once (Topic 8 Q7), and risks replication lag spiking on replicas trying to apply that one huge change. Batching (e.g., `UPDATE ... WHERE id BETWEEN x AND x+1000`, looping with small pauses) bounds lock duration and replication impact per batch, at the cost of the overall migration taking longer wall-clock time — an acceptable, deliberate tradeoff for production safety.

### Q6. How do migration tools (Flyway, Liquibase, Django/Rails migrations, Alembic) generally guarantee migrations run safely across a team/CI?
They maintain a version/history table in the DB itself recording which migrations have already run, apply pending migrations in a defined order, and typically wrap each migration in a transaction (where the DB supports transactional DDL — Postgres does; MySQL's DDL is mostly non-transactional) so a failed migration doesn't leave the schema half-changed. They also enable code review of schema changes as version-controlled files, and reproducible environments (spin up a fresh DB and replay all migrations to get an identical schema).

### Q7. What's the difference between how Postgres and MySQL handle DDL transactionality, and why does it matter operationally?
Postgres supports **transactional DDL** — you can run multiple schema changes inside a `BEGIN...COMMIT` and roll the whole thing back atomically if something fails partway. MySQL's DDL is largely **non-transactional** — most `ALTER TABLE` statements implicitly commit and can't be rolled back as part of a larger transaction; a failed multi-step MySQL migration can leave the schema in a partially-applied state that needs manual/tooled cleanup. This is a genuine, frequently-tested distinction that surprises engineers coming from Postgres to MySQL.

### Q8. How do you handle a MongoDB schema migration given there's no `ALTER TABLE` equivalent?
Since documents don't share an enforced schema, "migration" usually means either (a) **migrate-on-read**: application code handles both old and new document shapes at read time, converting old shape to new on the fly, with new writes always in the new shape (no big-bang migration needed, but code carries dual-shape-handling debt until fully migrated); or (b) a **background migration script** that iterates the collection in batches (`find().batchSize()` cursors, not loading the whole collection into memory) and rewrites documents to the new shape, tracked via a `schemaVersion` field so it's resumable/idempotent if interrupted.

### Q9. What's a rollback plan and why does every migration need one before it runs in production?
A rollback plan is the explicit, tested procedure to reverse a migration's effect if something goes wrong after deployment — for additive changes (new column/table) this is often trivial (just don't use it yet, or drop it), but for destructive changes (dropped column, changed type) a naive rollback can't recover data that's already been discarded. This is exactly why the expand-contract pattern (Q2) defers the destructive "contract" step until you're fully confident — it keeps a cheap rollback path available for as long as possible.

### Q10. Interviewer asks you to migrate a `status` column from a `VARCHAR` to an `ENUM`/reference table on a live table with billions of rows and no maintenance window — outline your approach out loud.
"I'd treat this as expand-contract: add a new `status_id` column referencing a new `statuses` lookup table; deploy code that writes both `status` (old) and `status_id` (new) on every write; backfill `status_id` for existing rows in small batches, throttled to avoid replication lag/lock contention, during low-traffic windows; once backfilled and verified consistent, deploy code that reads from `status_id`; monitor for a bake-in period; then stop dual-writing and finally drop the old `status` column in a later, separate migration once I'm confident there's no rollback need." This demonstrates the full pattern applied to a realistic prompt, which is usually exactly what's being tested.

**Red flags to avoid saying:**
- Proposing a single-step, blocking schema change on a "large table, zero downtime" prompt without mentioning expand-contract or batching.
- Not knowing MySQL DDL is largely non-transactional while Postgres's is.
- Forgetting a rollback plan when asked to walk through a risky migration.

---

## 19. Backup, Recovery & High Availability

**Primer:** Backups protect against data loss; high availability (HA) protects against
downtime — related but distinct concerns interviewers expect you to keep separate. Two
numbers anchor this whole topic: **RPO** and **RTO**.

### Q1. Define RPO and RTO and explain why they drive backup/HA architecture decisions.
**RPO (Recovery Point Objective)**: maximum acceptable data loss, measured in time (e.g., "we can lose at most 5 minutes of writes") — drives how frequently you back up/replicate. **RTO (Recovery Time Objective)**: maximum acceptable downtime to restore service (e.g., "we must be back up within 15 minutes") — drives whether you need hot standbys (fast failover) versus restoring from cold backups (slow). Every backup/HA design decision should trace back to an explicit RPO/RTO the business actually needs — over-engineering (e.g., synchronous multi-region replication for an internal admin tool) wastes money and complexity; under-engineering risks real incidents.

### Q2. What's the difference between a full backup, an incremental backup, and continuous WAL/binlog archiving, in terms of RPO?
**Full backup**: complete snapshot at a point in time — restoring gets you to exactly that moment (RPO = time since last full backup, could be up to 24h for nightly backups). **Incremental**: only changes since the last backup — faster/cheaper to take, but restore requires replaying the full chain (last full + all incrementals). **Continuous WAL/binlog archiving**: combined with a base backup, lets you replay to *any* point in time (point-in-time recovery, PITR) — RPO shrinks to essentially zero (or the archiving interval) since every committed transaction is captured.

### Q3. What is point-in-time recovery (PITR) and walk through how it works in Postgres.
Restoring the database to any specific timestamp (e.g., "5 minutes before the bad deploy ran a destructive query") rather than only to the moment of the last full backup. Mechanically: restore the most recent base backup taken before the target time, then replay archived WAL segments forward from that backup up to (but not past) the target timestamp using `recovery_target_time`. This is why continuous WAL archiving (Q2) is a prerequisite — without it you can only restore to full-backup granularity.

### Q4. What's the difference between backing up a database and having a replica — why isn't a replica a substitute for backups?
A replica protects against **hardware/node failure** (failover to a healthy copy) but not against **logical/human errors** — a bad `DELETE`, a buggy migration, or an application bug that corrupts data replicates to the replica near-instantly too, since replication faithfully copies every change including mistakes. Backups (especially with PITR) are what let you go back to a point *before* the mistake happened — the two solve different failure classes and you need both.

### Q5. How would you test that your backups actually work, and why is this step commonly skipped until it's too late?
Regularly perform an actual **restore drill** — spin up a new instance from the backup and verify the data is complete/consistent/queryable, ideally as an automated, scheduled job, not a manual one-off. It's commonly skipped because backups "running successfully" (the job exits 0) gives false confidence — a corrupted backup file, a missing WAL segment breaking PITR continuity, or a permissions issue on the restore target often only surfaces during an actual incident, which is the worst possible time to discover it.

### Q6. What's automatic failover and what are the risks of a poorly implemented one?
The system detects primary failure and automatically promotes a replica to primary without human intervention, minimizing RTO. Risks: **false positives** (a network blip mistaken for a real outage triggers unnecessary failover, causing its own brief disruption); **split-brain** (if the old primary comes back online without realizing it's been demoted, two nodes both think they're primary and accept writes, causing data divergence) — mitigated via **fencing** (forcibly ensuring the old primary can't accept writes, e.g., STONITH — "shoot the other node in the head") and consensus-based election requiring a majority (Topic 12 Q5's MongoDB election mechanism is a concrete example of avoiding split-brain via quorum).

### Q7. What is a multi-AZ vs multi-region HA setup, and what different failure modes does each protect against?
**Multi-AZ** (availability zone — physically separate data centers within one cloud region, low-latency links between them): protects against a single data center failure (power, hardware, rack-level issues) — typically supports synchronous or near-synchronous replication since latency is low. **Multi-region**: protects against an entire region going down (a rarer, bigger event) — cross-region latency is much higher, usually forcing asynchronous replication, meaning a region failover risks losing the last few seconds/minutes of unreplicated writes (a real RPO cost that must be explicitly accepted).

### Q8. How does MongoDB's replica set election interact with HA design — what's the minimum topology to survive one node failure without manual intervention?
A 3-node replica set (1 primary + 2 secondaries, or spread across 3 AZs) is the minimum that can survive one node failure while still maintaining a majority for elections and majority-write-concern acknowledgment — a 2-node set can't safely auto-elect after one node fails, since a lone remaining node can't get a majority vote by itself (this is why MongoDB recommends odd numbers or an arbiter for tie-breaking when full data nodes aren't justified).

### Q9. What's the difference between "backup retention policy" and "compliance/legal hold" requirements, and why might they conflict?
A retention policy is an operational/cost decision (e.g., "keep 30 days of backups, then delete to save storage"). Compliance or legal-hold requirements (e.g., financial records, active litigation) can mandate retaining specific data far longer than the operational policy would otherwise keep it — a conflict that needs an explicit process (tagging/excluding certain backups from automated deletion) rather than assuming a single retention number covers every case; worth mentioning as a real cross-functional concern beyond pure engineering.

### Q10. Interviewer: "Someone ran a destructive `UPDATE` without a `WHERE` clause on production 10 minutes ago — walk me through recovery."
"First, stop further writes to the affected table/rows if possible to prevent compounding the damage. Check if the mistaken statement was wrapped in a transaction that could still be rolled back (unlikely if already committed/10 minutes passed). Assuming it's committed: use point-in-time recovery — restore the most recent base backup to a *separate* instance, replay WAL up to just before the bad statement's timestamp, then extract/restore just the affected data from that recovered instance back into production, rather than restoring the whole production database over live traffic (which would lose the other 10 minutes of legitimate writes on unaffected tables). Afterward, add a safeguard — e.g., requiring explicit confirmation or a linter/proxy that blocks unscoped `UPDATE`/`DELETE` in production tooling." Shows both the technical recovery mechanism and the practical judgment to avoid making things worse.

**Red flags to avoid saying:**
- Treating a replica as equivalent to a backup for protecting against human/application error.
- Not knowing what RPO/RTO mean when discussing backup strategy.
- Proposing a full-database restore-over-production as the first response to a partial data-corruption incident.

---

## 20. Deadlocks & Performance Troubleshooting

**Primer:** This is the "live debugging" topic — interviewers often present a symptom
("queries are slow," "the app is hanging") and want to see your diagnostic process, not
just definitions.

### Q1. Walk through your general framework for diagnosing "the database is slow" from scratch.
"First, define slow precisely — which query/endpoint, since when, how much slower than baseline. Then check for the obvious high-signal causes in order: active long-running/blocking sessions (`pg_stat_activity`/`SHOW PROCESSLIST`), lock waits, a recent deploy/migration/data-shape change, resource saturation (CPU/memory/disk I/O/connections) via infra metrics, and replication lag if reads are affected. Only after ruling those out would I dig into a specific query's `EXPLAIN ANALYZE`." The framework itself — check cheap, high-signal things before deep query-level analysis — is what's being evaluated, not just the list.

### Q2. What's the difference between a query that's slow because of a bad plan vs one that's slow because of contention/locking — how do you tell them apart?
A **bad-plan** slow query is consistently slow regardless of concurrent load (check with `EXPLAIN ANALYZE` in isolation) — the fix is indexing/rewriting the query. A **contention** slow query runs fine in isolation but slows dramatically under concurrent load because it's waiting on locks held by other transactions — check `pg_locks`/`SHOW ENGINE INNODB STATUS` for lock waits, not the query plan itself, since the plan is fine; the wait time is what's inflating perceived latency.

### Q3. Explain the exact mechanics of a MySQL InnoDB deadlock error and what the app should do when it receives one.
InnoDB's deadlock detector runs a wait-for graph check; upon finding a cycle, it immediately aborts one transaction (rolling it back completely) and returns error 1213 (`ER_LOCK_DEADLOCK`) to that transaction's client — the *other* transaction(s) in the cycle proceed normally. The app must catch this specific error code and **retry the entire aborted transaction from the start** (not just the failed statement) after a short backoff, since the transaction was fully rolled back, not partially.

### Q4. How would you read `SHOW ENGINE INNODB STATUS` (or Postgres's equivalent) to find the actual cause of a recent deadlock?
MySQL's `SHOW ENGINE INNODB STATUS` includes a `LATEST DETECTED DEADLOCK` section showing both transactions involved — their SQL statements, which locks each held and which each was waiting for. Read it to identify the specific rows/tables and lock types in conflict, then trace back to the application code paths issuing those statements to find where lock-ordering (Topic 5 Q7) was violated. Postgres logs deadlock details to the server log (with `log_lock_waits`/deadlock detection) rather than a queryable status command, containing similar process/query/lock information.

### Q5. What's a common application-level cause of deadlocks that has nothing to do with the database itself?
Two code paths acquiring the same set of rows/tables in a different order — e.g., a "transfer funds" function locking account A then B, while a separate "merge accounts" function locks B then A; under concurrent execution these can deadlock even though each function is individually correct. The fix is a lock-ordering convention (Topic 5 Q7: always lock by ascending ID, or acquire locks in the same canonical order everywhere) enforced across the whole codebase, not just the two functions someone happened to notice.

### Q6. How do you distinguish a genuine deadlock from a session simply stuck waiting a long time (lock wait timeout) with no cycle?
A deadlock is detected proactively and near-instantly by the DB's cycle detector — no timeout period elapses. A plain lock wait (no cycle, just genuine contention — e.g., transaction A holds a row for a long time doing unrelated work, transaction B is simply waiting its turn) only errors out after `innodb_lock_wait_timeout`/`statement_timeout` elapses. Diagnostically: an immediate error = deadlock; a delayed timeout error = ordinary contention, and you should investigate *why* the blocking transaction held the lock so long (often an application bug — a forgotten commit, a slow external call inside a transaction) rather than treating it as a deadlock-retry scenario.

### Q7. What's "idle in transaction" and why is it one of the most common real production incidents?
A session that opened a transaction (`BEGIN`) but is neither committing, rolling back, nor actively running a statement — often from application code that opened a transaction, made an external API call or hit a bug mid-flow, and never reached the commit/rollback. It's dangerous because it holds locks and an old MVCC snapshot open indefinitely, blocking other writers and preventing vacuum from reclaiming dead tuples it might still need to see (Topic 9 Q6) — a very common real-world cause of "everything suddenly got slow" that traces back to one forgotten/leaked connection, not a DB capacity problem at all.

### Q8. How would you proactively catch idle-in-transaction or lock-pileup issues before they cause an incident?
Set `idle_in_transaction_session_timeout` (Postgres) to auto-kill sessions stuck open too long; alert on `pg_stat_activity`/`SHOW PROCESSLIST` session count and max transaction duration crossing a threshold; ensure application code uses a pattern that guarantees commit/rollback (e.g., a context manager/try-finally around every transaction, never manual begin/commit calls scattered without guaranteed cleanup); alert on connection pool utilization trending toward exhaustion as a leading indicator before it fully saturates.

### Q9. What's the difference between a slow query log and a query performance monitoring tool like `pg_stat_statements`, and when do you use each?
The **slow query log** captures individual query executions exceeding a duration threshold — good for catching specific bad occurrences, including one-off outliers. **`pg_stat_statements`** (or MySQL's `performance_schema`) aggregates statistics *per normalized query pattern* (total calls, total time, mean time, rows) — better for finding the query pattern responsible for the most *cumulative* load overall, which is often a moderately-slow-but-extremely-frequent query rather than the single slowest one-off outlier the slow query log highlights.

### Q10. Interviewer: "P99 latency on the checkout endpoint jumped from 100ms to 3s after a deploy 20 minutes ago — how do you triage in real time?"
"First check if it's a database problem at all versus application/network — compare DB-side query latency (via `pg_stat_statements`/APM DB spans) against total endpoint latency to isolate where the time is going. If DB-side: check `pg_stat_activity` for what's actually running/blocking right now, and diff the deploy for any new/changed query or migration in the checkout path — a new unindexed query or an accidentally-added lock (`SELECT FOR UPDATE`) on a hot row is the most common cause of exactly this pattern. Given it correlates tightly with a deploy 20 minutes ago, I'd treat the deploy as the prime suspect and consider rolling it back immediately to restore service, then root-cause offline from logs/traces rather than debugging live on a degraded production system." Shows time-pressure triage instinct, not just diagnostic knowledge.

**Red flags to avoid saying:**
- Diving straight into `EXPLAIN` on a specific query before checking for lock contention/blocking sessions as a cheaper first check.
- Not knowing "idle in transaction" is a real, extremely common production failure mode.
- No mention of correlating a performance regression with a recent deploy/migration when given an obvious timing clue in the prompt.

---

## 21. Stored Procedures, Triggers & Views

**Primer:** These are server-side logic constructs. Modern interviews care less about
syntax and more about *when it's actually appropriate* to push logic into the database
versus keeping it in application code — a debate with real, defensible tradeoffs on both sides.

### Q1. What's a stored procedure and what's the core argument for using one?
Precompiled SQL logic stored and executed inside the database, callable by name (`CALL proc_name(...)`). Core argument for: reduces network round-trips for multi-step logic (one call instead of several queries back and forth), can enforce business rules at the data layer regardless of which application/language touches the DB, and the DB can cache/optimize its execution plan across calls.

### Q2. What's the core argument against heavy use of stored procedures in a modern application?
Business logic split between application code and the database is harder to version-control cohesively, test with normal application test frameworks, and code-review alongside the feature it supports; it also couples you more tightly to a specific database vendor's procedural language (PL/pgSQL, T-SQL) and makes scaling application logic horizontally (stateless app servers) awkward if critical logic only exists inside the single DB instance. Most modern teams keep business logic in the application layer and reserve stored procedures for narrow, performance-critical, or data-integrity-critical cases.

### Q3. What's a trigger, and give a legitimate use case plus a use case where triggers cause real pain.
A trigger is code that automatically runs in response to a table event (`BEFORE`/`AFTER` `INSERT`/`UPDATE`/`DELETE`). Legitimate use: maintaining an audit log table automatically (`AFTER UPDATE` trigger inserting the old row into `audit_log`) so it can't be bypassed by forgetting to call an application function. Pain case: business logic hidden in triggers that application developers don't know exists — a debugging nightmare when an `UPDATE` has invisible side effects nobody on the current team wrote or documented; a very common real-world complaint about legacy systems.

### Q4. Give an example of a trigger for maintaining a denormalized counter, and name the concurrency risk.
```sql
CREATE TRIGGER trg_update_follower_count
AFTER INSERT ON follows
FOR EACH ROW
EXECUTE FUNCTION increment_follower_count();
```
Risk: high-concurrency inserts on the same target row (many people following the same popular account simultaneously) serialize on updating that one counter row, becoming a write hotspot/lock-contention point — the same "denormalized field must stay in sync" tradeoff from Topic 3 Q9, just automated via a trigger instead of application code, which doesn't remove the underlying contention risk.

### Q5. What's the difference between a regular (virtual) view and a materialized view?
A regular **view** is a saved query — no data is stored; every read re-executes the underlying query against live data (always fresh, but pays the full query cost every time). A **materialized view** (Topic 3 Q7) stores the computed result physically, trading freshness (needs explicit `REFRESH`) for read speed. Use a regular view purely for abstraction/simplification/security (hiding complex joins or restricting columns) without needing a performance win; use a materialized view when the underlying query is expensive and slight staleness is acceptable.

### Q6. What's a security/access-control use case for views?
Exposing a restricted subset of columns/rows to a role without granting direct table access — e.g., a `public_employee_directory` view selecting only `name`, `department`, `email` from an `employees` table that also holds `salary`/`ssn`, then granting `SELECT` on the view (not the base table) to a broader role. The base table's sensitive columns stay inaccessible to anyone using only the view.

### Q7. Are views updatable — can you `INSERT`/`UPDATE` through one?
Simple views (single table, no aggregation/`DISTINCT`/`GROUP BY`/window functions) are generally updatable by default in both Postgres and MySQL — writes pass through to the underlying table. Complex views (joins, aggregates) are generally not automatically updatable; Postgres allows making them updatable via `INSTEAD OF` triggers that define custom logic for how a write to the view should translate into writes against the base tables.

### Q8. Does MongoDB have an equivalent to stored procedures/triggers, and how do idiomatic MongoDB architectures typically replace that need?
MongoDB has limited server-side scripting (`$function`/`$accumulator` in aggregation, and historically `db.eval`, now largely discouraged/removed) — nowhere near as central as relational stored procedures/triggers. **Change Streams** are the more idiomatic MongoDB mechanism for "react to data changes" — an application (or serverless function, e.g., MongoDB Atlas Triggers) subscribes to a real-time feed of insert/update/delete events on a collection and runs logic externally, keeping "trigger-like" reactive logic in application/service code rather than embedded in the database itself.

### Q9. When would you actually recommend a stored procedure or trigger in a modern system, given the general preference for app-layer logic?
Narrow, defensible cases: (1) an invariant that absolutely must hold regardless of which client/service writes to the table, where relying on every application to remember to enforce it is too risky (e.g., an audit trail that must be tamper-evident even against a buggy or malicious application); (2) a genuinely proven performance bottleneck where round-trip latency for a tight multi-statement operation matters more than maintainability (rare, and I'd want a benchmark proving it before reaching for this); (3) database-native constraint enforcement (check constraints, foreign keys) which isn't really "procedure" logic but is the right, DB-enforced place for pure data-integrity rules.

### Q10. Candidate proposes putting all business logic (pricing calculations, discount rules) into stored procedures "for performance" — how would you push back in an interview discussion?
"I'd ask what specific latency/throughput number they're solving for, since round-trip cost for a well-designed application (batched queries, connection pooling) is rarely the actual bottleneck for business-rule logic like pricing. Moving pricing/discount logic into stored procedures makes it hard to unit test with normal application tooling, hard to code-review in the same PR as the feature, and creates a second place (alongside application code) where the same rule could drift out of sync if not updated together. I'd keep that logic in the application layer unless there's a measured, specific reason not to."

**Red flags to avoid saying:**
- Recommending heavy stored-procedure/trigger use as a default architecture without naming the maintainability tradeoff.
- Not knowing the difference between a view (always live) and a materialized view (stored, needs refresh).
- Claiming MongoDB has full trigger support equivalent to relational triggers — Change Streams are the idiomatic analog, not the same mechanism.

---

## 22. JSON & Full-Text Search in Relational DBs

**Primer:** Modern PostgreSQL and MySQL both support JSON columns and full-text search
natively — this topic tests whether you know when that native support is "good enough"
versus when you actually need a dedicated document store or search engine.

### Q1. What's the difference between Postgres's `JSON` and `JSONB` column types?
`JSON` stores the exact input text as-is (preserves whitespace/key order, re-parses on every read, no indexing support beyond basic operators). `JSONB` stores a parsed, binary decomposed representation (no whitespace/key-order preservation, slightly slower to write due to parse-on-insert, but much faster to query and supports indexing) — `JSONB` is the correct default choice for nearly all real use cases; `JSON` is really only useful when you need to preserve the exact original text byte-for-byte.

### Q2. How do you index a JSONB column for efficient querying in Postgres?
A **GIN index** on the JSONB column supports efficient containment (`@>`) and key-existence (`?`) queries across the whole document:
```sql
CREATE INDEX idx_metadata_gin ON products USING GIN (metadata);
SELECT * FROM products WHERE metadata @> '{"color": "blue"}';
```
For querying one specific, frequently-used key, a **B-Tree expression index** on that extracted key is more targeted/smaller:
```sql
CREATE INDEX idx_metadata_sku ON products ((metadata->>'sku'));
```

### Q3. When would you use a JSONB column instead of a proper normalized table for the same data?
When the data is genuinely semi-structured/sparse and varies per row in a way that would otherwise require an EAV pattern (Topic 15 Q8) or a proliferation of nullable columns — e.g., per-product custom attributes that differ wildly by category. Not appropriate for data with a stable, well-known shape shared across all rows, or data you need to join against/aggregate frequently by individual field — that's better as real columns/a normalized table where indexing, constraints, and query planning all work more naturally.

### Q4. What's MySQL's native full-text search and its main limitation compared to a dedicated search engine like Elasticsearch?
MySQL supports `FULLTEXT` indexes on `CHAR`/`VARCHAR`/`TEXT` columns with `MATCH() AGAINST()` queries, supporting natural-language relevance ranking and boolean-mode operators. Main limitation: no real support for fuzzy matching/typo tolerance, weak relevance-tuning controls, no faceting/aggregation-rich search UX, and it doesn't scale/distribute independently of the primary transactional database — fine for "search a blog's post titles," not appropriate for a product-search experience with filters, typo tolerance, and ranking tuning, where Elasticsearch/OpenSearch is the right dedicated tool.

### Q5. What's Postgres's `tsvector`/`tsquery` full-text search and how does it differ from a simple `LIKE '%term%'` query?
`tsvector` is a preprocessed, normalized representation of text (lowercased, stemmed — "running" and "run" both reduce to the same lexeme, stop-words like "the"/"a" removed), and `tsquery` represents a parsed search query in the same normalized form; matching (`@@`) and ranking (`ts_rank`) work on these normalized forms.
```sql
SELECT * FROM articles WHERE to_tsvector('english', body) @@ to_tsquery('english', 'database & performance');
```
`LIKE '%term%'` is a raw substring match — no stemming, no relevance ranking, and critically **can't use a B-Tree index** for a leading-wildcard pattern (forces a full scan), whereas a GIN index on a `tsvector` column makes full-text search genuinely index-accelerated.

### Q6. What's a generated/computed column and how does it relate to indexing JSON or full-text search fields efficiently?
A column whose value is automatically derived from other columns via an expression, stored (or virtual) and kept in sync automatically on write — e.g., `search_vector tsvector GENERATED ALWAYS AS (to_tsvector('english', title || ' ' || body)) STORED`. This lets you build a normal index directly on the generated column without needing an app-managed trigger to keep a separate `tsvector` column updated on every write — cleaner than manually maintaining it.

### Q7. When would you outgrow native Postgres/MySQL full-text search and need a dedicated search engine (Elasticsearch/OpenSearch/Algolia)?
When you need: typo-tolerant/fuzzy matching, faceted search (filter counts by category/price-range alongside results), complex relevance tuning (boosting certain fields, custom scoring), search-as-you-type/autocomplete at low latency, or search volume/query complexity high enough to want a system that scales independently of your primary transactional database rather than competing with OLTP load for the same resources.

### Q8. What's the operational cost of running a dedicated search engine alongside your primary database, and how do you keep the two in sync?
A second system to operate (indexing pipeline, cluster management, monitoring) and a second potential source of staleness — search results can lag behind the primary DB's true state if the sync mechanism lags or fails. Sync typically via: application dual-writes (write to DB, then to search index, accepting a small window of inconsistency), or more robustly, a change-data-capture pipeline (e.g., Postgres logical replication / MongoDB Change Streams / Debezium reading the binlog/WAL) that reliably streams every DB change into the search index asynchronously, decoupling search-index freshness from the request path entirely.

### Q9. How does MongoDB's `$text` index/search compare to Postgres/MySQL's native full-text search?
Similar tier of capability — MongoDB's `$text` index supports stemming, stop-words, and relevance scoring (`$meta: "textScore"`) much like Postgres's `tsvector`, with the same fundamental limitations versus a dedicated search engine (no typo tolerance, weaker relevance tuning, single-node-scoped performance). MongoDB Atlas additionally offers **Atlas Search**, a more Elasticsearch-like managed full-text/vector search layer built on Lucene, for teams needing more than the basic `$text` index without standing up a fully separate search cluster.

### Q10. Interviewer: "Our product search is slow and doesn't handle typos well — what's your recommendation and how would you justify it against 'just add a better index'?"
"A better index helps exact/prefix matching speed, but typo tolerance and relevance ranking are architecturally outside what a B-Tree or even a native full-text index (tsvector/FULLTEXT) is built to do well — those are solved problems in dedicated search engines via things like fuzzy/n-gram matching and tunable relevance scoring, not something you bolt onto a transactional database's indexing system. I'd recommend introducing Elasticsearch/OpenSearch fed by a CDC pipeline off the primary DB, keeping the primary DB as the source of truth and the search engine as a purpose-built, independently-scalable read path specifically for search — justified by the fact that the requirement (typo tolerance) is fundamentally a search-engine capability, not an indexing-depth problem."

**Red flags to avoid saying:**
- Recommending `JSON` over `JSONB` in Postgres without a specific reason to preserve exact text.
- Suggesting `LIKE '%term%'` scales for search — ignoring it can't use a standard index.
- Not knowing the difference between native DB full-text search and a dedicated search engine's capabilities (typo tolerance, faceting, relevance tuning).

---

## 23. Database Security (SQL Injection, Roles, Encryption)

**Primer:** Security fundamentals every senior engineer is expected to know cold — this
topic often comes up as a quick "do you actually know this" gate-check rather than a deep
dive, so precision matters more than length here.

### Q1. What is SQL injection and give a concrete vulnerable-vs-safe code example.
Untrusted input is concatenated directly into a SQL string, letting an attacker inject their own SQL logic.
```js
// VULNERABLE
db.query(`SELECT * FROM users WHERE email = '${input}'`);
// input = "' OR '1'='1" returns every row

// SAFE — parameterized query
db.query('SELECT * FROM users WHERE email = $1', [input]);
```
Parameterized queries/prepared statements are the fix — the DB driver sends the query structure and the input value separately, so the input is always treated as a literal value, never as executable SQL syntax, regardless of its content.

### Q2. Are ORMs automatically safe from SQL injection?
Mostly, for their standard query-builder methods — they parameterize under the hood. But **not** for raw-query escape hatches (`.raw()`, `$queryRawUnsafe`, string-built `.where()` clauses) if a developer concatenates user input into those directly — the ORM doesn't protect you from your own string concatenation. Always name this nuance in an interview rather than saying "ORMs prevent injection," which is only true for the paths that actually parameterize.

### Q3. What's the principle of least privilege as applied to database roles, with a concrete example?
Grant each application/service/user only the minimum permissions needed for its actual job — e.g., a reporting service's DB user gets `SELECT`-only access, never `INSERT`/`UPDATE`/`DELETE`/`DROP`; a service that only needs to write to two specific tables shouldn't have a role with blanket database-wide write access. Limits the blast radius if that specific service/credential is ever compromised — a compromised read-only reporting credential can't be used to modify or delete data.

### Q4. What's the difference between encryption at rest and encryption in transit, and do you need both?
**At rest**: data is encrypted on disk (e.g., Postgres's `pgcrypto`, cloud-provider disk encryption, MySQL's InnoDB tablespace encryption) — protects against someone gaining access to the raw disk/backup files. **In transit**: data is encrypted while moving over the network (TLS/SSL for client-DB connections, and for replication traffic) — protects against network-level eavesdropping/man-in-the-middle. Yes, you generally need both — they protect against entirely different attack vectors, and having one doesn't substitute for the other.

### Q5. What's column-level encryption and when would you use it beyond whole-database encryption at rest?
Encrypting specific sensitive columns (e.g., SSN, payment details) individually, often with application-managed keys, so the data stays encrypted even in memory/query results/logs unless explicitly decrypted by application code with the right key — protects against a broader threat model than disk-level encryption (which only protects against physical disk access, not against someone with DB query access, an accidental log dump, or a database-level breach). Tradeoff: encrypted columns generally can't be indexed/queried/filtered efficiently in their encrypted form, so you lose native DB query capability on that field.

### Q6. What's the risk of using a single shared database credential across your entire application fleet?
No per-service/per-actor audit trail (can't tell which service made a given query from DB logs alone), and a single point of compromise grants blanket access to everything that shared credential can do — versus per-service credentials scoped to least privilege (Q3), where a leak is both easier to trace to its source and inherently limited in blast radius. Also makes credential rotation harder (rotating one shared secret requires coordinating every consumer simultaneously, versus rotating one service's credential independently).

### Q7. What is a database audit log and what's it actually used for?
A record of who ran what query/change against the database and when — used for security forensics after an incident (what did a compromised credential actually do), compliance requirements (e.g., proving who accessed specific sensitive records), and detecting anomalous access patterns (a service account suddenly running queries it's never run before). Distinct from the general query/slow-query log — audit logging is specifically about the *who* and *what changed*, often required to be tamper-evident/immutable for compliance purposes.

### Q8. How does MongoDB's role-based access control (RBAC) compare conceptually to relational GRANT/REVOKE?
Conceptually similar — MongoDB has built-in roles (`read`, `readWrite`, `dbAdmin`, etc.) scoped per-database, and supports custom roles with fine-grained privilege sets (specific actions on specific collections), analogous to SQL's `GRANT SELECT ON table TO role`. The same least-privilege principle (Q3) applies identically regardless of database type — this is a good place in an interview to show the underlying security principle transfers across database technologies even when the exact commands differ.

### Q9. What's a connection string/credential leak risk specific to database configuration, and how do you mitigate it?
Hardcoded credentials in source code/config files committed to version control (a very common real-world incident source) — mitigate via secrets management (environment variables sourced from a vault/secrets manager, never committed), credential rotation policies, and scanning CI/pre-commit hooks for accidentally committed secrets. Also worth mentioning: even "internal-only" databases should require authentication and network-level restriction (VPC/security groups/firewall rules) — never rely on network obscurity alone as the only access control.

### Q10. Interviewer: "We found a service account with database superuser/root access that was only supposed to run read-only reports — how did this likely happen, and how do you prevent recurrence?"
"Most likely explanation: it was provisioned quickly during initial setup with an overly broad role 'to make it work' and never revisited/tightened afterward — a very common real-world drift pattern, not usually malicious. To prevent recurrence: audit all existing service credentials against actual least-privilege needs now; codify role grants as version-controlled infrastructure-as-code (so the *intended* permission set is explicit and reviewable in a PR, not a manual one-off `GRANT` someone ran once); and add a periodic automated audit that flags any credential with broader privileges than its documented/expected role." Shows both root-cause thinking and a durable process fix, not just a one-time revoke.

**Red flags to avoid saying:**
- "ORMs prevent SQL injection" as an absolute, unqualified statement.
- Not distinguishing encryption at rest from encryption in transit when asked to "explain database encryption."
- Suggesting a shared superuser credential is fine "since it's an internal service."

---

## 24. Distributed Transactions (2PC, Saga, Outbox)

**Primer:** Once data is split across multiple databases/services (polyglot persistence,
microservices, sharding), you lose single-database ACID transactions across that boundary.
This topic tests whether you know the real patterns for restoring consistency guarantees
without a shared transaction.

### Q1. Why can't you just wrap operations across two different databases (or two microservices) in one ACID transaction?
A normal `BEGIN...COMMIT` transaction only coordinates within a single database engine's own transaction manager — it has no way to atomically commit or roll back a *separate* database's or service's independent transaction. Two independent systems have two independent points of failure and no shared commit log, so "both succeed or both fail" isn't achievable with the same mechanism a single-DB transaction uses.

### Q2. Explain Two-Phase Commit (2PC) — what are the two phases?
**Phase 1 (prepare/vote)**: a coordinator asks every participant "can you commit this?" — each participant does whatever's needed to guarantee it *can* commit (locks resources, writes to a durable log) and votes yes/no, without actually committing yet. **Phase 2 (commit/abort)**: if all participants voted yes, the coordinator tells everyone to commit for real; if any voted no (or timed out), it tells everyone to abort/roll back. This gives atomicity across multiple systems, at the cost of every participant holding locks/resources throughout both phases.

### Q3. What are the practical problems with 2PC that make it unpopular in modern distributed/microservice systems?
It's **blocking**: if the coordinator crashes after phase 1 but before sending phase-2 decisions, participants are stuck holding locks indefinitely, unable to know whether to commit or abort — a real availability risk. It also requires all participants to be up and responsive simultaneously, coupling their availability together (the whole point of splitting into separate services was often to *decouple* their availability). For these reasons, most modern distributed systems avoid 2PC in favor of eventually-consistent patterns like Saga.

### Q4. Explain the Saga pattern and how it differs fundamentally from 2PC.
A Saga breaks a distributed transaction into a sequence of local transactions, each committing independently and immediately (no cross-service locking/waiting) — if a later step fails, previously completed steps are undone via explicit **compensating actions** (e.g., "cancel reservation" to undo "create reservation") rather than a coordinated rollback. Fundamentally different philosophy: 2PC achieves atomicity by blocking until consensus; Saga achieves eventual consistency by committing optimistically and compensating on failure — trading strict atomicity for availability and loose coupling.

### Q5. Give a concrete Saga example for an e-commerce order flow, including what happens on failure.
Steps: (1) create order (pending) → (2) reserve inventory → (3) charge payment → (4) confirm order. If step 3 (payment) fails: run compensating actions for the already-completed steps in reverse — release the inventory reservation from step 2, then mark the order as failed/cancelled instead of pending. Each step and its compensation are local transactions in their own service's database; nothing ever holds a lock across services waiting on the others.

### Q6. What's the difference between choreography-based and orchestration-based Sagas?
**Choreography**: each service publishes an event when its step completes, and the next service reacts to that event autonomously (no central coordinator) — simpler for a small number of steps, but the overall flow logic is implicit, spread across services, and harder to trace/debug as complexity grows. **Orchestration**: a central orchestrator service explicitly calls each step in sequence and decides what compensation to trigger on failure — the flow logic is centralized and explicit (easier to reason about/monitor), at the cost of that orchestrator being a new, more tightly-coupled dependency each participating service's flow relies on.

### Q7. What is the outbox pattern and what specific problem does it solve?
Solves the problem of atomically (1) writing to your local database AND (2) publishing an event/message about that write (e.g., to Kafka) — since those are two different systems, a naive "write to DB, then publish event" can fail between the two steps (DB write succeeds, event publish fails, or vice versa), leaving them inconsistent. Fix: write the event into an `outbox` table in the *same local transaction* as the actual data change (so it's atomic with the business write, since it's the same database), then a separate background process reads unpublished outbox rows and publishes them to the message broker, marking them published once confirmed — guaranteeing the event is eventually published if and only if the transaction committed.

### Q8. How does the outbox pattern relate to Change Data Capture (CDC)?
Instead of a separate polling process reading the outbox table, CDC tools (e.g., Debezium reading Postgres's WAL or MySQL's binlog) can capture the outbox table's inserts directly from the database's replication log and stream them to the message broker — removing the need for an explicit polling/publishing worker and reducing latency, while keeping the same core guarantee (the event only exists if the transaction that wrote it committed, since CDC reads from the durable replication log itself).

### Q9. What happens if a Saga's compensating action itself fails — how do real systems handle this edge case?
This is a genuinely hard, often under-addressed problem. Real systems typically: retry the compensating action with backoff (compensations should be designed to be idempotent/retriable); if retries are exhausted, escalate to a dead-letter queue / manual intervention alert rather than silently giving up, since an un-compensated step means real data inconsistency (e.g., inventory reserved but never released). Acknowledging this edge case explicitly in an interview is a strong signal — it shows you understand Saga isn't a magic "consistency solved" pattern, just a structured way to make inconsistency handling explicit and bounded.

### Q10. Interviewer: "We're splitting a monolith's order-processing logic (currently one DB transaction) into separate inventory and payment microservices — how do you preserve correctness?"
"I'd move away from expecting a single atomic transaction across the two services and instead design an explicit Saga: order-service orchestrates calling inventory-service to reserve stock, then payment-service to charge, with a defined compensating action (release reservation) if payment fails. Each service's own local DB write stays a normal local ACID transaction. For any event each service needs to publish about its own state change (e.g., 'inventory reserved'), I'd use the outbox pattern within that service's own database to guarantee the event is never lost or falsely published relative to its local commit. I'd explicitly design and test the compensation paths, including what happens if a compensation itself fails, before considering this done — not just the happy path."

**Red flags to avoid saying:**
- Proposing 2PC as the default answer for cross-service consistency without naming its blocking/availability-coupling downside.
- Describing Saga as giving the same atomicity guarantee as a real transaction — it gives eventual consistency via compensation, not atomicity.
- Not knowing what the outbox pattern solves when asked how to atomically write data and publish an event.

---

## 25. Data Modeling: Embedding vs Referencing (Mongo) & Relational Design

**Primer:** This topic is the practical synthesis of Topics 3, 10, and 15 — the single
most common MongoDB-specific system-design interview question, and relational data
modeling's equivalent judgment call on when to denormalize.

### Q1. State the core decision rule for embedding vs referencing in MongoDB.
Embed when the related data (a) is read together with the parent most of the time, (b) has a bounded, roughly-fixed-size relationship (won't grow unboundedly), and (c) doesn't need to be queried/updated independently of the parent very often. Reference when the related data is large/unbounded, needs independent querying/updates, or is shared across many parents (updating it in one place shouldn't require updating N embedded copies).

### Q2. Give a clear example each of "should embed" and "should reference," with reasoning.
**Embed**: a blog `post` document embedding its `author` snapshot (`{ name, avatarUrl }` at time of posting) and a small, bounded set of `tags` — read together on every page view, small and stable. **Reference**: an `order` document storing `customerId` (a reference) rather than embedding the full customer object — the customer is shared across potentially thousands of orders, is updated independently of any specific order (address changes shouldn't rewrite every past order), and embedding it everywhere would mean massive redundant updates on every customer profile change.

### Q3. What's "one-to-few," "one-to-many," and "one-to-squillions," and how does the right modeling choice differ across them?
- **One-to-few** (e.g., a person's 2-3 addresses): embed as an array directly in the parent document — small, bounded, always fits comfortably.
- **One-to-many** (e.g., a product's hundreds of reviews): usually reference — embedding hundreds of reviews inside one product document risks hitting the 16MB limit and causes write amplification on every new review; a separate `reviews` collection with a `productId` reference, indexed, is more scalable, possibly combined with embedding a small "recent/featured reviews" subset for fast display.
- **One-to-squillions** (e.g., a sensor's millions of readings, or a popular account's millions of followers): must reference, generally with the child holding the parent reference (`{ sensorId, reading, timestamp }`) rather than any attempt to list children on the parent — an array field of millions of IDs on the parent document isn't viable at all.

### Q4. How do you handle "the data changes but I embedded a snapshot of it" — is that always a bug?
Not necessarily — it depends on intent. Embedding a customer's *current* name inside every order and expecting it to reflect their *latest* name would be a bug (stale embedded copy). But embedding the *product name and price at the time of purchase* inside an order-item is often intentionally correct — an order should reflect what was actually purchased, not retroactively change if the product's current price changes (the same point from Topic 3 Q4). The key interview signal: explicitly state whether the embedded copy is meant to be a point-in-time snapshot or a live mirror, since they require completely different maintenance strategies (none needed vs active sync).

### Q5. How does the choice between embedding and referencing affect index design?
Embedded array fields become multikey indexes (Topic 11 Q5) when indexed — good for querying "find parents containing a specific embedded child value" but with the compound-multikey restriction. Referenced/separate-collection children get their own independent indexes exactly like a normal collection (indexed by the foreign-key-equivalent reference field for efficient `$lookup` or app-level fetch), and can be indexed/queried on their own attributes far more flexibly than data buried inside a parent's array ever could be.

### Q6. What's the "extended reference" (or "subset") pattern, and what problem does it solve?
A hybrid: reference the related entity (for the source of truth and independent access) but *also* embed a small, denormalized subset of its most commonly-needed fields directly in the parent, to avoid a `$lookup`/second query for the common case — e.g., an `order` document stores `customerId` (reference) plus `customerName`/`customerEmail` (a small embedded snapshot for display purposes on the order without a join). Solves the "constant `$lookup` for just 2 fields I need on every read" problem, while keeping the full customer record in one authoritative place — the same core tradeoff as relational denormalization (Topic 3), applied at the document-field level.

### Q7. How would you model a relational many-to-many relationship efficiently, and what indexes does it need?
A junction/join table with a composite primary key (or a unique composite index) on both foreign keys, e.g., `enrollments(student_id, course_id, enrolled_at)` with `PRIMARY KEY (student_id, course_id)` — this single composite index efficiently supports "find all courses for a student" (leftmost prefix on `student_id`) but querying "find all students for a course" efficiently needs an *additional* index with `course_id` first (or a second composite index `(course_id, student_id)`), since a composite index only serves queries matching its leftmost-prefix order (Topic 2 Q3).

### Q8. What's a hybrid relational approach to flexible/sparse attributes that avoids both a rigid wide table and a raw EAV table?
Use a `JSONB` column (Topic 22 Q1-Q3) for the genuinely variable/sparse attributes, alongside normal typed columns for the stable, universally-shared, frequently-queried fields (e.g., `name`, `price`, `category_id` as real columns; category-specific attributes like `{ "ram": "16GB" }` in a `attributes JSONB` column, indexed with GIN if queried). Gets you real column typing/constraints/indexing where the schema is genuinely stable, and flexible schema-on-read where it genuinely isn't — a pragmatic middle ground between full normalization and full EAV.

### Q9. How would you design the schema for a chat application's messages — walk through the embed/reference decision.
Conversations: a `conversations` collection holding metadata (`participants`, `createdAt`, maybe the last message preview embedded for a fast inbox-list view — extended reference pattern, Q6). Messages: a **separate** `messages` collection referencing `conversationId`, not embedded in the conversation document — because message count per conversation is unbounded (one-to-squillions, Q3), needs independent pagination/querying (Topic 11 Q8's cursor-based pagination), and is written far more frequently than the conversation metadata itself changes. Index `messages` on `(conversationId, createdAt)` for efficient "latest messages in this conversation" queries.

### Q10. Interviewer: "Design the schema for a MongoDB-backed e-commerce product catalog with reviews and inventory — explain your embed/reference choices out loud."
"Product document: embed core display data (`name`, `description`, `price`, `category-specific attributes` as a flexible sub-document) plus a small embedded `topReviews` array (3-5 featured/recent reviews, extended-reference-style) for fast product-page rendering without an extra query. Full reviews: separate `reviews` collection referencing `productId`, since review count is unbounded (one-to-many/squillions) and needs independent pagination/moderation/querying by review author. Inventory: I'd actually consider a *separate* `inventory` document per product/SKU rather than embedding stock count in the product document itself, since inventory updates are high-frequency, need strong consistency guarantees (Topic 5's `FOR UPDATE`-equivalent — MongoDB's atomic `findOneAndUpdate` with a quantity check) on every purchase, and embedding a fast-changing field inside an otherwise-mostly-static product document would cause unnecessary write contention on the whole document for unrelated product-detail reads." This response demonstrates applying nearly every embed/reference rule from this topic to one concrete design in real time, which is exactly the synthesis interviewers are testing for.

**Red flags to avoid saying:**
- Defaulting to embedding everything "because that's the Mongo way" without checking for unbounded growth (one-to-squillions).
- Not distinguishing an intentional point-in-time snapshot from an accidentally-stale embedded copy.
- Proposing a schema design without stating the actual read/write access pattern driving the embed/reference choice.

---

## Rapid-fire Recap

1. **ACID** — atomicity/consistency/isolation/durability; the contract a transaction gives you.
2. **Indexing** — B-Trees for range+equality, composite indexes follow leftmost-prefix, covering indexes skip the heap fetch.
3. **Normalization/Denormalization** — normalize for correctness by default, denormalize specific measured hot paths, own the sync cost.
4. **Isolation Levels/MVCC** — MySQL defaults Repeatable Read, Postgres defaults Read Committed; MVCC lets readers avoid blocking writers; watch for write skew.
5. **Locking** — row-level > table-level; pessimistic (`FOR UPDATE`) vs optimistic (version column); consistent lock ordering prevents deadlocks.
6. **Query Optimization** — `EXPLAIN ANALYZE` for real numbers; watch for `Seq Scan`/`type: ALL`, `Using filesort`, N+1 queries.
7. **Joins** — filter-in-`ON`-not-`WHERE` for LEFT JOINs; index your foreign keys (Postgres doesn't auto-index them).
8. **MySQL Storage Engines** — InnoDB is clustered by PK, row-level locking, buffer pool is the top tuning lever.
9. **Postgres Architecture** — VACUUM reclaims MVCC dead tuples and prevents XID wraparound; `idle in transaction` sessions are a common incident cause.
10. **MongoDB Document Model** — model around access patterns, not 1:1 relational translation; 16MB document limit rules out unbounded embedding.
11. **MongoDB Aggregation** — `$match`/`$sort` early for index use; avoid `.skip()` for deep pagination, use cursor-based instead.
12. **Replication** — async is the default/fast/riskier; sync/semi-sync trades latency for safety; replica lag breaks read-your-writes.
13. **Sharding/Partitioning** — partitioning is single-node, sharding is multi-node; shard key choice is nearly irreversible.
14. **CAP Theorem** — the tradeoff only applies *during a partition*; PACELC covers the everyday latency/consistency tradeoff too.
15. **SQL vs NoSQL** — choose based on access pattern and consistency needs, not "structured vs unstructured" folklore.
16. **Scaling** — connection pooling and caching before read replicas; read replicas don't help write throughput at all.
17. **Caching** — cache-aside is the default pattern; always design invalidation before you design the read path; watch for cache stampedes.
18. **Migrations** — expand-contract for zero-downtime breaking changes; batch large backfills; Postgres DDL is transactional, MySQL's mostly isn't.
19. **Backup/HA** — RPO/RTO drive the design; a replica is not a backup; test restores, don't just trust the backup job's exit code.
20. **Deadlocks/Troubleshooting** — check for blocking sessions before diving into query plans; "idle in transaction" is a top real-world incident cause.
21. **Stored Procs/Triggers/Views** — powerful but hide logic outside app code/version control; materialized views trade freshness for speed.
22. **JSON/Full-Text Search** — `JSONB` over `JSON` in Postgres; native full-text search is fine until you need typo tolerance/faceting, then use a real search engine.
23. **Security** — parameterized queries always; least-privilege roles; encryption at rest AND in transit, they cover different threats.
24. **Distributed Transactions** — 2PC blocks and couples availability; Saga trades atomicity for eventual consistency via compensation; outbox pattern solves atomic-write-plus-publish.
25. **Data Modeling (Mongo)** — embed bounded/read-together data, reference unbounded/independently-accessed data; extended reference pattern as a hybrid.

## Commonly Confused Pairs/Terms

| Term A | Term B | Key Difference |
|---|---|---|
| `EXPLAIN` | `EXPLAIN ANALYZE` | Estimated plan (not run) vs actual execution with real timing/row counts |
| Clustered index | Non-clustered/secondary index | Clustered = physical row order (one per table); secondary = separate structure needing a lookup |
| Deadlock | Lock wait timeout | Deadlock = detected cycle, aborted immediately; timeout = ordinary contention, waits then errors |
| Dirty read | Non-repeatable read | Dirty = reads uncommitted data; non-repeatable = same row changes value between two reads in one transaction |
| Dirty read | Dirty write | Dirty read = reading uncommitted data; dirty write = overwriting another transaction's uncommitted data (never allowed) |
| Optimistic locking | Pessimistic locking | Optimistic = check-at-commit via version column; pessimistic = lock upfront (`FOR UPDATE`) |
| Sharding | Partitioning | Sharding = across multiple nodes; partitioning = within one node/table |
| Replication | Backup | Replication protects against node failure; backup protects against logical/human error, and enables point-in-time recovery |
| View | Materialized view | View = always live, re-queries every read; materialized view = stored result, needs explicit refresh |
| Write-through cache | Write-behind cache | Write-through = write cache+DB synchronously together; write-behind = write cache now, flush to DB async later (data-loss risk) |
| 2PC | Saga pattern | 2PC = blocking, coordinated atomic commit across systems; Saga = independent local commits + compensating actions on failure |
| CAP theorem | PACELC | CAP = consistency vs availability tradeoff during a partition only; PACELC also covers the latency vs consistency tradeoff during normal operation |
| Embedding (Mongo) | Referencing (Mongo) | Embedding = related data stored inside the parent document; referencing = related data in a separate collection, linked by ID |
| JSON (Postgres) | JSONB (Postgres) | JSON = stores exact input text, re-parsed each read; JSONB = parsed binary form, indexable, faster to query |
| Vertical scaling | Horizontal scaling | Vertical = bigger single machine; horizontal = more machines (replicas/shards) |
| `WHERE` | `HAVING` | WHERE filters rows before aggregation; HAVING filters after aggregation, on aggregate results |
