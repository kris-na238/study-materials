# SQL Databases: SQL Language, MySQL and PostgreSQL

Each section explains **what it is, why it matters, how it works with real examples**, followed by **interview questions with model answers**.

Concepts like indexing theory, isolation levels and MVCC are explained in [01_DB_Fundamentals.md](01_DB_Fundamentals.md). This file covers:
- **Part 1:** the SQL language itself (works on any relational database; dialect differences are flagged)
- **Part 2:** what's **different** about MySQL and PostgreSQL internally

Practice problems are in [05_Query_Practice.md](05_Query_Practice.md).

**How to revise with this document**
- First pass: run every query against a local PostgreSQL or MySQL (Docker: `docker run -e POSTGRES_PASSWORD=pw -p 5432:5432 postgres`).
- Revision passes: read only the **Interview Questions**.
- Priority: ⭐ must-know · ⭐⭐ commonly asked · ⭐⭐⭐ senior / deep dive

> Versions: notes reflect **PostgreSQL up to 18** and **MySQL 8.0 / 8.4 LTS** (MySQL 8.0 reached end of life in April 2026; 8.4 is the current LTS). Version-specific features are tagged.

**Running example: ShopKart** (same as file 01)

```sql
users(id, name, email, city, created_at)
products(id, name, category, price, stock)
orders(id, user_id, status, total, created_at)
order_items(order_id, product_id, quantity, unit_price)
payments(id, order_id, amount, status, idempotency_key)
employees(id, name, department, salary, manager_id)   -- for hierarchy examples
```

---

## Table of Contents

