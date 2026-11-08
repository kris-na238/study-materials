# Query Practice: SQL and MongoDB

Interview-style query problems grouped by **pattern**, each with a worked solution. Once you recognise the pattern, most interview SQL questions become variations of these.

Syntax is explained in [02_SQL_Databases.md](02_SQL_Databases.md) (SQL) and [03_NoSQL_MongoDB.md](03_NoSQL_MongoDB.md) (MongoDB).

**How to practice with this document**
1. Read the **Question** and **Sample tables** only.
2. Write your own solution (on paper or in a database) **before** reading the answer.
3. Compare with the **Approach** and **SQL**; read the **Alternative** for the other way interviewers expect.
4. Revisit problems you got wrong after 2–3 days.
5. Daily target: 2–3 problems. Then practice more on **LeetCode SQL 50**, **DataLemur** and **StrataScratch**.

Solutions use **PostgreSQL** syntax; MySQL differences are noted where they matter.

Priority: ⭐ must-know · ⭐⭐ commonly asked · ⭐⭐⭐ senior / deep dive

**Pattern cheat sheet**

| If the question says... | Think... |
|---|---|
| "never", "without", "not in" | Anti-join: `NOT EXISTS` / `LEFT JOIN ... IS NULL` |
| "more than N per group" | `GROUP BY` + `HAVING` |
| "Nth highest", "top N per group", "latest per X" | `DENSE_RANK` / `ROW_NUMBER` in a subquery |
| "running", "cumulative", "so far" | `SUM() OVER (ORDER BY ...)` |
| "compared to previous", "growth" | `LAG()` |
| "consecutive", "streak" | Gaps and islands: `value − ROW_NUMBER()` |
| "hierarchy", "all reports under" | Recursive CTE |
| "for every", "all of" | Relational division |
| "each day, including days with zero" | Generate a date series + `LEFT JOIN` |
| "rows to columns" | Conditional aggregation `SUM(CASE WHEN ...)` |
| "median", "percentile" | `PERCENTILE_CONT` / window `ROW_NUMBER` + `COUNT` |

---

## Table of Contents

