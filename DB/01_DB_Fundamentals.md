# DB Fundamentals: Backend Interview Guide

Each section explains **what the concept is, why it exists, how it works (with real-life examples), and trade-offs**, followed by **frequently asked interview questions with model answers**.

This file is **database-agnostic**: the concepts apply whether you use MySQL, PostgreSQL, MongoDB or Redis. How each engine implements them is in:
- [02_SQL_Databases.md](02_SQL_Databases.md): SQL language, MySQL, PostgreSQL
- [03_NoSQL_MongoDB.md](03_NoSQL_MongoDB.md): MongoDB and other NoSQL stores
- [04_Redis.md](04_Redis.md): Redis and caching
- [05_Query_Practice.md](05_Query_Practice.md): practice problems
- [06_Scenario_Questions.md](06_Scenario_Questions.md): production scenarios

**How to revise with this document**
- First pass: read **What / Why / How it works** and run the SQL examples.
- Revision passes: read only the **Interview Questions**. Try answering out loud before reading the answer.
- Night before: [00_Revision_Cheatsheet.md](00_Revision_Cheatsheet.md).
- Priority: ⭐ must-know · ⭐⭐ commonly asked · ⭐⭐⭐ senior / deep dive

**Running example used throughout: ShopKart**, an e-commerce backend.

```
users(id, name, email, city, created_at)
products(id, name, category, price, stock)
orders(id, user_id, status, total, created_at)
order_items(order_id, product_id, quantity, unit_price)
payments(id, order_id, amount, status, idempotency_key)
accounts(id, user_id, balance)            -- wallet balance
```

---

## Table of Contents