**Part 1: SQL Language**
[1. SQL Basics](#1-sql-basics) · [2. Joins](#2-joins) · [3. Aggregation](#3-aggregation) · [4. Subqueries and Set Operations](#4-subqueries-and-set-operations) · [5. CTEs and Window Functions](#5-ctes-and-window-functions) · [6. Views, Procedures and Triggers](#6-views-procedures-and-triggers) · [7. Upserts](#7-upserts)

**Part 2: MySQL vs PostgreSQL**
[8. Architecture](#8-architecture) · [9. Storage and Clustered Index](#9-storage-and-clustered-index) · [10. Index Types](#10-index-types) · [11. Reading EXPLAIN](#11-reading-explain) · [12. Transactions and Locking](#12-transactions-and-locking) · [13. MVCC Implementation](#13-mvcc-implementation) · [14. Logs and Recovery](#14-logs-and-recovery) · [15. Replication and Scaling](#15-replication-and-scaling) · [16. JSON and Full-Text Search](#16-json-and-full-text-search) · [17. Engine-Specific Features](#17-engine-specific-features) · [18. Operations](#18-operations) · [19. MySQL vs PostgreSQL Summary](#19-mysql-vs-postgresql-summary)

---

# PART 1: SQL LANGUAGE

---

## 1. SQL Basics

**Priority:** ⭐ must-know

**What it is:** SQL (Structured Query Language) is the declarative language for relational databases: you describe *what* data you want, and the optimizer decides *how* to get it.

### 1.1 Command categories

| Category | Purpose | Commands |
|---|---|---|
| **DDL** (Data Definition) | Define structure | `CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME` |
| **DML** (Data Manipulation) | Change data | `INSERT`, `UPDATE`, `DELETE`, `MERGE` |
| **DQL** (Data Query) | Read data | `SELECT` |
| **DCL** (Data Control) | Permissions | `GRANT`, `REVOKE` |
| **TCL** (Transaction Control) | Transactions | `BEGIN`/`START TRANSACTION`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` |

Note: in MySQL (and Oracle), DDL statements cause an **implicit commit**, so you can't roll back a `CREATE TABLE` inside a transaction. PostgreSQL supports **transactional DDL**: you can `BEGIN; ALTER TABLE ...; ROLLBACK;`, which makes migrations safer.

### 1.2 Logical order of execution ⭐

You write SQL in one order, but it's evaluated in another:

```
Written order:                Logical execution order:
SELECT   ─────────────┐       1. FROM / JOIN      pick and combine tables
FROM                  │       2. WHERE            filter rows
WHERE                 │       3. GROUP BY         form groups
GROUP BY              │       4. HAVING           filter groups
HAVING                │       5. SELECT           compute columns, aliases, window functions
ORDER BY              │       6. DISTINCT         remove duplicates
LIMIT    ─────────────┘       7. ORDER BY         sort
                              8. LIMIT / OFFSET   cut
```

This explains several common errors:
```sql
-- ❌ Error: alias not known yet in WHERE (WHERE runs before SELECT)
SELECT price * quantity AS line_total FROM order_items WHERE line_total > 1000;
-- ✅
SELECT price * quantity AS line_total FROM order_items WHERE price * quantity > 1000;

-- ❌ Error: aggregates can't be used in WHERE (no groups exist yet)
SELECT user_id, COUNT(*) FROM orders WHERE COUNT(*) > 5 GROUP BY user_id;
-- ✅ Use HAVING
SELECT user_id, COUNT(*) FROM orders GROUP BY user_id HAVING COUNT(*) > 5;

-- ✅ ORDER BY runs after SELECT, so aliases work there
SELECT user_id, SUM(total) AS spent FROM orders GROUP BY user_id ORDER BY spent DESC;
```
(MySQL and PostgreSQL also allow aliases in `GROUP BY` as an extension.)

### 1.3 Filtering

```sql
SELECT * FROM products
WHERE category IN ('Mobiles', 'Laptops')          -- list membership
  AND price BETWEEN 10000 AND 50000               -- inclusive on both ends
  AND name LIKE 'Galaxy%'                          -- starts with (can use an index)
  AND name NOT LIKE '%Refurbished%'                -- contains (can't use a B-tree index)
  AND stock > 0;
```
- `LIKE`: `%` = any number of characters, `_` = exactly one. PostgreSQL also has `ILIKE` (case-insensitive). MySQL's default collations are already case-insensitive.
- `BETWEEN` with timestamps is a trap: `BETWEEN '2026-11-01' AND '2026-11-30'` misses everything on Nov 30 after midnight. Use `>= '2026-11-01' AND < '2026-12-01'`.

### 1.4 NULL handling ⭐

`NULL` means "unknown / missing". SQL uses **three-valued logic**: TRUE, FALSE, UNKNOWN.

```sql
NULL = NULL      → UNKNOWN (not TRUE!)
NULL <> 5        → UNKNOWN
NULL + 10        → NULL
TRUE AND NULL    → UNKNOWN
FALSE AND NULL   → FALSE
TRUE OR NULL     → TRUE
```
`WHERE` keeps only rows where the condition is **TRUE**; UNKNOWN rows are dropped.

```sql
-- ❌ Never matches anything
SELECT * FROM users WHERE city = NULL;
-- ✅
SELECT * FROM users WHERE city IS NULL;

-- Rows where city is NULL are excluded from BOTH of these
SELECT COUNT(*) FROM users WHERE city = 'Pune';
SELECT COUNT(*) FROM users WHERE city <> 'Pune';
```

**The `NOT IN` + NULL trap** (very common interview question):
```sql
-- Users who have never placed an order
SELECT * FROM users WHERE id NOT IN (SELECT user_id FROM orders);
-- If ANY orders.user_id is NULL, this returns ZERO rows.
-- Because: id NOT IN (1, 2, NULL) = id<>1 AND id<>2 AND id<>NULL → ... AND UNKNOWN → never TRUE

-- ✅ Use NOT EXISTS (NULL-safe, and usually optimized as an anti-join)
SELECT * FROM users u
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);
```

**NULL functions:**
```sql
COALESCE(phone, alt_phone, 'N/A')   -- first non-NULL value
NULLIF(discount, 0)                 -- returns NULL if discount = 0 (avoids divide-by-zero)
SELECT total / NULLIF(quantity, 0) FROM ...
-- MySQL: IFNULL(a, b); SQL Server: ISNULL(a, b); both are 2-argument COALESCE
```

**NULLs elsewhere:**
- Aggregates ignore NULLs: `AVG(rating)` averages only non-NULL ratings. `COUNT(col)` skips NULLs; `COUNT(*)` counts rows.
- `GROUP BY` puts all NULLs into one group.
- `ORDER BY`: PostgreSQL sorts NULLs **last** in ascending order; MySQL sorts them **first**. PostgreSQL supports `NULLS FIRST/LAST`.
- `UNIQUE` columns allow multiple NULLs in both (PostgreSQL 15+ offers `UNIQUE NULLS NOT DISTINCT` to allow only one).
- Null-safe equality: `a IS NOT DISTINCT FROM b` (standard, PostgreSQL; MySQL 8.0 doesn't support it) or `a <=> b` (MySQL).

### 1.5 DELETE vs TRUNCATE vs DROP ⭐

| | `DELETE` | `TRUNCATE` | `DROP` |
|---|---|---|---|
| Type | DML | DDL | DDL |
| Removes | Selected rows (`WHERE`) | All rows | The whole table (structure + data + indexes) |
| `WHERE` clause | Yes | No | No |
| Speed on big tables | Slow (row by row, logs each row) | Very fast (deallocates pages) | Very fast |
| Fires row triggers | Yes | No | No |
| Rollback | Yes | PostgreSQL: yes (in a transaction); MySQL: no (implicit commit) | PostgreSQL: yes; MySQL: no |
| Auto-increment | Keeps counting | Resets | Gone |
| Foreign keys | Checked per row | Blocked if referenced (PostgreSQL needs `CASCADE`) | Blocked if referenced |

### Interview Questions

**Q1. What's the order of execution of a SQL query?** ⭐
FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT. That's why you can't use a SELECT alias or an aggregate in WHERE, but you can use an alias in ORDER BY. (This is the *logical* order; the optimizer may physically execute steps differently as long as the result is the same.)

**Q2. What's the difference between WHERE and HAVING?** ⭐
WHERE filters individual rows **before** grouping and can't use aggregates. HAVING filters groups **after** GROUP BY and can use aggregates. Put conditions in WHERE whenever possible, since filtering early means less data to group.

**Q3. Why does `NOT IN` return no rows when the subquery has a NULL?** ⭐
`x NOT IN (a, b, NULL)` expands to `x<>a AND x<>b AND x<>NULL`. `x<>NULL` is UNKNOWN, so the whole condition can never be TRUE and every row is filtered out. Use `NOT EXISTS`, or filter NULLs inside the subquery.

**Q4. DELETE vs TRUNCATE vs DROP?** ⭐
DELETE removes chosen rows, logs each one, fires triggers and can be rolled back. TRUNCATE removes all rows quickly by deallocating storage, resets auto-increment and skips row triggers. DROP removes the table itself. In MySQL, TRUNCATE and DROP commit implicitly; in PostgreSQL both are transactional.

**Q5. What's the difference between `COUNT(*)`, `COUNT(1)` and `COUNT(column)`?** ⭐
`COUNT(*)` and `COUNT(1)` both count all rows and perform the same in modern databases. `COUNT(column)` counts only rows where that column is not NULL. `COUNT(DISTINCT column)` counts distinct non-NULL values.

---

## 2. Joins

**Priority:** ⭐ must-know

**What it is:** Combining rows from two or more tables based on a related column.

Sample data:
```
users                     orders
id | name                 id  | user_id | total
1  | Asha                 101 | 1       | 500
2  | Ravi                 102 | 1       | 300
3  | Meera                103 | 2       | 800
                          104 | 9       | 100    ← user 9 doesn't exist (bad data)
```

### 2.1 Join types

```sql
-- INNER JOIN: only matching rows on both sides
SELECT u.name, o.id FROM users u
INNER JOIN orders o ON o.user_id = u.id;
-- Asha 101, Asha 102, Ravi 103          (Meera and order 104 excluded)

-- LEFT JOIN: all rows from the left, matches from the right (NULL if none)
SELECT u.name, o.id FROM users u
LEFT JOIN orders o ON o.user_id = u.id;
-- Asha 101, Asha 102, Ravi 103, Meera NULL

-- RIGHT JOIN: all rows from the right (rarely used; swap tables and use LEFT)
-- ... Asha 101, Asha 102, Ravi 103, NULL 104

-- FULL OUTER JOIN: all rows from both sides
-- ... Asha 101, Asha 102, Ravi 103, Meera NULL, NULL 104

-- CROSS JOIN: every combination (3 users × 4 orders = 12 rows)
SELECT u.name, s.size FROM users u CROSS JOIN sizes s;   -- e.g. generate all combinations
```

```
INNER          LEFT           RIGHT          FULL
 ┌──┬──┐       ┌──┬──┐        ┌──┬──┐        ┌──┬──┐
 │  │██│       │██│██│        │  │██│██      │██│██│██
 └──┴──┘       └──┴──┘        └──┴──┘        └──┴──┘
```

**MySQL has no `FULL OUTER JOIN`.** Emulate it:
```sql
SELECT u.name, o.id FROM users u LEFT JOIN orders o ON o.user_id = u.id
UNION ALL
SELECT u.name, o.id FROM users u RIGHT JOIN orders o ON o.user_id = u.id
WHERE u.id IS NULL;
```

### 2.2 Self join

Joining a table to itself, typically for hierarchies or comparing rows:
```sql
-- Employees who earn more than their manager
SELECT e.name AS employee, m.name AS manager
FROM employees e
JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

### 2.3 Semi-join and anti-join

Not SQL keywords, but patterns optimizers recognise:
```sql
-- Semi-join: users who HAVE ordered (each user once, no duplicates)
SELECT * FROM users u WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);

-- Anti-join: users who have NEVER ordered
SELECT * FROM users u WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);
-- or
SELECT u.* FROM users u LEFT JOIN orders o ON o.user_id = u.id WHERE o.id IS NULL;
```

### 2.4 Common join mistakes ⭐

**Filtering a LEFT JOIN in WHERE turns it into an INNER JOIN:**
```sql
-- Goal: all users, with their DELIVERED orders if any
-- ❌ Meera disappears: o.status is NULL for her, and NULL = 'DELIVERED' is not TRUE
SELECT u.name, o.id FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE o.status = 'DELIVERED';

-- ✅ Put the condition on the right-side table in ON
SELECT u.name, o.id FROM users u
LEFT JOIN orders o ON o.user_id = u.id AND o.status = 'DELIVERED';
```

**Row multiplication (fan-out):** joining two one-to-many relations multiplies rows.
```sql
-- ❌ If a user has 3 orders and 2 addresses, each order total is counted twice
SELECT u.id, SUM(o.total)
FROM users u
JOIN orders o ON o.user_id = u.id
JOIN addresses a ON a.user_id = u.id
GROUP BY u.id;
-- ✅ Aggregate in a subquery first, then join
SELECT u.id, o.spent, a.address_count
FROM users u
LEFT JOIN (SELECT user_id, SUM(total) AS spent FROM orders GROUP BY user_id) o ON o.user_id = u.id
LEFT JOIN (SELECT user_id, COUNT(*) AS address_count FROM addresses GROUP BY user_id) a ON a.user_id = u.id;
```

**Joining on columns of different types** (`VARCHAR` vs `INT`) prevents index use and can give surprising matches in MySQL (implicit conversion).

### Interview Questions

**Q1. Explain the different types of joins.** ⭐
INNER returns only matching rows. LEFT returns all left rows plus matches (NULLs where none). RIGHT is the mirror. FULL OUTER returns all rows from both sides. CROSS returns the Cartesian product. A SELF join joins a table to itself, e.g. employees to their managers.

**Q2. How do you find rows in table A that have no match in table B?** ⭐
`NOT EXISTS` (preferred, NULL-safe), or `LEFT JOIN B ... WHERE B.id IS NULL`. Avoid `NOT IN` if the subquery column can be NULL.

**Q3. What happens if you put a condition on the right table of a LEFT JOIN in the WHERE clause?** ⭐
Rows without a match have NULLs in the right table's columns, so the WHERE condition is UNKNOWN and those rows are dropped; the LEFT JOIN behaves like an INNER JOIN. Put the condition in the ON clause instead.

**Q4. Why did my SUM double after adding a JOIN?** ⭐⭐
Joining another one-to-many table multiplied the rows (each order repeated per address). Aggregate each child table separately in subqueries/CTEs and then join the results, or use `COUNT(DISTINCT ...)` where appropriate.

**Q5. How do you do a FULL OUTER JOIN in MySQL?**
`LEFT JOIN` union `RIGHT JOIN` filtered to only unmatched rows (`WHERE left.id IS NULL`), using `UNION ALL`.

---

## 3. Aggregation

**Priority:** ⭐ must-know

### 3.1 GROUP BY and HAVING

```sql
-- Revenue and order count per user, only for users who spent more than 10,000
SELECT user_id,
       COUNT(*)        AS orders,
       SUM(total)      AS revenue,
       AVG(total)      AS avg_order_value,
       MAX(created_at) AS last_order_at
FROM orders
WHERE status <> 'CANCELLED'          -- filter rows first
GROUP BY user_id
HAVING SUM(total) > 10000            -- then filter groups
ORDER BY revenue DESC;
```

**Rule:** every column in `SELECT` must either be in `GROUP BY` or inside an aggregate function. Otherwise the database doesn't know which value to show for the group.
- PostgreSQL enforces this (but allows non-grouped columns if you grouped by the table's primary key, since they're functionally dependent).
- MySQL enforces it when `ONLY_FULL_GROUP_BY` is in `sql_mode` (the default since 5.7). Older setups without it returned an arbitrary value, a source of subtle bugs.

### 3.2 Conditional aggregation ⭐

Count different things in one pass using `CASE` (or PostgreSQL's `FILTER`):
```sql
SELECT DATE(created_at) AS day,
       COUNT(*)                                            AS total_orders,
       SUM(CASE WHEN status = 'DELIVERED' THEN 1 ELSE 0 END) AS delivered,
       SUM(CASE WHEN status = 'CANCELLED' THEN 1 ELSE 0 END) AS cancelled,
       ROUND(100.0 * SUM(CASE WHEN status = 'CANCELLED' THEN 1 ELSE 0 END) / COUNT(*), 2)
                                                           AS cancel_rate_pct
FROM orders
GROUP BY DATE(created_at);

-- PostgreSQL: cleaner
SELECT COUNT(*) FILTER (WHERE status = 'DELIVERED') AS delivered FROM orders;
```
Note the `100.0`: in integer division `1 / 3 = 0`. Multiply by a decimal first.

### 3.3 ROLLUP, CUBE, GROUPING SETS ⭐⭐

Subtotals and grand totals in one query:
```sql
-- Revenue by category and month, plus per-category subtotals and a grand total
SELECT category, DATE_TRUNC('month', o.created_at) AS month, SUM(oi.quantity * oi.unit_price)
FROM order_items oi
JOIN orders o ON o.id = oi.order_id
JOIN products p ON p.id = oi.product_id
GROUP BY ROLLUP (category, DATE_TRUNC('month', o.created_at));
```
- `ROLLUP (a, b)` → groups `(a, b)`, `(a)`, `()`: hierarchical subtotals.
- `CUBE (a, b)` → `(a, b)`, `(a)`, `(b)`, `()`: every combination.
- `GROUPING SETS ((a), (b))` → exactly the groupings you list.
- `GROUPING(col)` returns 1 when a row is a subtotal for that column (to tell it apart from a real NULL).
- MySQL syntax: `GROUP BY category, month WITH ROLLUP` (no CUBE/GROUPING SETS in MySQL 8.0).

### Interview Questions

**Q1. Can you SELECT a column that isn't in GROUP BY?** ⭐
Not in standard SQL, unless it's inside an aggregate or functionally dependent on the grouped columns (e.g. grouped by primary key). PostgreSQL enforces this; MySQL does with `ONLY_FULL_GROUP_BY` (default). Without that mode, older MySQL returned an arbitrary row's value, which caused bugs.

**Q2. How do you count rows matching different conditions in one query?** ⭐
Conditional aggregation: `SUM(CASE WHEN condition THEN 1 ELSE 0 END)` or `COUNT(CASE WHEN condition THEN 1 END)` per condition; PostgreSQL also has `COUNT(*) FILTER (WHERE condition)`. One table scan instead of several queries.

**Q3. What's the difference between ROLLUP and CUBE?** ⭐⭐
ROLLUP produces hierarchical subtotals following the column order (category → category+month → grand total). CUBE produces subtotals for every combination of the columns. GROUPING SETS lets you list exactly which groupings you want.

---

## 4. Subqueries and Set Operations

**Priority:** ⭐ must-know

### 4.1 Subquery types

```sql
-- Scalar subquery: returns one value
SELECT name, price, price - (SELECT AVG(price) FROM products) AS diff_from_avg
FROM products;

-- Row subquery: returns one row
SELECT * FROM products WHERE (category, price) = (SELECT category, MAX(price) FROM products WHERE id = 5 GROUP BY category);

-- Table subquery (derived table) in FROM
SELECT AVG(order_count)
FROM (SELECT user_id, COUNT(*) AS order_count FROM orders GROUP BY user_id) t;

-- Subquery in WHERE with IN
SELECT * FROM products WHERE id IN (SELECT product_id FROM order_items WHERE quantity > 10);
```

### 4.2 Correlated vs non-correlated ⭐

**Non-correlated:** runs independently, conceptually once.
```sql
SELECT * FROM products WHERE price > (SELECT AVG(price) FROM products);
```

**Correlated:** references the outer query, conceptually runs **once per outer row**.
```sql
-- Products priced above their own category's average
SELECT p.* FROM products p
WHERE p.price > (SELECT AVG(p2.price) FROM products p2 WHERE p2.category = p.category);
```
Modern optimizers often rewrite correlated subqueries into joins, but not always. If a correlated subquery is slow, rewrite it with a window function or a pre-aggregated join:
```sql
SELECT * FROM (
  SELECT p.*, AVG(price) OVER (PARTITION BY category) AS cat_avg FROM products p
) t WHERE price > cat_avg;
```

### 4.3 EXISTS vs IN ⭐

| | `IN` | `EXISTS` |
|---|---|---|
| Checks | Is the value in this list/result? | Does the subquery return at least one row? |
| Stops early | Depends | Yes, at the first match |
| NULL behaviour | `NOT IN` breaks with NULLs | `NOT EXISTS` is NULL-safe |
| Typical use | Small, fixed lists; non-correlated subqueries | Correlated existence checks |

In modern PostgreSQL and MySQL 8, the optimizer usually turns both `IN (subquery)` and `EXISTS` into the same semi-join plan, so performance is often identical. The real difference that matters is **`NOT IN` vs `NOT EXISTS` with NULLs**.

### 4.4 Set operations

```sql
-- UNION: combine and remove duplicates (needs a sort or hash → slower)
SELECT email FROM customers UNION SELECT email FROM newsletter_subscribers;

-- UNION ALL: combine and keep duplicates (faster; use it unless you need dedup)
SELECT 'order' AS type, created_at FROM orders
UNION ALL
SELECT 'refund', created_at FROM refunds;

-- INTERSECT: rows in both
SELECT user_id FROM orders INTERSECT SELECT user_id FROM reviews;

-- EXCEPT (Oracle: MINUS): rows in the first but not the second
SELECT id FROM users EXCEPT SELECT user_id FROM orders;
```
Rules: same number of columns, compatible types; column names come from the first query. `INTERSECT` and `EXCEPT` are available in MySQL from 8.0.31.

### Interview Questions

**Q1. What's a correlated subquery?** ⭐
A subquery that references columns of the outer query, so logically it's evaluated once per outer row (e.g. "products priced above *their* category's average"). It can be slow on large tables if the optimizer doesn't decorrelate it; rewriting with a JOIN on a pre-aggregated subquery or a window function usually helps.

**Q2. EXISTS or IN: which is faster?** ⭐
In modern optimizers they usually produce the same semi-join plan. EXISTS stops at the first match and is natural for correlated checks; IN is natural for literal lists. The important difference is negation: `NOT IN` returns nothing if the subquery yields a NULL, while `NOT EXISTS` behaves as expected.

**Q3. UNION vs UNION ALL?** ⭐
UNION removes duplicates, which requires sorting or hashing the combined result. UNION ALL keeps everything and is faster. Use UNION ALL unless you specifically need deduplication.

---

## 5. CTEs and Window Functions

**Priority:** ⭐ must-know (window functions are asked in most SQL rounds)

### 5.1 Common Table Expressions (CTEs)

A named temporary result defined with `WITH`, used to make complex queries readable step by step.

```sql
WITH monthly AS (
  SELECT user_id, DATE_TRUNC('month', created_at) AS month, SUM(total) AS spent
  FROM orders
  GROUP BY user_id, DATE_TRUNC('month', created_at)
),
ranked AS (
  SELECT *, RANK() OVER (PARTITION BY month ORDER BY spent DESC) AS rnk
  FROM monthly
)
SELECT * FROM ranked WHERE rnk <= 3;      -- top 3 spenders each month
```

Performance note: PostgreSQL before 12 always **materialized** CTEs (computed them separately, acting as an optimization fence). Since PostgreSQL 12, simple CTEs used once are inlined; force behaviour with `MATERIALIZED` / `NOT MATERIALIZED`. MySQL supports CTEs from 8.0.

### 5.2 Recursive CTEs ⭐⭐

For hierarchies (org charts, category trees, comment threads) and generating sequences.

```sql
-- Full reporting chain under the CEO (id = 1), with depth
WITH RECURSIVE org AS (
  -- anchor: the starting row(s)
  SELECT id, name, manager_id, 1 AS depth, name::TEXT AS path
  FROM employees
  WHERE id = 1

  UNION ALL

  -- recursive part: join back to the CTE itself
  SELECT e.id, e.name, e.manager_id, org.depth + 1, org.path || ' > ' || e.name
  FROM employees e
  JOIN org ON e.manager_id = org.id
)
SELECT * FROM org ORDER BY path;
```
How it runs: evaluate the anchor → feed the result into the recursive part → repeat with only the *new* rows → stop when no new rows are produced.

Guard against cycles (bad data where A manages B and B manages A): limit depth (`WHERE depth < 20`), track visited IDs in an array, or use PostgreSQL 14's `CYCLE` clause. MySQL limits recursion via `cte_max_recursion_depth` (default 1000).

```sql
-- Generate dates (useful for filling gaps in reports)
WITH RECURSIVE days AS (
  SELECT DATE '2026-11-01' AS d
  UNION ALL
  SELECT d + INTERVAL '1 day' FROM days WHERE d < DATE '2026-11-30'
)
SELECT d FROM days;
-- PostgreSQL shortcut: SELECT generate_series('2026-11-01'::date, '2026-11-30', '1 day');
```

### 5.3 Window functions ⭐

**What:** compute a value for each row using a "window" of related rows, **without collapsing rows** like `GROUP BY` does.

```
GROUP BY user_id → one row per user
OVER (PARTITION BY user_id) → every order row kept, each with its user's total alongside
```

**Syntax:**
```sql
function() OVER (
  PARTITION BY ...   -- split rows into groups (optional)
  ORDER BY ...       -- order within each group (needed for ranking, running totals)
  ROWS/RANGE ...     -- frame: which rows around the current row (optional)
)
```

**Ranking functions:**
```sql
SELECT name, department, salary,
  ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS row_num,
  RANK()       OVER (PARTITION BY department ORDER BY salary DESC) AS rnk,
  DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dense_rnk,
  NTILE(4)     OVER (ORDER BY salary DESC)                         AS quartile
FROM employees;
```

| salary | ROW_NUMBER | RANK | DENSE_RANK |
|---|---|---|---|
| 100 | 1 | 1 | 1 |
| 90 | 2 | 2 | 2 |
| 90 | 3 | 2 | 2 |
| 80 | 4 | **4** (skips 3) | **3** (no gap) |

- **ROW_NUMBER:** unique sequence, ties broken arbitrarily (add a tie-breaker column to make it deterministic). Use for deduplication, "latest row per group", pagination.
- **RANK:** ties share a rank, then a gap. Use for competition-style ranking.
- **DENSE_RANK:** ties share a rank, no gaps. Use for "Nth highest distinct salary".
- **NTILE(n):** splits rows into n roughly equal buckets (quartiles, percentiles).

**Top N per group** (very commonly asked):
```sql
-- Latest order of every user
SELECT * FROM (
  SELECT o.*, ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) AS rn
  FROM orders o
) t
WHERE rn = 1;
```
You can't use a window function directly in `WHERE` (it's computed in the SELECT step), hence the subquery/CTE. (Some databases support `QUALIFY`; MySQL and PostgreSQL don't.) PostgreSQL alternative: `SELECT DISTINCT ON (user_id) * FROM orders ORDER BY user_id, created_at DESC;`

**Offset (value) functions:**
```sql
SELECT month, revenue,
  LAG(revenue)  OVER (ORDER BY month) AS prev_month,
  LEAD(revenue) OVER (ORDER BY month) AS next_month,
  ROUND(100.0 * (revenue - LAG(revenue) OVER (ORDER BY month))
        / NULLIF(LAG(revenue) OVER (ORDER BY month), 0), 1) AS mom_growth_pct,
  FIRST_VALUE(revenue) OVER (ORDER BY month) AS first_month_revenue
FROM monthly_revenue;
```
`LAG(col, n, default)`: value from n rows before; useful for month-over-month growth, time between events, detecting changes.

**Aggregates as window functions:**
```sql
SELECT id, user_id, total,
  SUM(total) OVER (PARTITION BY user_id ORDER BY created_at) AS running_total,
  SUM(total) OVER (PARTITION BY user_id)                     AS user_total,
  ROUND(100.0 * total / SUM(total) OVER (PARTITION BY user_id), 2) AS pct_of_user_total,
  AVG(total) OVER (ORDER BY created_at ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS moving_avg_7
FROM orders;
```

### 5.4 Window frames ⭐⭐

The **frame** decides which rows in the partition the function looks at for the current row.

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW     -- running total
ROWS BETWEEN 6 PRECEDING AND CURRENT ROW             -- last 7 rows (moving average)
ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING   -- whole partition
RANGE BETWEEN INTERVAL '7 days' PRECEDING AND CURRENT ROW  -- last 7 *days* by value
```

- **ROWS:** counts physical rows.
- **RANGE:** uses values of the `ORDER BY` column; rows with the **same value (peers)** are treated together.

**The default frame trap:** when you specify `ORDER BY` without a frame, the default is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. Two consequences:
1. A running total with duplicate `ORDER BY` values jumps by both rows at once (peers are included together). Use `ROWS` for a strict row-by-row running total.
2. `LAST_VALUE(x) OVER (ORDER BY y)` returns the *current* row's value, not the partition's last, because the frame ends at the current row. Use `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`.

### Interview Questions

**Q1. Difference between ROW_NUMBER, RANK and DENSE_RANK?** ⭐
All number rows in order within a partition. ROW_NUMBER gives unique numbers even for ties. RANK gives ties the same number and then skips (1, 2, 2, 4). DENSE_RANK gives ties the same number without gaps (1, 2, 2, 3). Use DENSE_RANK for "Nth highest distinct value", ROW_NUMBER for "exactly one row per group".

**Q2. What's the difference between GROUP BY and window functions?** ⭐
GROUP BY collapses rows into one row per group. Window functions compute aggregates or rankings across related rows but keep every row, so you can show each order next to its user's total or running sum.

**Q3. How do you get the top 3 products by revenue in each category?** ⭐
Compute revenue per product, rank with `DENSE_RANK() OVER (PARTITION BY category ORDER BY revenue DESC)` (or ROW_NUMBER for exactly 3), and filter `rank <= 3` in an outer query or CTE, since window functions can't be used directly in WHERE.

**Q4. What's the difference between ROWS and RANGE in a window frame?** ⭐⭐
ROWS counts physical rows relative to the current row. RANGE works on the ORDER BY values and includes all peer rows with the same value. The default frame with ORDER BY is RANGE UNBOUNDED PRECEDING to CURRENT ROW, which surprises people with duplicate ordering values and with LAST_VALUE.

**Q5. When would you use a recursive CTE?** ⭐⭐
For hierarchical or graph data of unknown depth: org charts, category trees, threaded comments, bill of materials, or finding all reachable nodes. Also for generating sequences (dates, numbers). Always guard against cycles and runaway recursion.

**Q6. Do CTEs improve performance?**
Not by themselves; they mainly improve readability. Depending on the database and version they're either inlined (same as a subquery) or materialized once (which helps when reused, but can block optimizations like pushing filters inside). PostgreSQL 12+ lets you choose with `MATERIALIZED` / `NOT MATERIALIZED`.

---

## 6. Views, Procedures and Triggers

**Priority:** ⭐⭐ commonly asked

### 6.1 Views

A **view** is a saved query that behaves like a virtual table. It stores no data; querying it runs the underlying query.
```sql
CREATE VIEW active_customers AS
SELECT u.id, u.name, COUNT(o.id) AS order_count
FROM users u JOIN orders o ON o.user_id = u.id
WHERE o.created_at > now() - INTERVAL '90 days'
GROUP BY u.id, u.name;

SELECT * FROM active_customers WHERE order_count > 5;
```
**Uses:** hide complexity, give a stable interface while tables change underneath, restrict access (grant access to a view exposing only some columns, e.g. without `email`).

**Updatable views:** simple views on one table without aggregates/DISTINCT/GROUP BY can accept `INSERT/UPDATE/DELETE`. `WITH CHECK OPTION` prevents writes through the view that the view itself couldn't see.

### 6.2 Materialized views

Stores the query **result** physically, so reads are fast, but the data is a snapshot until refreshed.
```sql
-- PostgreSQL
CREATE MATERIALIZED VIEW daily_sales AS
SELECT DATE(created_at) AS day, SUM(total) AS revenue FROM orders GROUP BY 1;

CREATE UNIQUE INDEX ON daily_sales (day);
REFRESH MATERIALIZED VIEW CONCURRENTLY daily_sales;   -- CONCURRENTLY: readers aren't blocked (needs a unique index)
```
Real life: a dashboard showing daily revenue for 3 years. Computing it live scans millions of orders on every page load; a materialized view refreshed every 10 minutes makes it instant.

**MySQL has no materialized views.** Emulate with a summary table refreshed by a scheduled event, a cron job, or triggers.

### 6.3 Stored procedures vs functions

| | Function | Stored procedure |
|---|---|---|
| Returns | A value or table; usable inside `SELECT` | Optional OUT parameters/result sets; called with `CALL` |
| Transaction control (`COMMIT` inside) | No | Yes (PostgreSQL 11+, MySQL) |
| Typical use | Calculations, reusable expressions | Multi-step business operations, batch jobs |

```sql
-- PostgreSQL function
CREATE FUNCTION order_total(p_order_id BIGINT) RETURNS NUMERIC AS $$
  SELECT COALESCE(SUM(quantity * unit_price), 0) FROM order_items WHERE order_id = p_order_id;
$$ LANGUAGE sql STABLE;

SELECT id, order_total(id) FROM orders;
```

**Pros of logic in the database:** fewer network round trips, logic close to data, enforced for every client.
**Cons (why many teams avoid heavy use):** harder to version, test, debug and deploy; business logic split between app and database; ties you to one vendor; scales with the database (the hardest component to scale) instead of app servers.

### 6.4 Triggers

Code that runs automatically `BEFORE` or `AFTER` an `INSERT`, `UPDATE` or `DELETE`.
```sql
-- PostgreSQL: maintain updated_at automatically
CREATE FUNCTION set_updated_at() RETURNS trigger AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_products_updated_at
BEFORE UPDATE ON products
FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```
**Good uses:** `updated_at` timestamps, audit logs, enforcing complex invariants the constraint system can't express.
**When to avoid:** hidden business logic ("why did this row change?"), chains of triggers firing triggers, and slowing bulk writes (row triggers run per row). Don't call external systems from triggers.

### Interview Questions

**Q1. View vs materialized view?** ⭐⭐
A view stores only the query and runs it on every access, so it's always current but no faster than the query. A materialized view stores the result, so reads are fast, but data is stale until refreshed. Use materialized views for expensive aggregations that can tolerate some staleness, like dashboards. PostgreSQL supports them natively; in MySQL you build summary tables yourself.

**Q2. Stored procedure vs function?** ⭐⭐
A function returns a value and can be used within SQL expressions; it usually can't manage transactions. A procedure is invoked with CALL, can return multiple results and can commit/rollback inside. Both reduce round trips but put logic in the database, which is harder to test, version and scale.

**Q3. When would you use a trigger, and when not?** ⭐⭐
Use it for small, data-centric rules that must apply no matter which client writes: audit trails, `updated_at`, denormalized counters. Avoid it for business workflows, external calls or anything complex, because it's invisible to application developers, hard to debug and slows bulk operations.

---

## 7. Upserts

**Priority:** ⭐⭐ commonly asked

**What it is:** Insert a row, or update it if it already exists, in **one atomic statement**.

**Why it matters:** The naive approach is a race condition:
```javascript
// ❌ Two concurrent requests both see "not found" and both insert → duplicate or unique-key error
const existing = await db.query('SELECT * FROM cart_items WHERE user_id=$1 AND product_id=$2', [...]);
if (existing) await db.query('UPDATE ...'); else await db.query('INSERT ...');
```

**PostgreSQL: `INSERT ... ON CONFLICT`** (9.5+)
```sql
-- Add to cart: insert, or increase quantity if already in the cart
INSERT INTO cart_items (user_id, product_id, quantity)
VALUES (42, 100, 1)
ON CONFLICT (user_id, product_id)                     -- needs a unique constraint/index on these
DO UPDATE SET quantity = cart_items.quantity + EXCLUDED.quantity,   -- EXCLUDED = the row you tried to insert
              updated_at = now()
RETURNING *;

-- Ignore duplicates (idempotent insert)
INSERT INTO payments (idempotency_key, order_id, amount) VALUES (...)
ON CONFLICT (idempotency_key) DO NOTHING;
```

**MySQL: `INSERT ... ON DUPLICATE KEY UPDATE`**
```sql
INSERT INTO cart_items (user_id, product_id, quantity)
VALUES (42, 100, 1) AS new                               -- row alias (8.0.19+)
ON DUPLICATE KEY UPDATE quantity = cart_items.quantity + new.quantity;
-- Older syntax: quantity = quantity + VALUES(quantity)   (VALUES() deprecated in 8.0.20)

INSERT IGNORE INTO ...   -- skip duplicates (careful: also turns other errors into warnings)
```
Caveat: in MySQL it fires on **any** unique key conflict (primary key or any unique index), which can surprise you on tables with several unique keys.

**`REPLACE INTO` (MySQL):** deletes the existing row and inserts a new one. ⚠️ This fires delete triggers, can cascade-delete child rows via foreign keys, and resets columns you didn't specify to defaults. Prefer `ON DUPLICATE KEY UPDATE`.

**`MERGE`** (SQL standard; PostgreSQL 15+, SQL Server, Oracle; not in MySQL): sync a target table from a source with different actions per case:
```sql
MERGE INTO products p
USING staging_products s ON p.sku = s.sku
WHEN MATCHED AND s.discontinued THEN DELETE
WHEN MATCHED THEN UPDATE SET price = s.price, stock = s.stock
WHEN NOT MATCHED THEN INSERT (sku, name, price, stock) VALUES (s.sku, s.name, s.price, s.stock);
```
Note: PostgreSQL's `MERGE` isn't guaranteed to be race-free under concurrency the way `ON CONFLICT` is; for concurrent single-row upserts, prefer `ON CONFLICT`.

**MongoDB equivalent:** `updateOne(filter, update, { upsert: true })` (file 03). **Redis:** `SET` always upserts.

### Interview Questions

**Q1. How do you insert-or-update atomically?** ⭐⭐
PostgreSQL: `INSERT ... ON CONFLICT (cols) DO UPDATE SET ... = EXCLUDED....` or `DO NOTHING`. MySQL: `INSERT ... ON DUPLICATE KEY UPDATE`. Both need a unique constraint to detect the conflict. Avoid select-then-insert in application code, which races under concurrency.

**Q2. Why avoid `REPLACE INTO` in MySQL?**
It's implemented as DELETE + INSERT: it fires delete triggers, can cascade-delete child rows, generates a new auto-increment ID, and resets unspecified columns to defaults. `INSERT ... ON DUPLICATE KEY UPDATE` modifies the existing row in place.

**Q3. What's EXCLUDED in PostgreSQL's ON CONFLICT?**
A pseudo-table holding the row that was proposed for insertion, so the update can use the new values, e.g. `SET price = EXCLUDED.price`. The existing row is referenced by the table name.

---

# PART 2: MYSQL VS POSTGRESQL

> The concepts (MVCC, WAL, isolation, replication) are explained in file 01. This part covers **how each engine implements them differently**: the questions interviewers ask when your resume says "MySQL" or "PostgreSQL".

---

## 8. Architecture

**Priority:** ⭐⭐ commonly asked

### 8.1 MySQL

```
           Clients
              │
   ┌──────────▼───────────┐
   │ Connection handling  │  one THREAD per connection
   ├──────────────────────┤
   │ SQL layer            │  parser, optimizer, executor, binlog
   ├──────────────────────┤
   │ Storage engine API   │  pluggable
   ├──────┬───────┬───────┤
   │InnoDB│MyISAM │Memory │ ...
   └──────┴───────┴───────┘
```

**Pluggable storage engines:** the SQL layer is separate from how data is stored.

| | **InnoDB** (default since 5.5) | **MyISAM** (legacy) |
|---|---|---|
| Transactions | ✅ ACID | ❌ |
| Locking | Row-level | **Table-level** (one write blocks the whole table) |
| Foreign keys | ✅ | ❌ |
| Crash recovery | ✅ redo log | ❌ tables can corrupt; need `REPAIR TABLE` |
| MVCC | ✅ | ❌ |
| Clustered by PK | ✅ | ❌ (heap + separate indexes) |

Interview answer: **always use InnoDB**. MyISAM only survives in old systems; MySQL 8 even moved its own system tables to InnoDB.

**Thread per connection:** lighter than PostgreSQL's process per connection, so MySQL copes with more connections natively. A thread pool exists in MySQL Enterprise, Percona Server and MariaDB.

### 8.2 PostgreSQL

```
           Clients
              │
       ┌──────▼──────┐
       │ postmaster  │  listens, forks one backend PROCESS per connection
       └──────┬──────┘
   ┌──────────┼──────────┐
 backend   backend    backend       each ~5–10MB+ of memory
   └──────────┼──────────┘
     ┌────────▼─────────┐
     │  Shared memory   │  shared_buffers (page cache), WAL buffers, lock tables
     └────────┬─────────┘
  Background processes: checkpointer, background writer, WAL writer,
  autovacuum launcher + workers, stats, WAL senders (replication), archiver
```

**Process per connection:** strong isolation (a crashing backend doesn't corrupt others), but each connection costs real memory and creating one is slow. That's why PostgreSQL deployments almost always use **PgBouncer** or a similar pooler, and why `max_connections` is kept modest (default 100).

**One storage engine**, but highly **extensible**: custom data types, index types, operators, procedural languages, and extensions (PostGIS, pgvector, TimescaleDB, Citus).

PostgreSQL 18 (2025) added **asynchronous I/O** for reads (sequential scans, bitmap heap scans, VACUUM), a notable performance change for I/O-heavy workloads.

### Interview Questions

**Q1. Why is InnoDB preferred over MyISAM?** ⭐⭐
InnoDB supports ACID transactions, row-level locking, MVCC, foreign keys and crash recovery through its redo log. MyISAM has none of these: it locks the whole table on every write, which kills concurrency, and tables can corrupt after a crash. MyISAM's only historical advantages (full-text search, fast `COUNT(*)`) are gone or irrelevant now.

**Q2. Why does PostgreSQL need a connection pooler like PgBouncer?** ⭐⭐
Each connection is a separate OS process with its own memory, and forking one is relatively slow. Hundreds or thousands of connections from many app instances waste memory and add contention. PgBouncer in transaction mode multiplexes many client connections onto a small number of server connections.

**Q3. How does MySQL's architecture differ from PostgreSQL's?** ⭐⭐
MySQL separates the SQL layer from pluggable storage engines (InnoDB by default) and uses a thread per connection. PostgreSQL has a single integrated storage engine, uses a process per connection, and is designed for extensibility (custom types, index methods, extensions).

---

## 9. Storage and Clustered Index

**Priority:** ⭐⭐ commonly asked (the single most important internal difference)

### 9.1 MySQL InnoDB: the table *is* the primary key index

```
Primary key B+ tree (clustered index)
                 [ 500 | 1000 ]
               /       |        \
  leaf: [id=1 | Asha | Pune | ...full row...] [id=2 | ...] ...   ← rows live in the leaves, sorted by PK

Secondary index on email
  leaf: [asha@x.com → PK 1] [ravi@x.com → PK 2] ...              ← stores the PRIMARY KEY, not a disk address
```

**Lookup by secondary index = two B+ tree traversals:**
```
WHERE email = 'asha@x.com'
  → email index → finds PK = 1
  → primary key index → finds the row        ("bookmark lookup" / "back to table")
```

**Consequences:**
1. **Primary key choice matters a lot.**
   - Sequential PK (auto-increment, UUID v7): inserts append at the right edge; pages fill up neatly.
   - Random PK (UUID v4): inserts land on random pages → page splits, ~50–70% full pages, more I/O, poor cache use.
   - A wide PK (e.g. 36-char UUID string) is copied into **every** secondary index, making all of them bigger. Store UUIDs as `BINARY(16)`, not `CHAR(36)`.
2. **No primary key?** InnoDB uses the first `NOT NULL UNIQUE` index, otherwise it creates a hidden 6-byte row ID. Always define one explicitly.
3. **Range scans on the PK are very fast:** rows are physically adjacent.
4. **Covering secondary indexes get the PK for free:** an index on `(user_id)` can answer `SELECT id FROM orders WHERE user_id = ?` without touching the table.

**Page size:** 16KB by default.

### 9.2 PostgreSQL: heap + separate indexes

```
Heap (table file): rows in no particular order, addressed by ctid = (page, slot)
  page 0: [ctid (0,1): id=7 ...] [ctid (0,2): id=2 ...]
  page 1: [ctid (1,1): id=1 ...] ...

Primary key index:  [id=1 → (1,1)] [id=2 → (0,2)] [id=7 → (0,1)]
Email index:        [asha@x.com → (1,1)] ...       ← every index stores the physical location
```

- All indexes, including the primary key, are secondary and point to the **physical location** (ctid).
- **Every UPDATE creates a new row version at a new ctid** (MVCC, §13), so normally *every index* must get a new entry, even for indexes on columns that didn't change. **HOT updates** (§13) avoid this in many cases.
- `CLUSTER table USING index` physically reorders the table once, but the order isn't maintained afterwards.
- **Page size:** 8KB.

**TOAST** (The Oversized-Attribute Storage Technique) ⭐⭐⭐: rows must fit in a page, so values larger than about 2KB (long text, big JSONB) are compressed and/or moved to a separate TOAST table, with a pointer left in the row. Practical effect: `SELECT *` on tables with large JSON/text columns is expensive because it fetches the TOAST data; selecting only the small columns avoids that.

### 9.3 Side-by-side

| | MySQL InnoDB | PostgreSQL |
|---|---|---|
| Table storage | Clustered on PK (B+ tree) | Heap (unordered) |
| Secondary index points to | Primary key value | Physical location (ctid) |
| PK lookup | 1 tree traversal | Index + heap fetch |
| Secondary index lookup | 2 tree traversals | Index + heap fetch |
| Random PK impact | Severe (table itself fragments) | Moderate (only the PK index) |
| UPDATE of non-indexed column | In place (old version → undo log); secondary indexes untouched | New row version; indexes updated unless HOT |
| Page size | 16KB | 8KB |
| Large values | Off-page overflow pages | TOAST |

### Interview Questions

**Q1. How does InnoDB store a table?** ⭐⭐
As a B+ tree clustered on the primary key: the leaf pages hold full rows sorted by PK. Secondary indexes store the indexed columns plus the PK value, so a secondary lookup traverses the secondary index and then the PK index. That makes PK design important: it should be short and sequentially increasing.

**Q2. Why use UUID v7 instead of UUID v4 as a MySQL primary key?** ⭐⭐
UUID v4 is random, so inserts hit random leaf pages of the clustered index: frequent page splits, half-empty pages, poor buffer pool locality, more write I/O. UUID v7 starts with a timestamp, so new keys are roughly increasing and insert at the end like auto-increment, while still being globally unique. Also store it as `BINARY(16)`.

**Q3. Why can UPDATEs be more expensive in PostgreSQL than in MySQL?** ⭐⭐⭐
PostgreSQL's MVCC writes a whole new row version for every update. Unless the update qualifies as HOT (no indexed column changed and there's room on the same page), every index on the table needs a new entry pointing to the new version. InnoDB updates the row in place, keeps the old version in the undo log, and only touches indexes whose columns changed. This is the "write amplification" problem Uber cited when moving from PostgreSQL to MySQL in 2016.

**Q4. What is TOAST?** ⭐⭐⭐
PostgreSQL's mechanism for large column values: values over roughly 2KB are compressed and/or stored out of line in a separate TOAST table, with a pointer in the main row. It keeps rows small so more fit per page, but reading those columns costs extra I/O, another reason not to `SELECT *` on tables with big text/JSON columns.

---

## 10. Index Types

**Priority:** ⭐⭐ commonly asked

### 10.1 MySQL InnoDB

| Index | Use |
|---|---|
| **B+ tree** (default) | Almost everything: `=`, ranges, `ORDER BY`, prefix `LIKE 'abc%'` |
| **FULLTEXT** | Natural-language word search: `MATCH(name, description) AGAINST('wireless headphones')` |
| **Spatial (R-tree)** | Geometry types |
| **Multi-valued** (8.0.17+) | Index the values of a JSON array |
| **Functional / expression** (8.0.13+) | `CREATE INDEX idx ON users ((LOWER(email)));` |
| **Descending** (8.0+) | `(created_at DESC)` for mixed sort orders |
| **Invisible** (8.0+) | `ALTER TABLE t ALTER INDEX idx INVISIBLE;`: the optimizer ignores it. Test "what if I drop this index?" safely, then drop or make visible again |
| **Prefix index** | `INDEX (description(50))`: index only the first N characters of a long string |

InnoDB also has an **adaptive hash index**: it automatically builds in-memory hash entries for frequently accessed B+ tree pages. You don't create it; it can be disabled if it causes contention.

**Index Condition Pushdown (ICP):** with index `(user_id, status)` and `WHERE user_id = 42 AND status LIKE '%PEND%'`, MySQL checks the `status` condition using the index entries *before* fetching full rows, reducing table lookups. EXPLAIN shows `Using index condition`.

**Index merge:** combining several single-column indexes for `OR`/`AND` conditions. Shows as `index_merge` in EXPLAIN. Often a sign that a composite index would be better.

### 10.2 PostgreSQL

| Index | Supports | Real-life use |
|---|---|---|
| **B-tree** (default) | `=`, `<`, `>`, `BETWEEN`, `ORDER BY`, `LIKE 'abc%'` (with C collation or `text_pattern_ops`) | Almost everything |
| **Hash** | `=` only | Rarely better than B-tree; crash-safe since PostgreSQL 10 |
| **GIN** (Generalized Inverted Index) | "Contains" queries on multi-valued data | JSONB (`@>`, `?`), arrays (`@>`, `&&`), full-text search (`@@`), trigram `LIKE '%x%'` with `pg_trgm` |
| **GiST** (Generalized Search Tree) | Overlap, nearest-neighbour, ranges | PostGIS geometry, range types, exclusion constraints, `ORDER BY location <-> point` |
| **SP-GiST** | Partitioned search spaces | IP addresses, phone-number prefixes, quadtrees |
| **BRIN** (Block Range Index) | Ranges on naturally ordered data | Huge append-only tables ordered by time (logs, events, IoT): stores min/max per block range, so the index is tiny (KB instead of GB) |

**Special index features:**
```sql
-- Partial index: only index the rows you query
CREATE INDEX idx_pending_orders ON orders (created_at) WHERE status = 'PENDING';

-- Expression index
CREATE INDEX idx_users_lower_email ON users (LOWER(email));
SELECT * FROM users WHERE LOWER(email) = 'asha@x.com';   -- uses it

-- Covering index (PostgreSQL 11+)
CREATE INDEX idx_orders_user ON orders (user_id, created_at DESC) INCLUDE (status, total);

-- Build without blocking writes
CREATE INDEX CONCURRENTLY idx_orders_status ON orders (status);

-- GIN on JSONB
CREATE INDEX idx_products_attrs ON products USING GIN (attributes jsonb_path_ops);
SELECT * FROM products WHERE attributes @> '{"color": "black", "ram": "8GB"}';

-- Trigram index for "contains" search
CREATE EXTENSION pg_trgm;
CREATE INDEX idx_products_name_trgm ON products USING GIN (name gin_trgm_ops);
SELECT * FROM products WHERE name ILIKE '%phone%';      -- now uses the index
```

**Index-only scans and the visibility map:** a PostgreSQL index doesn't know if a row version is visible to your transaction (visibility info lives in the heap). An index-only scan can skip the heap only for pages marked **all-visible** in the **visibility map**, which VACUUM maintains. On a heavily updated table that hasn't been vacuumed recently, an "index-only scan" still visits the heap a lot (EXPLAIN ANALYZE shows `Heap Fetches`).

Recent B-tree improvements: **deduplication** of repeated keys (PostgreSQL 13) and **skip scan** on multi-column indexes when the leading column isn't filtered (PostgreSQL 18).

### Interview Questions

**Q1. What index types does PostgreSQL offer and when would you use GIN?** ⭐⭐
B-tree (default), Hash, GIN, GiST, SP-GiST and BRIN. GIN is an inverted index: it maps each element (a JSONB key/value, an array element, a word, a trigram) to the rows that contain it. Use it for JSONB containment (`@>`), array membership, full-text search and `LIKE '%term%'` with pg_trgm. GIN indexes are slower to update, so they suit read-heavy data.

**Q2. What is a BRIN index?** ⭐⭐⭐
A Block Range Index stores only the min and max value for each range of table blocks. It's tiny and cheap to maintain, and works well when the column's values correlate with physical order, e.g. `created_at` on an append-only events table. Queries skip block ranges whose min/max can't match. Useless for randomly ordered data.

**Q3. What's a partial index and when would you use it?** ⭐⭐
An index with a WHERE clause that covers only some rows. Use it when queries target a small subset: `WHERE status = 'PENDING'` (1% of rows) or active, non-deleted rows (`WHERE deleted_at IS NULL`). It's smaller and faster than a full index, and a partial UNIQUE index can enforce uniqueness only among active rows. (PostgreSQL supports it; MySQL doesn't.)

**Q4. What are invisible indexes in MySQL?** ⭐⭐
Indexes the optimizer ignores while they're still maintained. You make an index invisible to test whether removing it hurts any queries; if not, drop it, and if something gets slow, make it visible again instantly instead of rebuilding it.

**Q5. Why might an index-only scan in PostgreSQL still read the table?**
Visibility information isn't stored in the index. PostgreSQL skips the heap only for pages that the visibility map marks all-visible; otherwise it must check the heap row. Frequent updates without timely VACUUM leave many pages not all-visible.

---

## 11. Reading EXPLAIN

**Priority:** ⭐ must-know (the core skill for "fix this slow query")

### 11.1 MySQL

```sql
EXPLAIN SELECT o.id, o.total FROM orders o WHERE o.user_id = 42 AND o.status = 'SHIPPED' ORDER BY o.created_at DESC;
```
```
+----+-------------+-------+------+----------------------+--------------+---------+-------+------+-----------------------------+
| id | select_type | table | type | possible_keys        | key          | key_len | ref   | rows | Extra                       |
+----+-------------+-------+------+----------------------+--------------+---------+-------+------+-----------------------------+
|  1 | SIMPLE      | o     | ref  | idx_user,idx_user_st | idx_user     | 8       | const | 1200 | Using where; Using filesort |
+----+-------------+-------+------+----------------------+--------------+---------+-------+------+-----------------------------+
```

**The `type` column (access type), best to worst:**

| type | Meaning |
|---|---|
| `system` / `const` | At most one matching row, found via PK or unique index (`WHERE id = 5`) |
| `eq_ref` | One row per row of the previous table, via PK/unique index (typical join on PK) |
| `ref` | Several rows via a non-unique index (`WHERE user_id = 42`) |
| `range` | Index range scan (`BETWEEN`, `>`, `IN`, `LIKE 'abc%'`) |
| `index` | **Full index scan**: reads the whole index (better than ALL only if the index is smaller or covering) |
| `ALL` | **Full table scan** 🚩 |

**Other columns:**
- `possible_keys` vs `key`: indexes considered vs the one chosen. `key = NULL` means no index used.
- `key_len`: how many bytes of the index are used; tells you how many columns of a composite index are actually used.
- `rows`: estimated rows examined (estimate, not actual).
- `filtered`: estimated % of rows remaining after the table condition.

**The `Extra` column:**

| Extra | Meaning | Good/bad |
|---|---|---|
| `Using index` | Covering index: answered from the index only | ✅ |
| `Using where` | Rows filtered after being read | Neutral |
| `Using index condition` | Index Condition Pushdown | ✅ |
| `Using filesort` | Extra sort step (in memory or on disk), not satisfied by an index | ⚠️ on large results |
| `Using temporary` | Temporary table (GROUP BY / DISTINCT / UNION) | ⚠️ |
| `Using join buffer (hash join)` | Hash join | Fine for big joins; for OLTP, check join indexes |

Fixing the example: `Using filesort` with 1200 rows → create `INDEX (user_id, status, created_at)`, then `type = ref` on both columns and no filesort.

**Actual execution stats (MySQL 8.0.18+):**
```sql
EXPLAIN ANALYZE SELECT ...;     -- runs the query, shows the tree with actual time and rows
EXPLAIN FORMAT=TREE SELECT ...; -- tree format without running it
```

### 11.2 PostgreSQL

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT o.id, o.total FROM orders o
WHERE o.user_id = 42 AND o.status = 'SHIPPED' ORDER BY o.created_at DESC LIMIT 20;
```
```
Limit  (cost=0.43..85.20 rows=20 width=16) (actual time=0.041..0.210 rows=20 loops=1)
  Buffers: shared hit=24
  ->  Index Scan using idx_orders_user_created on orders o
        (cost=0.43..5100.50 rows=1203 width=16) (actual time=0.040..0.205 rows=20 loops=1)
        Index Cond: (user_id = 42)
        Filter: ((status)::text = 'SHIPPED'::text)
        Rows Removed by Filter: 3
        Buffers: shared hit=24
Planning Time: 0.150 ms
Execution Time: 0.240 ms
```

**How to read it:**
- It's a **tree**; read from the innermost (most indented) node outwards. Each node feeds its parent.
- `cost=startup..total`: the planner's estimate in arbitrary units (relative, not ms).
- `rows=`: **estimated** rows vs `actual ... rows=`: **real** rows. A large mismatch (estimated 10, actual 2,000,000) means bad statistics → bad plan. Run `ANALYZE table`.
- `loops=N`: the node ran N times (inner side of a nested loop); multiply time and rows by loops.
- `Buffers: shared hit` = pages from cache, `read` = from disk. High `read` = I/O bound.
- `Rows Removed by Filter`: rows read and thrown away. A large number means a better index is possible.

**Node types to know:**

| Node | Meaning |
|---|---|
| `Seq Scan` | Full table scan |
| `Index Scan` | Walk the index, fetch each row from the heap |
| `Index Only Scan` | Answer from the index (check `Heap Fetches`) |
| `Bitmap Index Scan` + `Bitmap Heap Scan` | Collect matching locations, then read pages in physical order; medium selectivity or combining indexes |
| `Nested Loop` / `Hash Join` / `Merge Join` | Join algorithms (file 01 §7) |
| `Sort` (`Sort Method: external merge Disk`) | Sort spilled to disk 🚩, consider more `work_mem` or an index providing the order |
| `HashAggregate` / `GroupAggregate` | GROUP BY implementations |

⚠️ `EXPLAIN ANALYZE` **actually executes** the query. For `UPDATE`/`DELETE`, wrap it: `BEGIN; EXPLAIN ANALYZE DELETE ...; ROLLBACK;`

Tools: explain.depesz.com, explain.dalibo.com visualize PostgreSQL plans.

### Interview Questions

**Q1. How do you read a MySQL EXPLAIN output?** ⭐
Check `type` first: `ALL` is a full table scan, `index` a full index scan; `range`, `ref`, `eq_ref` and `const` are progressively better. Check `key` to see which index was chosen (NULL = none), `key_len` to see how much of a composite index is used, `rows` for the estimated rows examined, and `Extra` for `Using filesort` / `Using temporary` (costly) or `Using index` (covering, good). Use `EXPLAIN ANALYZE` for actual timings.

**Q2. How do you read a PostgreSQL EXPLAIN ANALYZE plan?** ⭐
Read the tree from the inside out. Compare estimated vs actual rows at each node to spot bad estimates, look for Seq Scans on large tables, large "Rows Removed by Filter", sorts spilling to disk, and nested loops with high loop counts. Add `BUFFERS` to see cache hits vs disk reads. Find the node where most of the time goes and fix that.

**Q3. What does "Using filesort" mean? Is it always bad?** ⭐⭐
MySQL must sort the rows itself because no index provides the required order. It's not literally always on disk, and it's fine for small results. It becomes a problem when sorting many rows. Fix it with an index whose column order matches the WHERE equality columns followed by the ORDER BY columns.

**Q4. Estimated rows are 10 but actual rows are 1 million. What does that tell you?**
The planner's statistics are wrong or insufficient (stale stats, skewed data, correlated columns), so it probably chose a plan suited to tiny inputs, like a nested loop. Run `ANALYZE`, increase the statistics target for that column, or add extended statistics for correlated columns (PostgreSQL `CREATE STATISTICS`).

---

## 12. Transactions and Locking

**Priority:** ⭐ must-know (defaults) / ⭐⭐ (InnoDB locks)

### 12.1 Default isolation levels

| | MySQL InnoDB | PostgreSQL |
|---|---|---|
| Default | **REPEATABLE READ** | **READ COMMITTED** |
| Read Uncommitted | Supported (dirty reads possible) | Accepted, but behaves as Read Committed |
| Repeatable Read | Snapshot for plain SELECTs; **next-key locks** for locking reads/writes | Snapshot isolation; **aborts** on concurrent update of the same row |
| Serializable | Every plain SELECT becomes `SELECT ... FOR SHARE` (locking) | **SSI**: optimistic, aborts transactions on dangerous patterns |

```sql
-- MySQL
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
-- PostgreSQL
BEGIN ISOLATION LEVEL SERIALIZABLE;
```

Many high-traffic MySQL shops switch to **READ COMMITTED** to reduce gap locking and deadlocks.

### 12.2 InnoDB lock types ⭐⭐

| Lock | What it locks | Purpose |
|---|---|---|
| **Record lock** | One index record | Lock an existing row |
| **Gap lock** | The *gap* between index records (no rows) | Prevent inserts into a range → prevents phantoms |
| **Next-key lock** | Record + the gap before it | Default for locking reads/writes in REPEATABLE READ |
| **Insert intention lock** | A special gap lock taken by INSERT | Lets concurrent inserts into the same gap proceed if they don't conflict on the exact position |
| **Intention locks (IS/IX)** | Table level | Signal row locks inside the table |
| **AUTO-INC lock** | Table level, brief | Allocate auto-increment values (see `innodb_autoinc_lock_mode`) |

**InnoDB locks index records, not rows.** So:
```sql
-- Index on user_id exists → locks only matching index records (+ gaps)
UPDATE orders SET status = 'X' WHERE user_id = 42;

-- No index on coupon_code → full scan → locks EVERY row it examines (+ gaps)
UPDATE orders SET status = 'X' WHERE coupon_code = 'DIWALI';   -- effectively locks the table
```

**Gap lock example (REPEATABLE READ):**
```sql
-- orders.amount index has values 100, 200, 300
T1: SELECT * FROM orders WHERE amount BETWEEN 150 AND 250 FOR UPDATE;
    -- locks record 200 and the gaps (100,200] and (200,300)
T2: INSERT INTO orders (amount) VALUES (180);   -- BLOCKED by the gap lock (would be a phantom)
T2: INSERT INTO orders (amount) VALUES (400);   -- OK
```
Gap locks prevent phantoms but are a common cause of **unexpected blocking and deadlocks**, especially with inserts. Under READ COMMITTED, InnoDB mostly disables gap locking (except for foreign key and duplicate key checks).

**A MySQL RR subtlety:** plain `SELECT` reads the transaction's snapshot, but `UPDATE`/`DELETE`/`SELECT ... FOR UPDATE` read the **latest committed** version. So inside one REPEATABLE READ transaction, a SELECT can show 10 rows while an UPDATE with the same WHERE modifies 11 (a row committed by another transaction after your snapshot).

**Diagnosing:**
```sql
SHOW ENGINE INNODB STATUS\G                     -- "LATEST DETECTED DEADLOCK" section
SELECT * FROM performance_schema.data_locks;     -- current locks (8.0+)
SELECT * FROM sys.innodb_lock_waits;             -- who is blocking whom
-- innodb_print_all_deadlocks = ON → log every deadlock to the error log
```

### 12.3 PostgreSQL locking

**Row-level lock modes:**

| Mode | Taken by | Blocks |
|---|---|---|
| `FOR UPDATE` | `SELECT ... FOR UPDATE`, `DELETE`, `UPDATE` of key columns | All other row locks |
| `FOR NO KEY UPDATE` | Ordinary `UPDATE` (no key column change) | `FOR UPDATE`, `FOR NO KEY UPDATE`, `FOR SHARE` |
| `FOR SHARE` | `SELECT ... FOR SHARE` | Updates |
| `FOR KEY SHARE` | Foreign key checks | Only `FOR UPDATE` (deleting/changing the key) |

The weaker `NO KEY UPDATE` / `KEY SHARE` pair exists so that inserting an order (which checks the FK to `users`) doesn't block a concurrent update of the user's name.

**No gap locks.** PostgreSQL prevents phantoms through snapshots (in Repeatable Read) and SSI predicate tracking (in Serializable), not through gap locking.

**Repeatable Read behaviour:**
```
T1 (RR): BEGIN; SELECT stock FROM products WHERE id=1;   -- 10
T2:      UPDATE products SET stock = 9 WHERE id = 1; COMMIT;
T1:      UPDATE products SET stock = stock - 1 WHERE id = 1;
         ERROR: could not serialize access due to concurrent update
```
PostgreSQL refuses to let T1 overwrite a change it couldn't see, preventing the lost update. The app must **retry**. (MySQL RR would let T1's update apply on top of the latest version.)

**Serializable (SSI):** tracks read/write dependencies (using "SIRead" predicate locks that never block) and aborts one transaction when a cycle could produce a non-serializable result, e.g. the doctors' on-call write skew. Errors have SQLSTATE `40001`; retry them.

**Advisory locks** ⭐⭐⭐: application-defined locks on an arbitrary number, not tied to any row.
```sql
SELECT pg_advisory_xact_lock(hashtext('daily-report-job'));    -- held until transaction ends
SELECT pg_try_advisory_lock(12345);                             -- non-blocking; returns true/false
```
Use for: "only one instance of this cron job runs at a time", serializing work per customer, leader election among app instances. (MySQL equivalent: `GET_LOCK('name', timeout)`.)

**`SKIP LOCKED` / `NOWAIT`:** both databases support them (MySQL 8.0+).

**Table locks from DDL** ⭐⭐: many `ALTER TABLE` forms take an `ACCESS EXCLUSIVE` lock that blocks even reads. Worse, if the ALTER waits behind a long-running query, **every new query queues behind the ALTER**, and the app freezes. Always set a timeout for migrations:
```sql
SET lock_timeout = '5s';
ALTER TABLE orders ADD COLUMN notes TEXT;
```

**Diagnosing:**
```sql
SELECT pid, state, wait_event_type, query, now() - xact_start AS tx_age
FROM pg_stat_activity WHERE state <> 'idle' ORDER BY tx_age DESC;

SELECT pid, pg_blocking_pids(pid) AS blocked_by, query
FROM pg_stat_activity WHERE cardinality(pg_blocking_pids(pid)) > 0;
```

### Interview Questions

**Q1. What are the default isolation levels of MySQL and PostgreSQL, and why does it matter?** ⭐
MySQL InnoDB defaults to REPEATABLE READ; PostgreSQL to READ COMMITTED. The same code can behave differently: for example, under PostgreSQL RR a concurrent update raises a serialization error you must retry, while MySQL RR applies the update to the latest version. Gap locks in MySQL RR can also cause blocking and deadlocks that wouldn't happen in PostgreSQL.

**Q2. What are gap locks and next-key locks?** ⭐⭐
InnoDB-specific locks used in REPEATABLE READ. A gap lock locks the space between index records so no one can insert there; a next-key lock is a record lock plus the gap before it. Together they prevent phantom rows for locking reads and writes. Side effect: unexpected blocking and deadlocks, especially with concurrent inserts into nearby ranges. READ COMMITTED mostly disables them.

**Q3. Why can an UPDATE without an index lock the whole table in MySQL?** ⭐⭐
InnoDB locks the index records it scans, not just the ones it modifies. Without a usable index the UPDATE scans the clustered index, locking every row (and gaps in RR), which blocks all other writers on that table until the transaction ends.

**Q4. How does PostgreSQL handle a concurrent update in Repeatable Read?** ⭐⭐
If your transaction tries to update a row that another transaction modified and committed after your snapshot, PostgreSQL aborts it with "could not serialize access due to concurrent update". This prevents lost updates; the application must catch SQLSTATE 40001 and retry the transaction.

**Q5. What are PostgreSQL advisory locks used for?** ⭐⭐⭐
Application-level mutexes identified by a number, managed by the database but not tied to rows. Common uses: ensuring only one instance of a scheduled job runs across many app servers, or serializing operations for a given customer. Prefer transaction-scoped ones (`pg_advisory_xact_lock`) so they're released automatically. Note they don't work reliably through PgBouncer in transaction mode if session-scoped.

**Q6. A migration that adds a column froze the whole application. What happened?** ⭐⭐
The ALTER needed an exclusive table lock and had to wait for a long-running query or transaction on that table. While it waited in the lock queue, every new query on the table queued behind it, so the app stalled. Prevent it with `lock_timeout` (fail fast and retry), killing long-running transactions before migrations, and using online/lock-free DDL techniques.

---

## 13. MVCC Implementation

**Priority:** ⭐⭐ commonly asked (VACUUM is a favourite PostgreSQL question)

### 13.1 MySQL InnoDB: undo logs

```
Clustered index row (latest version)     Undo log (older versions)
┌─────────────────────────────┐
│ id=1 stock=8  trx_id=120    │──roll_ptr──► [stock=9 trx_id=110] ──► [stock=10 trx_id=90]
└─────────────────────────────┘
```
- The table always holds the **latest** version, updated in place. Each row has hidden columns: `DB_TRX_ID` (last transaction that modified it) and `DB_ROLL_PTR` (pointer to the previous version in the undo log).
- A reader whose **read view** (snapshot) can't see trx 120 follows the roll pointer and rebuilds an older version.
- The **purge** thread deletes undo records once no active read view needs them.
- **History list length** (`SHOW ENGINE INNODB STATUS`) = undo records waiting to be purged. If it keeps growing, a long-running transaction is blocking purge, and reads get slower because they walk longer version chains.

### 13.2 PostgreSQL: versions in the heap

```
Heap page
 ctid (0,1): id=1 stock=10  xmin=90   xmax=110   ← dead once no one needs it
 ctid (0,2): id=1 stock=9   xmin=110  xmax=120   ← dead once no one needs it
 ctid (0,3): id=1 stock=8   xmin=120  xmax=0     ← current version
```
- `xmin`: transaction that created this version. `xmax`: transaction that deleted/replaced it (0 = still live).
- **UPDATE = mark the old version's xmax + INSERT a new version.** DELETE just sets xmax.
- Visibility check per row: "Was xmin committed before my snapshot, and is xmax either empty, aborted, or not yet committed in my snapshot?"
- **Dead tuples** (old versions no transaction needs anymore) stay in the table until **VACUUM** removes them.

### 13.3 VACUUM and autovacuum ⭐⭐

**What VACUUM does:**
1. Removes dead tuples and makes their space **reusable** for new rows in the same table (plain VACUUM doesn't shrink the file).
2. Updates the **visibility map** (enables index-only scans) and **free space map**.
3. **Freezes** old transaction IDs (prevents wraparound, below).
4. `VACUUM ANALYZE` also refreshes planner statistics.

**`VACUUM FULL`:** rewrites the whole table compactly and returns space to the OS, but takes an **ACCESS EXCLUSIVE lock** (blocks reads and writes) for the duration. Avoid on busy production tables; use the `pg_repack` extension for online compaction instead.

**Autovacuum:** background workers that vacuum a table once its dead tuples exceed `autovacuum_vacuum_threshold + autovacuum_vacuum_scale_factor × rows` (defaults: 50 + 20%). On a 500-million-row table, 20% means waiting for **100 million** dead tuples, which is far too late. Tune it per table:
```sql
ALTER TABLE orders SET (autovacuum_vacuum_scale_factor = 0.01, autovacuum_vacuum_cost_limit = 2000);
```

**Bloat:** if dead tuples are produced faster than vacuum cleans them (or a long transaction prevents cleanup), tables and indexes grow, scans read more pages, and caches hold more garbage. Real life: a `jobs` queue table with constant inserts/updates/deletes is a classic bloat victim; queries that took 1 ms take 500 ms after a few weeks.

**What blocks VACUUM from cleaning:**
- Long-running transactions (including "idle in transaction" sessions)
- Abandoned replication slots
- Hot standby queries with `hot_standby_feedback = on`
- Old prepared transactions

### 13.4 HOT updates and fillfactor ⭐⭐⭐

**HOT (Heap-Only Tuple) update:** if an UPDATE changes **no indexed column** and the new version fits on the **same page**, PostgreSQL chains the new version from the old one within the page and **skips updating all indexes**. Huge write saving.

To help HOT happen on frequently updated tables, leave free space in each page:
```sql
ALTER TABLE products SET (fillfactor = 80);   -- fill pages only 80% on insert
```
Design tip: avoid indexing columns that change constantly (like `last_seen_at`, `view_count`) unless necessary, since any index on them prevents HOT.

### 13.5 Transaction ID wraparound ⭐⭐⭐

PostgreSQL transaction IDs are **32-bit**: about 4 billion values, compared in a circular way so each transaction sees ~2 billion IDs as "past" and ~2 billion as "future". If an old row's xmin falls more than ~2 billion transactions behind, it would suddenly look like it's from the *future* and become invisible: silent data loss.

**Prevention:** VACUUM **freezes** old rows, marking them as visible to everyone regardless of xmin. Autovacuum runs anti-wraparound vacuums automatically (`autovacuum_freeze_max_age`, default 200 million).

**If freezing can't keep up** (blocked by long transactions, or autovacuum disabled), PostgreSQL warns, and as a last resort **stops accepting writes** to protect data. This has caused real multi-hour outages (Sentry in 2015, Mailchimp's Mandrill in 2019).

Monitor: `SELECT datname, age(datfrozenxid) FROM pg_database;` and alert well before 1–1.5 billion.

### Interview Questions

**Q1. How is MVCC implemented in PostgreSQL vs MySQL?** ⭐⭐
PostgreSQL keeps every row version in the table itself: an UPDATE writes a new tuple and marks the old one with xmax; VACUUM later removes dead tuples. InnoDB updates rows in place and stores previous versions in the undo log, rebuilding older versions for snapshots by following roll pointers; a purge thread cleans the undo log. PostgreSQL's approach makes rollbacks instant but needs VACUUM; InnoDB's keeps tables compact but long version chains slow reads.

**Q2. What is VACUUM and why does PostgreSQL need it?** ⭐⭐
Because UPDATE and DELETE leave dead row versions in the table. VACUUM marks their space reusable, updates the visibility map (needed for index-only scans), and freezes old transaction IDs to prevent wraparound. Autovacuum does this automatically, but large or busy tables often need per-table tuning. VACUUM FULL compacts the table but locks it exclusively; use pg_repack for online compaction.

**Q3. What is table bloat and how do you fix it?** ⭐⭐
Space taken by dead tuples (and half-empty index pages) that vacuum hasn't reclaimed or can't return to the OS, so tables and indexes are much larger than their live data. Causes: autovacuum not aggressive enough for the write rate, long-running transactions, abandoned replication slots. Fix: remove the blocker, tune autovacuum per table, `REINDEX CONCURRENTLY` bloated indexes, and pg_repack for tables.

**Q4. What is transaction ID wraparound?** ⭐⭐⭐
PostgreSQL uses 32-bit transaction IDs compared circularly. Rows with xmin more than ~2 billion transactions old would appear to be in the future and vanish. VACUUM prevents this by freezing old rows. If freezing is blocked for too long, PostgreSQL forces read-only mode to protect data, which causes outages. Monitor `age(datfrozenxid)` and never let long transactions or disabled autovacuum block freezing.

**Q5. What is a HOT update?**
A Heap-Only Tuple update: when no indexed column changes and the new version fits on the same heap page, PostgreSQL skips inserting new index entries, which greatly reduces write amplification. Lowering fillfactor and not indexing frequently updated columns increase the HOT ratio (`pg_stat_user_tables.n_tup_hot_upd`).

**Q6. In MySQL, what does a growing "history list length" mean?** ⭐⭐⭐
Undo records are piling up because purge can't remove them, usually because a long-running transaction still has an old read view. Reads slow down (longer version chains) and the undo tablespace grows. Find and end the long transaction (`information_schema.innodb_trx`).

---

## 14. Logs and Recovery

**Priority:** ⭐⭐⭐ senior / deep dive (binlog vs redo log is ⭐⭐)

### 14.1 MySQL: redo log + undo log + binlog

| Log | Layer | Purpose | Format |
|---|---|---|---|
| **Redo log** | InnoDB | Crash recovery (WAL): replay committed changes | Physical ("page X changed at offset Y") |
| **Undo log** | InnoDB | Rollback + MVCC old versions | Logical old values |
| **Binary log (binlog)** | MySQL server (SQL layer) | Replication, point-in-time recovery, CDC | Logical (statements or row changes) |

**Why two "commit" logs?** Historical: the binlog belongs to the SQL layer and works for any storage engine; the redo log is InnoDB's own. They must agree, otherwise a crash could leave a transaction committed in InnoDB but missing from the binlog (so replicas never get it), or vice versa.

**Internal two-phase commit between redo log and binlog:**
```
1. Prepare: InnoDB writes the transaction's redo records + a PREPARE mark, flushes redo
2. Write the transaction to the binlog, flush binlog
3. Commit: InnoDB writes a COMMIT mark in the redo log
Crash recovery: a transaction in PREPARE state is committed if it's in the binlog, rolled back if not
```

**Durability settings** (the "double 1" configuration for full safety):
- `innodb_flush_log_at_trx_commit = 1`: flush redo on every commit (2 = write to OS cache each commit, flush once a second; can lose ~1s on an OS crash; 0 = flush once a second, can lose ~1s even on a MySQL crash).
- `sync_binlog = 1`: flush binlog on every commit.

**Binlog formats:**

| Format | Logs | Pros | Cons |
|---|---|---|---|
| `STATEMENT` | The SQL statement | Small | Non-deterministic statements (`NOW()`, `UUID()`, `LIMIT` without `ORDER BY`) can produce different results on replicas |
| `ROW` (default since 5.7.7) | Before/after images of each changed row | Exact, safe; needed for CDC (Debezium) | Large for bulk updates |
| `MIXED` | Statement, switching to row when unsafe | Compromise | Complexity |

**Doublewrite buffer** ⭐⭐⭐: InnoDB pages are 16KB but the OS/disk may write in 4KB units. A crash mid-write leaves a **torn page** that the redo log can't fix (redo records are changes, not whole pages). InnoDB first writes pages to a doublewrite area, then to their real location; on recovery, a torn page is restored from the doublewrite copy.

### 14.2 PostgreSQL: WAL

- **One log** for everything: crash recovery, physical replication, logical decoding (CDC) and PITR.
- **Torn pages** are handled by **full-page writes** (`full_page_writes = on`): after each checkpoint, the first change to any page writes a full image of that page to the WAL.
- `synchronous_commit`: `on` (default, wait for local WAL flush), `off` (return before flush; risk losing the last few hundred ms of commits on a crash, but **no corruption**), plus `remote_write` / `remote_apply` levels for synchronous replication. It can be set per transaction, e.g. off for analytics events, on for payments.
- WAL segments (16MB files) are recycled after checkpoints, unless needed by replicas, replication slots or archiving.

### 14.3 Backups and PITR

| | MySQL | PostgreSQL |
|---|---|---|
| Logical backup | `mysqldump`, `mysqlpump`, MySQL Shell dump utilities | `pg_dump` / `pg_dumpall` |
| Physical backup | Percona XtraBackup, MySQL Enterprise Backup | `pg_basebackup`, pgBackRest, Barman |
| PITR | Restore physical backup + replay binlogs up to a time/position (`mysqlbinlog --stop-datetime`) | Restore base backup + replay archived WAL (`recovery_target_time`) |
| Incremental | XtraBackup incremental | pgBackRest; native `pg_basebackup --incremental` (PostgreSQL 17+) |

### Interview Questions

**Q1. What's the difference between the redo log and the binlog in MySQL?** ⭐⭐
The redo log is InnoDB's physical write-ahead log used for crash recovery. The binlog is the server-level logical log of changes, used for replication, point-in-time recovery and CDC. MySQL keeps them consistent with an internal two-phase commit: InnoDB prepares, the binlog is written, then InnoDB commits; on recovery, prepared transactions are committed only if they're in the binlog.

**Q2. Statement-based vs row-based replication?** ⭐⭐
Statement-based logs SQL statements, which is compact but unsafe for non-deterministic statements (NOW(), RAND(), UUID(), LIMIT without ORDER BY), so replicas can diverge. Row-based logs actual row changes, which is exact and required for CDC tools, but larger for bulk updates. Row is the default and the recommended choice.

**Q3. What does `innodb_flush_log_at_trx_commit` control?** ⭐⭐
When the redo log is flushed to disk. 1 (default) flushes on every commit, fully durable. 2 writes to the OS cache at commit and flushes every second, so an OS crash or power loss can lose about a second of commits. 0 writes and flushes every second, so even a MySQL crash can lose commits. Teams sometimes use 2 on replicas or non-critical systems for throughput.

**Q4. What is `synchronous_commit = off` in PostgreSQL?** ⭐⭐⭐
The commit returns before its WAL is flushed. A crash can lose the most recent commits (bounded by a few times `wal_writer_delay`), but the database stays consistent; it never corrupts. Because it can be set per session or transaction, you can use it for low-value high-volume writes (analytics events) while keeping full durability for payments.

---

## 15. Replication and Scaling

**Priority:** ⭐⭐ commonly asked

### 15.1 MySQL replication

```
Primary:  writes → binlog
              │  (binlog dump thread)
              ▼
Replica:  I/O thread (receiver) → relay log → SQL thread(s) (applier) → replica's tables
```

- **Asynchronous** by default.
- **Semi-synchronous:** commit waits until at least one replica has *received* the events (written to its relay log), not necessarily applied. Falls back to async if no replica acknowledges within `rpl_semi_sync_source_timeout`.
- **GTID** (Global Transaction ID: `server_uuid:transaction_number`): every transaction has a unique ID, so a replica knows exactly which transactions it has applied. Makes failover and re-pointing replicas far easier than the old "binlog file + position" method.
- **Multi-threaded replicas:** apply transactions in parallel (`replica_parallel_workers`, default 4 in recent 8.0) to reduce lag.
- **Group Replication / InnoDB Cluster:** Paxos-based group with automatic failover; single-primary mode (recommended) or multi-primary. **InnoDB ClusterSet** for multi-region.
- Lag: `SHOW REPLICA STATUS` → `Seconds_Behind_Source`.

### 15.2 PostgreSQL replication

**Streaming (physical) replication:**
- The primary's **WAL sender** streams WAL to a **WAL receiver** on the standby, which replays it.
- Standby is a **byte-for-byte copy** of the whole cluster: same major version, all databases, no ability to write.
- **Hot standby:** standbys serve read-only queries.
- **Synchronous replication:** `synchronous_standby_names = 'FIRST 1 (replica_a, replica_b)'` or `'ANY 1 (...)'` (quorum).
- **Query conflicts on standbys:** a long read query on a replica may need rows that the primary has vacuumed away. Either the replica cancels the query ("canceling statement due to conflict with recovery") or, with `hot_standby_feedback = on`, the primary keeps those rows longer (causing bloat on the primary).

**Logical replication** (PostgreSQL 10+):
```sql
-- on the source
CREATE PUBLICATION shop_pub FOR TABLE orders, order_items;
-- on the target
CREATE SUBSCRIPTION shop_sub CONNECTION 'host=... dbname=shop' PUBLICATION shop_pub;
```
- Replicates **row changes for chosen tables**, not the whole cluster.
- Works **across major versions**: used for near-zero-downtime major version upgrades.
- The target is writable (can have extra tables and indexes).
- Doesn't replicate DDL or sequence values; schema changes must be applied on both sides, and sequences synced manually before a cutover.
- Also the basis for CDC tools (Debezium uses logical decoding).

**Replication slots** ⭐⭐: make the primary keep WAL until a consumer has received it, so a disconnected replica or CDC consumer can resume. **Danger:** an abandoned slot makes the primary keep WAL **forever**, filling the disk and taking the database down. Monitor `pg_replication_slots` and set `max_slot_wal_keep_size`.

**Failover:** PostgreSQL has no built-in automatic failover; use **Patroni** (with etcd/Consul), repmgr, or managed services (RDS, Cloud SQL, Aurora).

### 15.3 Partitioning

Both support **RANGE, LIST and HASH** partitioning with partition pruning.

| | MySQL | PostgreSQL |
|---|---|---|
| Syntax | `PARTITION BY RANGE (YEAR(created_at)) (PARTITION p2025 VALUES LESS THAN (2026), ...)` | Declarative: `PARTITION BY RANGE (created_at)` + `CREATE TABLE ... PARTITION OF` |
| Unique keys / PK | Must include **all** partition columns | Must include the partition key |
| Foreign keys | **Not supported** on partitioned InnoDB tables | Supported (PostgreSQL 12+ for referencing partitioned tables) |
| Sub-partitioning | Yes | Yes (partitions can themselves be partitioned) |
| Detach without blocking | `ALTER TABLE ... EXCHANGE PARTITION` | `DETACH PARTITION CONCURRENTLY` (14+) |
| Automatic partition creation | No | No (use the `pg_partman` extension) |

### 15.4 Sharding

Neither has built-in sharding in the core product:
- **MySQL:** **Vitess** (YouTube, Slack, GitHub, PlanetScale), ProxySQL rules, or application-level sharding.
- **PostgreSQL:** **Citus** (extension; distributed tables sharded by a column, e.g. `tenant_id`), or application-level sharding.
- Distributed SQL alternatives with built-in sharding: **TiDB** (MySQL-compatible), **CockroachDB** and **YugabyteDB** (PostgreSQL-compatible), **Aurora Limitless** / **Spanner** (managed).

### Interview Questions

**Q1. How does MySQL replication work?** ⭐⭐
The primary writes changes to the binlog. Each replica's I/O thread copies binlog events into its relay log, and SQL applier threads replay them. It's asynchronous by default; semi-sync makes the primary wait until a replica has received the events. GTIDs uniquely identify transactions, simplifying failover.

**Q2. Physical vs logical replication in PostgreSQL?** ⭐⭐
Physical (streaming) replication ships WAL and produces an exact read-only copy of the entire cluster; same major version required; used for HA and read replicas. Logical replication publishes row changes for selected tables to subscribers, which are writable, can run a different major version, and can have different indexes; used for upgrades, selective sync and CDC. It doesn't replicate DDL.

**Q3. What is a replication slot and what's the risk?** ⭐⭐
A slot makes the primary retain WAL until its consumer (a replica or CDC tool) confirms receiving it, so the consumer can catch up after disconnecting. If the consumer disappears and the slot isn't dropped, WAL accumulates until the disk fills and the primary goes down. Monitor slot lag and set `max_slot_wal_keep_size`.

**Q4. How would you do a major version upgrade of PostgreSQL with minimal downtime?** ⭐⭐⭐
Either `pg_upgrade --link` (fast, needs a short downtime), or for near-zero downtime: set up a new cluster on the new version, copy the schema, use logical replication to sync data, let it catch up, then briefly stop writes, sync sequences, and switch the application over. Managed services offer blue/green deployments built on the same idea.

**Q5. What's a limitation of partitioning in MySQL?** ⭐⭐
Every unique key, including the primary key, must include all partitioning columns, and partitioned InnoDB tables can't have or be referenced by foreign keys. This often forces schema changes (e.g. PK becomes `(id, created_at)`).

---

## 16. JSON and Full-Text Search

**Priority:** ⭐⭐ commonly asked

Real life: ShopKart's products have category-specific attributes: phones have RAM and screen size, shoes have size and material. Instead of 200 nullable columns or an EAV (entity-attribute-value) table, store them in a JSON column.

### 16.1 MySQL JSON

```sql
CREATE TABLE products (
  id BIGINT PRIMARY KEY,
  name VARCHAR(200),
  attributes JSON
);
INSERT INTO products VALUES (1, 'Galaxy S26', '{"brand": "Samsung", "ram_gb": 8, "colors": ["black", "silver"]}');

SELECT name, attributes->>'$.brand' AS brand          -- ->> extracts and unquotes
FROM products
WHERE attributes->>'$.brand' = 'Samsung'
  AND JSON_CONTAINS(attributes->'$.colors', '"black"');
```
- Stored in an optimized **binary format**, validated on insert.
- **No direct index on JSON.** Two ways to index:
```sql
-- 1. Generated column + index
ALTER TABLE products
  ADD COLUMN brand VARCHAR(50) GENERATED ALWAYS AS (attributes->>'$.brand') VIRTUAL,
  ADD INDEX idx_brand (brand);

-- 2. Functional index (8.0.13+)
CREATE INDEX idx_brand ON products ((CAST(attributes->>'$.brand' AS CHAR(50)) COLLATE utf8mb4_bin));

-- 3. Multi-valued index on an array (8.0.17+)
CREATE INDEX idx_colors ON products ((CAST(attributes->'$.colors' AS CHAR(20) ARRAY)));
SELECT * FROM products WHERE 'black' MEMBER OF (attributes->'$.colors');
```
- `JSON_TABLE()` turns JSON arrays into rows for joins.

### 16.2 PostgreSQL JSON and JSONB

| | `json` | `jsonb` |
|---|---|---|
| Storage | Exact text copy | Decomposed binary |
| Key order / duplicates / whitespace | Preserved | Not preserved (last duplicate key wins) |
| Insert speed | Slightly faster | Slightly slower (parsing) |
| Query speed | Slow (re-parse every time) | Fast |
| Indexing | No | **GIN** |
| **Use** | Rarely (exact logging of payloads) | **Almost always** |

```sql
CREATE TABLE products (id BIGSERIAL PRIMARY KEY, name TEXT, attributes JSONB);

SELECT name, attributes->>'brand' AS brand,            -- ->> returns text, -> returns jsonb
       (attributes->>'ram_gb')::int AS ram
FROM products
WHERE attributes @> '{"brand": "Samsung"}'             -- containment (uses GIN)
  AND attributes ? 'ram_gb'                            -- key exists
  AND attributes->'colors' ? 'black';                  -- array contains string

CREATE INDEX idx_attrs ON products USING GIN (attributes);                 -- supports @>, ?, ?|, ?&
CREATE INDEX idx_attrs_path ON products USING GIN (attributes jsonb_path_ops);  -- smaller/faster, only @>
CREATE INDEX idx_brand ON products ((attributes->>'brand'));               -- B-tree on one key

-- Update a nested field
UPDATE products SET attributes = jsonb_set(attributes, '{ram_gb}', '12') WHERE id = 1;
```
PostgreSQL also supports SQL/JSON path queries (`jsonb_path_query`, `@?`) and, from 17, `JSON_TABLE`.

### 16.3 When to use JSON columns

✅ Good:
- Variable attributes that differ per row (product specs, user preferences, feature flags)
- Storing third-party API payloads / webhooks as received
- Rarely queried metadata

❌ Bad:
- Fields you filter, join or aggregate on constantly → make them real columns
- Data needing foreign keys or strict constraints
- Large documents updated frequently (each update rewrites the whole value)

**Hybrid pattern:** core, frequently queried fields as columns (`id`, `name`, `category_id`, `price`) + a JSONB column for the variable attributes.

### 16.4 Full-text search ⭐⭐

**What it is:** searching text by *words and meaning* rather than by exact characters. "running shoes" should match a product described as "Lightweight shoe for runners", and the best matches should come first.

**Why `LIKE '%term%'` isn't enough:**
- **No index use:** a leading wildcard means a B-tree can't help, so every search is a full table scan. Fine at 10k products, painful at 10M.
- **No language awareness:** `LIKE '%run%'` doesn't know "runs", "running" and "ran" are the same word, and it matches "brunch".
- **No ranking:** every match is equally good, so you can't put the most relevant product first.
- **Case and accents:** needs `ILIKE`/`LOWER()` and still misses "café" vs "cafe".

**How it works (the inverted index):** text is split into **tokens**, normalized (lowercased, **stop words** like "the"/"for" dropped, words reduced to a root by **stemming**: "running" → "run"), then an **inverted index** maps each word to the rows that contain it, like the index at the back of a book. A search looks up each word and combines the row lists, then **ranks** them (how often and where the words appear).

**PostgreSQL: `tsvector` and `tsquery`**

```sql
-- tsvector = the normalized document; tsquery = the normalized search
SELECT to_tsvector('english', 'Lightweight running shoes for runners');
-- 'lightweight':1 'run':2 'runner':5 'shoe':3

SELECT to_tsquery('english', 'running & shoe');   -- 'run' & 'shoe'

-- Store it as a generated column so it stays in sync automatically, then index it with GIN
ALTER TABLE products ADD COLUMN search tsvector
  GENERATED ALWAYS AS (
    setweight(to_tsvector('english', coalesce(name, '')), 'A') ||          -- name matters most
    setweight(to_tsvector('english', coalesce(description, '')), 'B')
  ) STORED;
CREATE INDEX idx_products_search ON products USING GIN (search);

-- Search with ranking; websearch_to_tsquery accepts Google-style input: "red shoes" -leather
SELECT id, name, ts_rank(search, q) AS rank
FROM products, websearch_to_tsquery('english', 'running shoes -leather') AS q
WHERE search @@ q                       -- @@ = "matches"
ORDER BY rank DESC
LIMIT 20;
```
- Operators in `to_tsquery`: `&` (and), `|` (or), `!` (not), `<->` (followed by, for phrases), `:*` (prefix: `sho:*` for search-as-you-type).
- `ts_headline()` returns snippets with matched words highlighted.
- **Typos:** the `pg_trgm` extension (trigram similarity) handles "headphnoes" → "headphones" with `similarity()` / the `%` operator and a GIN/GiST trigram index. It also makes `LIKE '%term%'` indexable.

**MySQL: `FULLTEXT` indexes**

```sql
ALTER TABLE products ADD FULLTEXT INDEX ft_products (name, description);

-- Natural language mode (default): relevance-ranked
SELECT id, name, MATCH(name, description) AGAINST('running shoes') AS score
FROM products
WHERE MATCH(name, description) AGAINST('running shoes')
ORDER BY score DESC;

-- Boolean mode: + required, - excluded, * prefix, "..." phrase
SELECT * FROM products
WHERE MATCH(name, description) AGAINST('+shoe* -leather "running"' IN BOOLEAN MODE);
```
- Limits: minimum token length (`innodb_ft_min_token_size`, default 3), a built-in stop-word list, **no stemming**, basic ranking, no typo tolerance. The ngram parser is needed for Chinese/Japanese/Korean.
- The `MATCH` column list must match a `FULLTEXT` index exactly.

**When built-in search is enough vs when to move to Elasticsearch/OpenSearch**

| Need | Built-in (PG / MySQL) | Search engine |
|---|---|---|
| Keyword search on one or two tables, < a few million rows | ✅ | Overkill |
| Always consistent with the data (same transaction) | ✅ | ❌ eventually consistent (refresh lag) |
| Typo tolerance / fuzzy matching | Partial (`pg_trgm`) | ✅ built in |
| Synonyms ("tv" = "television"), autocomplete, "did you mean" | Manual | ✅ |
| Facets/filters with counts at scale ("Brand: Nike (120)") | Slow | ✅ aggregations |
| Custom relevance (boost by sales, rating, recency) | Limited | ✅ |
| Search load isolated from the transactional DB | ❌ shares CPU with checkout | ✅ |

**Rule of thumb:** start with PostgreSQL full-text search. Move to a search engine when search becomes a *product feature* (typos, synonyms, facets, relevance tuning) or search traffic starts hurting the primary database.

**Keeping the search engine in sync:** the database stays the **source of truth**; the search index is a derived copy that can be rebuilt.
- **Dual writes** (app writes DB, then ES): simple, but if the second write fails they drift apart. Avoid.
- **Outbox pattern or CDC** (Debezium reading the WAL/binlog → Kafka → indexer): reliable, no lost updates. Preferred. See [01 → Data Pipelines](01_DB_Fundamentals.md#19-data-pipelines-and-data-integrity).
- Have a **full reindex** job for recovery and mapping changes (build a new index, then switch an alias).
- Accept a small delay (seconds) between a product update and it appearing in search.

Real life: ShopKart launches with PostgreSQL `tsvector` search on 50k products. A year later, with 5M products, users complain that "iphnoe" returns nothing and the search page is 30% of DB CPU. They add OpenSearch fed by Debezium CDC from the `products` table, and keep PostgreSQL as the source of truth.

### Interview Questions

**Q1. JSON vs JSONB in PostgreSQL?** ⭐⭐
`json` stores the exact input text and must re-parse it on every access; `jsonb` stores a parsed binary form (no whitespace, key order or duplicate keys preserved), is much faster to query, and supports GIN indexes and containment operators. Use jsonb unless you need the exact original text.

**Q2. How do you index a JSON field in MySQL?** ⭐⭐
MySQL can't index a JSON column directly. Create a generated column that extracts the value and index it, or (8.0.13+) a functional index on the extracted expression. For arrays, use a multi-valued index (8.0.17+) with `MEMBER OF` / `JSON_CONTAINS`.

**Q3. Should I use PostgreSQL JSONB or MongoDB?** ⭐⭐
If most data is relational and only some attributes are flexible, JSONB in PostgreSQL gives you joins, transactions, constraints and flexible fields in one system. MongoDB fits better when the whole domain is document-shaped, documents are read and written as units, and you want built-in horizontal sharding and a document-first query language/driver ecosystem. Details in file 03.

**Q4. Why is `LIKE '%term%'` bad for search, and what does full-text search do differently?** ⭐⭐
The leading wildcard prevents B-tree index use, so it's a full scan; it has no idea of word forms ("run" vs "running"), matches inside other words, and can't rank results. Full-text search tokenizes and normalizes text (lowercase, stop words, stemming) into an inverted index mapping words to rows, so lookups are fast, match word variants, and return relevance-ranked results.

**Q5. How do you implement full-text search in PostgreSQL?** ⭐⭐
Build a `tsvector` from the text columns (ideally a `GENERATED ... STORED` column, with `setweight` so the name counts more than the description), put a GIN index on it, and query with `search @@ websearch_to_tsquery('english', :input)`, ordering by `ts_rank`. Add `pg_trgm` for typo tolerance and indexable `LIKE '%x%'`.

**Q6. What's MySQL's full-text search and its limitations?** ⭐
A `FULLTEXT` index on `CHAR`/`VARCHAR`/`TEXT` columns queried with `MATCH(...) AGAINST(...)`, in natural-language (ranked) or boolean mode (`+`, `-`, `*`, phrases). Limitations: no stemming, minimum word length and stop-word lists, basic ranking, no typo tolerance, and the `MATCH` columns must exactly match an index.

**Q7. "Our product search is slow and doesn't handle typos. Can't we just add a better index?"** ⭐⭐⭐
An index can make exact or prefix matching faster, but typo tolerance, synonyms and relevance ranking aren't something a B-tree provides. First step: replace `LIKE` with PostgreSQL full-text search plus `pg_trgm`, which may be enough. If search is a core product feature (fuzzy matching, synonyms, facets, ranking by popularity) or it's loading the primary DB, move it to Elasticsearch/OpenSearch, fed by CDC or an outbox from the database, which stays the source of truth. Mention the cost: another system to run, eventual consistency, and reindexing.

**Q8. How do you keep Elasticsearch in sync with the database?** ⭐⭐
Not with dual writes from the app (a failure between the two writes leaves them inconsistent). Use the outbox pattern or CDC (Debezium reading the WAL/binlog into Kafka, consumed by an indexer) so every committed change is delivered at least once, with idempotent indexing by primary key. Keep a full-reindex job (new index + alias swap) for recovery and mapping changes.

---

## 17. Engine-Specific Features

**Priority:** ⭐⭐ commonly asked

### 17.1 MySQL quirks to know

**`sql_mode`:** controls how strict MySQL is. Defaults in 8.0 include `ONLY_FULL_GROUP_BY`, `STRICT_TRANS_TABLES`, `NO_ZERO_DATE`, `ERROR_FOR_DIVISION_BY_ZERO`. Without strict mode, MySQL **silently truncates** too-long strings and converts invalid values (e.g. `'abc'` into an INT column becomes 0) with only a warning. Old apps that "worked" on lax settings break when upgraded to strict mode.

**Character sets:**
- `utf8` in MySQL is actually **`utf8mb3`**: max 3 bytes per character, so it **can't store emoji** (😀 is 4 bytes) or some CJK characters. Inserting them fails or truncates.
- Always use **`utf8mb4`** (the default since 8.0) with a collation like `utf8mb4_0900_ai_ci`.
- Collation suffixes: `_ci` case-insensitive, `_cs` case-sensitive, `_ai` accent-insensitive, `_bin` binary. With `_ci`, `'Asha' = 'asha'` is true, and a UNIQUE index treats them as duplicates.
- PostgreSQL comparisons are case-sensitive by default; use `LOWER()` with an expression index, the `citext` type, or a nondeterministic ICU collation.

**`AUTO_INCREMENT` behaviour:**
- **Gaps are normal:** rolled-back inserts, failed inserts, `INSERT ... ON DUPLICATE KEY UPDATE` and bulk inserts all consume values. Never assume IDs are contiguous.
- `innodb_autoinc_lock_mode = 2` (default in 8.0, "interleaved"): no table-level lock for inserts; fastest, but IDs from concurrent bulk inserts can interleave (needs row-based binlog).
- Before MySQL 8.0, the counter was kept in memory and reset to `MAX(id) + 1` on restart, which could reuse IDs of deleted rows. 8.0 persists it.

**Other MySQL-isms:** backticks for identifiers (`` `order` ``); `LIMIT offset, count` syntax; `IFNULL`; `GROUP_CONCAT` (PostgreSQL: `STRING_AGG`); `ON UPDATE CURRENT_TIMESTAMP` for auto-updated columns; no `RETURNING` (MariaDB has it; use `LAST_INSERT_ID()` in MySQL).

### 17.2 PostgreSQL features interviewers like

```sql
-- DISTINCT ON: first row per group (by the ORDER BY)
SELECT DISTINCT ON (user_id) user_id, id, total, created_at
FROM orders
ORDER BY user_id, created_at DESC;              -- latest order per user

-- RETURNING: get back generated values / changed rows without a second query
INSERT INTO orders (user_id, total) VALUES (42, 999) RETURNING id, created_at;
UPDATE products SET stock = stock - 1 WHERE id = 100 AND stock > 0 RETURNING stock;
DELETE FROM sessions WHERE expires_at < now() RETURNING user_id;

-- FILTER for conditional aggregates
SELECT COUNT(*) FILTER (WHERE status = 'CANCELLED') AS cancelled FROM orders;

-- LATERAL: a subquery that can reference columns from earlier FROM items (like a for-each)
SELECT u.id, recent.*
FROM users u
CROSS JOIN LATERAL (
  SELECT id, total FROM orders o WHERE o.user_id = u.id ORDER BY created_at DESC LIMIT 3
) recent;                                        -- top 3 orders per user, uses the index per user

-- generate_series: numbers or dates
SELECT d::date FROM generate_series('2026-11-01'::date, '2026-11-30', '1 day') d;

-- Arrays
SELECT * FROM products WHERE tags @> ARRAY['wireless'];      -- contains
SELECT unnest(tags) FROM products;                            -- array → rows
SELECT user_id, ARRAY_AGG(id ORDER BY created_at) FROM orders GROUP BY user_id;

-- Range types + exclusion constraint (no overlapping bookings)
CREATE TABLE room_bookings (
  room_id INT, during TSTZRANGE,
  EXCLUDE USING gist (room_id WITH =, during WITH &&)
);

-- IDENTITY columns (SQL standard; preferred over SERIAL)
CREATE TABLE t (id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY);

-- LISTEN / NOTIFY: lightweight pub/sub inside the database
NOTIFY order_created, '501';
```

MySQL equivalents: `LATERAL` exists in MySQL 8.0.14+; `MERGE`, `DISTINCT ON`, `RETURNING`, `FILTER`, arrays and exclusion constraints don't.

**`SERIAL` vs `IDENTITY`:** `SERIAL` is a PostgreSQL shorthand that creates a sequence owned by the column; `GENERATED ... AS IDENTITY` is the SQL-standard version with better permission and copy behaviour. Sequences, like auto-increment, have gaps (values used by rolled-back transactions aren't returned) and are **not** transactional.

### 17.3 PostgreSQL extensions ⭐⭐⭐

| Extension | What it adds | Real-life use |
|---|---|---|
| **PostGIS** | Geospatial types, functions and indexes | "Restaurants within 3km", delivery zones (Swiggy/Zomato-style) |
| **pgvector** | `vector` type, similarity search (HNSW, IVFFlat indexes) | Semantic search, RAG, recommendations, without a separate vector DB |
| **pg_trgm** | Trigram similarity, fast `LIKE '%x%'` and fuzzy matching | Search-as-you-type with typo tolerance |
| **pg_stat_statements** | Aggregated stats for every query shape | Finding the slowest/most frequent queries |
| **TimescaleDB** | Time-series hypertables, compression, continuous aggregates | Metrics, IoT |
| **Citus** | Distributed tables (sharding) | Multi-tenant SaaS at scale |
| **pg_partman** | Automatic partition creation and retention | Time-partitioned event tables |
| **pg_cron** | Cron jobs inside the database | Nightly cleanup, refreshing materialized views |
| **pg_repack** | Online table/index compaction | Removing bloat without locking |

```sql
-- pgvector example: 5 products most similar to a query embedding
CREATE EXTENSION vector;
ALTER TABLE products ADD COLUMN embedding vector(1536);
CREATE INDEX ON products USING hnsw (embedding vector_cosine_ops);
SELECT id, name FROM products ORDER BY embedding <=> $1 LIMIT 5;
```

### Interview Questions

**Q1. Why should you use utf8mb4 instead of utf8 in MySQL?** ⭐⭐
MySQL's `utf8` is an alias for `utf8mb3`, which stores at most 3 bytes per character, so it can't store emoji and some rare characters; inserts fail or truncate. `utf8mb4` is real UTF-8 (up to 4 bytes) and is the default since MySQL 8.0.

**Q2. Why are there gaps in my AUTO_INCREMENT / sequence IDs?** ⭐⭐
IDs are allocated before commit and never returned: rolled-back transactions, failed inserts, upserts that end up updating, and bulk-insert pre-allocation all consume values. This is by design so ID generation doesn't need locks held until commit. Never rely on IDs being contiguous; use `ROW_NUMBER()` if you need sequential numbering for display.

**Q3. What does DISTINCT ON do in PostgreSQL?** ⭐⭐
It returns the first row for each distinct value of the given expression(s), with "first" decided by ORDER BY, which must start with the same expressions. Example: the latest order per user. It's a concise, often fast alternative to `ROW_NUMBER() ... WHERE rn = 1`.

**Q4. What is a LATERAL join?** ⭐⭐
A subquery in FROM that can reference columns from tables listed before it, so it's evaluated per row, like a for-each loop. Classic use: "top N per group" (each user's 3 latest orders with `ORDER BY ... LIMIT 3` inside the lateral subquery), which uses an index per user and can beat window functions when groups are large.

**Q5. What is MySQL's sql_mode and why does it matter?** ⭐⭐
A set of flags controlling SQL strictness and syntax. Strict modes (`STRICT_TRANS_TABLES`) make invalid or too-long data raise errors instead of being silently truncated/converted; `ONLY_FULL_GROUP_BY` rejects ambiguous GROUP BY queries. Lax modes hide data-corruption bugs, so production should run strict.

---

## 18. Operations

**Priority:** ⭐⭐ commonly asked

### 18.1 Finding slow queries

**MySQL:**
```sql
-- Slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 0.5;               -- seconds
-- analyze with: pt-query-digest /var/lib/mysql/slow.log

-- What's running now
SHOW FULL PROCESSLIST;
KILL 12345;                                      -- kill a connection/query

-- Top queries by total time (performance_schema / sys schema)
SELECT * FROM sys.statement_analysis ORDER BY total_latency DESC LIMIT 10;
SELECT * FROM sys.schema_unused_indexes;
SELECT * FROM information_schema.innodb_trx ORDER BY trx_started;   -- long transactions
```

**PostgreSQL:**
```sql
-- Top queries by total time (needs pg_stat_statements in shared_preload_libraries)
SELECT query, calls, total_exec_time, mean_exec_time, rows
FROM pg_stat_statements ORDER BY total_exec_time DESC LIMIT 10;

-- What's running now, and for how long
SELECT pid, usename, state, now() - query_start AS runtime, wait_event_type, query
FROM pg_stat_activity WHERE state <> 'idle' ORDER BY runtime DESC;

SELECT pg_cancel_backend(12345);      -- cancel the query
SELECT pg_terminate_backend(12345);   -- kill the connection

-- Safety nets
ALTER DATABASE shop SET statement_timeout = '30s';
ALTER DATABASE shop SET idle_in_transaction_session_timeout = '60s';

-- Log slow queries
ALTER SYSTEM SET log_min_duration_statement = '500ms';

-- Unused indexes
SELECT relname, indexrelname, idx_scan FROM pg_stat_user_indexes WHERE idx_scan = 0;
```
The **auto_explain** extension logs execution plans of slow queries automatically.

Optimize by **total time** (calls × mean), not just the slowest single query: a 5 ms query called 2 million times an hour matters more than a 10 s report run once a day.

### 18.2 Key configuration parameters

| Purpose | MySQL | PostgreSQL |
|---|---|---|
| Page cache | `innodb_buffer_pool_size`: **50–75% of RAM** on a dedicated server | `shared_buffers`: **~25% of RAM** (relies on OS cache too) |
| Planner's view of cache | — | `effective_cache_size`: ~50–75% of RAM (a hint, allocates nothing) |
| Per-operation memory (sorts, hashes) | `sort_buffer_size`, `join_buffer_size` | `work_mem`, **per sort/hash node per query**, so 64MB × 3 nodes × 200 connections can exhaust RAM |
| Maintenance memory | — | `maintenance_work_mem` (VACUUM, CREATE INDEX) |
| Redo/WAL size | `innodb_redo_log_capacity` (8.0.30+) | `max_wal_size` (larger = fewer checkpoints) |
| Connections | `max_connections` (default 151) | `max_connections` (default 100) |
| Durability | `innodb_flush_log_at_trx_commit`, `sync_binlog` | `synchronous_commit`, `fsync` (never turn off) |
| SSD tuning | `innodb_io_capacity` | `random_page_cost = 1.1` (default 4 assumes spinning disks) |

### 18.3 Online schema changes ⭐⭐

**MySQL Online DDL:**
```sql
ALTER TABLE orders ADD COLUMN notes TEXT, ALGORITHM=INSTANT;          -- metadata only, instant
ALTER TABLE orders ADD INDEX idx_status (status), ALGORITHM=INPLACE, LOCK=NONE;  -- concurrent DML allowed
```
| Algorithm | How | Blocks writes? |
|---|---|---|
| `INSTANT` (8.0.12+) | Change metadata only | No (brief metadata lock). Adding columns (any position from 8.0.29), dropping columns, renaming, changing defaults |
| `INPLACE` | Rebuild/modify in the engine | Usually no (`LOCK=NONE`); brief metadata locks at start/end |
| `COPY` | Create a new table, copy all rows | **Yes**; e.g. changing a column's data type |

Specify `ALGORITHM=` explicitly: if the requested algorithm isn't possible, MySQL returns an error instead of silently doing a blocking COPY.

For huge tables or changes that need COPY, use **gh-ost** (GitHub; reads the binlog, no triggers, can be paused/throttled) or **pt-online-schema-change** (Percona; uses triggers): they build a shadow table, copy rows in chunks, keep it in sync, then atomically swap names.

Even INSTANT/INPLACE operations need a brief **metadata lock**; if a long transaction holds the table, the ALTER waits, and everything queues behind it (`Waiting for table metadata lock`). Set `lock_wait_timeout` low for migrations.

**PostgreSQL:**

| Operation | Safe way |
|---|---|
| Add an index | `CREATE INDEX CONCURRENTLY` (slower, can't run in a transaction; if it fails, drop the INVALID index and retry) |
| Add a column | Instant, including with a **constant default** (PostgreSQL 11+). A *volatile* default (`DEFAULT random()`) rewrites the table |
| Add `NOT NULL` | Add `CHECK (col IS NOT NULL) NOT VALID` → `VALIDATE CONSTRAINT` (no long lock) → `SET NOT NULL` (12+ uses the validated check, skipping the scan) |
| Add a foreign key | `ADD CONSTRAINT ... NOT VALID`, then `VALIDATE CONSTRAINT` separately |
| Change column type | Usually a full rewrite with an exclusive lock → expand/contract with a new column instead |
| Rebuild a bloated index | `REINDEX INDEX CONCURRENTLY` (12+) |
| Every DDL | `SET lock_timeout = '5s'` and retry, to avoid queueing everything behind you |

### 18.4 Row-level security (PostgreSQL) ⭐⭐⭐

```sql
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON orders
  USING (tenant_id = current_setting('app.tenant_id')::bigint);

-- App sets the tenant per transaction
SET LOCAL app.tenant_id = '42';
SELECT * FROM orders;   -- automatically filtered to tenant 42
```
The database enforces tenant isolation even if a developer forgets a `WHERE tenant_id = ?`. Table owners and superusers bypass RLS unless `FORCE ROW LEVEL SECURITY` is set. Supabase builds its whole authorization model on it. MySQL has no native RLS (use views or application filters).

### Interview Questions

**Q1. How do you find the slowest queries in production?** ⭐
PostgreSQL: `pg_stat_statements` ordered by total execution time, the slow query log (`log_min_duration_statement`), and auto_explain for plans. MySQL: the slow query log analysed with pt-query-digest, and `performance_schema` / `sys.statement_analysis`. Prioritize by total time (frequency × duration), then EXPLAIN ANALYZE the top offenders.

**Q2. How do you add an index to a large production table without downtime?** ⭐⭐
PostgreSQL: `CREATE INDEX CONCURRENTLY`, which builds without blocking writes (and check for an INVALID index if it fails). MySQL: `ALTER TABLE ... ADD INDEX ..., ALGORITHM=INPLACE, LOCK=NONE`, or gh-ost/pt-online-schema-change for very large tables. In both, set a lock timeout and avoid running it while long transactions hold the table.

**Q3. How should you size `innodb_buffer_pool_size` and `shared_buffers`?** ⭐⭐
On a dedicated MySQL server, 50–75% of RAM for the buffer pool, since InnoDB does its own caching (often with O_DIRECT). PostgreSQL relies on both shared_buffers and the OS page cache, so ~25% of RAM for shared_buffers and `effective_cache_size` set to ~50–75% to tell the planner how much caching to expect. Then watch hit ratios.

**Q4. Why can a large `work_mem` crash a PostgreSQL server?** ⭐⭐⭐
`work_mem` is allocated per sort or hash operation, per query, per connection, and a single complex query can have several. With many connections, total usage can be many times `work_mem` and exhaust RAM, triggering the OOM killer. Keep the global value modest and raise it per session for specific heavy queries.

**Q5. What is gh-ost and why use it?** ⭐⭐⭐
GitHub's online schema migration tool for MySQL. It creates a shadow table with the new schema, copies rows in small chunks, applies ongoing changes by reading the binlog (no triggers), and finally swaps the tables atomically. It can be throttled or paused based on replica lag and load, which makes changes on huge tables safe when native online DDL would block or overload the server.

---

## 19. MySQL vs PostgreSQL Summary

**Priority:** ⭐ must-know ("Why PostgreSQL over MySQL?" is a common question)

| Area | MySQL (InnoDB) | PostgreSQL |
|---|---|---|
| Architecture | Thread per connection, pluggable engines | Process per connection, extensible single engine |
| Table storage | Clustered on primary key | Heap + separate indexes |
| MVCC | Undo log, in-place updates, purge | Row versions in heap, VACUUM |
| Default isolation | Repeatable Read (gap locks) | Read Committed |
| Serializable | Lock-based | SSI (optimistic) |
| Transactional DDL | No | **Yes** |
| Index types | B+ tree, FULLTEXT, spatial, multi-valued, functional, invisible | B-tree, Hash, GIN, GiST, SP-GiST, BRIN, partial, expression, covering |
| JSON | JSON (index via generated/functional columns) | JSONB with GIN, very capable |
| SQL features | Good (CTEs, windows since 8.0) | Richer: `DISTINCT ON`, `RETURNING`, `FILTER`, arrays, ranges, exclusion constraints, `MERGE` |
| Materialized views | No | Yes |
| Extensions | Plugins, limited | PostGIS, pgvector, TimescaleDB, Citus, ... |
| Replication | Binlog (async, semi-sync), GTID, Group Replication; mature, simple | Physical streaming + logical; failover via Patroni |
| Write-heavy updates | Often more efficient (no full-row rewrite, fewer index updates) | Write amplification unless HOT |
| Sharding ecosystem | Vitess, PlanetScale | Citus |
| Licence | GPL (Oracle-owned), with commercial editions | PostgreSQL licence (permissive, community-run) |
| Typical users | Facebook/Meta (MyRocks), Shopify, GitHub, Uber (post-2016), Booking.com | Instagram, Apple, Reddit, Supabase, Notion, many startups |

**How to answer "Which would you choose?":**
- **PostgreSQL** when you need rich SQL, complex queries, JSONB, geospatial/vector extensions, strict standards and data integrity. It's the default choice for most new projects today.
- **MySQL** when the team and tooling already know it, for very high-volume simple read/write OLTP with mature replication and sharding (Vitess), or when using managed MySQL-compatible platforms (Aurora MySQL, PlanetScale, TiDB).
- Both are excellent; the operational experience of the team usually matters more than the feature list.

### Interview Questions

**Q1. MySQL vs PostgreSQL: what are the key differences?** ⭐
Storage (clustered index vs heap), MVCC (undo log vs in-heap versions with VACUUM), default isolation (Repeatable Read vs Read Committed), features (PostgreSQL has richer SQL, JSONB, more index types, transactional DDL and extensions), and process model (threads vs processes). MySQL has historically had simpler replication and strong sharding tools (Vitess); PostgreSQL is more extensible and standards-compliant.

**Q2. Why did Uber move from PostgreSQL to MySQL (2016)?** ⭐⭐⭐
Their write-heavy workload suffered from PostgreSQL's write amplification: every update created a new row version and, with many indexes, new index entries, which also inflated WAL and replication traffic. They also cited replication issues across major versions and the cost of PostgreSQL's process-per-connection model. MySQL's in-place updates with secondary indexes pointing to primary keys suited their pattern better. The counterpoint: PostgreSQL has improved since (HOT, logical replication), and many companies run very large PostgreSQL fleets. A good answer frames it as workload fit, not "one is better".