**Part 1: SQL**
[1. Filtering and Joins](#1-filtering-and-joins) · [2. Aggregation](#2-aggregation) · [3. Ranking](#3-ranking) · [4. Running Calculations](#4-running-calculations) · [5. Hierarchies](#5-hierarchies) · [6. Gaps and Islands](#6-gaps-and-islands) · [7. Pivoting](#7-pivoting) · [8. Statistics](#8-statistics) · [9. Product Analytics](#9-product-analytics) · [10. Advanced Patterns](#10-advanced-patterns)

**Part 2: MongoDB**
[11. Basic Queries](#11-basic-queries) · [12. Aggregation](#12-aggregation) · [13. Advanced Aggregation](#13-advanced-aggregation)

---

# PART 1: SQL

---

## 1. Filtering and Joins

**Priority:** ⭐ must-know

### 1.1 Customers who never ordered ⭐

**Question:** List all users who have never placed an order.

**Sample tables:**
```
users                 orders
id | name             id  | user_id
1  | Asha             101 | 1
2  | Ravi             102 | 1
3  | Meera            103 | 3
4  | Kabir            104 | NULL     ← guest checkout
```
Expected: `Ravi`, `Kabir`.

**Approach:** anti-join. Watch out for NULLs in `orders.user_id`.

**SQL:**
```sql
SELECT u.id, u.name
FROM users u
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);
```

**Alternative:**
```sql
SELECT u.id, u.name
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE o.id IS NULL;
```
**Trap:** `WHERE u.id NOT IN (SELECT user_id FROM orders)` returns **nothing** here because of the NULL `user_id` in order 104. If you use `NOT IN`, add `WHERE user_id IS NOT NULL` inside the subquery.

---

### 1.2 Employees earning more than their manager ⭐

**Question:** Find employees whose salary is higher than their manager's.

**Sample table:**
```
employees
id | name  | salary | manager_id
1  | Anil  | 90000  | NULL
2  | Bina  | 95000  | 1
3  | Chetan| 60000  | 1
4  | Divya | 70000  | 3
```
Expected: `Bina` (95000 > 90000), `Divya` (70000 > 60000).

**Approach:** self-join employees to employees on `manager_id = id`.

**SQL:**
```sql
SELECT e.name AS employee, e.salary, m.name AS manager, m.salary AS manager_salary
FROM employees e
JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

---

### 1.3 Paid orders without a successful payment record ⭐

**Question:** Find orders marked `PAID` that have no payment with status `SUCCESS` (a data-integrity check).

**Approach:** anti-join with a condition on the right table: the condition belongs in the subquery/`ON`, not the outer `WHERE`.

**SQL:**
```sql
SELECT o.id, o.total
FROM orders o
WHERE o.status = 'PAID'
  AND NOT EXISTS (
    SELECT 1 FROM payments p
    WHERE p.order_id = o.id AND p.status = 'SUCCESS'
  );
```

**Alternative:**
```sql
SELECT o.id, o.total
FROM orders o
LEFT JOIN payments p ON p.order_id = o.id AND p.status = 'SUCCESS'   -- condition in ON
WHERE o.status = 'PAID' AND p.id IS NULL;
```

---

### 1.4 Products never sold in the last 30 days ⭐

**SQL:**
```sql
SELECT p.id, p.name
FROM products p
WHERE NOT EXISTS (
  SELECT 1
  FROM order_items oi
  JOIN orders o ON o.id = oi.order_id
  WHERE oi.product_id = p.id
    AND o.created_at >= CURRENT_DATE - INTERVAL '30 days'
);
```
MySQL: `CURRENT_DATE - INTERVAL 30 DAY`.

---

## 2. Aggregation

**Priority:** ⭐ must-know

### 2.1 Departments with more than N employees ⭐

**Question:** Departments with more than 5 employees, with their average salary.

**SQL:**
```sql
SELECT department, COUNT(*) AS headcount, ROUND(AVG(salary), 2) AS avg_salary
FROM employees
GROUP BY department
HAVING COUNT(*) > 5
ORDER BY headcount DESC;
```

---

### 2.2 Daily order stats with cancellation rate ⭐

**Question:** For each day, show total orders, delivered orders, cancelled orders and cancellation rate (%).

**Approach:** conditional aggregation; one pass over the table.

**SQL:**
```sql
SELECT
  DATE(created_at) AS day,
  COUNT(*) AS total_orders,
  COUNT(*) FILTER (WHERE status = 'DELIVERED') AS delivered,
  COUNT(*) FILTER (WHERE status = 'CANCELLED') AS cancelled,
  ROUND(100.0 * COUNT(*) FILTER (WHERE status = 'CANCELLED') / COUNT(*), 2) AS cancel_rate_pct
FROM orders
GROUP BY DATE(created_at)
ORDER BY day;
```

**MySQL / portable version:**
```sql
SUM(CASE WHEN status = 'CANCELLED' THEN 1 ELSE 0 END) AS cancelled
-- MySQL shortcut: SUM(status = 'CANCELLED')  (boolean is 1/0)
```
**Trap:** `100 * cancelled / total` with integers gives integer division in PostgreSQL. Use `100.0`.

---

### 2.3 Find duplicate emails ⭐

**Question:** Find emails that appear more than once in `users`.

**SQL:**
```sql
SELECT LOWER(TRIM(email)) AS email, COUNT(*) AS occurrences
FROM users
GROUP BY LOWER(TRIM(email))
HAVING COUNT(*) > 1;
```
Normalizing with `LOWER(TRIM())` catches `Asha@x.com ` vs `asha@x.com`, which is what real duplicates look like.

To see the full duplicate rows:
```sql
SELECT *
FROM (
  SELECT u.*, COUNT(*) OVER (PARTITION BY LOWER(email)) AS cnt
  FROM users u
) t
WHERE cnt > 1
ORDER BY LOWER(email), id;
```

---

### 2.4 Delete duplicates, keeping the oldest row ⭐

**Question:** Remove duplicate users by email, keeping the row with the smallest `id`.

**Approach:** number rows within each duplicate group and delete everything except row 1.

**SQL (PostgreSQL):**
```sql
DELETE FROM users
WHERE id IN (
  SELECT id FROM (
    SELECT id, ROW_NUMBER() OVER (PARTITION BY LOWER(email) ORDER BY id) AS rn
    FROM users
  ) t
  WHERE rn > 1
);
```

**Alternative (self-join delete):**
```sql
-- PostgreSQL
DELETE FROM users a
USING users b
WHERE LOWER(a.email) = LOWER(b.email) AND a.id > b.id;

-- MySQL
DELETE a FROM users a
JOIN users b ON LOWER(a.email) = LOWER(b.email) AND a.id > b.id;
```
**MySQL note:** MySQL doesn't allow selecting from the same table you're deleting from in a plain subquery (error 1093); wrap the subquery in another derived table, or use the join form.

**Follow-up:** prevent it from happening again with a unique index: `CREATE UNIQUE INDEX ON users (LOWER(email));` (and in practice, re-point child rows such as orders to the kept user before deleting).

---

## 3. Ranking

**Priority:** ⭐ must-know (the single most common SQL interview category)

### 3.1 Second / Nth highest salary ⭐

**Question:** Find the second-highest **distinct** salary. Return NULL if it doesn't exist.

```
salaries: 100, 90, 90, 80
second highest distinct = 90, third = 80
```

**Approach 1: DENSE_RANK (generalizes to N)**
```sql
SELECT MAX(salary) AS second_highest      -- MAX() returns NULL when there are no rows
FROM (
  SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
  FROM employees
) t
WHERE rnk = 2;
```

**Approach 2: LIMIT / OFFSET**
```sql
SELECT (
  SELECT DISTINCT salary FROM employees
  ORDER BY salary DESC
  LIMIT 1 OFFSET 1          -- N-1
) AS second_highest;        -- outer SELECT makes it return NULL instead of no row
```

**Approach 3: subquery (no window functions)**
```sql
SELECT MAX(salary) FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

**Approach 4: correlated count (works for N, classic interview answer)**
```sql
SELECT DISTINCT salary FROM employees e1
WHERE 2 - 1 = (SELECT COUNT(DISTINCT salary) FROM employees e2 WHERE e2.salary > e1.salary);
```

**Why DENSE_RANK and not RANK or ROW_NUMBER?** With salaries 100, 90, 90, 80: `ROW_NUMBER` gives 90 as both 2nd and 3rd; `RANK` gives 1, 2, 2, **4**, so "rank 3" doesn't exist; `DENSE_RANK` gives 1, 2, 2, 3, which matches "third highest distinct salary = 80".

---

### 3.2 Top N per group ⭐

**Question:** Top 3 highest-paid employees in each department (include ties).

**SQL:**
```sql
SELECT department, name, salary
FROM (
  SELECT e.*, DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rnk
  FROM employees e
) t
WHERE rnk <= 3
ORDER BY department, salary DESC;
```
- **Exactly 3 rows per group** (no ties): `ROW_NUMBER()` with a tie-breaker: `ORDER BY salary DESC, id`.
- **Include ties, but at most 3 distinct salary levels:** `DENSE_RANK()` (as above).
- **Ties that "use up" places (sports-style):** `RANK()`.

**Alternative (PostgreSQL LATERAL, efficient with an index on `(department, salary DESC)`):**
```sql
SELECT d.department, top.name, top.salary
FROM (SELECT DISTINCT department FROM employees) d
CROSS JOIN LATERAL (
  SELECT name, salary FROM employees e
  WHERE e.department = d.department
  ORDER BY salary DESC
  LIMIT 3
) top;
```

---

### 3.3 Latest row per group ⭐

**Question:** Each user's most recent order.

**SQL:**
```sql
SELECT *
FROM (
  SELECT o.*, ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC, id DESC) AS rn
  FROM orders o
) t
WHERE rn = 1;
```

**Alternatives:**
```sql
-- PostgreSQL
SELECT DISTINCT ON (user_id) * FROM orders ORDER BY user_id, created_at DESC, id DESC;

-- Without window functions (careful: returns several rows if two orders share the max timestamp)
SELECT o.* FROM orders o
JOIN (SELECT user_id, MAX(created_at) AS max_at FROM orders GROUP BY user_id) m
  ON m.user_id = o.user_id AND m.max_at = o.created_at;
```

---

### 3.4 Top product by revenue in each category ⭐⭐

**Question:** For each category, the product with the highest total revenue.

**SQL:**
```sql
WITH product_revenue AS (
  SELECT p.category, p.id, p.name, SUM(oi.quantity * oi.unit_price) AS revenue
  FROM order_items oi
  JOIN products p ON p.id = oi.product_id
  GROUP BY p.category, p.id, p.name
)
SELECT category, name, revenue
FROM (
  SELECT pr.*, RANK() OVER (PARTITION BY category ORDER BY revenue DESC) AS rnk
  FROM product_revenue pr
) t
WHERE rnk = 1;
```
Pattern: **aggregate first (CTE), then rank.** You can't put a window function over an aggregate in the same `WHERE`.

---

## 4. Running Calculations

**Priority:** ⭐ must-know

### 4.1 Running total ⭐

**Question:** For each user, show every order with the running total of their spending.

**SQL:**
```sql
SELECT user_id, id, created_at, total,
  SUM(total) OVER (
    PARTITION BY user_id
    ORDER BY created_at, id
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS running_total
FROM orders
ORDER BY user_id, created_at;
```
Why `ROWS ...` and the `id` tie-breaker: with the default `RANGE` frame, two orders with the same `created_at` would both show the sum including each other (peer rows), so the running total would jump.

**Without window functions (older MySQL, interview follow-up):**
```sql
SELECT o1.user_id, o1.id, o1.total,
  (SELECT SUM(o2.total) FROM orders o2
   WHERE o2.user_id = o1.user_id
     AND (o2.created_at, o2.id) <= (o1.created_at, o1.id)) AS running_total
FROM orders o1;
```
O(n²): fine for small data only.

---

### 4.2 7-day moving average of daily revenue ⭐

**SQL:**
```sql
WITH daily AS (
  SELECT DATE(created_at) AS day, SUM(total) AS revenue
  FROM orders
  GROUP BY DATE(created_at)
)
SELECT day, revenue,
  ROUND(AVG(revenue) OVER (ORDER BY day ROWS BETWEEN 6 PRECEDING AND CURRENT ROW), 2) AS moving_avg_7d
FROM daily
ORDER BY day;
```
**Trap:** `ROWS BETWEEN 6 PRECEDING` means 6 *rows*, not 6 *days*. If some days have no orders, the window covers more than 7 calendar days. Fix: fill missing days first (§10.2), or use `RANGE BETWEEN INTERVAL '6 days' PRECEDING AND CURRENT ROW` (PostgreSQL 11+).

---

### 4.3 Cumulative percentage (Pareto: which products make 80% of revenue?) ⭐⭐

**SQL:**
```sql
WITH product_revenue AS (
  SELECT product_id, SUM(quantity * unit_price) AS revenue
  FROM order_items
  GROUP BY product_id
),
cumulative AS (
  SELECT product_id, revenue,
    SUM(revenue) OVER (ORDER BY revenue DESC, product_id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
      / SUM(revenue) OVER () * 100 AS cumulative_pct
  FROM product_revenue
)
SELECT * FROM cumulative
WHERE cumulative_pct - (revenue / (SELECT SUM(revenue) FROM product_revenue) * 100) < 80
ORDER BY revenue DESC;
```
The condition keeps every product whose cumulative share *before* adding it was under 80%, so the product that crosses 80% is included. A simpler but slightly different answer is `WHERE cumulative_pct <= 80`.

`SUM(revenue) OVER ()` with an empty `OVER()` = grand total on every row.

---

### 4.4 Month-over-month revenue growth ⭐

**SQL:**
```sql
WITH monthly AS (
  SELECT DATE_TRUNC('month', created_at) AS month, SUM(total) AS revenue
  FROM orders
  WHERE status <> 'CANCELLED'
  GROUP BY DATE_TRUNC('month', created_at)
)
SELECT month, revenue,
  LAG(revenue) OVER (ORDER BY month) AS prev_revenue,
  ROUND(100.0 * (revenue - LAG(revenue) OVER (ORDER BY month))
        / NULLIF(LAG(revenue) OVER (ORDER BY month), 0), 2) AS mom_growth_pct
FROM monthly
ORDER BY month;
```
MySQL: `DATE_FORMAT(created_at, '%Y-%m-01')` instead of `DATE_TRUNC`.

**Year-over-year:** `LAG(revenue, 12) OVER (ORDER BY month)` (assumes no missing months), or join each month to the same month last year.

---

## 5. Hierarchies

**Priority:** ⭐⭐ commonly asked

### 5.1 Full reporting chain (all employees under a manager) ⭐⭐

**Question:** List everyone who reports (directly or indirectly) to employee 1, with their level.

**SQL:**
```sql
WITH RECURSIVE reports AS (
  SELECT id, name, manager_id, 0 AS level
  FROM employees
  WHERE id = 1                                   -- anchor

  UNION ALL

  SELECT e.id, e.name, e.manager_id, r.level + 1
  FROM employees e
  JOIN reports r ON e.manager_id = r.id          -- step down one level
  WHERE r.level < 20                             -- safety against cycles
)
SELECT * FROM reports ORDER BY level, name;
```

### 5.2 Path from an employee up to the CEO ⭐⭐

**SQL:**
```sql
WITH RECURSIVE chain AS (
  SELECT id, name, manager_id, name::TEXT AS path
  FROM employees WHERE id = 4                    -- start at Divya

  UNION ALL

  SELECT m.id, m.name, m.manager_id, c.path || ' → ' || m.name
  FROM employees m
  JOIN chain c ON m.id = c.manager_id            -- step UP to the manager
)
SELECT path FROM chain WHERE manager_id IS NULL; -- the row that reached the top
-- Divya → Chetan → Anil
```
MySQL: use `CAST(name AS CHAR(1000))` in the anchor and `CONCAT(c.path, ' → ', m.name)`.

### 5.3 Number of direct and total reports per manager ⭐⭐⭐

```sql
WITH RECURSIVE tree AS (
  SELECT id AS root_id, id AS emp_id FROM employees
  UNION ALL
  SELECT t.root_id, e.id
  FROM tree t JOIN employees e ON e.manager_id = t.emp_id
)
SELECT r.id, r.name,
  (SELECT COUNT(*) FROM employees d WHERE d.manager_id = r.id) AS direct_reports,
  COUNT(*) - 1 AS total_reports                   -- minus self
FROM tree t JOIN employees r ON r.id = t.root_id
GROUP BY r.id, r.name
ORDER BY total_reports DESC;
```

---

## 6. Gaps and Islands

**Priority:** ⭐⭐ commonly asked (looks hard, easy once you know the trick)

### 6.1 Longest login streak per user ⭐⭐

**Question:** Given `logins(user_id, login_date)` (possibly several logins per day), find each user's longest streak of consecutive days.

```
user 1: Nov 1, Nov 2, Nov 3, Nov 5, Nov 6   → streaks: 3 days, 2 days → longest 3
```

**The trick:** for consecutive dates, `date − row_number` is **constant** within a streak.
```
date    rn   date - rn
Nov 1    1   Oct 31    ┐
Nov 2    2   Oct 31    ├ same "group" → one island
Nov 3    3   Oct 31    ┘
Nov 5    4   Nov 1     ┐ new group
Nov 6    5   Nov 1     ┘
```

**SQL:**
```sql
WITH days AS (
  SELECT DISTINCT user_id, login_date::date AS d       -- one row per user per day
  FROM logins
),
grouped AS (
  SELECT user_id, d,
    d - (ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY d))::int AS grp
  FROM days
),
streaks AS (
  SELECT user_id, grp, MIN(d) AS start_date, MAX(d) AS end_date, COUNT(*) AS length
  FROM grouped
  GROUP BY user_id, grp
)
SELECT DISTINCT ON (user_id) user_id, start_date, end_date, length
FROM streaks
ORDER BY user_id, length DESC, start_date DESC;
```
MySQL: `DATE_SUB(d, INTERVAL ROW_NUMBER() OVER (...) DAY)` for the group key; replace `DISTINCT ON` with a `ROW_NUMBER()` filter.

---

### 6.2 Three or more consecutive rows with the same value ⭐⭐

**Question (LeetCode "Consecutive Numbers"):** `logs(id, num)` with consecutive ids. Find numbers that appear at least 3 times in a row.

**Approach 1: LAG/LEAD**
```sql
SELECT DISTINCT num
FROM (
  SELECT num,
    LAG(num)  OVER (ORDER BY id) AS prev_num,
    LEAD(num) OVER (ORDER BY id) AS next_num
  FROM logs
) t
WHERE num = prev_num AND num = next_num;
```

**Approach 2: islands (generalizes to "at least N")**
```sql
SELECT DISTINCT num
FROM (
  SELECT num,
    ROW_NUMBER() OVER (ORDER BY id) - ROW_NUMBER() OVER (PARTITION BY num ORDER BY id) AS grp
  FROM logs
) t
GROUP BY num, grp
HAVING COUNT(*) >= 3;
```
(If `id` can have gaps, use `ROW_NUMBER() OVER (ORDER BY id)` instead of `id` itself for the "position".)

---

### 6.3 Find missing IDs (gaps) ⭐⭐

**Question:** Invoice numbers should be continuous. Find the missing ranges.

**SQL:**
```sql
SELECT id + 1 AS gap_start, next_id - 1 AS gap_end
FROM (
  SELECT id, LEAD(id) OVER (ORDER BY id) AS next_id
  FROM invoices
) t
WHERE next_id - id > 1;
```

**Alternative (list every missing number, PostgreSQL):**
```sql
SELECT s.n AS missing_id
FROM generate_series((SELECT MIN(id) FROM invoices), (SELECT MAX(id) FROM invoices)) AS s(n)
LEFT JOIN invoices i ON i.id = s.n
WHERE i.id IS NULL;
```

---

## 7. Pivoting

**Priority:** ⭐⭐ commonly asked

### 7.1 Rows to columns ⭐⭐

**Question:** Monthly revenue per category as columns.

```
Want:
month    | Mobiles | Laptops | Audio
2026-09  | 120000  | 450000  | 30000
```

**SQL (conditional aggregation, works everywhere):**
```sql
SELECT
  DATE_TRUNC('month', o.created_at) AS month,
  SUM(CASE WHEN p.category = 'Mobiles' THEN oi.quantity * oi.unit_price ELSE 0 END) AS mobiles,
  SUM(CASE WHEN p.category = 'Laptops' THEN oi.quantity * oi.unit_price ELSE 0 END) AS laptops,
  SUM(CASE WHEN p.category = 'Audio'   THEN oi.quantity * oi.unit_price ELSE 0 END) AS audio
FROM orders o
JOIN order_items oi ON oi.order_id = o.id
JOIN products p ON p.id = oi.product_id
GROUP BY DATE_TRUNC('month', o.created_at)
ORDER BY month;
```
Columns must be known in advance. For a **dynamic** list of categories, build the SQL in application code (or with dynamic SQL in a procedure). PostgreSQL's `crosstab()` (tablefunc extension) and SQL Server's `PIVOT` are alternatives.

---

### 7.2 Columns to rows ⭐⭐

**Question:** `quarterly_sales(product_id, q1, q2, q3, q4)` → one row per product per quarter.

**SQL (portable):**
```sql
SELECT product_id, 'Q1' AS quarter, q1 AS sales FROM quarterly_sales
UNION ALL
SELECT product_id, 'Q2', q2 FROM quarterly_sales
UNION ALL
SELECT product_id, 'Q3', q3 FROM quarterly_sales
UNION ALL
SELECT product_id, 'Q4', q4 FROM quarterly_sales;
```

**Alternative (PostgreSQL, single scan):**
```sql
SELECT qs.product_id, v.quarter, v.sales
FROM quarterly_sales qs
CROSS JOIN LATERAL (VALUES ('Q1', qs.q1), ('Q2', qs.q2), ('Q3', qs.q3), ('Q4', qs.q4)) AS v(quarter, sales);
```

---

## 8. Statistics

**Priority:** ⭐⭐ commonly asked

### 8.1 Median ⭐⭐

**Question:** Median order value overall, and per user.

**SQL (PostgreSQL):**
```sql
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY total) AS median_total FROM orders;

SELECT user_id, PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY total) AS median_total
FROM orders GROUP BY user_id;
```
- `PERCENTILE_CONT`: interpolates (median of 10, 20 = 15).
- `PERCENTILE_DISC`: returns an actual value from the data (10).

**Portable version (MySQL has no PERCENTILE_CONT as an aggregate):**
```sql
SELECT AVG(total) AS median_total
FROM (
  SELECT total,
    ROW_NUMBER() OVER (ORDER BY total) AS rn,
    COUNT(*) OVER () AS cnt
  FROM orders
) t
WHERE rn IN (FLOOR((cnt + 1) / 2.0), CEIL((cnt + 1) / 2.0));
```
(The `2.0` matters in PostgreSQL: with integer division `(cnt + 1) / 2` is already rounded down, so `CEIL` would pick the same row twice for even counts.)
For odd `cnt` both expressions give the same middle row; for even `cnt` they give the two middle rows, which are averaged. Add `PARTITION BY user_id` to both window functions for per-user medians.

### 8.2 Percentiles and buckets ⭐⭐

```sql
-- p90 and p99 delivery time in hours
SELECT
  PERCENTILE_CONT(0.9)  WITHIN GROUP (ORDER BY delivery_hours) AS p90,
  PERCENTILE_CONT(0.99) WITHIN GROUP (ORDER BY delivery_hours) AS p99
FROM deliveries;

-- Split customers into 4 spending quartiles
SELECT user_id, total_spent, NTILE(4) OVER (ORDER BY total_spent DESC) AS quartile
FROM (SELECT user_id, SUM(total) AS total_spent FROM orders GROUP BY user_id) t;

-- Histogram: order counts by value band
SELECT
  CASE WHEN total < 500 THEN '0-499'
       WHEN total < 2000 THEN '500-1999'
       WHEN total < 10000 THEN '2000-9999'
       ELSE '10000+' END AS band,
  COUNT(*)
FROM orders
GROUP BY 1
ORDER BY MIN(total);
```

---

## 9. Product Analytics

**Priority:** ⭐⭐ commonly asked (very common at product companies)

### 9.1 Day-1 retention ⭐⭐

**Question:** For each signup date, what % of users came back the day after signing up?

**Tables:** `users(id, created_at)`, `events(user_id, event_at)`

**Approach:** find the users who had any event on the day after signup, then LEFT JOIN all users to that set and count per signup date.

**SQL:**
```sql
WITH returned AS (
  SELECT DISTINCT e.user_id
  FROM events e JOIN users u ON u.id = e.user_id
  WHERE DATE(e.event_at) = DATE(u.created_at) + 1
)
SELECT DATE(u.created_at) AS signup_date,
  COUNT(*) AS signups,
  COUNT(r.user_id) AS returned_day1,
  ROUND(100.0 * COUNT(r.user_id) / COUNT(*), 2) AS day1_retention_pct
FROM users u
LEFT JOIN returned r ON r.user_id = u.id
GROUP BY DATE(u.created_at)
ORDER BY signup_date;
```

---

### 9.2 Monthly cohort retention ⭐⭐⭐

**Question:** Group customers by the month of their first order (cohort). For each cohort, what % ordered again in month 1, 2, 3... after their first month?

**SQL:**
```sql
WITH first_order AS (
  SELECT user_id, DATE_TRUNC('month', MIN(created_at)) AS cohort_month
  FROM orders GROUP BY user_id
),
activity AS (
  SELECT DISTINCT o.user_id, f.cohort_month,
    (EXTRACT(YEAR FROM o.created_at) - EXTRACT(YEAR FROM f.cohort_month)) * 12
    + (EXTRACT(MONTH FROM o.created_at) - EXTRACT(MONTH FROM f.cohort_month)) AS month_number
  FROM orders o JOIN first_order f ON f.user_id = o.user_id
),
cohort_size AS (
  SELECT cohort_month, COUNT(*) AS users FROM first_order GROUP BY cohort_month
)
SELECT a.cohort_month, a.month_number, COUNT(*) AS active_users,
  ROUND(100.0 * COUNT(*) / c.users, 1) AS retention_pct
FROM activity a JOIN cohort_size c ON c.cohort_month = a.cohort_month
GROUP BY a.cohort_month, a.month_number, c.users
ORDER BY a.cohort_month, a.month_number;
```
Result rows `(cohort, month_number, retention_pct)` are usually pivoted into the familiar triangle chart (§7.1).

---

### 9.3 Funnel conversion ⭐⭐

**Question:** From `events(user_id, event_name, event_at)` with events `view_product → add_to_cart → checkout → purchase`, how many users reached each step **in order**, and what's the conversion between steps?

**SQL:**
```sql
WITH steps AS (
  SELECT user_id,
    MIN(event_at) FILTER (WHERE event_name = 'view_product') AS viewed_at,
    MIN(event_at) FILTER (WHERE event_name = 'add_to_cart')  AS carted_at,
    MIN(event_at) FILTER (WHERE event_name = 'checkout')     AS checkout_at,
    MIN(event_at) FILTER (WHERE event_name = 'purchase')     AS purchased_at
  FROM events
  GROUP BY user_id
),
funnel AS (
  SELECT
    COUNT(viewed_at) AS viewed,
    COUNT(*) FILTER (WHERE carted_at   > viewed_at) AS carted,
    COUNT(*) FILTER (WHERE checkout_at > carted_at   AND carted_at > viewed_at) AS checked_out,
    COUNT(*) FILTER (WHERE purchased_at > checkout_at AND checkout_at > carted_at AND carted_at > viewed_at) AS purchased
  FROM steps
)
SELECT *,
  ROUND(100.0 * carted / NULLIF(viewed, 0), 1)          AS view_to_cart_pct,
  ROUND(100.0 * checked_out / NULLIF(carted, 0), 1)     AS cart_to_checkout_pct,
  ROUND(100.0 * purchased / NULLIF(checked_out, 0), 1)  AS checkout_to_purchase_pct
FROM funnel;
```
Simplification: uses each user's *first* time at each step. Real funnels are often per session and within a time window.

---

### 9.4 Sessionization ⭐⭐⭐

**Question:** A new session starts when a user is inactive for more than 30 minutes. Assign a session number to each event and compute session lengths.

**Approach:** flag events where the gap from the previous event > 30 min (`LAG`), then a running `SUM` of the flags gives the session number.

**SQL:**
```sql
WITH flagged AS (
  SELECT user_id, event_at,
    CASE
      WHEN LAG(event_at) OVER (PARTITION BY user_id ORDER BY event_at) IS NULL
        OR event_at - LAG(event_at) OVER (PARTITION BY user_id ORDER BY event_at) > INTERVAL '30 minutes'
      THEN 1 ELSE 0
    END AS is_new_session
  FROM events
),
sessions AS (
  SELECT user_id, event_at,
    SUM(is_new_session) OVER (PARTITION BY user_id ORDER BY event_at
                              ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS session_no
  FROM flagged
)
SELECT user_id, session_no,
  MIN(event_at) AS session_start, MAX(event_at) AS session_end,
  MAX(event_at) - MIN(event_at) AS duration, COUNT(*) AS events
FROM sessions
GROUP BY user_id, session_no
ORDER BY user_id, session_no;
```
The **"flag + running sum"** trick is reusable for any "start a new group when a condition happens" problem.

---

## 10. Advanced Patterns

**Priority:** ⭐⭐⭐ senior / deep dive

### 10.1 Relational division: customers who bought every product in a category ⭐⭐⭐

**Question:** Users who have bought **all** products in the `Kitchen` category.

**Approach 1: count comparison**
```sql
SELECT o.user_id
FROM orders o
JOIN order_items oi ON oi.order_id = o.id
JOIN products p ON p.id = oi.product_id
WHERE p.category = 'Kitchen'
GROUP BY o.user_id
HAVING COUNT(DISTINCT p.id) = (SELECT COUNT(*) FROM products WHERE category = 'Kitchen');
```

**Approach 2: double NOT EXISTS ("there is no Kitchen product this user hasn't bought")**
```sql
SELECT u.id
FROM users u
WHERE NOT EXISTS (
  SELECT 1 FROM products p
  WHERE p.category = 'Kitchen'
    AND NOT EXISTS (
      SELECT 1 FROM orders o JOIN order_items oi ON oi.order_id = o.id
      WHERE o.user_id = u.id AND oi.product_id = p.id
    )
);
```
Edge case: if the category has **zero** products, approach 2 returns every user (vacuous truth: "bought all of nothing"), while approach 1 returns no one. Mention it.

---

### 10.2 Daily revenue including days with no orders ⭐⭐

**Question:** Revenue for every day of November 2026, showing 0 for days without orders.

**SQL (PostgreSQL):**
```sql
SELECT d::date AS day, COALESCE(SUM(o.total), 0) AS revenue
FROM generate_series('2026-11-01'::date, '2026-11-30'::date, INTERVAL '1 day') AS d
LEFT JOIN orders o
  ON o.created_at >= d AND o.created_at < d + INTERVAL '1 day'
GROUP BY d
ORDER BY d;
```

**MySQL (recursive CTE for the dates):**
```sql
WITH RECURSIVE days AS (
  SELECT DATE '2026-11-01' AS d
  UNION ALL
  SELECT d + INTERVAL 1 DAY FROM days WHERE d < '2026-11-30'
)
SELECT days.d, COALESCE(SUM(o.total), 0) AS revenue
FROM days
LEFT JOIN orders o ON o.created_at >= days.d AND o.created_at < days.d + INTERVAL 1 DAY
GROUP BY days.d
ORDER BY days.d;
```
Many companies keep a permanent **calendar/date dimension table** for this.

**Note:** the join uses a range on `created_at` rather than `DATE(o.created_at) = d`, so an index on `created_at` can be used.

---

### 10.3 Mutual friends and friend recommendations ⭐⭐⭐

**Table:** `friendships(user_id, friend_id)`, stored in **both directions** (if 1–2 are friends, both `(1,2)` and `(2,1)` exist).

**Mutual friends of users 1 and 2:**
```sql
SELECT f1.friend_id AS mutual_friend
FROM friendships f1
JOIN friendships f2 ON f1.friend_id = f2.friend_id
WHERE f1.user_id = 1 AND f2.user_id = 2;
-- or: SELECT friend_id FROM friendships WHERE user_id = 1
--     INTERSECT
--     SELECT friend_id FROM friendships WHERE user_id = 2;
```

**"People you may know" for user 1 (friends of friends, ranked by mutual count):**
```sql
SELECT f2.friend_id AS suggested_user, COUNT(*) AS mutual_friends
FROM friendships f1
JOIN friendships f2 ON f2.user_id = f1.friend_id      -- friends of my friends
WHERE f1.user_id = 1
  AND f2.friend_id <> 1                                -- not me
  AND NOT EXISTS (                                     -- not already my friend
    SELECT 1 FROM friendships x WHERE x.user_id = 1 AND x.friend_id = f2.friend_id
  )
GROUP BY f2.friend_id
ORDER BY mutual_friends DESC
LIMIT 10;
```
If friendships are stored **once** (only `user_id < friend_id`), first normalize with `SELECT user_id, friend_id FROM friendships UNION ALL SELECT friend_id, user_id FROM friendships`.

Follow-up: at social-network scale this is a job for a graph database or precomputed recommendations (file 03 §12.3).

---

### 10.4 Year-over-year comparison by joining a table to itself ⭐⭐

```sql
WITH monthly AS (
  SELECT DATE_TRUNC('month', created_at) AS month, SUM(total) AS revenue
  FROM orders GROUP BY 1
)
SELECT cur.month, cur.revenue, prev.revenue AS revenue_last_year,
  ROUND(100.0 * (cur.revenue - prev.revenue) / NULLIF(prev.revenue, 0), 2) AS yoy_pct
FROM monthly cur
LEFT JOIN monthly prev ON prev.month = cur.month - INTERVAL '1 year'
ORDER BY cur.month;
```
Safer than `LAG(revenue, 12)`, which silently compares the wrong months if any month is missing.

---

# PART 2: MONGODB

All examples use the ShopKart collections from file 03 (`users`, `products`, `orders` with embedded `items`).

---

## 11. Basic Queries

**Priority:** ⭐ must-know

### 11.1 Filter, project, sort, limit ⭐

**Question:** The 5 cheapest in-stock Samsung phones under ₹50,000: name and price only.

```javascript
db.products.find(
  { category: "Mobiles", "attributes.brand": "Samsung", price: { $lt: 50000 }, stock: { $gt: 0 } },
  { _id: 0, name: 1, price: 1 }
).sort({ price: 1 }).limit(5);
```
**Index:** `{ category: 1, "attributes.brand": 1, price: 1 }` (equality, equality, then sort/range on price per the ESR rule). `stock > 0` is filtered after the index scan.

### 11.2 Orders containing a product with quantity ≥ 2 ⭐

```javascript
db.orders.find({ items: { $elemMatch: { productId: 100, qty: { $gte: 2 } } } });
```
Without `$elemMatch`, the two conditions could match different items (file 03 §3.3).

### 11.3 Users with no address saved ⭐

```javascript
db.users.find({ $or: [ { addresses: { $exists: false } }, { addresses: { $size: 0 } } ] });
```

### 11.4 Orders in the last 7 days with total above ₹10,000, newest first ⭐

```javascript
db.orders.find({
  createdAt: { $gte: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000) },
  total: { $gt: 10000 }
}).sort({ createdAt: -1 });
```

---

## 12. Aggregation

**Priority:** ⭐ must-know

### 12.1 Order count and revenue per status ⭐

```javascript
db.orders.aggregate([
  { $group: { _id: "$status", orders: { $sum: 1 }, revenue: { $sum: "$total" } } },
  { $sort: { revenue: -1 } }
]);
```
SQL: `SELECT status, COUNT(*), SUM(total) FROM orders GROUP BY status ORDER BY 3 DESC;`

### 12.2 Users who spent more than ₹1,00,000 (HAVING) ⭐

```javascript
db.orders.aggregate([
  { $match: { status: { $ne: "CANCELLED" } } },            // WHERE
  { $group: { _id: "$userId", spent: { $sum: "$total" }, orders: { $sum: 1 } } },
  { $match: { spent: { $gt: 100000 } } },                   // HAVING
  { $sort: { spent: -1 } }
]);
```

### 12.3 Top 3 best-selling products (by quantity) ⭐

```javascript
db.orders.aggregate([
  { $match: { status: { $in: ["PAID", "SHIPPED", "DELIVERED"] } } },
  { $unwind: "$items" },
  { $group: { _id: "$items.productId", name: { $first: "$items.name" }, unitsSold: { $sum: "$items.qty" } } },
  { $sort: { unitsSold: -1 } },
  { $limit: 3 }
]);
```

### 12.4 Top 2 most expensive products per category ⭐⭐

```javascript
// MongoDB 5.2+
db.products.aggregate([
  { $group: {
      _id: "$category",
      top2: { $topN: { n: 2, sortBy: { price: -1 }, output: { name: "$name", price: "$price" } } }
  } }
]);

// Older versions: sort, group with $push, then slice
db.products.aggregate([
  { $sort: { category: 1, price: -1 } },
  { $group: { _id: "$category", products: { $push: { name: "$name", price: "$price" } } } },
  { $project: { top2: { $slice: ["$products", 2] } } }
]);
```

### 12.5 Join: orders with customer name and city ($lookup) ⭐

```javascript
db.orders.aggregate([
  { $match: { status: "SHIPPED" } },
  { $lookup: { from: "users", localField: "userId", foreignField: "_id", as: "user" } },
  { $unwind: "$user" },                                     // turns it into an inner join
  { $project: { _id: 1, total: 1, customer: "$user.name", city: "$user.city" } }
]);
```

### 12.6 Users who never ordered (anti-join) ⭐⭐

```javascript
db.users.aggregate([
  { $lookup: {
      from: "orders",
      let: { uid: "$_id" },
      pipeline: [ { $match: { $expr: { $eq: ["$userId", "$$uid"] } } }, { $limit: 1 } ],
      as: "anyOrder" } },
  { $match: { anyOrder: { $size: 0 } } },
  { $project: { name: 1, email: 1 } }
]);
```
Index `orders.userId` so each lookup is an index probe.

### 12.7 Revenue per day for a month ⭐

```javascript
db.orders.aggregate([
  { $match: { createdAt: { $gte: ISODate("2026-11-01"), $lt: ISODate("2026-12-01") }, status: { $ne: "CANCELLED" } } },
  { $group: {
      _id: { $dateTrunc: { date: "$createdAt", unit: "day", timezone: "Asia/Kolkata" } },   // 5.0+
      revenue: { $sum: "$total" }, orders: { $sum: 1 } } },
  { $sort: { _id: 1 } }
]);
```
Note the **time zone**: grouping by UTC day would put orders placed between midnight and 05:30 IST into the previous day. Older versions: `$dateToString: { format: "%Y-%m-%d", date: "$createdAt", timezone: "Asia/Kolkata" }`.

### 12.8 Find duplicate emails ⭐

```javascript
db.users.aggregate([
  { $group: { _id: { $toLower: "$email" }, count: { $sum: 1 }, ids: { $push: "$_id" } } },
  { $match: { count: { $gt: 1 } } }
]);
// Delete all but the first: for each group, deleteMany({ _id: { $in: ids.slice(1) } })
```

---

## 13. Advanced Aggregation

**Priority:** ⭐⭐ commonly asked

### 13.1 Running total of each user's spending ($setWindowFields) ⭐⭐

```javascript
db.orders.aggregate([
  { $setWindowFields: {
      partitionBy: "$userId",
      sortBy: { createdAt: 1 },
      output: {
        runningTotal: { $sum: "$total", window: { documents: ["unbounded", "current"] } },
        orderNumber:  { $documentNumber: {} }        // like ROW_NUMBER()
      } } },
  { $project: { userId: 1, createdAt: 1, total: 1, runningTotal: 1, orderNumber: 1 } }
]);
```

### 13.2 Latest order per user ⭐⭐

```javascript
db.orders.aggregate([
  { $sort: { userId: 1, createdAt: -1 } },
  { $group: { _id: "$userId", latestOrder: { $first: "$$ROOT" } } },
  { $replaceRoot: { newRoot: "$latestOrder" } }
]);
```
With an index on `{ userId: 1, createdAt: -1 }`, the `$sort` + `$group` with `$first` can be optimized to use the index (a "distinct scan").

### 13.3 Product listing page with facets ($facet) ⭐⭐

```javascript
db.products.aggregate([
  { $match: { category: "Mobiles", stock: { $gt: 0 } } },
  { $facet: {
      results: [ { $sort: { price: 1 } }, { $skip: 0 }, { $limit: 20 }, { $project: { name: 1, price: 1 } } ],
      brands:  [ { $group: { _id: "$attributes.brand", count: { $sum: 1 } } }, { $sort: { count: -1 } } ],
      priceBands: [ { $bucket: { groupBy: "$price", boundaries: [0, 15000, 30000, 60000, 1000000],
                                  default: "Other", output: { count: { $sum: 1 } } } } ],
      total: [ { $count: "count" } ]
  } }
]);
```

### 13.4 Month-over-month growth ⭐⭐

```javascript
db.orders.aggregate([
  { $match: { status: { $ne: "CANCELLED" } } },
  { $group: { _id: { $dateTrunc: { date: "$createdAt", unit: "month" } }, revenue: { $sum: "$total" } } },
  { $sort: { _id: 1 } },
  { $setWindowFields: {
      sortBy: { _id: 1 },
      output: { prevRevenue: { $shift: { output: "$revenue", by: -1 } } }   // like LAG()
  } },
  { $project: {
      month: "$_id", revenue: 1, prevRevenue: 1,
      momGrowthPct: { $cond: [
        { $gt: ["$prevRevenue", 0] },
        { $round: [ { $multiply: [ { $divide: [ { $subtract: ["$revenue", "$prevRevenue"] }, "$prevRevenue" ] }, 100 ] }, 2 ] },
        null ] }
  } }
]);
```

### 13.5 Customers who bought every Kitchen product (division) ⭐⭐⭐

```javascript
const kitchenIds = db.products.distinct("_id", { category: "Kitchen" });

db.orders.aggregate([
  { $unwind: "$items" },
  { $match: { "items.productId": { $in: kitchenIds } } },
  { $group: { _id: "$userId", bought: { $addToSet: "$items.productId" } } },
  { $match: { $expr: { $eq: [ { $size: "$bought" }, kitchenIds.length ] } } }
]);
```

### 13.6 Save results as a materialized view ($merge) ⭐⭐

```javascript
// Nightly job: refresh daily sales summary
db.orders.aggregate([
  { $match: { createdAt: { $gte: ISODate("2026-11-07"), $lt: ISODate("2026-11-08") } } },
  { $group: { _id: { $dateTrunc: { date: "$createdAt", unit: "day" } }, revenue: { $sum: "$total" }, orders: { $sum: 1 } } },
  { $merge: { into: "daily_sales", on: "_id", whenMatched: "replace", whenNotMatched: "insert" } }
]);
```
Dashboards then read the small `daily_sales` collection instead of aggregating millions of orders on every request.