**[The Big Picture](#the-big-picture)**

**Part 1: Foundations**
[1. Basics](#1-basics) · [2. Data Modeling](#2-data-modeling) · [3. Normalization](#3-normalization) · [4. Relational Algebra](#4-relational-algebra)

**Part 2: Storage and Access**
[5. Storage Internals](#5-storage-internals) · [6. Indexing](#6-indexing) · [7. Query Processing and Optimization](#7-query-processing-and-optimization) · [8. Connection Management](#8-connection-management) · [9. ORMs and the Application Layer](#9-orms-and-the-application-layer)

**Part 3: Transactions and Durability**
[10. Transactions and ACID](#10-transactions-and-acid) · [11. Isolation Levels](#11-isolation-levels) · [12. Concurrency Control and Locking](#12-concurrency-control-and-locking) · [13. MVCC](#13-mvcc) · [14. Recovery and Durability](#14-recovery-and-durability)

**Part 4: Scaling and Distributed Systems**
[15. Replication](#15-replication) · [16. Partitioning and Sharding](#16-partitioning-and-sharding) · [17. Distributed Database Theory](#17-distributed-database-theory) · [18. SQL vs NoSQL](#18-sql-vs-nosql)

**Part 5: Production Concerns**
[19. Data Pipelines and Data Integrity](#19-data-pipelines-and-data-integrity) · [20. Data Lifecycle](#20-data-lifecycle) · [21. Security](#21-security) · [22. Applied Design](#22-applied-design)

---

## The Big Picture

Database topics form a pipeline. Every interview question sits somewhere on it:

```
  DESIGN            STORE             ACCESS              PROTECT                 SCALE
┌──────────┐    ┌───────────┐    ┌──────────────┐    ┌────────────────┐    ┌────────────────┐
│ Modeling │ →  │ Pages     │ →  │ Indexes      │ →  │ Transactions   │ →  │ Replication    │
│ Keys     │    │ Buffer    │    │ Optimizer    │    │ Isolation      │    │ Sharding       │
│ Normal-  │    │ pool      │    │ Connections  │    │ Locks / MVCC   │    │ CAP            │
│ ization  │    │ Row/col   │    │ ORMs         │    │ WAL / Recovery │    │ Caching        │
└──────────┘    └───────────┘    └──────────────┘    └────────────────┘    └────────────────┘
   Part 1          Part 2             Part 2               Part 3                 Part 4
```

**The single idea that ties everything together:** *every database feature is a trade-off between correctness, speed and scale.*
- Indexes trade write speed and disk space for read speed.
- Normalization trades read speed (more joins) for consistency (no duplicates).
- Stronger isolation trades throughput for correctness.
- Synchronous replication trades latency for durability.
- Sharding trades simplicity for capacity.

When you don't know an answer in an interview, saying *"it's a trade-off between X and Y, and here's when I'd pick each"* is almost always the right shape.

**Threads that cross sections (interviewers love these):**
1. **Disk is slow → pages → buffer pool → indexes → B+ trees.** Everything in storage design (§5, §6) exists to reduce disk reads.
2. **Concurrent users → anomalies → isolation levels → locks or MVCC.** §11, §12 and §13 are three views of the same problem.
3. **Crashes → WAL → replication → CDC.** The same log that recovers a crash (§14) feeds replicas (§15) and change pipelines (§19).
4. **One machine isn't enough → replication (reads) → sharding (writes) → CAP trade-offs.** §15 → §16 → §17.
5. **App code → ORM → N+1 → indexes → connection pool.** Most production slowness starts in the application layer (§8, §9), not the database.

---

# PART 1: FOUNDATIONS

---

## 1. Basics

**Priority:** ⭐ must-know

**What it is:** A **database** is an organized collection of data. A **DBMS** (Database Management System) is the software that stores, retrieves and protects it: MySQL, PostgreSQL, MongoDB, Redis.

**Why it exists:** Before databases, applications stored data in flat files. Imagine ShopKart storing orders in `orders.csv`:
- **Concurrency:** two customers checking out at once both rewrite the file, and one order is lost.
- **Crashes:** the server dies halfway through a write, and the file is corrupted.
- **Redundancy and inconsistency:** the customer's address is copied into every order line; when it changes, half the copies are stale.
- **Querying:** "total sales by city last month" means writing custom parsing code every time.
- **Security:** you can't give the support team read access to orders but not to payments.

A DBMS solves all five: concurrency control, crash recovery, a single source of truth, a query language, and access control.

**How it works**

*DBMS components:*

```
         SQL query
             │
   ┌─────────▼──────────┐
   │  Query Processor   │  parser → optimizer → executor
   └─────────┬──────────┘
   ┌─────────▼──────────┐
   │ Transaction Manager│  locks, MVCC, isolation
   └─────────┬──────────┘
   ┌─────────▼──────────┐
   │  Buffer Manager    │  caches disk pages in RAM
   └─────────┬──────────┘
   ┌─────────▼──────────┐
   │  Storage Manager   │  files, pages, indexes, WAL
   └────────────────────┘
```

*Three-schema architecture:* separates how users see data from how it's stored.

| Level | What it describes | ShopKart example |
|---|---|---|
| **External (view)** | What each user/app sees | Support team's view shows orders without card details |
| **Logical (conceptual)** | All tables, columns, relationships | `orders`, `users`, foreign keys |
| **Physical (internal)** | Files, pages, indexes on disk | B+ tree index on `orders.user_id`, 8KB pages |

This gives **data independence**:
- **Physical independence:** add an index or move data to SSD without changing queries.
- **Logical independence:** add a column to `users` without breaking apps that use a view (harder to achieve in practice).

*Types of databases:*

| Type | Data shape | Examples | Real-life use |
|---|---|---|---|
| Relational | Tables with rows and columns | PostgreSQL, MySQL | Orders, payments, banking |
| Key-value | Key → value | Redis, DynamoDB | Sessions, caches, carts |
| Document | JSON-like documents | MongoDB | Product catalogs with varying attributes |
| Wide-column | Rows with dynamic columns, partitioned | Cassandra, HBase | Messaging history, IoT at huge write volume |
| Graph | Nodes and edges | Neo4j | Social networks, fraud rings, recommendations |
| Search | Inverted index | Elasticsearch, OpenSearch | Product search, log search |
| Time-series | Timestamped measurements | TimescaleDB, InfluxDB | Metrics, stock ticks, sensor data |
| Vector | Embeddings | pgvector, Pinecone, Qdrant | Semantic search, RAG for LLMs |

*OLTP vs OLAP:*

| | OLTP (transactional) | OLAP (analytical) |
|---|---|---|
| Workload | Many small reads/writes | Few huge read queries |
| Example | "Place order #123" | "Revenue by category per month for 3 years" |
| Rows touched | A few | Millions to billions |
| Schema | Normalized | Denormalized (star schema) |
| Storage | Row store | Column store |
| Examples | PostgreSQL, MySQL | Snowflake, BigQuery, ClickHouse, Redshift |

Real life: ShopKart's checkout runs on PostgreSQL (OLTP). Every night, orders are copied into BigQuery (OLAP) where the business team runs dashboards. Running those dashboards on the production database would slow down checkout.

### Interview Questions

**Q1. Why use a database instead of storing data in files?** ⭐
Files give you none of: safe concurrent access, crash recovery, a query language, indexing, integrity constraints or access control. You'd end up writing a buggy partial database yourself. A DBMS provides all of these with ACID guarantees: two users can't overwrite each other's orders, a crash mid-write doesn't corrupt data, and `SELECT ... WHERE` replaces custom parsing code.

**Q2. What is data independence and why does it matter?**
Changing one level of the schema without affecting the level above. Physical independence means you can add an index, partition a table or change storage without rewriting queries. That's why a DBA can tune a production database without a code deploy. Logical independence (changing tables without breaking apps) is partially achieved with views.

**Q3. What's the difference between OLTP and OLAP? Can one database do both?** ⭐
OLTP handles many short transactions on a few rows (checkout, login); OLAP handles a few long queries scanning millions of rows (reports). They need opposite storage layouts: row stores are fast for fetching one whole order, column stores are fast for summing one column across millions of rows. Some systems try to do both ("HTAP": TiDB, SingleStore, PostgreSQL with columnar extensions), but the common pattern is OLTP in PostgreSQL/MySQL, replicated into a warehouse for analytics.

**Q4. Which database would you pick for a new backend service?**
Default to **PostgreSQL** unless there's a specific reason not to. It handles relational data, JSON, full-text search and even vectors well. Add **Redis** for caching and rate limiting. Move to specialised stores only when a real need appears: MongoDB for highly variable document shapes, Cassandra for massive write volume, Elasticsearch for advanced search. Mention the trade-off: every extra database is another system to operate, back up and keep consistent.

---

## 2. Data Modeling

**Priority:** ⭐ must-know

**What it is:** Deciding which **entities** your system has, what **attributes** they have, and how they **relate**, then turning that into tables, columns, keys and constraints.

**Why it matters:** Your schema outlives your code. A bad model (wrong keys, missing constraints, duplicated data) causes bugs that no amount of application code can fully fix. Interviewers test this with "design the schema for X" questions.

**How it works**

### 2.1 The ER (Entity-Relationship) model

| Term | Meaning | ShopKart example |
|---|---|---|
| **Entity** | A "thing" you store data about | User, Product, Order |
| **Attribute** | A property of an entity | User.email, Product.price |
| **Simple attribute** | Can't be split | `email` |
| **Composite attribute** | Made of parts | `address` = street + city + pincode |
| **Multivalued attribute** | Several values | A user's multiple phone numbers |
| **Derived attribute** | Computed from others | `age` from `date_of_birth`, `order.total` from items |
| **Relationship** | How entities connect | User *places* Order |
| **Weak entity** | Can't exist without its owner; identified partly by owner's key | `order_items` only exist inside an `order` |

**Cardinality** (how many on each side):
- **1:1**: User ↔ UserProfile
- **1:N**: User → many Orders
- **M:N**: Orders ↔ Products (an order has many products; a product appears in many orders)

**Participation:**
- **Total**: every Order *must* belong to a User (`user_id NOT NULL`)
- **Partial**: a User *may* have zero Orders

### 2.2 Mapping relationships to tables

```sql
-- 1:1  → foreign key with UNIQUE (or share the primary key)
CREATE TABLE user_profiles (
  user_id BIGINT PRIMARY KEY REFERENCES users(id),
  bio TEXT,
  avatar_url TEXT
);

-- 1:N  → foreign key on the "many" side
CREATE TABLE orders (
  id BIGINT PRIMARY KEY,
  user_id BIGINT NOT NULL REFERENCES users(id),
  status VARCHAR(20) NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- M:N  → junction (bridge) table with a composite key
CREATE TABLE order_items (
  order_id   BIGINT REFERENCES orders(id),
  product_id BIGINT REFERENCES products(id),
  quantity   INT NOT NULL CHECK (quantity > 0),
  unit_price DECIMAL(10,2) NOT NULL,      -- price at time of purchase
  PRIMARY KEY (order_id, product_id)
);
```

Note `unit_price` in `order_items`: it looks like duplication of `products.price`, but it isn't. It records the price **at the time of purchase**. If the product price changes tomorrow, old orders must not change. This is a classic interview point: *not all apparent duplication is bad; historical facts must be captured.*

Multivalued attributes (several phone numbers) become their own table: `user_phones(user_id, phone)`.

### 2.3 Keys

| Key | Definition | Example |
|---|---|---|
| **Super key** | Any set of columns that uniquely identifies a row | `{id}`, `{id, email}`, `{email}` |
| **Candidate key** | A *minimal* super key (no column can be removed) | `{id}`, `{email}` |
| **Primary key** | The candidate key you choose as the main identifier | `id` |
| **Alternate key** | Candidate keys not chosen as primary | `email` (enforce with UNIQUE) |
| **Foreign key** | Column referencing another table's key | `orders.user_id → users.id` |
| **Composite key** | Key made of multiple columns | `(order_id, product_id)` |
| **Surrogate key** | Artificial ID with no business meaning | Auto-increment `id`, UUID |
| **Natural key** | Real-world identifier | Email, PAN number, ISBN |

**Surrogate vs natural keys:** natural keys change in real life (people change emails, countries re-issue IDs) and can be long. Use a **surrogate primary key** plus a **UNIQUE constraint** on the natural key. You get stable joins and still prevent duplicates.

**Auto-increment vs UUID:**

| | Auto-increment (BIGINT) | UUID v4 (random) | UUID v7 / ULID (time-ordered) |
|---|---|---|---|
| Size | 8 bytes | 16 bytes | 16 bytes |
| Generated | By database | Anywhere | Anywhere |
| Index insert pattern | Sequential (fast) | Random (page splits, slower) | Mostly sequential (fast) |
| Guessable | Yes (`/orders/1001` → try 1002) | No | Partially (timestamp visible) |
| Merging data across shards/systems | Collisions | Safe | Safe |

Real life: expose a UUID in public URLs so users can't enumerate orders, and prefer **UUID v7** over v4 when it's the primary key, because random UUIDs scatter inserts across the index (see §6).

### 2.4 Integrity constraints

```sql
CREATE TABLE products (
  id       BIGSERIAL PRIMARY KEY,                  -- entity integrity: unique, not null
  sku      VARCHAR(40) NOT NULL UNIQUE,            -- alternate key
  name     TEXT NOT NULL,
  price    DECIMAL(10,2) NOT NULL CHECK (price >= 0),  -- domain integrity
  category_id BIGINT REFERENCES categories(id)     -- referential integrity
    ON DELETE RESTRICT
);
```

**Referential actions** (what happens to children when the parent is deleted or updated):

| Action | Behaviour | Use when |
|---|---|---|
| `RESTRICT` / `NO ACTION` | Block the delete | Deleting a category that still has products should fail |
| `CASCADE` | Delete children too | Deleting an order deletes its `order_items` |
| `SET NULL` | Set FK to NULL | Deleting a salesperson keeps their deals, unassigned |
| `SET DEFAULT` | Set FK to default | Rare |

Be careful with `CASCADE`: deleting one user could silently delete thousands of orders. Many teams avoid cascades on important data and use soft deletes instead (§20).

**Constraints in the database vs in application code:** application validation gives friendly error messages, but only database constraints are guaranteed. Two app servers checking "is this email taken?" at the same moment can both say "no" and both insert. A `UNIQUE` constraint catches it. **Always enforce invariants in the database too.**

### 2.5 Choosing data types

| Choice | Rule | Why |
|---|---|---|
| Money | `DECIMAL(12,2)` or integer paise/cents, **never FLOAT** | `0.1 + 0.2 = 0.30000000000000004` in floating point |
| Text | `VARCHAR(n)` when there's a real limit, else `TEXT` | `CHAR(n)` pads with spaces; rarely useful |
| Timestamps | Store in **UTC** with a time-zone-aware type | Users and servers are in different time zones |
| Booleans | `BOOLEAN` | Not `'Y'/'N'` strings |
| Status values | `VARCHAR` + `CHECK`, or a lookup table | Native ENUMs are hard to change in some databases |
| IDs | `BIGINT` | `INT` runs out at ~2.1 billion; Basecamp and others have hit this |

Real life: a team used `INT` for an `events.id` column. At 2,147,483,647 rows, inserts started failing at 3 AM. Changing an `INT` primary key to `BIGINT` on a huge table requires rewriting the whole table. Use `BIGINT` from day one.

### Interview Questions

**Q1. What's the difference between a primary key and a unique key?** ⭐
Both enforce uniqueness. A table has **one** primary key, which can't be NULL and is the main identifier other tables reference. It can have **many** unique keys, which usually allow NULLs (in most databases, multiple NULLs are allowed because NULL ≠ NULL). In MySQL InnoDB the primary key also decides the physical order of the table (clustered index).

**Q2. Should you use a natural key or a surrogate key as primary key?** ⭐
Usually a surrogate key, plus a UNIQUE constraint on the natural key. Natural keys change (emails, phone numbers), are often long (slower joins and bigger indexes), and sometimes aren't truly unique (two people can share a name and date of birth). A surrogate never changes and is compact. Exception: small lookup tables with stable codes, like `country_code = 'IN'`.

**Q3. How do you model a many-to-many relationship?** ⭐
With a junction table holding foreign keys to both sides, and a composite primary key (or unique constraint) on the pair to prevent duplicates. Example: `order_items(order_id, product_id, quantity)`. The junction table often carries its own attributes, like quantity or price at the time of purchase.

**Q4. UUID or auto-increment for primary keys?** ⭐⭐
Auto-increment is compact and gives sequential inserts, but it leaks business volume (order 1001 → 1500 tells competitors your sales) and is hard to merge across shards. Random UUIDs (v4) are globally unique and unguessable, but they cause random index inserts and page splits, which hurt write performance, especially in MySQL where the table is clustered on the primary key. A good modern answer: **UUID v7 or ULID**, which are globally unique *and* time-ordered. Another common pattern: internal BIGINT primary key plus a public UUID column.

**Q5. Should constraints live in the database or the application?** ⭐
Both. The application validates for a good user experience. The database enforces the rule because it's the only place that sees every write, including concurrent requests, scripts, migrations and other services. A check in application code can't prevent two concurrent requests from inserting the same email; a UNIQUE constraint can.

**Q6. Why does `order_items` store `unit_price` when `products` already has `price`?**
Because it's a different fact: the price *when the order was placed*. Products change price; historical orders must not. Same for shipping address on an order: copy it at the time of the order, don't reference the user's current address.

---

## 3. Normalization

**Priority:** ⭐ must-know

**What it is:** A step-by-step process for organizing tables so that **each fact is stored in exactly one place**. Each "normal form" removes a specific kind of redundancy.

**Why it exists:** Redundant data causes **anomalies**. Consider this unnormalized ShopKart table:

| order_id | user_id | user_name | user_city | product_id | product_name | qty |
|---|---|---|---|---|---|---|
| 1 | 10 | Asha | Pune | 100 | Phone | 1 |
| 1 | 10 | Asha | Pune | 101 | Case | 2 |
| 2 | 10 | Asha | Pune | 100 | Phone | 1 |

- **Update anomaly:** Asha moves to Mumbai. You must update 3 rows; miss one and the data contradicts itself.
- **Insert anomaly:** You can't add a new product until someone orders it.
- **Delete anomaly:** Delete order 2, which is the only order containing some product, and you lose that product's name entirely.

**How it works**

### 3.1 Functional dependencies (FDs)

`A → B` means "if you know A, you know B." In the table above:
- `user_id → user_name, user_city`
- `product_id → product_name`
- `(order_id, product_id) → qty`

**Closure** `X+`: all attributes you can determine from X using the FDs. If `X+` contains every column, X is a super key. The **candidate key** here is `(order_id, product_id)`.

**Armstrong's axioms** (used to derive FDs):
- **Reflexivity:** if Y ⊆ X then X → Y
- **Augmentation:** if X → Y then XZ → YZ
- **Transitivity:** if X → Y and Y → Z then X → Z

**Finding candidate keys (exam-style):** attributes that never appear on the right side of any FD *must* be in every key. Start from those, compute the closure, and add attributes until the closure covers everything.

### 3.2 The normal forms

**1NF: atomic values, no repeating groups.**

❌ `users(id, name, phones = "98..., 99...")`
✅ `users(id, name)` + `user_phones(user_id, phone)`

**2NF: 1NF + no partial dependency** (no non-key column depends on only *part* of a composite key).

In `order_lines(order_id, product_id, qty, product_name)`, `product_name` depends only on `product_id`, which is part of the key. Move it out:
✅ `order_lines(order_id, product_id, qty)` + `products(product_id, product_name)`

*2NF only matters for composite keys.* A table with a single-column key that's in 1NF is automatically in 2NF.

**3NF: 2NF + no transitive dependency** (non-key columns depend only on the key, not on other non-key columns).

In `orders(order_id, user_id, user_city)`: `order_id → user_id → user_city`. City depends on the user, not the order. Move it out:
✅ `orders(order_id, user_id)` + `users(user_id, user_city)`

*Memory hook:* every non-key attribute must depend on **"the key, the whole key, and nothing but the key."* (1NF, 2NF, 3NF.)

**BCNF (Boyce-Codd): for every FD X → Y, X must be a super key.**

3NF allows one exception: if Y is part of a candidate key. BCNF removes it. Classic example, a tutoring booking table:

`bookings(student, subject, tutor)` with FDs:
- `(student, subject) → tutor`
- `tutor → subject` (each tutor teaches one subject)

`tutor → subject`, but `tutor` isn't a super key. That breaks BCNF. Split:
✅ `tutor_subjects(tutor, subject)` + `student_tutors(student, tutor)`

**4NF: no multivalued dependencies.** ⭐⭐⭐
`employee_skills_languages(emp, skill, language)` where skills and languages are independent. You'd need every skill × language combination. Split into `emp_skills(emp, skill)` and `emp_languages(emp, language)`.

**5NF: no join dependencies** that can't be implied by keys. Rarely asked beyond the definition.

### 3.3 Decomposition properties

When you split a table, two properties matter:
- **Lossless join** (mandatory): joining the pieces back gives *exactly* the original rows, no fake extra rows. Guaranteed if the common columns form a key of at least one piece.
- **Dependency preservation** (desirable): every FD can still be checked within a single table, without a join.

3NF decomposition can always keep both. BCNF can always be lossless but sometimes loses dependency preservation. That's why many real systems stop at 3NF.

### 3.4 Denormalization

**What:** deliberately adding redundancy to make reads faster.

**Real-life examples:**
- Storing `orders.total` instead of summing `order_items` every time the order list loads.
- Storing `posts.comment_count` instead of `COUNT(*)` on comments for every feed item.
- Copying `user_name` into a `comments` table in a read-heavy social app.

**Cost:** now you must keep copies in sync (in a transaction, with triggers, or eventually via events). Denormalize when reads vastly outnumber writes and you've measured that joins are the bottleneck.

### 3.5 Star and snowflake schemas ⭐⭐

Used in data warehouses (OLAP), not OLTP:

```
                 dim_date
                    │
 dim_customer ── fact_sales ── dim_product
                    │
                 dim_store
```

- **Fact table:** measurable events (`fact_sales(date_id, product_id, customer_id, qty, revenue)`). Huge, append-only.
- **Dimension tables:** descriptive context (`dim_product(product_id, name, category, brand)`).
- **Star schema:** dimensions are denormalized (one table each). Fewer joins, faster queries.
- **Snowflake schema:** dimensions are normalized (`dim_product → dim_category → dim_department`). Less redundancy, more joins.

### Interview Questions

**Q1. What is normalization and why do we need it?** ⭐
Organizing data so each fact is stored once, by removing dependencies that cause redundancy. It prevents update anomalies (changing a user's city in one row but not another), insert anomalies (can't add a product without an order) and delete anomalies (deleting the last order loses the product). It also saves space and keeps writes simple.

**Q2. Explain 1NF, 2NF and 3NF with an example.** ⭐
- 1NF: atomic values. A `phones` column holding "98..., 99..." breaks it; move phones to their own table.
- 2NF: no column depends on part of a composite key. In `order_lines(order_id, product_id, qty, product_name)`, `product_name` depends only on `product_id`, so move it to `products`.
- 3NF: no column depends on another non-key column. In `orders(id, user_id, user_city)`, city depends on user, not order, so move it to `users`.

**Q3. What's the difference between 3NF and BCNF?** ⭐⭐
BCNF is stricter: for every dependency X → Y, X must be a super key. 3NF also allows X → Y when Y is part of some candidate key. Example: `bookings(student, subject, tutor)` with `tutor → subject` is in 3NF but not BCNF, because `tutor` isn't a super key. Converting to BCNF can lose dependency preservation, which is why 3NF is the usual practical target.

**Q4. When would you denormalize?** ⭐
When reads dominate, the joins are measurably slow, and you can tolerate the work of keeping copies in sync. Examples: cached counters (`comment_count`), precomputed totals, read models for dashboards, or document databases where you embed related data. Always mention how you keep the copy correct: same transaction, trigger, or asynchronous event with eventual consistency.

**Q5. Is a fully normalized database always better?**
No. Highly normalized schemas need many joins for common screens, which costs CPU and latency. Real systems normalize to 3NF for the source of truth, then add targeted denormalization (counters, summary tables, caches, search indexes) for hot read paths.

**Q6. Given R(A, B, C, D) with A → B, B → C, find the candidate key and the highest normal form.** ⭐⭐
D never appears on the right side, and neither does A, so both must be in the key. `{A, D}+ = {A, D, B, C}`, which is everything, so the candidate key is **(A, D)**. `A → B` is a partial dependency (A is only part of the key), so the relation is **only in 1NF**. Decompose into `R1(A, B)`, `R2(B, C)`, `R3(A, D)`.

---

## 4. Relational Algebra

**Priority:** ⭐⭐ commonly asked (mainly in campus/fresher interviews and theory rounds)

**What it is:** The mathematical language of operations on tables (relations). Each operation takes tables as input and produces a table as output. SQL is built on it, and query optimizers internally rewrite SQL as relational algebra.

**Why it matters:** It explains *what* SQL does and why optimizers can reorder operations. For example, "filter before join" is the algebra rule σ(R ⋈ S) = σ(R) ⋈ S when the filter only touches R.

**How it works**

| Operation | Symbol | SQL equivalent | ShopKart example |
|---|---|---|---|
| Selection | σ (sigma) | `WHERE` | σ city='Pune' (users) |
| Projection | π (pi) | `SELECT col` (with DISTINCT) | π name, email (users) |
| Union | ∪ | `UNION` | Customers ∪ Sellers emails |
| Difference | − | `EXCEPT` | All users − users who ordered |
| Cartesian product | × | `CROSS JOIN` | Every user paired with every product |
| Rename | ρ (rho) | `AS` | ρ u(users) |
| Natural join | ⋈ | `JOIN ... USING` | users ⋈ orders |
| Intersection | ∩ | `INTERSECT` | Users who ordered ∩ users who reviewed |
| Division | ÷ | `NOT EXISTS ... NOT EXISTS` | Users who bought **every** product in a category |

Note: relational algebra works on **sets** (no duplicates). SQL works on **bags/multisets** (duplicates allowed), which is why `SELECT` needs `DISTINCT` to match projection exactly.

**Division** is the one interviewers like, because it answers "for all" questions: *Which users have bought every product in the 'Kitchen' category?*

```sql
-- "There is no kitchen product that this user has NOT bought"
SELECT u.id
FROM users u
WHERE NOT EXISTS (
  SELECT 1 FROM products p
  WHERE p.category = 'Kitchen'
    AND NOT EXISTS (
      SELECT 1 FROM orders o
      JOIN order_items oi ON oi.order_id = o.id
      WHERE o.user_id = u.id AND oi.product_id = p.id
    )
);

-- Same thing with counting (often easier to write)
SELECT o.user_id
FROM orders o
JOIN order_items oi ON oi.order_id = o.id
JOIN products p ON p.id = oi.product_id
WHERE p.category = 'Kitchen'
GROUP BY o.user_id
HAVING COUNT(DISTINCT p.id) = (SELECT COUNT(*) FROM products WHERE category = 'Kitchen');
```

*Tuple and domain relational calculus* are declarative alternatives ("describe what you want" rather than "the steps to get it"). SQL is closer to tuple calculus in style. Knowing that they exist and have equal expressive power to relational algebra (Codd's theorem) is usually enough.

### Interview Questions

**Q1. What's the difference between selection and projection?** ⭐
Selection (σ) picks **rows** that satisfy a condition, like `WHERE`. Projection (π) picks **columns**, like the `SELECT` list, and removes duplicates in pure algebra.

**Q2. How do you express "users who bought all products in a category" in SQL?** ⭐⭐
That's relational division. Either use double `NOT EXISTS` ("there's no product in the category this user hasn't bought"), or group by user and compare `COUNT(DISTINCT product_id)` to the total number of products in the category.

**Q3. Why does relational algebra matter for query optimization?**
Because algebraic equivalences let the optimizer rewrite a query into a cheaper but equivalent form: pushing selections below joins (filter early), reordering joins (join small results first), and removing unneeded columns early.

---

# PART 2: STORAGE AND ACCESS

---

## 5. Storage Internals

**Priority:** ⭐⭐ commonly asked

**What it is:** How a database physically lays out data on disk and in memory.

**Why it matters:** **Disk access is the bottleneck.** Reading from RAM takes about 100 nanoseconds; a random read from SSD takes about 100 microseconds (1,000× slower); from spinning disk about 10 milliseconds (100,000× slower). Almost every database design decision, from pages to B+ trees to buffer pools, exists to **minimize disk reads**. Once you understand this, indexing and query plans make much more sense.

**How it works**

### 5.1 Pages (blocks)

A database never reads one row from disk. It reads a whole **page**: a fixed-size block (PostgreSQL 8KB, MySQL InnoDB 16KB). A page holds many rows.

```
Table file on disk
┌──────────┬──────────┬──────────┬──────────┐
│ Page 0   │ Page 1   │ Page 2   │ Page 3   │ ...
└──────────┴──────────┴──────────┴──────────┘

Inside one page (slotted page layout)
┌──────────────────────────────────────────┐
│ Header (page id, free space pointers)    │
│ Slot array: [ptr1][ptr2][ptr3] →         │
│                                          │
│              free space                  │
│                                          │
│           ← [row 3][row 2][row 1]        │
└──────────────────────────────────────────┘
```

The **slot array** grows from the top, rows grow from the bottom. A row's address is `(page number, slot number)`. In PostgreSQL this is called the **TID/ctid**.

**Consequence:** the cost of a query is roughly *the number of pages it touches*, not the number of rows. Fetching 100 rows that sit in 1 page is cheap; fetching 100 rows scattered across 100 pages is 100× more expensive.

### 5.2 Buffer pool

The **buffer pool** (MySQL `innodb_buffer_pool_size`, PostgreSQL `shared_buffers`) is a cache of pages in RAM.

```
Query needs page 42
   │
   ▼
In buffer pool? ──yes──► return it (fast, "cache hit")
   │ no
   ▼
Read from disk → put in buffer pool (evict an old page if full) → return it
```

- **Dirty page:** a page modified in memory but not yet written to disk. Written later by a background process (with WAL protecting it; see §14).
- **Eviction:** usually a variant of **LRU** (least recently used). Plain LRU has a problem: one big table scan would flush out all your hot pages. So databases use smarter versions: InnoDB splits its LRU into "young" and "old" parts; PostgreSQL uses a **clock-sweep** algorithm.
- **Cache hit ratio:** for OLTP you want 99%+. If it drops, your working set doesn't fit in RAM and queries slow down sharply.

Real life: ShopKart's database has 200GB of data but only 20GB of "hot" data (recent orders, active products). With a 32GB buffer pool, almost every query is served from RAM. When a marketing team runs a report scanning 3 years of orders, it pollutes the cache and checkout latency spikes, which is another reason to send analytics to a replica or warehouse.

### 5.3 Heap tables vs clustered tables

| | Heap table (PostgreSQL) | Clustered / index-organized table (MySQL InnoDB, SQL Server clustered) |
|---|---|---|
| Row order on disk | Wherever there's space (insertion order, roughly) | Sorted by primary key, stored *inside* the primary key B+ tree |
| Primary key lookup | Index → TID → heap page (2 steps) | B+ tree leaf *is* the row (1 step) |
| Secondary index stores | TID (physical location) | Primary key value |
| Secondary index lookup | Index → heap | Secondary index → PK → PK B+ tree (double lookup) |
| Random PK inserts | Fine for the table, only the index suffers | Cause page splits in the table itself |
| Range scan on PK | Rows may be scattered | Rows are physically adjacent (fast) |

This is why "use a sequential primary key" matters so much more in MySQL than in PostgreSQL.

### 5.4 Row store vs column store

```
Row store (OLTP: PostgreSQL, MySQL)
Page 1: [1, Asha, Pune, 500] [2, Ravi, Delhi, 300] [3, Meera, Pune, 800]

Column store (OLAP: ClickHouse, BigQuery, Redshift, Parquet files)
id:     [1, 2, 3]
name:   [Asha, Ravi, Meera]
city:   [Pune, Delhi, Pune]
amount: [500, 300, 800]
```

- `SELECT * FROM orders WHERE id = 2` → row store wins (one page has the whole row).
- `SELECT SUM(amount) FROM orders` over 1 billion rows → column store wins: it reads only the `amount` column, and similar values compress extremely well (Pune, Pune, Pune → "Pune × 3").

### 5.5 Sequential vs random I/O

Reading pages in order (sequential) is far faster than jumping around (random), especially on spinning disks, and still noticeably on SSDs. That's why:
- A **full table scan** can beat an index when you need a large fraction of the table: it reads sequentially, while the index causes random reads.
- Databases write changes to an append-only **log** (sequential) first and update data pages later (§14).
- LSM trees (§6) turn random writes into sequential writes.

### Interview Questions

**Q1. Why do databases read data in pages instead of rows?** ⭐⭐
Because disk I/O has a high fixed cost per operation and transfers data in blocks anyway. Reading 8–16KB at once amortizes that cost across many rows, and nearby rows are often needed together. The page is also the unit of caching in the buffer pool and the unit of locking/latching for many internal operations.

**Q2. What is a buffer pool and why does its size matter so much?** ⭐⭐
It's the database's RAM cache of disk pages. Every read goes through it; a hit avoids a disk read. If your hot data fits in the buffer pool, most queries never touch disk. A common sizing rule for a dedicated MySQL server is 50–75% of RAM for the buffer pool; PostgreSQL uses a smaller `shared_buffers` (around 25% of RAM) because it also relies on the OS page cache.

**Q3. What's the difference between a clustered and a heap table?** ⭐⭐
In a clustered table (InnoDB), rows are stored inside the primary key B+ tree, sorted by key. Primary key lookups and range scans are very fast, but secondary indexes store the primary key and need a second lookup, and random primary keys cause page splits. In a heap table (PostgreSQL), rows are stored unordered and every index, including the primary key, points to the physical row location.

**Q4. Why are column stores better for analytics?** ⭐⭐
Analytical queries touch few columns but many rows. A column store reads only the needed columns, compresses each column very well (same type, repeated values), and can process values in batches with CPU vector instructions. A row store would read every column of every row from disk.

---

## 6. Indexing

**Priority:** ⭐ must-know (one of the most-asked database topics)

**What it is:** An index is a separate data structure that lets the database find rows **without scanning the whole table**, like the index at the back of a book.

**Why it exists:** Without an index, `SELECT * FROM orders WHERE user_id = 42` must read every page of a 50-million-row table (a **full table scan**). With an index on `user_id`, it reads a few index pages and jumps straight to the matching rows.

**The trade-off:** indexes make reads faster but:
- Every `INSERT`, `UPDATE` and `DELETE` must also update every affected index (slower writes).
- They take disk space and buffer pool memory.
- Too many indexes confuse the optimizer and bloat the database.

**How it works**

### 6.1 B-tree and B+ tree

The default index type in almost every relational database is a **B+ tree**: a balanced, wide, shallow tree.

```
                    ┌──────────────┐
                    │   [50 | 100] │                 root (internal node)
                    └──┬────┬────┬─┘
          ┌────────────┘    │    └──────────────┐
   ┌──────▼─────┐   ┌───────▼──────┐   ┌───────▼──────┐
   │ [10|20|35] │   │ [60|75|90]   │   │ [110|150]    │  internal nodes
   └─┬──────────┘   └──────────────┘   └──────────────┘
     ▼
 ┌────────────┐   ┌────────────┐   ┌────────────┐
 │ 1,5,8 →row │ ⇄ │10,12,18→row│ ⇄ │20,25,30→row│ ⇄ ...   leaf nodes, linked list
 └────────────┘   └────────────┘   └────────────┘
```

**B-tree vs B+ tree:**

| | B-tree | B+ tree |
|---|---|---|
| Data stored in | All nodes | **Leaf nodes only** |
| Internal nodes | Keys + data | Keys only, so more keys fit per page (higher fan-out) |
| Leaves linked? | No | **Yes**, doubly linked list |
| Range queries | Need tree traversal | Find the start, then walk the leaf list |
| Tree height | Taller | Shorter |

**Why B+ trees win for databases:**
- **High fan-out:** one 16KB page can hold hundreds of keys. With fan-out 500, a tree of height 3 holds 500³ = **125 million** entries. So a lookup in a 100-million-row table is just **3–4 page reads**, and the top levels are almost always cached in RAM.
- **Range queries** (`WHERE created_at BETWEEN ...`, `ORDER BY`) walk the linked leaves sequentially.
- **Always balanced:** every lookup costs the same.

**Insert and page splits:** when a leaf page is full, it splits into two half-full pages and a new key goes up to the parent. With **sequential keys** (auto-increment, UUID v7), inserts always go to the rightmost leaf and pages fill completely. With **random keys** (UUID v4), inserts land on random pages all over the tree, causing many splits, half-empty pages and more disk I/O.

### 6.2 Other index types

| Type | How it works | Good for | Not good for |
|---|---|---|---|
| **Hash** | Hash(key) → bucket | Exact match `=` | Ranges, sorting, prefix search |
| **Bitmap** | One bit-array per distinct value | Low-cardinality columns in warehouses (`gender`, `status`), combining with AND/OR | Frequent updates (OLTP) |
| **Inverted** | Word → list of documents containing it | Full-text search, JSON keys, arrays | Simple equality on scalars |
| **R-tree / GiST** | Bounding boxes | Geospatial: "restaurants within 2km" | Ordinary values |
| **LSM tree** | See 6.8 | Write-heavy workloads | Read-heavy point lookups (slightly) |

### 6.3 Clustered vs non-clustered indexes

- **Clustered index:** the table rows themselves are stored in index order. Only **one** per table (rows can only be physically sorted one way). In InnoDB, the primary key *is* the clustered index.
- **Non-clustered (secondary) index:** a separate structure with key → pointer to row. Many per table.

**Dense vs sparse:**
- **Dense:** an index entry for every row (secondary indexes are always dense).
- **Sparse:** an entry for every *page*, only possible when data is sorted by that key (a clustered index's internal levels are effectively sparse).

### 6.4 Composite indexes and the leftmost-prefix rule ⭐

A **composite index** covers several columns, sorted by the first column, then by the second within it, and so on, like a phone book sorted by (last name, first name).

```sql
CREATE INDEX idx_orders_user_status_date ON orders (user_id, status, created_at);
```

| Query | Uses index? | Why |
|---|---|---|
| `WHERE user_id = 42` | ✅ | Leftmost column |
| `WHERE user_id = 42 AND status = 'SHIPPED'` | ✅ | Leftmost prefix |
| `WHERE user_id = 42 AND status = 'SHIPPED' ORDER BY created_at` | ✅ fully, no sort needed | All three in order |
| `WHERE status = 'SHIPPED'` | ❌ (mostly) | Skips the first column; like finding everyone named "Ravi" in a phone book sorted by last name |
| `WHERE user_id = 42 AND created_at > '2026-01-01'` | ⚠️ partly | Uses `user_id`, but `status` is skipped so `created_at` can't narrow the search through the index |
| `WHERE user_id > 40 AND status = 'SHIPPED'` | ⚠️ partly | After a **range** condition, later columns can't be used to seek |

**Column order rules of thumb:**
1. Equality columns first (`user_id = ?`, `status = ?`).
2. Then the column you sort by or filter with a range (`created_at`).
3. Among equality columns, order mainly by which queries need to share the index.

(MongoDB calls this the **ESR rule**: Equality, Sort, Range. It applies to SQL too.)

Some newer optimizers can do a **skip scan** (jumping through distinct values of the first column), but don't rely on it in interview answers.

### 6.5 Covering indexes and index-only scans

If the index contains **every column the query needs**, the database never visits the table: an **index-only scan**.

```sql
-- "My orders" page shows id, status, date for one user
CREATE INDEX idx_orders_cover ON orders (user_id, created_at DESC) INCLUDE (status, total);

SELECT id, status, total, created_at
FROM orders
WHERE user_id = 42
ORDER BY created_at DESC
LIMIT 20;
-- Reads only index pages: no table lookups, no sort
```

(`INCLUDE` is PostgreSQL/SQL Server syntax; in MySQL you'd add the columns to the key itself. In InnoDB, the primary key `id` is automatically in every secondary index.)

### 6.6 Selectivity and cardinality

- **Cardinality:** number of distinct values in a column. `email`: very high. `status` (5 values): low.
- **Selectivity:** fraction of rows a condition matches. `WHERE email = ?` → 1 row (very selective). `WHERE status = 'DELIVERED'` → 80% of rows (not selective).

An index helps when the condition is **selective**. If a query will return 80% of the table, reading the index *plus* doing random lookups into the table is slower than one sequential scan, so the optimizer ignores the index. That's correct behaviour, not a bug.

Exception: a low-cardinality column can still benefit from an index if you query the **rare** value, e.g. `WHERE status = 'PAYMENT_FAILED'` (0.1% of rows). A **partial index** is ideal here:

```sql
CREATE INDEX idx_failed_orders ON orders (created_at) WHERE status = 'PAYMENT_FAILED';
```

### 6.7 When an index is NOT used ⭐

| Mistake | Example | Fix |
|---|---|---|
| Function on the column | `WHERE YEAR(created_at) = 2026` | `WHERE created_at >= '2026-01-01' AND created_at < '2027-01-01'`, or an expression index |
| Leading wildcard | `WHERE name LIKE '%phone'` | Full-text / trigram index; `LIKE 'phone%'` *can* use a B-tree |
| Implicit type conversion | `phone` is VARCHAR, query uses `WHERE phone = 9876543210` (number) | Match types: `WHERE phone = '9876543210'` |
| Arithmetic on column | `WHERE price * 1.18 > 1000` | `WHERE price > 1000 / 1.18` |
| OR across different columns | `WHERE user_id = 1 OR email = 'x'` | `UNION` of two indexed queries (some optimizers do this automatically) |
| Skipped leading column | Index `(a, b)`, query `WHERE b = 1` | Add an index on `b` or reorder |
| `!=` / `NOT IN` | `WHERE status != 'DELIVERED'` | Usually not selective anyway; rewrite as `IN (...)` of the rare values |
| Low selectivity | `WHERE is_active = true` (99% true) | Index isn't worth it |
| Stale statistics | Optimizer thinks the table is tiny | Run `ANALYZE` |

### 6.8 B+ tree vs LSM tree ⭐⭐

**LSM tree (Log-Structured Merge tree)** is used by Cassandra, RocksDB, ScyllaDB, LevelDB, and storage engines like MyRocks.

```
Write → [WAL on disk] + [Memtable in RAM, sorted]
                              │ when full, flush
                              ▼
                  SSTable (immutable, sorted file on disk)  L0
                  SSTable  SSTable                          L1
                  SSTable  SSTable  SSTable  SSTable        L2
                  (background compaction merges files and removes old versions)

Read → memtable → newest SSTables → older SSTables
       (Bloom filters skip files that definitely don't contain the key)
```

| | B+ tree | LSM tree |
|---|---|---|
| Write pattern | Update in place (random I/O) | Append only (sequential I/O) |
| Write throughput | Good | **Excellent** |
| Read (point lookup) | **Predictable**, few page reads | May check several files (Bloom filters help) |
| Range scans | Excellent | Good (merge across files) |
| Space | Fragmentation from splits | Old versions until compaction |
| Background work | Little | Compaction (uses CPU and I/O, can cause latency spikes) |
| Used by | PostgreSQL, MySQL InnoDB, MongoDB WiredTiger (default) | Cassandra, RocksDB, ScyllaDB, HBase |

**Bloom filter:** a compact bit array that answers "is this key *possibly* in this file?" It can give false positives but **never false negatives**, so a "no" lets you skip reading the file entirely.

Real life: a messaging app storing billions of chat messages per day (mostly writes, reads by conversation) fits an LSM store like Cassandra. A banking ledger with lots of point lookups and strong consistency fits a B+ tree database.

### Interview Questions

**Q1. How does an index speed up a query? What's the cost?** ⭐
It's a sorted structure (usually a B+ tree) that lets the database jump to matching rows in O(log n) page reads instead of scanning every page. The costs: slower writes (each index must be updated), extra disk space and memory, and more work for the optimizer. Index the columns used in frequent `WHERE`, `JOIN` and `ORDER BY` clauses, and remove unused indexes.

**Q2. Why do databases use B+ trees instead of binary search trees or hash tables?** ⭐
Binary trees have fan-out 2, so a million keys need about 20 levels, meaning 20 disk reads. A B+ tree node is a whole page with hundreds of keys, so 3–4 levels cover hundreds of millions of rows. Hash tables only support equality, not ranges or sorting. B+ trees keep data only in linked leaves, which makes range scans and `ORDER BY` efficient.

**Q3. Explain the leftmost-prefix rule.** ⭐
A composite index on `(a, b, c)` is sorted by a, then b, then c. It can be used for conditions on `a`, `a,b` or `a,b,c`, but not efficiently for `b` or `c` alone, just as a phone book sorted by last name can't find everyone with a given first name. Also, once a column is used with a range (`a > 5`), the following columns can't be used for seeking.

**Q4. What is a covering index?** ⭐
An index that contains every column a query needs, so the database answers from the index alone without reading the table (an index-only scan). Example: index on `(user_id, created_at) INCLUDE (status)` for "list this user's recent orders with status". It saves the random table lookups, which are often the most expensive part.

**Q5. Clustered vs non-clustered index?** ⭐
A clustered index determines the physical order of rows; the leaf level contains the rows themselves, so there can be only one. A non-clustered index is separate and points to the row (by physical address in PostgreSQL, by primary key in InnoDB). In InnoDB, every table is clustered on its primary key.

**Q6. I added an index but the query is still slow. Why?** ⭐
Check with `EXPLAIN`. Common reasons: a function or type cast on the column (`WHERE DATE(created_at) = ...`), a leading wildcard `LIKE '%x'`, the wrong column order in a composite index, low selectivity (the optimizer correctly prefers a scan), stale statistics, or the index is used but the query still fetches too many rows or sorts a large result. Sometimes the problem isn't the index at all: lock waits, network or N+1 queries.

**Q7. Should you index every column used in a WHERE clause?**
No. Index based on actual query patterns, preferring composite indexes that serve several queries. Each index slows writes. Skip low-selectivity columns unless querying a rare value (use a partial index). Periodically find and drop unused indexes (`pg_stat_user_indexes`, `sys.schema_unused_indexes` in MySQL).

**Q8. B+ tree vs LSM tree: when would you choose each?** ⭐⭐
B+ trees update data in place: predictable fast reads, good range scans, moderate write throughput. Best for general OLTP. LSM trees buffer writes in memory and flush sorted immutable files, turning random writes into sequential ones: very high write throughput, but reads may check several files and background compaction uses resources. Best for write-heavy workloads like logs, metrics, messaging and IoT.

**Q9. Why are random UUID primary keys bad for MySQL performance?** ⭐⭐
InnoDB stores rows inside the primary key B+ tree. Random UUIDs insert into random leaf pages, causing frequent page splits, half-empty pages, poor buffer pool locality and more disk writes. Sequential keys always append to the rightmost page. Also, every secondary index stores the 16-byte UUID instead of an 8-byte BIGINT, making all indexes bigger. Use BIGINT, or time-ordered UUID v7/ULID.

---

## 7. Query Processing and Optimization

**Priority:** ⭐⭐ commonly asked

**What it is:** The path a query takes from SQL text to results, and the decisions the database makes along the way.

**Why it matters:** SQL is *declarative*: you say what you want, not how to get it. The **optimizer** chooses how. Reading its choices (with `EXPLAIN`) is the core skill for fixing slow queries.

**How it works**

### 7.1 Query lifecycle

```
SQL text
  │ 1. Parser: check syntax, build a parse tree
  ▼
  │ 2. Analyzer/binder: resolve table and column names, check permissions
  ▼
  │ 3. Rewriter: expand views, simplify expressions, flatten subqueries
  ▼
  │ 4. Optimizer: generate candidate plans, estimate the cost of each, pick the cheapest
  ▼
  │ 5. Executor: run the plan, pulling rows through operators
  ▼
Results
```

### 7.2 The cost-based optimizer

For a query joining 3 tables, there are many plans: which table first, which join algorithm, which index. The optimizer estimates the cost of each (mainly I/O pages and CPU) using **statistics**:
- Number of rows and pages per table
- Number of distinct values per column
- Histograms / most-common values (data distribution)

**Cardinality estimation** (guessing how many rows each step returns) is the hardest part and the source of most bad plans. If statistics say `status = 'PENDING'` matches 10 rows but it really matches 2 million, the optimizer might pick a nested loop join that runs for minutes.

**Fix:** keep statistics fresh (`ANALYZE` in PostgreSQL, `ANALYZE TABLE` in MySQL; usually automatic), and be suspicious when `EXPLAIN ANALYZE` shows estimated rows far from actual rows.

### 7.3 Scan types

| Scan | What it does | When chosen |
|---|---|---|
| **Full table scan** (Seq Scan) | Reads every page in order | No usable index, or query returns a large part of the table |
| **Index scan** | Walk the index, fetch each matching row from the table | Selective condition |
| **Index-only scan** | Answer from the index alone | Covering index |
| **Bitmap scan** (PostgreSQL) | Collect matching row locations from index(es), sort by page, then read pages in order | Medium selectivity, or combining two indexes with AND/OR |

### 7.4 Join algorithms ⭐⭐

**Nested loop join**
```
for each row r in outer table:
    for each row s in inner table where s.key = r.key:   ← fast if inner has an index
        output (r, s)
```
Best when the outer side is small and the inner side has an index on the join key. Example: 20 orders on a page, look up each user by primary key.

**Hash join**
```
Build: hash table from the smaller input, keyed on join column
Probe: scan the larger input, look up each row in the hash table
```
Best for large, unsorted inputs with equality joins (`=`). Needs memory; spills to disk if the hash table doesn't fit (PostgreSQL `work_mem`). MySQL added hash joins in 8.0.18.

**Sort-merge join**
```
Sort both inputs on the join key (or read them already sorted from an index)
Walk both lists together, like merging two sorted arrays
```
Best when both inputs are already sorted (from indexes) or the output must be sorted anyway. Also works for range joins.

| | Nested loop | Hash join | Merge join |
|---|---|---|---|
| Best for | Small outer + indexed inner | Large inputs, equality | Pre-sorted inputs |
| Join condition | Any | Equality only | Equality and ranges |
| Memory | Low | Hash table | Sort buffers |

### 7.5 Common performance problems in application queries

**`SELECT *`**
- Transfers unnecessary data over the network (think large `description TEXT` columns).
- Prevents index-only scans.
- Breaks when columns are added or reordered.
Select only what you need.

**OFFSET pagination** ⭐
```sql
-- Page 5000: the database still reads and throws away 99,980 rows
SELECT * FROM orders ORDER BY created_at DESC LIMIT 20 OFFSET 99980;
```
Cost grows with the page number. Also, rows inserted during browsing shift pages, causing duplicates or skipped items.

**Keyset (seek / cursor) pagination:**
```sql
-- Remember the last row of the previous page
SELECT id, created_at, total FROM orders
WHERE (created_at, id) < ('2026-11-01 10:00:00', 98765)    -- tie-breaker on id
ORDER BY created_at DESC, id DESC
LIMIT 20;
```
Uses the index to jump straight to the right place: page 1 and page 5000 cost the same. Trade-off: you can't jump to an arbitrary page number, only next/previous. This is how infinite scroll feeds (Twitter/X, Instagram) work.

**Counting at scale:** `SELECT COUNT(*) FROM orders` on 500 million rows is slow in MVCC databases, because each row's visibility must be checked. Options: approximate counts from statistics, maintain a counter table, or cache the count. Product teams usually accept "10,000+" instead of an exact number.

**N+1 queries:** covered in §9, since it's an application-layer problem.

### Interview Questions

**Q1. How do you approach a slow query?** ⭐
1. Reproduce it and get the actual plan with `EXPLAIN ANALYZE` (PostgreSQL) or `EXPLAIN ANALYZE` / `EXPLAIN FORMAT=TREE` (MySQL 8).
2. Look for full scans on big tables, large gaps between estimated and actual rows, expensive sorts, and nested loops over large inputs.
3. Fix in order of effort: add or adjust an index, rewrite the query (remove functions on columns, avoid `SELECT *`, use keyset pagination), update statistics, then consider schema changes (denormalize, partition) or caching.
4. Verify with the plan again and with production-like data.

**Q2. What's a query execution plan?** ⭐
The tree of operations the optimizer chose to run a query: scans, joins, sorts, aggregations, with estimated costs and row counts. `EXPLAIN` shows the plan; `EXPLAIN ANALYZE` actually runs it and shows real timings and row counts, which is what you need for diagnosis.

**Q3. Explain nested loop, hash and merge joins.** ⭐⭐
Nested loop: for each outer row, look up matches in the inner table; great with a small outer side and an index on the inner side. Hash join: build a hash table on the smaller input and probe it with the larger one; great for large unsorted equality joins. Merge join: walk two sorted inputs together; great when both are already sorted by the join key. The optimizer chooses based on estimated sizes, indexes and available memory.

**Q4. Why is OFFSET pagination slow and what's the alternative?** ⭐
`OFFSET N` still reads and discards N rows, so deep pages get slower and slower. It also gives duplicate or missing rows if data changes between page loads. Keyset pagination filters on the last seen sort key (`WHERE (created_at, id) < (?, ?)`), using the index to jump directly to the next page, so every page costs the same. The trade-off is no "jump to page 50".

**Q5. The optimizer chose a bad plan. What could cause it?**
Usually wrong cardinality estimates: stale statistics, correlated columns (city = 'Mumbai' and state = 'Maharashtra' aren't independent, but the optimizer assumes they are), skewed data, or parameterized queries planned for a different value. Fixes: `ANALYZE`, extended statistics (PostgreSQL `CREATE STATISTICS`), rewriting the query, and only as a last resort, optimizer hints.

---

## 8. Connection Management

**Priority:** ⭐⭐ commonly asked (very common in backend interviews)

**What it is:** How the application opens, reuses and closes connections to the database.

**Why it matters:** Opening a database connection is expensive: TCP handshake, TLS handshake, authentication, and the database allocating memory (PostgreSQL even starts a whole new OS **process** per connection). Doing that for every HTTP request adds tens of milliseconds and can overwhelm the database.

**How it works**

### 8.1 Connection pooling

A **pool** keeps a set of open connections and lends them to requests:

```
HTTP request 1 ─┐                    ┌── conn 1 ──┐
HTTP request 2 ─┼─► borrow ─► Pool ──┼── conn 2 ──┼──► Database
HTTP request 3 ─┘   (wait if busy)   └── conn 3 ──┘
                    return when done
```

Key settings (names vary by library: HikariCP, node-postgres `Pool`, Prisma, SQLAlchemy):

| Setting | Meaning |
|---|---|
| `max` / pool size | Maximum open connections |
| `min` / idle | Connections kept open when idle |
| Acquire timeout | How long a request waits for a free connection before failing |
| Idle timeout | Close connections idle longer than this |
| Max lifetime | Recycle connections periodically (avoids stale connections, helps after failover) |

### 8.2 Pool sizing

**Bigger is not better.** A database server with 8 CPU cores can only truly run about 8 queries at the same time. With 500 active connections, they fight over CPU, locks and memory, and throughput *drops*.

A widely used starting formula (from the PostgreSQL wiki / HikariCP): `connections ≈ (CPU cores × 2) + number of disks`. For a typical 8-core database, that's around **20 connections in total**, often far fewer than people expect.

**The multiplication problem:** total connections = pool size × number of app instances.

```
20 app pods × pool size 20 = 400 connections
PostgreSQL default max_connections = 100  →  "FATAL: too many connections"
```

Real life: ShopKart autoscales from 5 to 40 Node.js pods during a sale. Each pod has a pool of 10. At 40 pods, that's 400 connections, the database refuses new ones and checkout fails, *at exactly the moment traffic is highest*. Fix: smaller per-pod pools and an external pooler.

### 8.3 External poolers

A proxy that sits between many app instances and the database, multiplexing many client connections onto few database connections:
- **PgBouncer** (PostgreSQL): in *transaction pooling* mode, a server connection is assigned only for the duration of a transaction. 2,000 client connections can share 50 database connections.
- **ProxySQL** (MySQL), **RDS Proxy** (AWS, both).

Caveat: in transaction pooling mode, session-level features (session variables, `SET`, advisory locks held across transactions, some prepared statement setups) don't work as expected, because consecutive transactions may run on different server connections.

### 8.4 Connection leaks and pool exhaustion

A **leak** is a connection borrowed and never returned, usually because of an error path:

```javascript
// ❌ Leak: if the query throws, release() never runs
const client = await pool.connect();
const res = await client.query('SELECT ...');
client.release();

// ✅ Always release in finally
const client = await pool.connect();
try {
  const res = await client.query('SELECT ...');
  return res.rows;
} finally {
  client.release();
}
```

Symptoms: everything works for hours, then requests start timing out with "timeout acquiring connection", and restarting the app "fixes" it. Other causes of exhaustion: long transactions holding connections, slow queries under load, or making HTTP calls while holding a connection inside a transaction.

### 8.5 Serverless

AWS Lambda / Vercel functions can scale to thousands of concurrent instances, each opening its own connection. Solutions: an external pooler (RDS Proxy, PgBouncer), serverless-friendly drivers that use HTTP (Neon, PlanetScale), or a pool size of 1 per function with a connection created outside the handler so it's reused across invocations.

### Interview Questions

**Q1. Why do we need connection pooling?** ⭐
Creating a connection costs a network round trip, TLS, authentication and server-side memory (a whole process in PostgreSQL). Pooling reuses a fixed set of open connections, cutting latency per request and protecting the database from being flooded with connections.

**Q2. How do you size a connection pool?** ⭐⭐
Start from what the database can handle, not what the app wants: roughly `2 × CPU cores` of active connections for the whole database. Divide that across app instances, remembering that total = pool size × instances (including autoscaling peaks). Then load-test and watch acquire-wait times and database CPU. If you need more client connections than the database can handle, add PgBouncer/RDS Proxy rather than raising `max_connections`.

**Q3. Our app gets "too many connections" errors after scaling out. What do you do?** ⭐
Total connections = instances × pool size. Short term: lower the per-instance pool size. Proper fix: an external pooler (PgBouncer in transaction mode, RDS Proxy) so many app connections share few database connections. Also check for leaks and long transactions that hold connections, and set `max_connections` based on database memory rather than just raising it.

**Q4. What's a connection leak and how do you find it?**
A connection that's borrowed and never returned, usually because an exception skipped the release. The pool slowly empties until requests time out. Find it with pool metrics (active vs idle count), leak-detection settings (HikariCP `leakDetectionThreshold`), and database-side views like `pg_stat_activity` showing connections stuck "idle in transaction". Fix with `try/finally` or framework-managed transactions.

---

## 9. ORMs and the Application Layer

**Priority:** ⭐ must-know for backend roles

> Examples: **Prisma, TypeORM, Sequelize** (Node.js) · **SQLAlchemy, Django ORM** (Python)

**What it is:** An ORM (Object-Relational Mapper) maps database rows to objects in your programming language, so you write `user.orders` instead of SQL.

**Why it matters:** Most real-world database performance problems in backend services come from how the application uses the ORM, not from the database itself. Interviewers ask about N+1, transactions and migrations to check that you understand what SQL your code actually generates.

**How it works**

### 9.1 ORM vs query builder vs raw SQL

| | ORM | Query builder | Raw SQL |
|---|---|---|---|
| Examples | Prisma, TypeORM, Django ORM, SQLAlchemy ORM | Knex, Kysely, SQLAlchemy Core | `pg`, `mysql2`, `psycopg` |
| Productivity | High | Medium | Lower |
| Type safety | Good (Prisma, Kysely especially) | Good | Manual |
| Control over SQL | Low | High | Full |
| Complex queries (windows, CTEs) | Awkward, often drop to raw SQL | Good | Best |
| Risk | Hidden N+1, inefficient SQL | Some boilerplate | SQL injection if you concatenate strings |

Common practical answer: use the ORM for CRUD and simple relations, and raw SQL (parameterized) for complex reports and performance-critical paths.

### 9.2 The N+1 query problem ⭐

```javascript
// Show 50 orders with the customer's name
const orders = await Order.findAll({ limit: 50 });     // 1 query
for (const order of orders) {
  const user = await order.getUser();                  // 50 more queries!
  console.log(order.id, user.name);
}
// Total: 1 + N = 51 queries
```

Each query might take only 1–2 ms, but 51 round trips add up, and with nested relations (orders → items → products) it becomes hundreds of queries per page.

**How to spot it:** enable query logging in development and look for the same query repeated with different IDs; APM tools (Datadog, New Relic) flag it; Django has `django-debug-toolbar`.

**Fixes:**
```javascript
// Prisma: eager load with include → 2 queries (orders, then users WHERE id IN (...))
const orders = await prisma.order.findMany({ take: 50, include: { user: true } });

// Sequelize: include → 1 query with JOIN
const orders = await Order.findAll({ limit: 50, include: [User] });
```
```python
# Django
Order.objects.select_related("user")[:50]        # JOIN (for FK / one-to-one)
User.objects.prefetch_related("orders")          # second query with IN (...) (for one-to-many / M:N)

# SQLAlchemy
select(Order).options(joinedload(Order.user))    # JOIN
select(User).options(selectinload(User.orders))  # SELECT ... WHERE user_id IN (...)
```
In GraphQL APIs, use **DataLoader** to batch the lookups made by many resolvers into one `IN (...)` query.

### 9.3 Lazy vs eager loading

| | Lazy loading | Eager loading |
|---|---|---|
| When related data loads | When you first access it | Upfront, with the main query |
| Risk | N+1 queries; "lazy load outside session" errors | Loading data you don't need; huge JOINs |
| Good for | Rarely accessed relations | Relations you know you'll use |

Eager loading with a JOIN on a one-to-many relation duplicates the parent row for every child ("cartesian explosion"); loading several collections that way multiplies rows. That's why ORMs offer a separate-query strategy (`selectinload`, `prefetch_related`, Prisma's default).

### 9.4 Transactions in application code ⭐

```javascript
// Prisma interactive transaction: transfer money between wallets
await prisma.$transaction(async (tx) => {
  const from = await tx.account.update({
    where: { id: fromId },
    data: { balance: { decrement: amount } },
  });
  if (from.balance < 0) throw new Error('Insufficient funds');   // rolls back
  await tx.account.update({
    where: { id: toId },
    data: { balance: { increment: amount } },
  });
});
```
```python
# Django
from django.db import transaction
with transaction.atomic():
    ...
```

**Rules:**
1. **Keep transactions short.** A transaction holds locks and (in PostgreSQL) prevents cleanup of old row versions for as long as it's open.
2. **Never make network calls inside a transaction.** If you call a payment gateway that takes 10 seconds, you hold locks and a connection for 10 seconds. If it times out, you don't know whether the payment happened, but your DB transaction rolls back anyway. Instead: write a `PENDING` record, commit, call the external service, then update the status in a new transaction (and use idempotency keys; see §19).
3. **Be explicit about boundaries.** Each ORM call without a transaction runs in its own auto-commit transaction. Two separate `save()` calls are *not* atomic.
4. **Retry on serialization failures and deadlocks** (with a limit), since databases are allowed to abort transactions under concurrency.

### 9.5 Optimistic locking in ORMs

Add a `version` column; every update checks and increments it:
```sql
UPDATE products SET stock = 9, version = 6
WHERE id = 100 AND version = 5;
-- 0 rows affected → someone else updated it first → retry or tell the user
```
TypeORM (`@VersionColumn`), Sequelize (`version: true`), Hibernate (`@Version`) and SQLAlchemy (`version_id_col`) support this natively. See §12 for when to choose optimistic vs pessimistic locking.

### 9.6 Migrations

Schema changes as versioned code files, applied in order and tracked in a table in the database (`_prisma_migrations`, `django_migrations`, `alembic_version`).

Tools: Prisma Migrate, TypeORM migrations, Sequelize CLI, Knex, Alembic (SQLAlchemy), Django migrations, Flyway/Liquibase.

**Good practices:**
- Review the generated SQL before running it in production; ORMs sometimes generate a table-locking operation.
- Make migrations **backward-compatible** so old and new app versions can run at the same time during a deploy (expand → migrate → contract; see §22).
- Separate schema migrations from large data backfills; backfill in batches.
- Don't rely on "down" migrations in production; roll forward with a fix instead.

### 9.7 Bulk operations

```javascript
// ❌ 10,000 INSERT statements, 10,000 round trips
for (const p of products) await prisma.product.create({ data: p });

// ✅ One statement (or a few batches)
await prisma.product.createMany({ data: products });
```
Same for updates: `UPDATE ... WHERE id IN (...)` or `UPDATE ... FROM (VALUES ...)` rather than a loop. For very large loads, use the database's bulk loader (`COPY` in PostgreSQL, `LOAD DATA` in MySQL).

### 9.8 See the SQL your ORM generates

Always know how: Prisma `log: ['query']`, Sequelize `logging: console.log`, TypeORM `logging: true`, Django `connection.queries` / `str(queryset.query)`, SQLAlchemy `echo=True`. Then run `EXPLAIN` on it.

### Interview Questions

**Q1. What is the N+1 problem and how do you fix it?** ⭐
Loading a list with one query, then running one extra query per item to load a relation: 1 + N queries. It's usually caused by lazy loading inside a loop. Fix by eager loading the relation with a JOIN or a single `WHERE id IN (...)` query (`include` in Prisma/Sequelize, `select_related`/`prefetch_related` in Django, `joinedload`/`selectinload` in SQLAlchemy), or by batching with DataLoader in GraphQL. Detect it through query logs or APM tools.

**Q2. ORM or raw SQL?** ⭐
Both, for different jobs. ORMs speed up CRUD work, give type safety and handle migrations. Raw SQL (always parameterized) is better for complex reporting queries, window functions, bulk operations and hot paths where you need full control. The key skill is knowing what SQL the ORM generates and checking it with `EXPLAIN`.

**Q3. Why shouldn't you call an external API inside a database transaction?** ⭐
The transaction holds row locks and a pooled connection for the entire duration of the network call, which can be seconds. Under load this blocks other requests and exhausts the pool. It's also not atomic anyway: if the API call succeeds but the commit fails, or the call times out, the database and the external system disagree. Use a status field (`PENDING` → `CONFIRMED`), idempotency keys and, for reliability, the outbox pattern.

**Q4. Lazy vs eager loading: which should be the default?**
Neither blindly. Lazy loading is convenient but causes N+1 queries in loops, so many teams disable or warn on it. Eager loading avoids N+1 but can fetch too much or create huge JOIN results. Choose per query: eager-load what the endpoint will definitely use, and use separate batched queries (`selectinload`, `prefetch_related`) for collections.

**Q5. How do you deploy a schema migration safely?** ⭐⭐
Make it backward-compatible with the currently running code, apply it before deploying code that depends on it, avoid long table locks (check what the generated SQL does; use online DDL tools or concurrent index creation), and backfill data in batches. For destructive changes like renaming or dropping a column, use expand-and-contract over multiple deploys.

---

# PART 3: TRANSACTIONS AND DURABILITY

---

## 10. Transactions and ACID

**Priority:** ⭐ must-know

**What it is:** A **transaction** is a group of operations that the database treats as a single unit: either all of them happen, or none of them do.

**Why it exists:** The classic example is a wallet transfer on ShopKart: Asha pays Ravi ₹500.

```sql
UPDATE accounts SET balance = balance - 500 WHERE user_id = 1;   -- Asha
-- 💥 server crashes here
UPDATE accounts SET balance = balance + 500 WHERE user_id = 2;   -- Ravi
```

Without a transaction, ₹500 disappears. With a transaction, a crash in the middle means *neither* update is kept.

```sql
BEGIN;
UPDATE accounts SET balance = balance - 500 WHERE user_id = 1;
UPDATE accounts SET balance = balance + 500 WHERE user_id = 2;
COMMIT;      -- or ROLLBACK; to undo everything
```

**How it works: ACID**

| Property | Meaning | ShopKart example | How the database enforces it |
|---|---|---|---|
| **Atomicity** | All or nothing | Debit and credit both happen, or neither | Undo log / rollback; MVCC versions |
| **Consistency** | A transaction moves the database from one valid state to another | Balance never goes negative (`CHECK (balance >= 0)`), every order has a valid user (FK) | Constraints, triggers, *plus your application logic* |
| **Isolation** | Concurrent transactions don't see each other's half-done work | Two checkouts buying the last phone don't both succeed | Locks, MVCC, isolation levels (§11–13) |
| **Durability** | Once committed, data survives crashes | After "Payment successful", a power cut doesn't lose the order | Write-ahead log flushed to disk on commit (§14) |

Note: "Consistency" in ACID means *integrity rules are respected*. It's **not** the same as "consistency" in CAP (§17), which means all replicas return the same latest value. Interviewers like this trap.

**Transaction states:**
```
          ┌─────────► Partially committed ──► Committed
 Active ──┤                    │
          └─────────► Failed ◄─┘ ──► Aborted (rolled back) ──► restart or give up
```

**Savepoints** let you roll back part of a transaction:
```sql
BEGIN;
INSERT INTO orders ...;
SAVEPOINT before_coupon;
UPDATE coupons SET used = true WHERE code = 'DIWALI50';   -- fails
ROLLBACK TO before_coupon;                                -- keep the order, skip the coupon
COMMIT;
```

### 10.1 Schedules and serializability ⭐⭐

A **schedule** is the order in which operations from concurrent transactions actually execute.

- **Serial schedule:** transactions run one after another. Always correct, but no concurrency.
- **Serializable schedule:** interleaved, but the result equals *some* serial order. This is the gold standard of correctness.

**Conflicting operations:** two operations from different transactions on the same item where at least one is a write (R-W, W-R, W-W).

**Conflict serializability test (precedence graph):**
1. One node per transaction.
2. Draw an edge T1 → T2 if an operation of T1 conflicts with and comes *before* an operation of T2.
3. **No cycle → conflict serializable.** A cycle means there's no equivalent serial order.

```
Schedule: R1(A) W2(A) W1(A)
R1(A) before W2(A) → edge T1 → T2
W2(A) before W1(A) → edge T2 → T1
Cycle T1 ⇄ T2 → NOT conflict serializable
```

**View serializability** is a weaker (broader) condition; every conflict-serializable schedule is view-serializable, but not the other way round. Testing it is NP-complete, so databases don't use it in practice.

**Recoverability** (what happens when a transaction aborts):
- **Recoverable:** a transaction commits only after every transaction it read from has committed.
- **Cascadeless:** transactions only read *committed* data, so one abort can't force others to abort (a "cascading rollback").
- **Strict:** no one reads or overwrites an item until the transaction that last wrote it has finished. What strict 2PL gives you (§12).

### Interview Questions

**Q1. Explain ACID with a real example.** ⭐
A bank transfer: debit A, credit B. **Atomicity**: both or neither. **Consistency**: constraints hold (no negative balance, total money unchanged). **Isolation**: a concurrent report never sees money "in flight" (debited but not yet credited). **Durability**: once the app shows "Transfer successful", a crash can't undo it, because the commit was written to the log on disk.

**Q2. How does a database guarantee atomicity and durability?** ⭐⭐
With a **write-ahead log**: every change is recorded in a sequential log before data pages are modified, and on commit the log is flushed to disk. After a crash, the database replays committed changes (redo) and reverses uncommitted ones (undo). See §14.

**Q3. Is the "C" in ACID the same as the "C" in CAP?** ⭐
No. ACID consistency means the transaction respects integrity rules (constraints, invariants). CAP consistency means linearizability: every read sees the most recent write across all replicas.

**Q4. What's a serializable schedule? How do you test for it?** ⭐⭐
A concurrent schedule whose result is the same as running the transactions one after another in some order. Test for conflict serializability by building a precedence graph (edge Ti → Tj when an operation of Ti conflicts with a later operation of Tj); it's conflict-serializable if and only if the graph has no cycle.

**Q5. Does MongoDB / Redis support ACID transactions?**
MongoDB: single-document operations are always atomic; multi-document ACID transactions exist since 4.0 (replica sets) and 4.2 (sharded clusters), with a performance cost. Redis: `MULTI/EXEC` runs commands atomically and in isolation (single-threaded), but has **no rollback** if a command fails, and durability depends on persistence settings. Details in files 03 and 04.

---

## 11. Isolation Levels

**Priority:** ⭐ must-know (asked in almost every backend interview)

**What it is:** How much concurrent transactions are allowed to see of each other's changes. The SQL standard defines four levels; each one prevents more **anomalies** but allows less concurrency.

**Why it exists:** Full isolation (running transactions one at a time) is correct but slow. Isolation levels let you trade some correctness for throughput when your use case allows it.

**How it works**

### 11.1 The anomalies (read phenomena)

**Dirty write:** overwriting another transaction's *uncommitted* write. This is the one anomaly that **no isolation level allows**, not even Read Uncommitted, because every database holds a write lock on a modified row until the transaction ends.
```
T1: UPDATE orders SET status = 'SHIPPED', courier = 'Delhivery' WHERE id = 7;   (not committed)
T2: UPDATE orders SET status = 'CANCELLED', courier = NULL WHERE id = 7;         ← would overwrite T1's uncommitted row
T1: ROLLBACK;   ← roll back to what? T2's change is on top of T1's
```
If it were allowed, rollback would be impossible to do correctly, and rows updated by two transactions could end up with a mix of both (status from one, courier from the other). In practice T2 simply **waits** for T1's row lock.

**Dirty read:** reading another transaction's *uncommitted* data.
```
T1: UPDATE accounts SET balance = 0 WHERE id = 1;    (not committed)
T2: SELECT balance FROM accounts WHERE id = 1;  → 0   ← reads uncommitted value
T1: ROLLBACK;                                         ← the 0 never really existed
```
Real life: a fraud check approves a payment based on a balance that was rolled back.

**Non-repeatable read:** reading the same row twice in one transaction and getting different values, because another transaction committed an update in between.
```
T1: SELECT price FROM products WHERE id = 100;   → 1000
T2: UPDATE products SET price = 1200 WHERE id = 100; COMMIT;
T1: SELECT price FROM products WHERE id = 100;   → 1200   ← changed within my transaction
```
Real life: an invoice is generated with the price read twice: once for the line item, once for the total, and they don't match.

**Phantom read:** running the same *range* query twice and getting different **sets** of rows, because another transaction inserted or deleted matching rows.
```
T1: SELECT COUNT(*) FROM orders WHERE status = 'PENDING';   → 10
T2: INSERT INTO orders (..., status) VALUES (..., 'PENDING'); COMMIT;
T1: SELECT COUNT(*) FROM orders WHERE status = 'PENDING';   → 11   ← a "phantom" row
```

**Lost update:** two transactions read the same value, both modify it, and one overwrites the other.
```
Stock = 10
T1: read stock → 10            T2: read stock → 10
T1: write stock = 10 - 1 = 9   T2: write stock = 10 - 1 = 9
Final: 9  (should be 8; one sale was "lost")
```
Real life: two customers buy the last units of an item and the stock count goes wrong, leading to overselling.

**Write skew:** two transactions read overlapping data, make decisions based on it, and write *different* rows, together breaking a rule neither broke alone.
```
Rule: at least one doctor must be on call. Alice and Bob are both on call.
T1 (Alice): SELECT COUNT(*) on-call → 2, so it's OK for me to leave → UPDATE alice SET on_call = false
T2 (Bob):   SELECT COUNT(*) on-call → 2, so it's OK for me to leave → UPDATE bob   SET on_call = false
Both commit → zero doctors on call
```
ShopKart version: two admins each remove a different product from a promotion that must keep at least one product. Or two users each book a different seat but together exceed a group booking limit.

### 11.2 The four levels

| Isolation level | Dirty read | Non-repeatable read | Phantom read | Typical implementation |
|---|---|---|---|---|
| **Read Uncommitted** | ⚠️ possible | ⚠️ possible | ⚠️ possible | Rarely used |
| **Read Committed** | ✅ prevented | ⚠️ possible | ⚠️ possible | Each *statement* sees a fresh snapshot of committed data |
| **Repeatable Read** | ✅ prevented | ✅ prevented | ⚠️ possible per SQL standard (prevented in practice by PostgreSQL and mostly by MySQL InnoDB) | One snapshot for the whole *transaction* |
| **Serializable** | ✅ prevented | ✅ prevented | ✅ prevented | Strict 2PL, or SSI (PostgreSQL) |

Lost updates and write skew aren't in the original SQL standard table, but interviewers ask about them:
- **Lost update:** prevented by Serializable; PostgreSQL's Repeatable Read also detects it (aborts one transaction). In MySQL's Repeatable Read it can still happen with a plain read-then-write, so use `SELECT ... FOR UPDATE` or atomic updates.
- **Write skew:** only prevented by Serializable (or explicit locking).

**Default levels:**
- **PostgreSQL, Oracle, SQL Server:** Read Committed
- **MySQL InnoDB:** Repeatable Read
- **MongoDB:** behaves like Read Committed for individual operations; multi-document transactions use snapshot isolation.

### 11.3 Snapshot isolation ⭐⭐

Most modern databases implement Repeatable Read using **snapshot isolation (SI)**: when a transaction starts, it sees a frozen snapshot of all committed data as of that moment, using MVCC (§13). Readers never block writers and writers never block readers.

SI prevents dirty reads, non-repeatable reads and phantoms, but **allows write skew**, because two transactions writing *different* rows don't conflict with each other.

PostgreSQL's **Serializable** level uses **Serializable Snapshot Isolation (SSI)**: it runs like SI but tracks read/write dependencies between transactions and **aborts** one if a dangerous pattern appears. Your application must be ready to **retry** on `could not serialize access` errors.

### 11.4 Preventing anomalies without changing the isolation level

Most real systems stay on Read Committed and fix specific cases:

```sql
-- 1. Atomic update (no read-then-write in the app): fixes lost update
UPDATE products SET stock = stock - 1 WHERE id = 100 AND stock > 0;
-- check rows affected: 0 means out of stock

-- 2. Lock the rows you read: fixes lost update and write skew
BEGIN;
SELECT * FROM doctors WHERE on_call = true FOR UPDATE;
-- decide, then update
COMMIT;

-- 3. Optimistic version check (§12)
UPDATE products SET stock = 9, version = version + 1 WHERE id = 100 AND version = 5;

-- 4. A constraint the database enforces
ALTER TABLE accounts ADD CONSTRAINT positive_balance CHECK (balance >= 0);
```

### Interview Questions

**Q1. Explain the isolation levels and the anomalies each one prevents.** ⭐
Read Uncommitted allows dirty reads. Read Committed prevents dirty reads, but rows can change between statements (non-repeatable reads) and new rows can appear (phantoms). Repeatable Read keeps the same snapshot for the whole transaction, preventing non-repeatable reads (and in PostgreSQL/InnoDB, phantoms in practice). Serializable guarantees the result equals some serial order, preventing everything including write skew. Higher levels cost more blocking or more aborted transactions.

**Q2. What's the difference between a non-repeatable read and a phantom read?** ⭐
A non-repeatable read is an *existing row's value* changing between two reads. A phantom is the *set of rows* matching a condition changing because rows were inserted or deleted. Locking the rows you read stops non-repeatable reads but not phantoms; preventing phantoms needs range/predicate locks (gap locks in InnoDB) or snapshots.

**Q3. What's the default isolation level in MySQL and PostgreSQL?** ⭐
MySQL InnoDB: Repeatable Read. PostgreSQL: Read Committed. A practical consequence: the same application code can behave differently under concurrency on the two databases.

**Q4. What is write skew? Give an example.** ⭐⭐
Two transactions read the same data, each decides it's safe to proceed, and each writes a *different* row, together breaking an invariant. Example: two on-call doctors both check "2 doctors on call, I can leave" and both go off call. Snapshot isolation doesn't catch it because the writes don't touch the same row. Fix with Serializable isolation, `SELECT ... FOR UPDATE` on the rows that were read, or modeling the invariant so it becomes a single-row update or a constraint.

**Q5. How do you prevent overselling the last item in stock?** ⭐
Simplest and best: an atomic conditional update, `UPDATE products SET stock = stock - 1 WHERE id = ? AND stock > 0`, then check that one row was affected. Alternatives: `SELECT ... FOR UPDATE` then update (pessimistic), or a version column (optimistic). Add `CHECK (stock >= 0)` as a final safety net. At very high scale (flash sales), pre-allocate stock in Redis with an atomic `DECR` and reconcile with the database afterwards.

**Q6. Why not always use Serializable?** ⭐⭐
It reduces throughput: with locking, more blocking and deadlocks; with SSI, more transactions aborted and retried under contention. Every write path must handle retries. Most applications get correctness more cheaply with Read Committed plus targeted fixes: atomic updates, row locks or constraints on the few operations that need them.

**Q7. What is a dirty write, and why does no isolation level allow it?** ⭐⭐
A dirty write is one transaction overwriting a row that another transaction has modified but not yet committed. If it were allowed, rollback couldn't work (undoing T1 would also wipe T2's change, or leave a half-mixed row), so even Read Uncommitted prevents it: a writer holds an exclusive row lock until commit or rollback, and the second writer waits. Isolation levels only differ in what *reads* can see.

---

## 12. Concurrency Control and Locking

**Priority:** ⭐⭐ commonly asked

**What it is:** The mechanisms a database uses to let many transactions run at once while keeping data correct.

**Why it exists:** Isolation levels (§11) describe *what* guarantees you get. Locks and MVCC are *how* the database delivers them.

**How it works**

### 12.1 Lock types

| Lock | Also called | Allows others to | Taken by |
|---|---|---|---|
| **Shared (S)** | Read lock | Read, but not write | `SELECT ... FOR SHARE` |
| **Exclusive (X)** | Write lock | Nothing | `UPDATE`, `DELETE`, `SELECT ... FOR UPDATE` |

**Compatibility matrix:**

| | S held | X held |
|---|---|---|
| Request S | ✅ | ❌ wait |
| Request X | ❌ wait | ❌ wait |

**Lock granularity:** row, page, or table. Row locks allow the most concurrency but cost more memory to track. Some databases **escalate** many row locks into a table lock (SQL Server does; PostgreSQL and InnoDB don't).

**Intent locks** (IS, IX): placed on the *table* when a transaction locks rows inside it. Without them, a transaction wanting to lock the whole table would have to check every row for locks. With them, it just checks the table's intent lock.

### 12.2 Two-phase locking (2PL)

A transaction has two phases:
1. **Growing phase:** acquire locks, release none.
2. **Shrinking phase:** release locks, acquire none.

```
locks held
   ▲        ┌───────┐
   │       ╱         ╲
   │      ╱           ╲
   │     ╱             ╲
   └────┴───────────────┴──► time
       growing   lock    shrinking
                 point
```

2PL guarantees conflict serializability, but it can cause **cascading rollbacks** (others may read data you later roll back, if you release locks before committing).

**Strict 2PL** (what real databases use): hold all **exclusive** locks until commit or rollback. No one can read uncommitted data, which prevents cascading aborts.
**Rigorous 2PL:** hold *all* locks (shared too) until the end.

Neither form prevents **deadlocks**.

### 12.3 Deadlocks ⭐

Two transactions each hold a lock the other needs:
```
T1: UPDATE accounts ... WHERE id = 1;   -- locks row 1
T2: UPDATE accounts ... WHERE id = 2;   -- locks row 2
T1: UPDATE accounts ... WHERE id = 2;   -- waits for T2
T2: UPDATE accounts ... WHERE id = 1;   -- waits for T1  → deadlock
```
Real life: two transfers at the same moment, Asha → Ravi and Ravi → Asha.

**Detection:** the database maintains a **wait-for graph** (edge T1 → T2 means T1 waits for T2). A cycle is a deadlock. The database picks a **victim** (usually the transaction that's cheapest to roll back), aborts it with a deadlock error, and the other continues. PostgreSQL checks after `deadlock_timeout` (1s default); InnoDB detects immediately.

**Prevention schemes** (timestamp-based, mostly theory questions):
- **Wait-Die:** an older transaction may wait for a younger one; a younger one requesting a lock held by an older one dies (aborts) and restarts with its original timestamp.
- **Wound-Wait:** an older transaction "wounds" (aborts) a younger one holding the lock; a younger one waits for an older one.
- **Timeouts:** give up after waiting N seconds (InnoDB `innodb_lock_wait_timeout`, 50s default).

**Prevention in practice (what to say in an interview):**
1. **Lock in a consistent order.** For transfers, always lock the lower account ID first:
   ```sql
   SELECT * FROM accounts WHERE id IN (1, 2) ORDER BY id FOR UPDATE;
   ```
2. **Keep transactions short** and don't wait for user input or network calls inside them.
3. **Index the columns in `WHERE`** of `UPDATE`/`DELETE`; otherwise the database scans and locks far more rows than needed (especially InnoDB with gap locks).
4. **Retry** the aborted transaction; deadlocks can't be fully eliminated, so code must handle them.

### 12.4 Optimistic vs pessimistic concurrency control ⭐

**Pessimistic:** assume conflicts will happen; lock first.
```sql
BEGIN;
SELECT stock FROM products WHERE id = 100 FOR UPDATE;   -- others wait here
UPDATE products SET stock = stock - 1 WHERE id = 100;
COMMIT;
```

**Optimistic:** assume conflicts are rare; don't lock, but detect conflicts at write time with a version number or timestamp.
```sql
SELECT stock, version FROM products WHERE id = 100;          -- stock 10, version 5
-- ... user edits for a while, no locks held ...
UPDATE products SET stock = 9, version = 6
WHERE id = 100 AND version = 5;
-- 1 row affected → success. 0 rows → someone changed it; reload and retry
```

| | Pessimistic | Optimistic |
|---|---|---|
| Best when | Conflicts are frequent (hot rows, flash sales, seat booking) | Conflicts are rare (editing a user profile, admin edits) |
| Cost | Waiting, possible deadlocks | Retries when conflicts happen |
| Works across HTTP requests? | No (can't hold a DB lock between requests) | **Yes**: send `version` to the client, check it on save |
| Example | `SELECT ... FOR UPDATE` | `WHERE version = ?`, HTTP `ETag` / `If-Match` |

Real life: a CMS where two editors open the same product page. With optimistic locking, the second to save gets "This product was changed by someone else, please reload" instead of silently overwriting the first editor's changes.

**`SKIP LOCKED` and `NOWAIT`:**
```sql
-- Job queue: each worker grabs a different pending job, without waiting on each other
SELECT * FROM jobs WHERE status = 'PENDING'
ORDER BY created_at LIMIT 1
FOR UPDATE SKIP LOCKED;

-- Fail immediately instead of waiting if the row is locked
SELECT * FROM seats WHERE id = 42 FOR UPDATE NOWAIT;
```

### 12.5 Timestamp ordering ⭐⭐⭐

Every transaction gets a timestamp; operations must happen in timestamp order. Each data item tracks the largest read timestamp and write timestamp. If a transaction tries to read something already overwritten by a younger transaction, or to write something a younger transaction already read, it's aborted and restarted. No locks, so no deadlocks, but more aborts. The **Thomas write rule** lets the database silently ignore outdated writes instead of aborting. This idea is the foundation of MVCC.

### Interview Questions

**Q1. What is a deadlock and how do you prevent it?** ⭐
Two or more transactions each waiting for a lock held by another, so none can proceed. Databases detect them with a wait-for graph and abort one transaction. To reduce them: acquire locks in a consistent order (e.g. sort IDs before locking), keep transactions short, index the columns used in update conditions so fewer rows are locked, and always retry deadlock errors in application code.

**Q2. Optimistic vs pessimistic locking: when do you use each?** ⭐
Pessimistic (`SELECT ... FOR UPDATE`) when conflicts are likely and retrying is expensive: inventory during a sale, seat booking, money movement. Optimistic (version column) when conflicts are rare and you can't hold locks, e.g. a user editing a form for minutes. Optimistic locking also works across HTTP requests and in distributed systems.

**Q3. What is two-phase locking? Does it prevent deadlocks?** ⭐⭐
A transaction acquires all its locks before releasing any (a growing phase, then a shrinking phase), which guarantees serializable schedules. Strict 2PL holds write locks until commit, preventing cascading rollbacks. It does **not** prevent deadlocks; it can cause them.

**Q4. How would you build a job queue on a relational database?** ⭐⭐
A `jobs` table with a status column and an index on `(status, created_at)`. Workers run `SELECT ... WHERE status = 'PENDING' ORDER BY created_at LIMIT 1 FOR UPDATE SKIP LOCKED`, mark the job as running, and commit. `SKIP LOCKED` lets many workers take different jobs without blocking each other. Handle crashed workers with a lease/timeout column. It works well up to moderate volumes; beyond that, use a dedicated queue (SQS, RabbitMQ, Kafka).

**Q5. Why does a missing index make locking worse?**
To update rows matching a `WHERE` clause, the database has to find them. Without an index it scans the table, and in some engines (InnoDB) it locks every row it examines, not just the ones it changes. One `UPDATE ... WHERE email = ?` without an index on `email` can lock most of the table and block every other writer.

---

## 13. MVCC

**Priority:** ⭐⭐ commonly asked

**What it is:** **Multi-Version Concurrency Control** keeps **multiple versions** of each row. Each transaction sees the version that was committed when its snapshot was taken.

**Why it exists:** With pure locking, a long report reading the `orders` table would block every checkout trying to update an order, and vice versa. MVCC's key promise: **readers don't block writers, and writers don't block readers.** Only writers updating the *same row* wait for each other.

**How it works**

```
Row: product 100

 version 1: stock=10   created by T5   (deleted by T9)
 version 2: stock=9    created by T9   (deleted by T12)
 version 3: stock=8    created by T12  (current)

Transaction T10 started after T9 committed but before T12 → sees stock = 9
A new transaction now → sees stock = 8
```

Each row version records **which transaction created it** and **which transaction deleted/replaced it**. When a transaction reads, it checks each version against its **snapshot**: "Which transactions had committed when I started?" It sees the newest version created by a committed-before-me transaction and not yet deleted by one.

**What an UPDATE does:** it doesn't overwrite the row. It creates a new version and marks the old one as replaced.

**Cleaning up old versions:** old versions are needed only while some running transaction might still need them. After that they're garbage.
- **PostgreSQL:** old versions stay in the table itself; **VACUUM** removes them later. Dead rows cause **bloat**.
- **MySQL InnoDB:** the table holds the latest version; older versions are rebuilt from the **undo log**; a **purge** thread removes undo records that are no longer needed.
- **MongoDB (WiredTiger):** keeps versions in memory (cache) for active snapshots.

**The long-transaction problem:** a transaction that stays open for hours (a forgotten `BEGIN` in a SQL console, a stuck worker) keeps an old snapshot alive, so *no version created after it started can be cleaned up anywhere*. Tables and undo logs bloat, and queries get slower. Real life: a developer opens a transaction in a DB GUI tool, goes to lunch, and the production database's disk usage climbs for hours. Monitor and kill "idle in transaction" sessions.

**MVCC + isolation levels:**
- **Read Committed:** a new snapshot for **each statement**.
- **Repeatable Read / Snapshot:** one snapshot for the **whole transaction**.

**Write conflicts still need locks:** if two transactions update the same row, the second waits for the first to commit or roll back. Then, depending on isolation level, it either proceeds with the new version (Read Committed) or aborts with a serialization error (PostgreSQL Repeatable Read).

### Interview Questions

**Q1. What is MVCC and why is it useful?** ⭐⭐
The database keeps multiple versions of rows so each transaction reads a consistent snapshot without taking read locks. Readers and writers don't block each other, which greatly improves concurrency for mixed workloads. The costs are storage for old versions and background cleanup (VACUUM in PostgreSQL, purge in InnoDB).

**Q2. How does MVCC implement Read Committed vs Repeatable Read?** ⭐⭐
Same mechanism, different snapshot timing. Read Committed takes a new snapshot at the start of every statement, so each statement sees the latest committed data. Repeatable Read takes one snapshot at the start of the transaction (in PostgreSQL, at the first statement), so every read in the transaction sees the same data.

**Q3. Why are long-running transactions bad in an MVCC database?** ⭐⭐
They pin an old snapshot, so cleanup can't remove any row versions newer than that snapshot, across the whole database. This causes table and index bloat (PostgreSQL), a growing undo history (InnoDB "history list length"), slower queries, and in PostgreSQL it can eventually threaten transaction ID wraparound. They also hold locks and connections.

**Q4. Does MVCC remove the need for locks?**
No. MVCC removes read locks for normal `SELECT`s. Writes to the same row still need row locks, and you still use explicit locks (`SELECT ... FOR UPDATE`) to prevent lost updates or write skew at lower isolation levels.

---

## 14. Recovery and Durability

**Priority:** ⭐⭐⭐ senior / deep dive (WAL itself is ⭐⭐)

**What it is:** How a database guarantees committed data survives crashes, and how it returns to a correct state after a failure.

**Why it exists:** Writing data pages directly to disk on every commit would be slow (random writes, many pages per transaction) and unsafe (a crash halfway through writing a page corrupts it). Databases need a way to make commits durable *quickly* and recover *correctly*.

**How it works**

### 14.1 Failure types

| Failure | Example | Handled by |
|---|---|---|
| Transaction failure | Constraint violation, deadlock victim | Rollback using undo information |
| System crash | Power cut, OS crash, OOM kill; RAM lost, disk OK | WAL replay on restart |
| Media failure | Disk dies | Backups + archived logs, or replicas |

### 14.2 Write-Ahead Logging (WAL) ⭐⭐

**The rule:** before a modified data page is written to disk, the **log record** describing the change must already be on disk. And a transaction is committed only when its **commit record** is flushed to the log.

```
UPDATE accounts SET balance = 500 WHERE id = 1;

1. Change the page in the buffer pool (RAM)          → page is now "dirty"
2. Append a log record to the WAL buffer              → "T7: page 42, balance 1000 → 500"
3. COMMIT → flush WAL to disk (fsync)                 → sequential write, fast
4. Tell the client "success"
5. Later, a background writer flushes dirty page 42  → can be batched, ordered
```

**Why it's fast:** the log is **append-only and sequential**; one small fsync per commit (and databases group several commits into one fsync, "group commit"). The random page writes happen later in the background.

**Redo vs undo:**
- **Redo:** reapply committed changes whose pages hadn't reached disk before the crash. ("Make sure what was committed is there.")
- **Undo:** reverse changes from uncommitted transactions whose pages *had* reached disk. ("Make sure what wasn't committed is gone.")

### 14.3 Checkpoints

If recovery had to replay the log from the beginning of time, restart would take forever. A **checkpoint** periodically flushes dirty pages and records "everything before log position X is safely on disk." Recovery starts from the last checkpoint.

Trade-off: frequent checkpoints = faster recovery but more I/O during normal operation.

### 14.4 ARIES (overview) ⭐⭐⭐

The classic recovery algorithm (used, in variations, by most databases). Three passes after a crash:
1. **Analysis:** scan from the last checkpoint to find which transactions were active and which pages were dirty.
2. **Redo:** replay *all* logged changes from the oldest needed point ("repeat history"), restoring the exact state at the crash, including uncommitted changes.
3. **Undo:** roll back the transactions that hadn't committed, writing *compensation log records* (CLRs) so undo isn't repeated if the system crashes again during recovery.

Key ideas: **steal** (dirty pages of uncommitted transactions may be written to disk, so undo is needed) and **no-force** (committed pages needn't be written at commit time, so redo is needed). This combination gives the best runtime performance.

**Shadow paging** (an alternative): never modify pages in place; write changes to new copies and atomically switch a pointer at commit. Simple recovery, but fragments data and makes concurrency hard. Used in some embedded databases and as an idea in copy-on-write file systems.

### 14.5 The same log, many uses

The WAL isn't only for crash recovery:
- **Replication:** stream the log to replicas (§15).
- **Point-in-time recovery:** restore a backup and replay archived logs up to a moment.
- **Change Data Capture:** read the log to publish changes to Kafka (§19).

| Database | Log name |
|---|---|
| PostgreSQL | WAL |
| MySQL InnoDB | Redo log (crash recovery) + binlog (replication, PITR) |
| MongoDB | Journal (crash recovery) + oplog (replication) |
| Redis | AOF (append-only file) |

### 14.6 Backups and point-in-time recovery

| Type | What it is | Restore speed | Size |
|---|---|---|---|
| **Full** | Complete copy | Fastest restore | Largest |
| **Incremental** | Changes since the *last backup of any kind* | Slowest (need the full + every incremental) | Smallest |
| **Differential** | Changes since the *last full* backup | Medium (full + latest differential) | Grows over time |
| **Logical** | SQL/JSON dump (`pg_dump`, `mysqldump`, `mongodump`) | Slow for large data | Portable across versions |
| **Physical** | Copy of data files (`pg_basebackup`, Percona XtraBackup, snapshots) | Fast | Tied to the version |

**Point-in-time recovery (PITR):** restore the latest full backup, then replay archived WAL/binlogs up to a specific moment.

Real life: at 14:32 someone runs `DELETE FROM orders` without a `WHERE`. With PITR you restore to 14:31:59. Without it, you lose everything since last night's backup.

**Two numbers to know:**
- **RPO (Recovery Point Objective):** how much data you can afford to lose (e.g. 5 minutes).
- **RTO (Recovery Time Objective):** how long you can afford to be down (e.g. 1 hour).

**A backup you haven't restored is not a backup.** Test restores regularly. (GitLab's 2017 incident: several backup methods were found not to be working only when they were needed.)

**Replicas are not backups:** a `DELETE` on the primary is replicated to every replica within milliseconds.

### Interview Questions

**Q1. What is a write-ahead log and why is it needed?** ⭐⭐
An append-only log where every change is recorded *before* the data pages are modified on disk. On commit, only the log needs to be flushed (a fast sequential write). After a crash, the database replays the log to redo committed changes and undo uncommitted ones. It gives durability and atomicity without writing every data page synchronously, and the same log feeds replication and point-in-time recovery.

**Q2. How does a database recover after a crash?** ⭐⭐
Starting from the last checkpoint, it analyzes the log to find active transactions and dirty pages, redoes all logged changes to rebuild the state at the crash, then undoes changes from transactions that never committed. That's the ARIES approach.

**Q3. What's the difference between redo and undo logs?**
Redo logs record the new values so committed changes can be reapplied if their pages didn't reach disk. Undo logs record old values so uncommitted changes can be reversed; in InnoDB, undo logs also serve MVCC by rebuilding older row versions for snapshots.

**Q4. How would you design a backup strategy for a production database?** ⭐⭐
Start from RPO and RTO. Typically: automated daily full physical backups plus continuous WAL/binlog archiving for point-in-time recovery, stored in a different region/account (protects against ransomware and account compromise), encrypted, with retention rules. Run regular automated restore tests. Keep replicas for availability, but don't count them as backups since they replicate mistakes instantly.

**Q5. Someone ran `DELETE` without `WHERE` in production. What do you do?**
Stop further damage first (stop the job, put the app in maintenance mode if needed). Then restore: PITR to a new instance just before the delete, export the deleted rows and re-insert them into production (or switch over, depending on how much has changed since). Afterwards: postmortem, restrict direct production write access, require reviewed scripts, use `BEGIN` + check row count before `COMMIT`, and consider delayed replicas.

---

# PART 4: SCALING AND DISTRIBUTED SYSTEMS

---

## 15. Replication

**Priority:** ⭐⭐ commonly asked

**What it is:** Keeping copies of the same data on multiple servers.

**Why it exists:**
- **High availability:** if the primary dies, a replica takes over.
- **Read scaling:** spread read queries across replicas.
- **Geographic latency:** put a replica near users in another region.
- **Isolating workloads:** run heavy reports on a replica instead of the primary.

**How it works**

### 15.1 Topologies

**Primary-replica (leader-follower)**: the most common.
```
             writes
App ───────────────────► Primary ──(log stream)──► Replica 1  ◄── reads
                                 └─(log stream)──► Replica 2  ◄── reads
```
All writes go to one primary; it streams its change log (WAL / binlog / oplog) to replicas, which replay it.

**Multi-primary (multi-leader):** several nodes accept writes and replicate to each other. Useful for multiple data centers or offline-capable apps, but **write conflicts** become possible (two users edit the same row in different regions at the same time) and must be resolved (§17). Avoid unless you really need it.

**Leaderless:** clients write to several nodes directly and read from several (Cassandra, DynamoDB). Uses quorums (§17).

### 15.2 Synchronous vs asynchronous

| | Synchronous | Asynchronous | Semi-synchronous |
|---|---|---|---|
| Commit returns when | Replica(s) confirmed the write | Primary wrote it locally | At least one replica *received* it |
| Write latency | Higher (network round trip) | Lowest | Medium |
| Data loss if primary dies | None | Recent writes may be lost | Very little |
| Risk | A slow replica slows all writes | Replica lag | Falls back to async on timeout (MySQL) |

Common setup: one synchronous (or semi-sync) replica in the same region for durability, plus asynchronous replicas elsewhere.

### 15.3 Replication lag and its effects ⭐

With asynchronous replication, replicas are slightly behind, usually milliseconds, but seconds or minutes under heavy load or long-running queries.

**Problems it causes:**
- **Read-your-writes violation:** Asha updates her address (goes to primary), the page reloads and reads from a replica, and she sees her *old* address. She thinks the save failed.
- **Monotonic reads violation:** two reads hit different replicas, and data appears to go back in time.
- **Inventory/booking checks** against a lagging replica can make wrong decisions.

**Solutions:**
1. Read from the primary for data the user just modified (e.g. for 10 seconds after a write, or for "my profile" pages).
2. Track the write's log position and only read from replicas that have caught up to it.
3. Pin a user's session to one replica (helps monotonic reads).
4. Always read from the primary for critical checks (balances, stock).

### 15.4 Failover and split-brain

**Failover:** detect the primary is down → choose the most up-to-date replica → promote it → point applications to it. Tools: Patroni (PostgreSQL), Orchestrator / InnoDB Cluster (MySQL), built-in elections (MongoDB replica sets, Redis Sentinel), or managed services (AWS RDS Multi-AZ).

**Risks:**
- With async replication, writes not yet copied to the new primary are **lost**.
- **Split-brain:** the old primary wasn't actually dead (just a network partition), and now *two* nodes accept writes, so data diverges. Prevention: require a **majority quorum** to elect a primary, and **fence** the old primary (STONITH: "shoot the other node in the head", or revoke its access) so it can't accept writes.

### 15.5 Hot, warm and cold standby

How ready the backup system is to take over when the primary fails. The trade-off is cost vs recovery time (RTO) and data loss (RPO).

| | Hot standby | Warm standby | Cold standby |
|---|---|---|---|
| State | Running, continuously replicating, often serving reads | Running or provisioned, receiving data periodically (log shipping), not serving traffic | Not running; just backups and a plan (or infrastructure-as-code) to build a server |
| Failover time (RTO) | Seconds (automatic) | Minutes (promote, maybe catch up on logs) | Hours (provision, restore, replay) |
| Data loss (RPO) | None to seconds | Minutes (since the last shipped log) | Since the last backup / archived log |
| Cost | Highest (full second server, always on) | Medium | Lowest |
| Example | PostgreSQL streaming replica, RDS Multi-AZ, MongoDB secondary | Replica in another region fed by WAL archive shipping, a scaled-down DR instance | Nightly backups in S3 plus a runbook |

Note the naming in PostgreSQL: **hot standby** specifically means a standby that accepts read-only queries while it replays WAL; a standby that replays but doesn't serve queries is a warm standby.

Real life: ShopKart's checkout database has a hot standby in the same region (payments can't be down for long), a warm standby in a second region for regional disasters, and the analytics database is only backed up nightly (cold), because a few hours of downtime there is acceptable.

### Interview Questions

**Q1. Why replicate a database?** ⭐
High availability (failover if the primary dies), read scaling (spread reads across replicas), lower latency for distant users, and isolating heavy analytics from production traffic. Replication does **not** help with write scaling, since every replica must apply every write; for that you need sharding.

**Q2. Synchronous or asynchronous replication?** ⭐⭐
Synchronous guarantees no data loss on failover but adds latency to every commit, and a slow or dead replica can stall writes. Asynchronous is fast but replicas lag and a failover can lose recent commits. Many production setups use one synchronous/semi-sync replica nearby for durability and asynchronous ones for reads and other regions.

**Q3. A user updates their profile but sees the old data after refresh. Why, and how do you fix it?** ⭐
Replication lag: the write went to the primary and the read went to a replica that hadn't applied it yet. Fixes: read-your-writes routing (read from the primary for a short time after the user writes, or for the user's own data), wait for the replica to reach the write's log position, or use session stickiness.

**Q4. What is split-brain and how do you prevent it?** ⭐⭐
Two nodes both believe they're the primary and accept writes, so data diverges. It happens when a network partition makes a healthy primary look dead. Prevent it with majority-based elections (a node can only become primary with votes from a majority, so only one side of a partition can win) and fencing the old primary.

**Q5. What's the difference between a hot, warm and cold standby?** ⭐⭐
They differ in how ready they are to take over. A hot standby is running and continuously replicating, so failover takes seconds with little or no data loss, at the cost of a second full server. A warm standby is provisioned and receives data periodically (log shipping) but doesn't serve traffic, so failover takes minutes and loses minutes of data. A cold standby is just backups plus a runbook, so recovery takes hours and you lose everything since the last backup, but it's cheapest. You choose per system based on its RPO and RTO.

---

## 16. Partitioning and Sharding

**Priority:** ⭐⭐ commonly asked

**What it is:**
- **Partitioning:** splitting one big table into smaller pieces **inside one database server**.
- **Sharding:** splitting data across **multiple database servers**, each holding a subset (a shard). Also called horizontal partitioning across machines.

**Why it exists:**
- **Vertical scaling** (bigger machine: more CPU, RAM, faster disk) is simple but has a ceiling and gets expensive.
- **Read replicas** scale reads, not writes or storage.
- When writes or data size exceed one machine, you must split the data.

**How it works**

### 16.1 Vertical vs horizontal partitioning

- **Vertical partitioning:** split *columns* into separate tables. Example: move the rarely used `products.description_html` (large) into `product_details`, so the hot `products` table stays small.
- **Horizontal partitioning:** split *rows*. Example: `orders_2024`, `orders_2025`, `orders_2026`.

### 16.2 Partitioning within one server

```sql
-- PostgreSQL declarative partitioning
CREATE TABLE orders (
  id BIGINT, user_id BIGINT, created_at TIMESTAMPTZ, total DECIMAL(10,2)
) PARTITION BY RANGE (created_at);

CREATE TABLE orders_2026_10 PARTITION OF orders
  FOR VALUES FROM ('2026-10-01') TO ('2026-11-01');
CREATE TABLE orders_2026_11 PARTITION OF orders
  FOR VALUES FROM ('2026-11-01') TO ('2026-12-01');
```

**Benefits:**
- **Partition pruning:** `WHERE created_at >= '2026-11-01'` only scans the November partition.
- **Cheap data deletion:** dropping last year's partition is instant, compared to `DELETE` of 100 million rows (§20).
- Smaller indexes per partition; maintenance (VACUUM, index rebuilds) per partition.

**Costs:** queries that don't filter on the partition key scan every partition; unique constraints must include the partition key; too many partitions add planning overhead.

### 16.3 Sharding strategies ⭐

| Strategy | How rows are assigned | Pros | Cons | Example |
|---|---|---|---|---|
| **Range** | Key ranges: users A–M → shard 1, N–Z → shard 2 | Range queries stay on one shard | **Hot spots** (all new orders go to the "latest" shard) | Time-series, alphabetical |
| **Hash** | `hash(key) % N` | Even distribution | Range queries hit all shards; changing N moves almost everything | User data by `user_id` |
| **Consistent hashing** | Keys and shards placed on a ring | Adding a shard moves only ~1/N of keys | More complex | Cassandra, DynamoDB, caches |
| **Directory / lookup** | A lookup table maps key → shard | Full flexibility, easy to move tenants | Lookup service is a bottleneck / single point of failure | Multi-tenant SaaS |
| **Geo** | By region | Data residency (GDPR), low latency | Uneven sizes | EU users in EU shard |

### 16.4 Consistent hashing ⭐⭐

```
                 0
          ┌──────────────┐
     Node C            Node A          keys are hashed onto the same ring;
       │                  │            each key belongs to the next node clockwise
     key3               key1
       │                  │
          └──── Node B ───┘
               key2
```

With plain `hash(key) % N`, going from 4 to 5 servers changes the result for about 80% of keys, so nearly all data must move. With consistent hashing, adding a node only takes over keys from its neighbour: about 1/N of the data moves.

**Virtual nodes:** each physical server appears at many points on the ring, giving more even distribution and spreading the load of a failed node across many servers.

### 16.5 Choosing a shard key ⭐

The most important decision; it's very hard to change later.
- **High cardinality:** many distinct values (so data can spread). `country` is a bad key if 70% of users are in India.
- **Even distribution of load**, not just of data.
- **Matches the most common queries**, so they hit one shard. For ShopKart, `user_id` keeps a user's orders, cart and addresses together: "my orders" is a single-shard query.
- **Not monotonically increasing** for range sharding (timestamps, auto-increment IDs send every new write to the last shard). Hash them instead.

### 16.6 Problems sharding introduces

- **Cross-shard queries:** "top 10 products by sales across all users" must ask every shard and merge (scatter-gather).
- **Cross-shard joins** aren't possible in the database; do them in the application or denormalize.
- **Cross-shard transactions** need 2PC or sagas (§17).
- **Global unique IDs:** auto-increment per shard collides. Use UUIDs, Snowflake-style IDs (timestamp + machine ID + sequence), or ID ranges per shard.
- **Hot spots / celebrity problem:** one key gets huge traffic (a celebrity's posts). Split hot keys further, e.g. append a random suffix and aggregate on read.
- **Resharding:** moving data when adding shards, while serving traffic.
- **Operational complexity:** backups, migrations and monitoring across N databases.

**Rule:** shard last. Before sharding, try: query/index optimization, caching, read replicas, vertical scaling, partitioning, archiving old data, and splitting by service (orders DB vs users DB). Tools that reduce the pain: Vitess (MySQL; YouTube, Slack), Citus (PostgreSQL), or databases with built-in sharding (MongoDB, Cassandra, CockroachDB, YugabyteDB, TiDB).

### Interview Questions

**Q1. What's the difference between partitioning and sharding?** ⭐
Partitioning splits a table into pieces within one database server: it helps query pruning and maintenance but not the server's capacity. Sharding spreads data across multiple servers so you scale writes and storage beyond one machine, at the cost of cross-shard queries, transactions and operational complexity.

**Q2. How do you choose a shard key?** ⭐
High cardinality, even distribution of data *and* traffic, aligned with the main access pattern so most queries hit a single shard, and not monotonically increasing (or hashed if it is). For an e-commerce app, `user_id` (or `tenant_id` in SaaS) is a common choice because a user's orders, cart and profile stay together.

**Q3. What is consistent hashing and why is it used?** ⭐⭐
Both keys and nodes are hashed onto a ring; each key belongs to the next node clockwise. When a node is added or removed, only the keys between it and its neighbour move, about 1/N of the data, instead of nearly all keys as with `hash % N`. Virtual nodes give smoother balancing. It's used in Cassandra, DynamoDB, Redis Cluster-like systems and distributed caches.

**Q4. When would you shard? What would you try first?** ⭐⭐
Only when a single primary can't handle the write load or data size even after optimization. First: fix queries and indexes, add caching, add read replicas, scale vertically, partition and archive old data, and split databases by service. Sharding permanently complicates every query, migration and transaction, so delay it until it's clearly needed.

**Q5. How do you generate unique IDs across shards?** ⭐⭐
Options: UUIDs (v7 for ordering), Twitter Snowflake-style 64-bit IDs (timestamp + worker ID + per-worker sequence; time-ordered and compact), a central ticket server handing out ID blocks, or per-shard ranges/offsets. Snowflake-style or UUID v7 is the common modern answer.

---

## 17. Distributed Database Theory

**Priority:** ⭐⭐ commonly asked (CAP is ⭐)

**What it is:** The rules and trade-offs that apply when data lives on more than one machine connected by an unreliable network.

**Why it matters:** Once you replicate or shard, networks can drop or delay messages, and nodes can fail independently. You can't have perfect consistency, availability and low latency all at once.

**How it works**

### 17.1 CAP theorem ⭐

In a distributed system, during a **network Partition**, you must choose between:
- **Consistency (C):** every read returns the most recent write (linearizability), or an error.
- **Availability (A):** every request to a non-failed node gets a (non-error) response, possibly stale.

```
     Region 1              ✂ network cut ✂            Region 2
  ┌─────────────┐                               ┌─────────────┐
  │ Node A      │                               │ Node B      │
  │ stock = 1   │                               │ stock = 1   │
  └─────────────┘                               └─────────────┘
 User buys the last item on A.  Another user tries to buy it on B.

 CP choice: B refuses (error/timeout) because it can't confirm with A → correct but unavailable
 AP choice: B accepts → available, but the item is now sold twice
```

**Common mistakes:**
- "Pick 2 of 3" is misleading. Partitions *will* happen, so P isn't optional. The real choice is **C or A during a partition**. When there's no partition, you can have both.
- CAP's "consistency" is not ACID's "consistency" (§10).

| System | Behaviour during partition |
|---|---|
| Relational database with synchronous replication, MongoDB (majority writes), HBase, etcd/ZooKeeper | CP: some requests fail rather than return stale data |
| Cassandra, DynamoDB (default), Riak, CouchDB | AP: always answer, reconcile later (tunable) |

Real life: a bank balance or seat booking → prefer CP. A social media like counter or shopping cart → AP is fine (Amazon's Dynamo paper: a cart that occasionally shows a removed item is better than a cart that won't load).

### 17.2 PACELC ⭐⭐⭐

Extends CAP to normal operation: **if Partition**, choose **A or C**; **Else**, choose **Latency or Consistency**. Even without failures, waiting for replicas to confirm (consistency) adds latency.
- DynamoDB, Cassandra: PA/EL (available, low latency)
- Traditional relational with sync replication, Spanner: PC/EC (consistent, pay latency)
- MongoDB: PC/EC by default with majority settings, tunable

### 17.3 Consistency models

From strongest to weakest:

| Model | Guarantee | Example |
|---|---|---|
| **Linearizable (strong)** | Behaves like one copy; once a write completes, every later read sees it | Bank balance, distributed lock |
| **Sequential** | All nodes see operations in the same order (not necessarily real-time order) | |
| **Causal** | If A caused B (reply to a comment), everyone sees A before B | Comment threads, chat |
| **Read-your-writes** | You always see your own writes | Profile updates |
| **Monotonic reads** | You never see data go backwards in time | Refreshing a feed |
| **Eventual** | If writes stop, all replicas eventually agree | Like counts, view counts, DNS |

### 17.4 Quorums

With **N** replicas, write to **W** of them and read from **R** of them. If **R + W > N**, every read set overlaps every write set in at least one node, so reads see the latest write.

```
N = 3, W = 2, R = 2  →  2 + 2 > 3  ✅ strongly consistent reads (in the normal case)
N = 3, W = 1, R = 1  →  fast, but reads may be stale
N = 3, W = 3, R = 1  →  fast reads, but writes fail if any node is down
```
Cassandra lets you choose per query (`ONE`, `QUORUM`, `ALL`).

### 17.5 Distributed transactions ⭐⭐

**Two-Phase Commit (2PC)**:
```
Coordinator                      Participants (Orders DB, Payments DB)
   │── 1. PREPARE ──────────────────► each locks resources, writes "prepared" to its log
   │◄──────────────── YES / NO ──────│
   │── 2. COMMIT (if all YES) ───────► commit
   │   or ABORT (if any NO)          │
```
- ✅ Atomic across databases.
- ❌ **Blocking:** if the coordinator crashes after PREPARE, participants hold locks and can't decide alone.
- ❌ Slow (multiple round trips, locks held throughout); poor fit for microservices.

**3PC** adds a pre-commit phase to reduce blocking but doesn't handle network partitions; rarely used in practice.

**Saga pattern:** a sequence of **local** transactions, each with a **compensating action** to undo it.
```
ShopKart checkout saga:
1. Order service:     create order (PENDING)        compensate: cancel order
2. Inventory service: reserve stock                 compensate: release stock
3. Payment service:   charge card                   compensate: refund
4. Order service:     mark CONFIRMED

If step 3 fails → run "release stock", then "cancel order"
```
- **Choreography:** each service listens for events and reacts (no central controller). Simple for few steps, hard to follow for many.
- **Orchestration:** a central orchestrator tells each service what to do (Temporal, AWS Step Functions). Easier to reason about and monitor.

Sagas give **eventual consistency**, not isolation: other users might briefly see the PENDING order or reserved stock. Compensations must be **idempotent**, and some actions can't be truly undone (an email already sent), so you design semantic compensations (send a "sorry, cancelled" email).

### 17.6 Consensus ⭐⭐⭐

How a group of nodes agrees on a value (who's the leader, what's the next log entry) even when some fail.
- **Raft:** an elected leader receives writes, appends them to its log, replicates them to followers, and commits an entry once a **majority** has it. If the leader dies, followers time out and elect a new one; a candidate needs a majority of votes. Used in etcd, Consul, CockroachDB, TiDB, MongoDB's replica set protocol (Raft-like).
- **Paxos:** older, equivalent guarantees, notoriously hard to understand and implement. Used in Google Chubby and Spanner.

**Why majority?** Any two majorities overlap, so two leaders can't both be elected and committed data can't be lost. A cluster of 2f + 1 nodes tolerates f failures: 3 nodes tolerate 1, 5 tolerate 2. That's why clusters use odd sizes.

### 17.7 Conflict resolution ⭐⭐⭐

For multi-leader or leaderless systems where two writes to the same item can happen concurrently:
- **Last-write-wins (LWW):** keep the write with the latest timestamp. Simple, but silently **loses data**, and clock skew between servers can pick the wrong winner.
- **Vector clocks:** each replica keeps a counter per node; comparing vectors tells you whether one write happened before another or they were concurrent. Concurrent writes are kept as "siblings" for the application to merge (Amazon Dynamo, Riak).
- **CRDTs** (Conflict-free Replicated Data Types): data structures designed to merge automatically and correctly, e.g. a counter where each node increments its own slot and the total is the sum. Used in collaborative editors and Redis Enterprise active-active.
- **Application-level merge:** e.g. shopping cart = union of items.

### Interview Questions

**Q1. Explain the CAP theorem.** ⭐
In a distributed system, when a network partition happens, you must choose between consistency (every read sees the latest write, or errors) and availability (every request gets a response, possibly stale). Partitions are unavoidable, so the real choice is CP or AP *during* a partition. Banking and inventory usually choose CP; social feeds, carts and counters can choose AP.

**Q2. Is MongoDB CP or AP? Cassandra?** ⭐⭐
MongoDB with a replica set is generally CP: only the primary accepts writes, and during a partition the side without a majority has no primary. (Reading from secondaries relaxes this.) Cassandra is AP by default: any replica accepts writes and reconciles later, though you can tune it toward consistency with `QUORUM` reads and writes. It's more accurate to say these systems are *tunable*.

**Q3. What is eventual consistency? Give a real example.** ⭐
If no new writes happen, all replicas will eventually return the same value, but in the meantime reads may be stale. Example: you like a post and your friend sees the old like count for a second. Or DNS: a change takes minutes to reach everyone. It's acceptable when stale reads cause no real harm.

**Q4. How do you maintain consistency across microservices without a distributed transaction?** ⭐⭐
Use the Saga pattern: a series of local transactions, each publishing an event that triggers the next step, with compensating actions if a step fails. Make steps idempotent, use the outbox pattern so the database change and the event publication can't diverge (§19), and design the UI around intermediate states like "Order pending".

**Q5. What's 2PC and why is it avoided in microservices?** ⭐⭐
Two-phase commit: a coordinator asks every participant to prepare (lock and promise to commit), then tells all of them to commit or abort. It's atomic, but blocking: if the coordinator fails after prepare, participants hold locks indefinitely. It also adds latency and couples services tightly, and many systems (message brokers, third-party APIs) don't support it. Sagas trade atomicity for availability and loose coupling.

**Q6. Why do consensus clusters use an odd number of nodes?** ⭐⭐
Decisions need a majority. A 3-node cluster tolerates 1 failure; a 4-node cluster still needs 3 votes and so also tolerates only 1. The extra even node adds cost and no fault tolerance. 5 nodes tolerate 2 failures.

---

## 18. SQL vs NoSQL

**Priority:** ⭐ must-know

**What it is:** Choosing between relational databases and the various NoSQL families.

**Why it matters:** "Why did you choose MongoDB/PostgreSQL for this?" is asked in almost every system design and backend interview. A good answer is about **data shape, access patterns, consistency needs and scale**, not about trends.

**How it works**

### 18.1 ACID vs BASE

| ACID (most relational databases) | BASE (many NoSQL systems) |
|---|---|
| **A**tomic, **C**onsistent, **I**solated, **D**urable | **B**asically **A**vailable, **S**oft state, **E**ventually consistent |
| Correctness first | Availability and scale first |
| Bank transfers, orders | Feeds, analytics events, caches |

The line is blurry now: MongoDB has multi-document ACID transactions, PostgreSQL has JSONB and can scale out with Citus, and "NewSQL" databases (CockroachDB, Spanner, TiDB, YugabyteDB) offer SQL + ACID + horizontal scaling.

### 18.2 Schema-on-write vs schema-on-read

- **Schema-on-write (relational):** the database rejects data that doesn't match the schema. Bad data never gets in; changing the schema requires migrations.
- **Schema-on-read (document stores):** the database accepts any shape; the *application* interprets it when reading. Flexible, but old documents with old shapes live forever, and every reader must handle them. MongoDB does support optional schema validation.

### 18.3 Decision framework

| Choose relational (PostgreSQL/MySQL) when | Choose a NoSQL store when |
|---|---|
| Data is relational with many joins (users, orders, products, payments) | Data is naturally a self-contained document read as a whole (product catalog with varying attributes, CMS content) → **document** |
| You need multi-row ACID transactions | You need key-based lookups at very high speed (sessions, carts, rate limits) → **key-value** |
| Ad-hoc queries and reporting matter | You have huge write volume with simple, known queries (chat messages, IoT, logs) → **wide-column** |
| Data integrity is critical (money, inventory) | Relationships *are* the data (social graph, fraud rings) → **graph** |
| The schema is fairly stable | Full-text relevance search → **search engine** |

**Polyglot persistence** (common real-world answer): ShopKart uses PostgreSQL for orders and payments (source of truth), Redis for sessions, carts and caching, Elasticsearch for product search, and a warehouse for analytics, each fed from the main database.

**Common misconceptions:**
- "NoSQL is faster." Only for the access pattern it's designed for. A key lookup in PostgreSQL by primary key is also sub-millisecond.
- "NoSQL has no schema." There's always a schema; it's just enforced by the application instead of the database.
- "SQL doesn't scale." Relational databases scale vertically and with replicas a long way; Shopify, GitHub and Instagram run on sharded MySQL/PostgreSQL.

### Interview Questions

**Q1. SQL or NoSQL: how do you decide?** ⭐
Look at data shape, access patterns, consistency requirements and scale. Relational for structured, related data needing joins, transactions and integrity (orders, payments). Document stores for self-contained, variable-shaped records read as a unit. Key-value for simple, very fast lookups. Wide-column for massive write throughput with known query patterns. Graph for relationship-heavy queries. The default for a new service is often PostgreSQL, adding others only for specific needs.

**Q2. What does BASE mean?** ⭐
Basically Available (the system keeps responding even during failures), Soft state (state may change over time without new input, as replicas sync), Eventually consistent (replicas converge if writes stop). It's the trade-off many distributed NoSQL databases make to favor availability and partition tolerance over immediate consistency.

**Q3. Can a relational database store flexible JSON data?** ⭐⭐
Yes. PostgreSQL's JSONB (with GIN indexes) and MySQL's JSON type let you store semi-structured attributes alongside normal columns. A common pattern: core fields as columns (with constraints and indexes), variable attributes (product specs) in a JSONB column. This often removes the need for a separate document database.

---

# PART 5: PRODUCTION CONCERNS

---

## 19. Data Pipelines and Data Integrity

**Priority:** ⭐⭐⭐ senior / deep dive (idempotency is ⭐⭐)

**What it is:** Moving data reliably between the database and other systems (search index, cache, analytics, other microservices), and keeping it correct along the way.

**Why it matters:** In real backends, the database is rarely alone. When an order is placed, ShopKart must also update Elasticsearch, invalidate the cache, send an email and notify the warehouse service. If any of these drifts out of sync, users see wrong data.

**How it works**

### 19.1 The dual-write problem

```javascript
await db.orders.insert(order);          // 1. succeeds
await kafka.publish('order.created');   // 2. 💥 fails / app crashes
// Database has the order, but no one else knows about it
```
Reversing the order doesn't help: then the event can exist for an order that was never saved. You can't atomically write to two different systems without a distributed transaction.

### 19.2 Transactional outbox pattern ⭐⭐

Write the event into an **outbox table in the same database transaction** as the business data. A separate process publishes it.

```sql
BEGIN;
INSERT INTO orders (id, user_id, total, status) VALUES (501, 42, 1999.00, 'PLACED');
INSERT INTO outbox (id, aggregate_id, event_type, payload, created_at)
VALUES (gen_random_uuid(), 501, 'order.created', '{"orderId":501,...}', now());
COMMIT;   -- both rows or neither
```
```
outbox table ──► relay (polling job or CDC) ──► Kafka / RabbitMQ ──► consumers
                 marks rows as published
```
The relay may publish the same event twice (e.g. it crashes after publishing but before marking it done), so this gives **at-least-once** delivery, and consumers must be idempotent.

### 19.3 Change Data Capture (CDC)

Instead of the application publishing events, read the database's own **replication log** and turn every committed change into an event.
- **MySQL:** binlog (row format)
- **PostgreSQL:** logical decoding of the WAL
- **MongoDB:** change streams (from the oplog)
- Tool: **Debezium** → Kafka

Uses: keeping search indexes and caches in sync, feeding the data warehouse, relaying outbox rows, migrating between databases with no downtime.

### 19.4 Delivery guarantees and idempotency ⭐⭐

| Guarantee | Meaning | Risk |
|---|---|---|
| **At-most-once** | Sent once, never retried | Messages can be lost |
| **At-least-once** | Retried until acknowledged | Duplicates |
| **Exactly-once** | Each message processed once | Very hard end-to-end; usually built as *at-least-once + idempotent processing* |

**Idempotent** means processing the same message twice has the same effect as processing it once.

```sql
-- Idempotent payment: the client sends an idempotency key with each request
CREATE TABLE payments (
  id BIGSERIAL PRIMARY KEY,
  idempotency_key UUID NOT NULL UNIQUE,
  order_id BIGINT NOT NULL,
  amount DECIMAL(10,2) NOT NULL,
  status VARCHAR(20) NOT NULL
);

INSERT INTO payments (idempotency_key, order_id, amount, status)
VALUES ('6f1c...', 501, 1999.00, 'INITIATED')
ON CONFLICT (idempotency_key) DO NOTHING;   -- retry → no second payment
```

Other techniques: a `processed_events(event_id)` table with a unique constraint, checked in the same transaction as the side effect; naturally idempotent operations (`SET status = 'SHIPPED'` rather than `INCREMENT count`).

Real life: Stripe's API accepts an `Idempotency-Key` header for exactly this reason. A user double-clicks "Pay", or the mobile network retries the request, and the customer is charged once.

### 19.5 Poison messages ⭐⭐⭐

A **poison message** (poison pill) is a message that fails every time it's processed, so it's retried forever, blocks the queue partition behind it, or crashes consumers in a loop.

**Causes:**
- Malformed data (invalid JSON, a missing required field)
- **Schema drift:** the producer added/renamed a field and the consumer's deserializer breaks
- Data that violates a constraint downstream (an order referencing a product that doesn't exist yet)
- A bug triggered by a specific edge-case value (an emoji in a name, a NULL, a huge payload)

**Real life:** a CDC pipeline streams order changes into the search index. One product has a 2MB description that exceeds the search engine's field limit. The consumer fails, retries, fails... and because Kafka processes a partition in order, every product update behind it is stuck. Search results go stale for hours.

**Handling:**
1. **Retry with exponential backoff and a maximum attempt count**: transient failures (network blips, a locked row) usually succeed on retry.
2. After N failures, move the message to a **dead letter queue (DLQ)** so the main flow continues.
3. **Alert** on DLQ growth; inspect, fix the bug or the data, then **replay** the messages.
4. Distinguish **retryable** errors (timeouts) from **non-retryable** ones (validation errors); send non-retryable ones to the DLQ immediately.
5. **Prevent:** validate at the producer, use a **schema registry** with compatibility rules (Avro/Protobuf/JSON Schema), and write tolerant consumers that ignore unknown fields.
6. In databases, the equivalent is a **quarantine table**: rows that fail ETL validation are stored with the error message instead of failing the whole batch.

Engine examples: RabbitMQ and SQS have built-in DLQs (SQS `maxReceiveCount` + redrive policy); Kafka consumers implement retry topics + DLQ topics; Redis Streams track delivery counts in the pending entries list (file 04); SQL Server Service Broker disables a queue after 5 consecutive rollbacks.

### 19.6 Data quality

| Problem | Example | Prevention |
|---|---|---|
| Dirty data | `"  ASHA@Gmail.com "` vs `asha@gmail.com` | Normalize on input; `CHECK` constraints; case-insensitive unique index |
| Orphan records | `order_items` pointing to deleted orders | Foreign keys; soft deletes |
| Duplicate ingestion | The same event imported twice | Idempotency keys, unique constraints, upserts |
| Schema drift | A source system silently changes a column type | Schema registry; contract tests; validation at ingestion |
| Late/out-of-order events | "Shipped" arrives before "Paid" | Event timestamps/versions; ignore older versions |

### Interview Questions

**Q1. What is the dual-write problem and how do you solve it?** ⭐⭐
Writing to a database and a message broker (or cache, or search index) separately: if one succeeds and the other fails, the systems disagree, and there's no atomic way to write both. Solve with the transactional outbox (write the event to an outbox table in the same transaction, publish it asynchronously) or CDC (derive events from the database's change log).

**Q2. What does idempotency mean and how do you implement it?** ⭐⭐
An operation is idempotent if doing it several times has the same effect as doing it once. Implement it with a client-supplied idempotency key stored under a unique constraint (a second request returns the first result), a processed-events table checked in the same transaction, or by designing operations as "set to value" instead of "increment". It's essential because networks, queues and users all retry.

**Q3. What is a poison message and how do you handle it?** ⭐⭐⭐
A message that always fails processing, so retries never succeed and it can block the queue or crash consumers repeatedly. Handle it with limited retries and exponential backoff, classify errors as retryable or not, move failing messages to a dead letter queue, alert on it, fix and replay. Prevent it with producer-side validation, schema registries with compatibility checks, and tolerant consumers.

**Q4. What's the difference between at-least-once and exactly-once delivery?** ⭐⭐
At-least-once retries until acknowledged, so duplicates are possible. Exactly-once means each message affects the system once; true end-to-end exactly-once is very hard across independent systems, so in practice it's achieved as at-least-once delivery plus idempotent consumers (or transactional processing within one system, like Kafka transactions).

---

## 20. Data Lifecycle

**Priority:** ⭐⭐ commonly asked

**What it is:** What happens to data over time: deleting it, expiring it, archiving it and complying with regulations.

**Why it matters:** Tables grow forever unless you plan otherwise. Old data slows queries, inflates backups and costs money, and laws (GDPR, India's DPDP Act) require deleting personal data on request.

**How it works**

### 20.1 Soft delete vs hard delete ⭐

```sql
-- Hard delete: the row is gone
DELETE FROM products WHERE id = 100;

-- Soft delete: mark it
ALTER TABLE products ADD COLUMN deleted_at TIMESTAMPTZ;
UPDATE products SET deleted_at = now() WHERE id = 100;
SELECT * FROM products WHERE deleted_at IS NULL;   -- every query must remember this
```

| | Soft delete | Hard delete |
|---|---|---|
| Undo / restore | Easy | Need backups |
| Audit history | Kept | Lost |
| Foreign keys | Old orders still reference the product | Must cascade, set null, or block |
| Query complexity | Every query needs `deleted_at IS NULL`, and forgetting it is a common bug | Simple |
| Unique constraints | A deleted `email` still blocks re-registration | No issue |
| Table size | Keeps growing | Smaller |
| Privacy law | Data still exists, which may violate erasure requests | Compliant |

**Fixes for soft-delete problems:**
```sql
-- Unique email only among non-deleted users (PostgreSQL partial unique index)
CREATE UNIQUE INDEX uniq_active_email ON users (email) WHERE deleted_at IS NULL;

-- Hide deleted rows by default
CREATE VIEW active_products AS SELECT * FROM products WHERE deleted_at IS NULL;
```
ORMs help too: Sequelize `paranoid: true`, TypeORM `@DeleteDateColumn` + `softDelete()`, Django managers that filter deleted rows by default.

Alternative: **archive on delete**: move the row to a `deleted_products` table (or an audit log) and hard delete it from the main table. Main queries stay simple and history is kept.

### 20.2 TTL and automatic expiry

Some data should disappear automatically: sessions, OTPs, password-reset tokens, carts abandoned for 30 days.
- **Redis:** `SET otp:42 123456 EX 300` (expires in 5 minutes)
- **MongoDB:** TTL index, `db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 })`
- **Cassandra / DynamoDB:** per-row TTL
- **MySQL / PostgreSQL:** no built-in row TTL; use a scheduled job (MySQL Event Scheduler, `pg_cron`, an app cron job) or drop partitions.

### 20.3 Deleting big data safely

```sql
-- ❌ One giant delete: long transaction, locks, huge WAL, replica lag, bloat
DELETE FROM events WHERE created_at < '2025-01-01';      -- 300 million rows

-- ✅ Delete in batches with pauses
DELETE FROM events
WHERE id IN (SELECT id FROM events WHERE created_at < '2025-01-01' LIMIT 5000);
-- repeat until 0 rows affected, sleeping briefly between batches

-- ✅✅ Best: partition by time, drop whole partitions (instant, no bloat)
DROP TABLE events_2024_12;   -- or ALTER TABLE ... DETACH PARTITION first
```

### 20.4 Retention and archiving

- Define **retention policies** per data type: logs 30 days, orders 8 years (tax law), chat messages until the user deletes their account.
- **Hot → warm → cold storage:** recent data in the main database, older data in cheaper storage (an archive database, or Parquet files in S3 queryable with Athena/BigQuery).
- Archive before deleting if the business or law needs the data.

**Retention policy vs legal hold:**
- A **retention policy** is the routine rule: "delete chat logs after 1 year", "keep backups for 35 days". It exists to save cost and to satisfy privacy laws that say *don't keep personal data longer than needed*.
- A **legal hold** is an exception ordered for a specific case: when there's a lawsuit, investigation or regulator request, all data that might be relevant must be preserved, **even if the retention policy says delete it**. Deleting it after a hold is issued can be treated as destroying evidence.
- **The conflict:** the retention job (and a user's erasure request under GDPR/DPDP) says delete; the hold says keep. **The hold wins** for the data it covers, and it lasts until legal releases it.
- **How systems handle it:** a `legal_hold` flag (or a holds table keyed by user/account/date range) that every purge job, TTL cleanup, erasure workflow and backup-expiry rule must check before deleting; object storage with hold/lock features (S3 Object Lock legal hold); and an audit log of when holds were placed and released.

Real life: ShopKart deletes customer support chats after 12 months. A customer sues over a faulty refund, so legal places a hold on that customer's chats, orders and payment records. The nightly purge job skips them until the case closes, then the normal retention rule applies again.

### 20.5 Right to be forgotten ⭐⭐⭐

GDPR (EU) and India's DPDP Act give users the right to have personal data erased. Challenges:
- Personal data is copied everywhere: replicas, caches, search indexes, analytics warehouses, logs, **backups**.
- You may legally need to keep some records (invoices for tax).

Common approaches:
- **Anonymize/pseudonymize** instead of deleting rows: replace name and email with placeholders, keep the order amounts for accounting.
- **Crypto-shredding:** encrypt each user's personal data with a per-user key; to "delete" it, delete the key. Copies in backups become unreadable.
- Keep a deletion log so restores from backup can re-apply deletions.

### Interview Questions

**Q1. Soft delete or hard delete?** ⭐
Soft delete when you need undo, audit history, or old records reference the row (old orders → deleted product). Hard delete when data must truly be gone (privacy laws) or the table would grow unboundedly. Soft delete's costs: every query must filter deleted rows, unique constraints need partial indexes, and tables grow. A clean middle ground is moving deleted rows to an archive table.

**Q2. How do you delete 500 million old rows from a production table?** ⭐⭐
Not in one statement: that creates a huge transaction, long locks, massive log volume, replica lag and bloat. Delete in small batches (a few thousand rows by primary key) with pauses, during low traffic, while watching replica lag. The better long-term design is time-based partitioning, so you drop old partitions instantly.

**Q3. How do you implement expiring data like OTPs or sessions?**
In Redis, set a TTL on the key. In MongoDB, a TTL index. In SQL, store an `expires_at` column, always check it on read (`WHERE expires_at > now()`), and clean up with a scheduled job. Never rely only on the cleanup job for correctness, since it may run late.

**Q4. What's the difference between a retention policy and a legal hold, and how do you handle the conflict?** ⭐⭐
A retention policy is the routine rule for how long data is kept before it's deleted (for cost and privacy compliance). A legal hold is a case-specific order to preserve relevant data because of litigation or an investigation, and it overrides the retention policy and even user deletion requests for the data it covers, until it's released. Technically: store holds explicitly (a flag or holds table), make every purge job, TTL cleanup, erasure flow and backup-expiry rule check them, use storage-level locks (e.g. S3 Object Lock) for archived data, and audit when holds are placed and lifted.

---

## 21. Security

**Priority:** ⭐⭐ commonly asked (SQL injection is ⭐)

**What it is:** Protecting data from unauthorized access, modification and leakage.

**How it works**

### 21.1 Authentication, authorization and least privilege

- **Authentication:** who are you? (password, IAM auth, certificates)
- **Authorization:** what can you do? (`GRANT`/`REVOKE`, roles)

```sql
CREATE ROLE app_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_readonly;

CREATE USER reporting_service WITH PASSWORD '...';
GRANT app_readonly TO reporting_service;

-- The application user can read/write data but not drop tables or change schema
GRANT SELECT, INSERT, UPDATE, DELETE ON orders, order_items TO shopkart_app;
```

**Least privilege:** the app's runtime user shouldn't own the schema; migrations run with a separate, more powerful user. The analytics tool gets read-only access to a replica. Developers don't have write access to production by default.

### 21.2 SQL injection ⭐

Building SQL by concatenating user input:
```javascript
// ❌ Vulnerable
const q = `SELECT * FROM users WHERE email = '${req.body.email}'`;
// Attacker sends email:  ' OR '1'='1
// → SELECT * FROM users WHERE email = '' OR '1'='1'      ← returns every user
// Or:  '; DROP TABLE users; --
```

**Fix: parameterized queries (prepared statements).** The SQL text and the data are sent separately, so input is always treated as a value, never as code.
```javascript
// ✅ node-postgres
await pool.query('SELECT * FROM users WHERE email = $1', [req.body.email]);
```
```python
# ✅ psycopg
cur.execute("SELECT * FROM users WHERE email = %s", (email,))
```

Traps:
- Placeholders can't be used for **identifiers** (table/column names, `ORDER BY` column, `ASC/DESC`). Use an **allow-list**: `const col = ['price', 'created_at'].includes(input) ? input : 'created_at'`.
- ORMs are safe by default, but their raw-query escape hatches (`sequelize.query`, `prisma.$queryRawUnsafe`, Django `.raw()` / `extra()`) are vulnerable if you concatenate strings.
- **NoSQL injection** exists too: in MongoDB, passing `{"$ne": null}` as a password from JSON input turns the query into "password is not null". Validate input types (file 03).

Defense in depth: least-privilege database users (so injection can't drop tables), input validation, a WAF.

### 21.3 Encryption

- **In transit:** TLS between the app and database (`sslmode=require`/`verify-full` in PostgreSQL).
- **At rest:** encrypted disks/volumes (cloud providers do this by default), and encrypted backups.
- **Column-level / application-level encryption** for very sensitive fields (Aadhaar/SSN, card data): the database only sees ciphertext. Trade-off: you can't search or index the plaintext easily.
- **Passwords are hashed, never encrypted:** bcrypt, scrypt or Argon2 with a salt. Never MD5 or SHA-1.
- **Card numbers:** don't store them; use a payment provider's tokens (PCI DSS).

### 21.4 Auditing

Record who did what and when: database audit logs (pgAudit, MySQL Enterprise Audit / Percona audit plugin), `created_by`/`updated_by` columns, or audit tables populated by triggers or CDC. Needed for compliance (SOX, PCI, HIPAA) and incident investigation.

### 21.5 Other practices

- Don't expose the database to the internet; put it in a private subnet.
- Store credentials in a secrets manager (AWS Secrets Manager, Vault), not in code; rotate them.
- Mask or anonymize production data before copying it to dev/staging.
- **Row-level security** (PostgreSQL RLS) for multi-tenant apps: the database itself enforces "users only see their tenant's rows".

### 21.6 Data poisoning ⭐⭐⭐

An attacker inserts malicious or misleading data that a downstream system later trusts:
- **ML training data:** fake reviews or labels skew a recommendation or fraud model.
- **Vector databases / RAG:** a malicious document is added to the knowledge base, and when retrieved it injects instructions into an LLM's prompt (indirect prompt injection) or spreads false information.

Defenses: control and validate who can write to training data and knowledge bases, track provenance (where each record came from), detect anomalies, and treat retrieved content as untrusted data, not instructions.

### Interview Questions

**Q1. What is SQL injection and how do you prevent it?** ⭐
An attack where user input is concatenated into a SQL string and changes the query's meaning, e.g. `' OR '1'='1` returning every row, or `; DROP TABLE`. Prevent it with parameterized queries/prepared statements so input is always data, allow-lists for dynamic identifiers like sort columns, ORMs used correctly, and least-privilege database users to limit damage.

**Q2. How should passwords be stored?** ⭐
Hashed with a slow, salted algorithm designed for passwords: Argon2, bcrypt or scrypt. Never plaintext, never reversible encryption, never fast hashes like MD5/SHA-256 alone (they can be brute-forced at billions per second on GPUs).

**Q3. What does least privilege mean for databases?** ⭐⭐
Each user or service gets only the permissions it needs: the app can read/write its tables but not alter the schema or read other services' data; analytics is read-only on a replica; migrations use a separate privileged account; humans don't have routine production write access. It limits the damage from bugs, leaked credentials and injection.

**Q4. How do you isolate tenants' data in a multi-tenant app?**
Options: a `tenant_id` column on every table (cheapest; enforce it with PostgreSQL row-level security or a mandatory filter in the data access layer), a schema per tenant, or a database per tenant (strongest isolation, most expensive). See §22.

---

## 22. Applied Design

**Priority:** ⭐⭐ commonly asked (in system design and backend rounds)

**What it is:** Applying everything above to design schemas for real systems.

**How to approach any "design the schema for X" question:**
1. **Clarify requirements:** main entities, key screens/APIs, read vs write ratio, scale, consistency needs.
2. **List entities and relationships**, then tables, keys and constraints.
3. **Design for the main queries:** which indexes do they need?
4. **Identify concurrency hazards** (double booking, overselling, double payment) and how you'll prevent them.
5. **Discuss scale:** caching, replicas, partitioning, sharding key, archiving.

### 22.1 E-commerce (ShopKart) core

```sql
users(id PK, email UNIQUE, name, created_at)
addresses(id PK, user_id FK, line1, city, pincode, is_default)
products(id PK, sku UNIQUE, name, category_id FK, price, stock CHECK (stock >= 0), version)
orders(id PK, user_id FK, status, total, shipping_address JSONB /* snapshot */, created_at)
order_items(order_id FK, product_id FK, quantity, unit_price, PRIMARY KEY (order_id, product_id))
payments(id PK, order_id FK, idempotency_key UNIQUE, amount, status, provider_ref)

INDEX orders (user_id, created_at DESC)     -- "My orders"
INDEX products (category_id, price)          -- category listing sorted by price
```
Points to mention: snapshot prices and addresses on the order; atomic stock decrement; idempotent payments; order status as a state machine (`PLACED → PAID → SHIPPED → DELIVERED`, `CANCELLED`); search in Elasticsearch; cart in Redis.

### 22.2 Booking system (movie seats / hotel rooms): preventing double booking ⭐

```sql
show_seats(show_id, seat_id, status, held_by, held_until, booking_id,
           PRIMARY KEY (show_id, seat_id))
bookings(id PK, user_id, show_id, status, idempotency_key UNIQUE, created_at)
```
Flow:
1. **Hold** seats with a conditional update, which is atomic and only succeeds if the seat is free:
   ```sql
   UPDATE show_seats
   SET status = 'HELD', held_by = 42, held_until = now() + interval '10 minutes'
   WHERE show_id = 7 AND seat_id IN ('A1','A2')
     AND (status = 'AVAILABLE' OR (status = 'HELD' AND held_until < now()));
   -- rows affected must equal 2, otherwise roll back
   ```
2. User pays within 10 minutes.
3. **Confirm**: set `status = 'BOOKED'` where `held_by = 42`.
4. Expired holds are released automatically by the `held_until` check (plus a cleanup job).

For hotel *date ranges*, PostgreSQL can enforce "no overlapping bookings for the same room" with an **exclusion constraint**:
```sql
ALTER TABLE room_bookings ADD CONSTRAINT no_overlap
  EXCLUDE USING gist (room_id WITH =, daterange(check_in, check_out) WITH &&);
```

### 22.3 URL shortener

```sql
links(short_code VARCHAR(10) PRIMARY KEY, long_url TEXT NOT NULL,
      user_id BIGINT, created_at, expires_at)
```
Very read-heavy (100:1). Generate codes from a unique ID with base62 encoding, or random codes with a unique constraint and retry on collision. Cache hot codes in Redis. Shard by `short_code` hash. Count clicks asynchronously (events to a queue → analytics store), not with an `UPDATE` per click.

### 22.4 Chat app

```sql
conversations(id, type, created_at)
conversation_members(conversation_id, user_id, last_read_message_id)
messages(conversation_id, message_id /* time-ordered */, sender_id, body, created_at,
         PRIMARY KEY (conversation_id, message_id))
```
Main query: latest messages in a conversation, so the key is `(conversation_id, message_id)`. Huge write volume suits a wide-column store (Discord moved messages from MongoDB to Cassandra, and later to ScyllaDB) partitioned by conversation (and time bucket for very active chats). Unread count = messages after `last_read_message_id`.

### 22.5 Ledger / wallet ⭐⭐

Never just update a balance column; record every movement as an **immutable entry** (double-entry bookkeeping):
```sql
ledger_entries(id, transaction_id, account_id, amount /* + credit, - debit */, created_at)
-- every transaction's entries sum to zero
-- balance = SUM(amount) for the account (cache it in accounts.balance, updated in the same transaction)
```
Mistakes are fixed with **reversal entries**, never by updating or deleting rows. This gives a full audit trail and makes reconciliation possible. Use `DECIMAL` or integer minor units, idempotency keys and Serializable or row-locking for balance checks.

### 22.6 Audit tables and history

```sql
order_status_history(id, order_id, from_status, to_status, changed_by, changed_at)
```
Populate from application code, triggers or CDC. For "what did this row look like on date X", use history tables or temporal tables (SQL:2011 system-versioned tables in MariaDB/SQL Server).

### 22.7 Multi-tenancy patterns ⭐⭐

| Pattern | Isolation | Cost | Ops effort | Good for |
|---|---|---|---|---|
| Shared tables + `tenant_id` column | Weakest (one missing `WHERE` leaks data; use RLS) | Lowest | Lowest | Many small tenants (most SaaS) |
| Schema per tenant | Medium | Medium | Migrations × N schemas | Hundreds of tenants |
| Database per tenant | Strongest | Highest | Highest | Few large / regulated enterprise tenants |

Also watch for **noisy neighbours** (one big tenant slowing everyone) and make `tenant_id` the leading column of indexes and the shard key.

### 22.8 Zero-downtime schema migrations (expand → migrate → contract) ⭐⭐

Example: renaming `users.name` to `users.full_name` with no downtime:
1. **Expand:** add `full_name` (nullable). Deploy code that writes to **both** columns and still reads `name`.
2. **Migrate:** backfill `full_name` from `name` in batches.
3. Deploy code that reads `full_name` (still writing both).
4. **Contract:** deploy code that stops writing `name`, then drop the `name` column.

Each step is backward-compatible with the version of the code running before it, so rolling deploys and rollbacks are safe.

Other rules: add indexes concurrently/online; add `NOT NULL` constraints in steps (add as not validated, then validate); avoid operations that rewrite the whole table while holding locks. Engine-specific details are in file 02.

### 22.9 Choosing a database for a use case

| Use case | Typical choice | Why |
|---|---|---|
| Orders, payments, users | PostgreSQL / MySQL | ACID, relations, constraints |
| Session store, cache, rate limiter, leaderboard | Redis | In-memory speed, TTL, data structures |
| Product catalog with varied attributes | PostgreSQL JSONB or MongoDB | Flexible schema |
| Product search with typo tolerance | Elasticsearch / OpenSearch | Inverted index, relevance ranking |
| Chat messages, activity feeds at huge scale | Cassandra / ScyllaDB / DynamoDB | High write throughput, partition by conversation/user |
| Metrics, monitoring | TimescaleDB, InfluxDB, Prometheus | Time-series compression, downsampling |
| Friend-of-friend, fraud rings | Neo4j | Graph traversal |
| Analytics dashboards | BigQuery, Snowflake, ClickHouse | Column store |
| Semantic search / RAG | pgvector, Pinecone, Qdrant | Vector similarity search |

### Interview Questions

**Q1. How do you prevent double booking of the same seat?** ⭐
Make claiming a seat a single atomic, conditional database operation: `UPDATE ... SET status = 'HELD' WHERE seat = ? AND status = 'AVAILABLE'` and check rows affected; or a unique constraint on `(show_id, seat_id)` in a bookings table, so the second insert fails. Add a hold with expiry for the payment window, and idempotency keys for payment retries. For date ranges, use an exclusion constraint (PostgreSQL) or lock the room's rows with `SELECT ... FOR UPDATE`.

**Q2. How would you design a wallet / ledger?** ⭐⭐
An append-only ledger of entries (double-entry: each transaction's entries sum to zero), with the balance derived from entries or cached in an account row updated in the same transaction. Never update or delete entries; fix errors with reversal entries. Use exact decimal types, idempotency keys on every money movement, and row locks or Serializable isolation when checking balances before debiting.

**Q3. How do you rename a column with zero downtime?** ⭐⭐
Expand and contract: add the new column, dual-write from the application, backfill in batches, switch reads to the new column, stop writing the old one, and finally drop it. Each step is compatible with the previous code version, so rolling deploys work.

**Q4. Shared-table or database-per-tenant multi-tenancy?** ⭐⭐
Shared tables with `tenant_id` are cheapest and simplest to operate, and suit many small tenants, but you must enforce tenant filtering everywhere (row-level security helps) and watch for noisy neighbours. Database-per-tenant gives the strongest isolation and per-tenant backup/restore, which suits large or regulated customers, but multiplies operational work. Many SaaS products use shared tables by default and move big enterprise customers to dedicated databases.
