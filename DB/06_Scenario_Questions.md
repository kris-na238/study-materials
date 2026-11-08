# Scenario and Troubleshooting Questions (Backend)

Open-ended "what would you do?" questions that combine several concepts. Interviewers use them to see whether you can **apply** what's in files 01–04 under realistic pressure.

Study these **after** files 01–04. Each scenario links back to the concepts it uses.

**How to answer any scenario question (the structure interviewers look for):**
1. **Clarify** before solving: scale, what changed recently, which database, how urgent.
2. **Stabilise first** if it's a live incident: stop the bleeding (kill a query, roll back a deploy, shed load), *then* find the root cause.
3. **Investigate with evidence**: name the specific tools and metrics you'd check.
4. **List likely causes**, most probable first.
5. **Fix short-term → long-term**, and explain how you'd **prevent it** happening again (monitoring, alerts, tests, process).

Priority: ⭐ must-know · ⭐⭐ commonly asked · ⭐⭐⭐ senior / deep dive

---

## Table of Contents

[1. Performance](#1-performance) · [2. Concurrency](#2-concurrency) · [3. Connections and Load](#3-connections-and-load) · [4. Replication and Consistency](#4-replication-and-consistency) · [5. Caching](#5-caching) · [6. Schema and Data Changes](#6-schema-and-data-changes) · [7. Scaling](#7-scaling) · [8. Messaging and Pipelines](#8-messaging-and-pipelines) · [9. Reliability and Recovery](#9-reliability-and-recovery)

---

# 1. Performance

### 1.1 A query that used to be fast is now slow in production ⭐

**Clarifying questions:**
- Slow always, or only sometimes / at certain times?
- What changed: a deploy, data volume, a new index, a migration, a database upgrade?
- One query or everything on the database?

**How to investigate:**
1. Find it and confirm: `pg_stat_statements` / MySQL slow log or `sys.statement_analysis`; APM traces.
2. Get the **actual** plan: `EXPLAIN (ANALYZE, BUFFERS)` in PostgreSQL, `EXPLAIN ANALYZE` in MySQL, with the real parameter values.
3. Compare **estimated vs actual rows** at each plan node.
4. Check whether it's waiting rather than working: lock waits (`pg_stat_activity.wait_event`, `sys.innodb_lock_waits`), I/O (`Buffers: read` high), CPU.
5. Check system health: CPU, disk I/O, buffer pool hit ratio, replication, autovacuum activity.

**Likely causes:**
- **Data growth:** a plan that was fine at 10K rows (nested loop, full scan) is terrible at 10M.
- **Plan change** from stale or skewed statistics (estimated 10 rows, actual 2M).
- A dropped or unused index; a new query shape that can't use the index (a function on a column, a type mismatch, `LIKE '%x'`).
- **Table/index bloat** (PostgreSQL dead tuples) or fragmented indexes.
- **Lock contention** from a new job or long transaction.
- Working set no longer fits in memory → disk reads.
- Parameter-sensitive plan: plan cached for a common value, now used for a rare one (or vice versa).

**Fixes:**
- Short-term: `ANALYZE` the table; add the right index (`CREATE INDEX CONCURRENTLY`); kill the blocking session; roll back the deploy that changed the query.
- Long-term: rewrite the query (keyset pagination, remove functions on columns, select fewer columns), tune autovacuum, partition or archive old data, cache the result, add **plan regression monitoring** and slow-query alerts.

**Concepts used:** [01 §6 Indexing](01_DB_Fundamentals.md#6-indexing), [01 §7 Query Processing](01_DB_Fundamentals.md#7-query-processing-and-optimization), [02 §11 EXPLAIN](02_SQL_Databases.md#11-reading-explain), [02 §13 VACUUM](02_SQL_Databases.md#13-mvcc-implementation)

---

### 1.2 An API endpoint makes hundreds of queries per request ⭐

**How to investigate:** enable ORM query logging or look at the APM trace for the request: the same `SELECT ... WHERE id = ?` repeated with different IDs is the N+1 signature.

**Likely cause:** lazy-loading a relation inside a loop (orders → user, orders → items → product), or a GraphQL resolver per item.

**Fixes:**
- Eager load: Prisma `include`, Sequelize/TypeORM `include`/`relations`, Django `select_related`/`prefetch_related`, SQLAlchemy `joinedload`/`selectinload`.
- Batch with `WHERE id IN (...)`; DataLoader for GraphQL.
- For list screens, a single purpose-built query (JOIN + selected columns), possibly cached.
- Prevent: query-count assertions in tests (e.g. Django `assertNumQueries`), APM alerts on queries-per-request.

**Concepts used:** [01 §9 ORMs](01_DB_Fundamentals.md#9-orms-and-the-application-layer)

---

### 1.3 Pagination gets slower on later pages ⭐

**Cause:** `LIMIT 20 OFFSET 100000` reads and discards 100,000 rows before returning 20. Cost grows linearly with page number. Also causes duplicates/skips when data changes between page loads.

**Fixes:**
- **Keyset (cursor) pagination:** `WHERE (created_at, id) < (:last_created_at, :last_id) ORDER BY created_at DESC, id DESC LIMIT 20` with an index on `(created_at, id)`. Every page costs the same. Return an opaque cursor to the client.
- If users really need page numbers: cap the maximum page (search engines do), or use a deferred join (paginate over the narrow index first, then join for full rows).
- Approximate total counts instead of exact `COUNT(*)`.

**Concepts used:** [01 §7.5](01_DB_Fundamentals.md#7-query-processing-and-optimization)

---

### 1.4 `COUNT(*)` on a large table is too slow ⭐⭐

**Why it's slow:** in MVCC databases (PostgreSQL, InnoDB), there's no single row count; each row's visibility to your transaction must be checked, so the database scans a whole index or the table.

**Fixes, depending on the need:**
- **Approximate is fine** (dashboards, "about 1.2M results"): PostgreSQL `SELECT reltuples FROM pg_class WHERE relname = 'orders'`; MySQL `information_schema.TABLES.TABLE_ROWS`; or `EXPLAIN` row estimates.
- **Exact, frequently needed:** maintain a counter (a counter table updated in the same transaction, or Redis `INCR`), accepting write contention on hot counters (shard the counter into N rows and sum them).
- **Filtered counts:** make sure an index covers the filter so an index-only scan is possible (PostgreSQL also needs a well-vacuumed visibility map).
- **UI:** show "10,000+" instead of the exact number.

**Concepts used:** [01 §7.5](01_DB_Fundamentals.md#7-query-processing-and-optimization), [01 §13 MVCC](01_DB_Fundamentals.md#13-mvcc)

---

### 1.5 I added an index, but the query didn't get faster ⭐

**Check with EXPLAIN whether the index is used at all.** If not:
- Function or cast on the column (`WHERE DATE(created_at) = ...`, `WHERE phone = 98765` on a VARCHAR).
- Leading wildcard `LIKE '%term'`.
- Composite index in the wrong order (leftmost-prefix rule), or a range column before the column you need.
- Low selectivity: the optimizer correctly prefers a full scan when the query returns a large share of the table.
- Stale statistics → run `ANALYZE`.
- `OR` across different columns.
- (PostgreSQL) the index build failed and left an `INVALID` index after `CREATE INDEX CONCURRENTLY`.

**If it is used but still slow:**
- The query fetches too many rows anyway (needs pagination or aggregation).
- Many table lookups → make it a covering index.
- An expensive sort or join elsewhere in the plan.
- The real bottleneck isn't the query: lock waits, network, N+1, connection pool waits.

**Concepts used:** [01 §6.7](01_DB_Fundamentals.md#6-indexing)

---

### 1.6 Writes are slow on a table with many indexes ⭐⭐

**Cause:** every insert/update must update every index (and in PostgreSQL, non-HOT updates add entries to *every* index). Twelve indexes ≈ thirteen structures to modify per write, plus more WAL and replication traffic.

**Investigate:** list indexes and their usage (`pg_stat_user_indexes.idx_scan`, MySQL `sys.schema_unused_indexes`, `sys.schema_redundant_indexes`).

**Fixes:**
- Drop unused and redundant indexes (an index on `(a)` is redundant if `(a, b)` exists). Test first with MySQL invisible indexes / MongoDB hidden indexes.
- Combine several single-column indexes into a few composite ones.
- PostgreSQL: avoid indexing frequently updated columns (prevents HOT updates); lower `fillfactor`.
- Batch writes (multi-row inserts, `COPY`), and use sequential primary keys.
- Move analytics-only indexes to a replica or warehouse.

**Concepts used:** [01 §6](01_DB_Fundamentals.md#6-indexing), [02 §13.4 HOT](02_SQL_Databases.md#13-mvcc-implementation)

---

# 2. Concurrency

### 2.1 Two users booked the same seat (double booking) ⭐

**Root cause:** a check-then-act race:
```
Request A: SELECT status FROM seats WHERE id = 42   → AVAILABLE
Request B: SELECT status FROM seats WHERE id = 42   → AVAILABLE
Request A: UPDATE seats SET status = 'BOOKED' ...
Request B: UPDATE seats SET status = 'BOOKED' ...   → both "succeed"
```

**Fixes (pick one, best first):**
1. **Atomic conditional update:** `UPDATE seats SET status='HELD', held_by=:user WHERE id=42 AND status='AVAILABLE'` → check rows affected = 1.
2. **Unique constraint:** `bookings` with `UNIQUE (show_id, seat_id)` so the second insert fails.
3. **Pessimistic lock:** `SELECT ... FOR UPDATE` on the seat row inside a transaction.
4. **Optimistic lock:** version column.
5. Hotels/date ranges: PostgreSQL **exclusion constraint** on `(room_id, daterange)`.

Also: a hold with expiry (`held_until`) during payment, idempotency keys on the booking API, and a background job releasing expired holds.

**Concepts used:** [01 §11.4](01_DB_Fundamentals.md#11-isolation-levels), [01 §12](01_DB_Fundamentals.md#12-concurrency-control-and-locking), [01 §22.2](01_DB_Fundamentals.md#22-applied-design)

---

### 2.2 An account balance went wrong under concurrent updates (lost update) ⭐

**Root cause:** read-modify-write in application code:
```javascript
const acc = await db.query('SELECT balance FROM accounts WHERE id = $1', [id]);
await db.query('UPDATE accounts SET balance = $1 WHERE id = $2', [acc.balance - 100, id]);
// two concurrent requests both read 500 → both write 400 → one debit lost
```

**Fixes:**
- **Atomic update in SQL:** `UPDATE accounts SET balance = balance - 100 WHERE id = $1 AND balance >= 100` (check rows affected).
- `SELECT ... FOR UPDATE` before computing, within one transaction.
- Optimistic version column, with retry.
- `CHECK (balance >= 0)` constraint as a safety net.
- For money: a **ledger** (append-only entries) instead of mutating a balance directly; idempotency keys on every transfer.

**Concepts used:** [01 §11.1 lost update](01_DB_Fundamentals.md#11-isolation-levels), [01 §22.5 ledger](01_DB_Fundamentals.md#22-applied-design)

---

### 2.3 The application is getting deadlocks ⭐

**How to investigate:**
- MySQL: `SHOW ENGINE INNODB STATUS` → "LATEST DETECTED DEADLOCK"; enable `innodb_print_all_deadlocks`.
- PostgreSQL: the server log prints both statements and the lock cycle; `log_lock_waits = on`.
- Identify the two statements and the **order** in which each transaction locks rows.

**Likely causes:**
- Two code paths lock the same rows in **opposite order** (transfer A→B and B→A).
- Missing index → an UPDATE/DELETE scans and locks many rows (and gaps in InnoDB RR).
- InnoDB **gap locks** on concurrent inserts into nearby ranges (REPEATABLE READ).
- Long transactions holding locks while doing other work.
- Foreign key checks taking shared locks on parent rows.

**Fixes:**
- **Lock in a consistent order** (e.g. `ORDER BY id FOR UPDATE`).
- Add indexes so only the needed rows are locked.
- Keep transactions short; no network calls inside.
- Consider READ COMMITTED in MySQL to reduce gap locking.
- **Retry** on deadlock errors (MySQL 1213, PostgreSQL 40P01) with a small randomized backoff; deadlocks can't be eliminated completely.

**Concepts used:** [01 §12.3](01_DB_Fundamentals.md#12-concurrency-control-and-locking), [02 §12.2 InnoDB locks](02_SQL_Databases.md#12-transactions-and-locking)

---

### 2.4 Inventory goes negative during a flash sale ⭐⭐

**Cause:** the same check-then-act race as 2.1, at very high concurrency, often with a cache in front showing stale stock.

**Fixes:**
- Database: `UPDATE products SET stock = stock - :qty WHERE id = :id AND stock >= :qty` + `CHECK (stock >= 0)`.
- At flash-sale scale, a single hot row becomes a lock bottleneck (every buyer waits on the same row). Options:
  - **Redis atomic counter** (`DECR` or Lua check-then-decrement) as the gatekeeper; only successful reservations proceed to create orders asynchronously via a queue.
  - **Split stock into N buckets/rows** (e.g. 10 rows of 100 units) and pick one at random to reduce contention.
- Never use the cached stock value for the final decision; read/decrement the source of truth at checkout.
- Release reservations that aren't paid within a timeout; reconcile Redis and the database after the sale.

**Concepts used:** [04 §12.6](04_Redis.md#12-use-case-designs), [01 §11](01_DB_Fundamentals.md#11-isolation-levels)

---

### 2.5 A payment was processed twice after a retry ⭐⭐

**Cause:** the client (or a load balancer, or a queue) retried a request that had actually succeeded, e.g. the response timed out. Without idempotency, each retry creates a new charge.

**Fixes:**
- **Idempotency key** per logical operation, generated by the client (UUID) and sent with every retry.
- Server stores it with a **unique constraint**: `INSERT ... ON CONFLICT (idempotency_key) DO NOTHING`; if it already exists, return the original result instead of processing again.
- Handle the "in progress" state (first request still running) by returning 409 or waiting.
- Pass the same key to the payment provider (Stripe, Razorpay support idempotency keys).
- Queue consumers: a processed-message table checked in the same transaction as the side effect.

**Concepts used:** [01 §19.4 idempotency](01_DB_Fundamentals.md#19-data-pipelines-and-data-integrity)

---

# 3. Connections and Load

### 3.1 "Too many connections" errors after scaling the app ⭐

**Cause:** total connections = **app instances × pool size per instance** (+ cron jobs, admin tools). Autoscaling from 10 to 40 pods with a pool of 20 = 800 connections; PostgreSQL's default `max_connections` is 100, MySQL's 151.

**Fixes:**
- Short-term: reduce the per-instance pool size; restart leaking instances.
- Proper fix: an **external pooler** (PgBouncer in transaction mode, ProxySQL, RDS Proxy) multiplexing many client connections onto a small number of database connections.
- Size the database side from its cores (~2 × cores active connections), not from app demand.
- Raising `max_connections` is a last resort: each connection costs memory (a whole process in PostgreSQL) and more active connections reduce throughput.

**Concepts used:** [01 §8](01_DB_Fundamentals.md#8-connection-management)

---

### 3.2 The connection pool is exhausted and requests time out ⭐

**Symptoms:** "timeout acquiring a connection from the pool", while the database itself looks idle or moderately loaded.

**How to investigate:**
- Pool metrics: active vs idle vs waiting.
- Database: `pg_stat_activity` showing many `idle in transaction` sessions; MySQL `SHOW PROCESSLIST` with long `Sleep` connections inside transactions.
- Leak detection (HikariCP `leakDetectionThreshold`, logging borrow stack traces).

**Likely causes:**
- **Connection leaks:** a code path that doesn't release the connection on error (missing `finally`).
- **Long transactions:** holding a connection while calling an external API or doing heavy computation.
- **Slow queries** under load keep connections busy longer (a lock wait cascades into pool exhaustion).
- Pool too small for legitimate concurrency (less common than the above).

**Fixes:** `try/finally` or framework-managed transactions; move external calls outside transactions; set statement and `idle_in_transaction_session_timeout`; fix the slow query; then tune pool size.

**Concepts used:** [01 §8.4](01_DB_Fundamentals.md#8-connection-management), [01 §9.4](01_DB_Fundamentals.md#9-orms-and-the-application-layer)

---

### 3.3 Database CPU is at 100% ⭐

**Stabilise:** identify and kill runaway queries (`pg_cancel_backend`, `KILL QUERY`); if a recent deploy caused it, roll back; enable rate limiting / shed non-critical traffic.

**How to investigate:**
- **Top queries by total time** (`pg_stat_statements`, `sys.statement_analysis`): CPU is usually consumed by a few query shapes running very often, not one huge query.
- Look for full scans, large sorts/hash joins, missing indexes, N+1 storms.
- Check for a traffic spike, a cache outage (all reads now hit the database), or a new batch job/report.

**Fixes:**
- Index or rewrite the top offenders; cache hot reads; move analytics to a replica; fix N+1.
- Long-term: capacity planning, read replicas, query regression checks in CI, alerts on CPU and on top-query changes.

---

### 3.4 Read traffic is overwhelming the primary ⭐⭐

**Options, in order:**
1. Remove waste: fix N+1, unnecessary queries, `SELECT *`, missing indexes.
2. **Cache** hot reads in Redis (cache-aside) and at the HTTP/CDN layer.
3. **Read replicas** for read-heavy endpoints, routing writes to the primary; handle replication lag (read-your-writes for the user's own data).
4. Precompute expensive reads (materialized views, summary tables).
5. If writes also don't fit: partitioning, then sharding.

**Concepts used:** [01 §15](01_DB_Fundamentals.md#15-replication), [04 §9](04_Redis.md#9-caching-patterns)

---

# 4. Replication and Consistency

### 4.1 A user updates their profile but sees old data on refresh ⭐

**Cause:** **replication lag**. The write went to the primary; the read went to a replica that hasn't applied it yet. (Or: a stale cache entry that wasn't invalidated.)

**Investigate:** confirm which node served the read; check replica lag (`pg_stat_replication`, `SHOW REPLICA STATUS`); check whether a cache sits in front.

**Fixes:**
- **Read-your-writes routing:** read from the primary for the user's own data, or for N seconds after they write (flag in session/cookie).
- Wait until the replica has caught up to the write's position (PostgreSQL LSN, MySQL GTID) before reading.
- Session stickiness to one replica (monotonic reads).
- If it's the cache: invalidate on write (delete after DB update).

**Concepts used:** [01 §15.3](01_DB_Fundamentals.md#15-replication), [04 §10](04_Redis.md#10-cache-consistency)

---

### 4.2 Replication lag keeps growing ⭐⭐

**How to investigate:** lag metrics over time; what's running on the replica; write volume on the primary; replica resources (CPU, disk I/O, network).

**Likely causes:**
- **Large transactions / bulk updates** on the primary (one `UPDATE` of 50M rows must be replayed as a whole).
- Replica hardware weaker than the primary, or disk I/O saturated.
- Single-threaded apply (older MySQL configs).
- **Long queries on the replica** conflicting with replay (PostgreSQL may pause replay or cancel queries).
- Network bandwidth between regions.
- Tables without primary keys in MySQL row-based replication (each row change needs a full scan on the replica!).

**Fixes:** batch large writes into small chunks; enable parallel apply (`replica_parallel_workers`); upgrade replica hardware; move heavy analytics to a dedicated replica; add primary keys; for PostgreSQL, tune `max_standby_streaming_delay` / `hot_standby_feedback`.

---

### 4.3 Data was lost after a failover ⭐⭐⭐

**Cause:** **asynchronous replication**: the primary acknowledged commits that hadn't reached the replica before the primary failed; the promoted replica doesn't have them. (Or split-brain: the old primary kept accepting writes after being cut off, and those were discarded.)

**Fixes / prevention:**
- Synchronous or semi-synchronous replication to at least one replica (PostgreSQL `synchronous_standby_names`, MySQL semi-sync); MongoDB `w: "majority"`.
- Majority-based failover with fencing of the old primary (Patroni, Orchestrator, Group Replication) to prevent split-brain.
- Redis: `min-replicas-to-write`, `WAIT`, and not using Redis as the only store for critical data.
- Recovery: compare the old primary's log with the new primary to extract lost transactions (MongoDB writes rollback files; MySQL binlogs can be diffed by GTID) and reconcile manually.

**Concepts used:** [01 §15.2, §15.4](01_DB_Fundamentals.md#15-replication), [03 §8.3](03_NoSQL_MongoDB.md#8-replication), [04 §7.3](04_Redis.md#7-replication-and-high-availability)

---

# 5. Caching

### 5.1 The cache and database are out of sync ⭐

**Investigate:** which keys are stale; how they're written (cache-aside? write-through?); what the invalidation path is; whether a deploy or an outage coincided.

**Likely causes:**
- Updating the cache instead of deleting it, with concurrent writers applying out of order.
- Deleting the cache *before* the DB update (a concurrent read re-caches the old value).
- The rare read/write race even with "update DB then delete cache".
- A failed cache delete (network error) with no retry.
- A code path (admin tool, script, another service) that writes the database without invalidating.
- No TTL, so stale values live forever.

**Fixes:**
- DB first, then **delete** the cache; retry failed deletes (or queue them).
- TTLs on everything as a safety net; delayed double delete for hot keys.
- **CDC-based invalidation** (Debezium → consumer deletes keys) so *every* write path invalidates, including scripts and other services.
- Version cache key prefixes to purge quickly; never use cached values for critical decisions.

**Concepts used:** [04 §10](04_Redis.md#10-cache-consistency)

---

### 5.2 The database crashed when a popular cache key expired ⭐

**This is cache breakdown / stampede:** thousands of requests missed the same hot key at the same moment and all queried the database.

**Fixes:**
- **Single-flight rebuild:** a Redis lock (`SET NX PX`) so one request rebuilds; others wait briefly or get stale data.
- **Logical expiry / stale-while-revalidate:** keep serving the old value while one background refresh runs.
- Don't expire known hot keys; refresh them proactively.
- **TTL jitter** to avoid synchronized expiry across many keys (avalanche).
- Protect the database: circuit breaker, rate limiting, a small in-process cache.

**Concepts used:** [04 §11](04_Redis.md#11-cache-problems)

---

### 5.3 Redis memory is full ⭐

**Investigate:** `INFO memory` (`used_memory` vs `maxmemory`, fragmentation), `INFO stats` (`evicted_keys`), `--bigkeys` / `--memkeys`, keys without TTLs (sample with `SCAN` + `TTL`), and the eviction policy.

**Likely causes:**
- Keys without TTLs accumulating (sessions or cache entries that never expire).
- A few **big keys** (an unbounded list/hash).
- `noeviction` policy on a cache instance → writes failing with OOM.
- A `volatile-*` policy, but most keys have no TTL → nothing evictable.
- Memory fragmentation; fork copy-on-write overhead during RDB/AOF rewrite.

**Fixes:** set TTLs; choose `allkeys-lru`/`allkeys-lfu` for caches; split or trim big keys; separate caches from must-not-lose data (different instances/policies); enable `activedefrag`; leave headroom for forks; scale up or out (Cluster).

**Concepts used:** [04 §3](04_Redis.md#3-expiry-and-eviction), [04 §13](04_Redis.md#13-operations-and-troubleshooting)

---

### 5.4 Redis latency spiked suddenly ⭐⭐

**Investigate:** `SLOWLOG GET`, `LATENCY DOCTOR`, `INFO` (`latest_fork_usec`, `mem_fragmentation_ratio`, `connected_clients`, `blocked_clients`), host CPU/network/swap.

**Likely causes:**
- A blocking command: `KEYS *`, `HGETALL`/`SMEMBERS` on a huge key, `DEL` of a big key, a slow Lua script.
- **Fork latency** from RDB snapshots / AOF rewrites on a large dataset (and transparent huge pages enabled).
- **Swapping** (fragmentation ratio < 1): the OS moved Redis memory to disk.
- A **hot key** saturating one node in a cluster.
- Network saturation from large values; too many connections (no client-side pooling).
- AOF `appendfsync always` on slow disks.

**Fixes:** replace blocking commands (`SCAN`, `UNLINK`, paging); split big keys; schedule or move persistence to replicas; disable THP; add memory headroom; spread hot keys with local caching or replicas.

**Concepts used:** [04 §1.1](04_Redis.md#1-fundamentals), [04 §13](04_Redis.md#13-operations-and-troubleshooting)

---

# 6. Schema and Data Changes

### 6.1 Add a column / index to a 500-million-row table without downtime ⭐⭐

**Index:**
- PostgreSQL: `CREATE INDEX CONCURRENTLY` (check for an `INVALID` index if it fails, drop and retry).
- MySQL: `ALTER TABLE ... ADD INDEX ..., ALGORITHM=INPLACE, LOCK=NONE`, or gh-ost / pt-online-schema-change for very large or busy tables.
- MongoDB: index builds don't block reads/writes (4.2+), but run off-peak; Atlas rolling builds.

**Column:**
- PostgreSQL 11+: `ADD COLUMN ... DEFAULT <constant>` is instant. A volatile default or a type change rewrites the table → use a new nullable column + batched backfill instead.
- MySQL 8.0: `ALGORITHM=INSTANT` for adding columns.
- `NOT NULL`: add a `CHECK ... NOT VALID` constraint, `VALIDATE` it, then `SET NOT NULL` (PostgreSQL 12+).

**Always:**
- Set `lock_timeout` (PostgreSQL) / `lock_wait_timeout` (MySQL) so the DDL fails fast instead of queueing every query behind it.
- Check for long-running transactions first.
- Backfill data in small batches, watching replication lag.
- Test on a production-sized copy.

**Concepts used:** [02 §18.3](02_SQL_Databases.md#18-operations), [02 §12.3](02_SQL_Databases.md#12-transactions-and-locking)

---

### 6.2 Rename a column without breaking the running app ⭐⭐

**Expand → migrate → contract** across several deploys:
1. Add the new column `full_name`.
2. Deploy code that **writes both** columns and still reads the old one.
3. Backfill `full_name` from `name` in batches.
4. Deploy code that **reads the new** column (still writing both, for rollback safety).
5. Deploy code that stops writing the old column.
6. Drop `name`.

Each step is compatible with the previous code version, so rolling deploys (old and new pods running together) and rollbacks are safe. A direct `RENAME COLUMN` breaks every running instance of the old code immediately.

**Concepts used:** [01 §22.8](01_DB_Fundamentals.md#22-applied-design)

---

### 6.3 Delete or archive old data from a huge table ⭐⭐

**Don't** run `DELETE FROM events WHERE created_at < '2025-01-01'` on 300M rows: one huge transaction, long locks, massive WAL/binlog, replica lag, bloat, and a rollback that takes as long as the delete if it fails.

**Do:**
- **Batch deletes:** a few thousand rows per transaction by primary key, with short sleeps, during low traffic, monitoring replica lag.
- Archive first if needed: copy batches to an archive table, S3 (Parquet) or cold storage, then delete.
- **Long-term: partition by time** and `DROP`/`DETACH` old partitions (instant, no bloat).
- PostgreSQL: afterwards, VACUUM (and pg_repack if you need disk space back).
- MongoDB: TTL indexes for automatic expiry; or `deleteMany` in batches.

**Concepts used:** [01 §20.3](01_DB_Fundamentals.md#20-data-lifecycle), [01 §16.2](01_DB_Fundamentals.md#16-partitioning-and-sharding)

---

### 6.4 Move from one database to another with no downtime ⭐⭐⭐

(e.g. MySQL → PostgreSQL, a self-hosted → managed database, or a major version upgrade.)

**Steps:**
1. **Schema translation** and testing on a copy.
2. **Initial bulk copy** (snapshot) into the new database.
3. **Continuous sync** of changes using CDC (Debezium, AWS DMS) or logical replication, until lag ≈ 0.
4. **Validate**: row counts, checksums, sample comparisons, and shadow reads (read from both, compare results, serve from the old).
5. Optionally **dual writes** behind a feature flag (carefully, with the old DB as source of truth).
6. **Cutover:** brief write freeze (seconds), confirm sync, switch writes to the new DB, sync sequences.
7. Keep **reverse replication** (new → old) running for a while so rollback is possible.
8. Decommission after a confidence period.

**Concepts used:** [01 §19.3 CDC](01_DB_Fundamentals.md#19-data-pipelines-and-data-integrity), [02 §15.2 logical replication](02_SQL_Databases.md#15-replication-and-scaling)

---

# 7. Scaling

### 7.1 One table has grown too large: partition or shard? ⭐⭐

**Clarify:** is the problem query speed, maintenance (VACUUM, index builds, backups), storage cost, or write throughput beyond one server?

| Problem | Solution |
|---|---|
| Queries mostly touch recent data; old data rarely used | **Partition by time** (pruning), archive old partitions |
| Deleting old data is painful | Partition by time, drop partitions |
| Table maintenance takes too long | Partitioning (per-partition VACUUM/indexes) |
| Reads overload the server | Indexes, caching, read replicas |
| **Writes or total data exceed one server** | **Sharding** (Citus, Vitess, application-level, or a distributed database) |

Partition first (it stays within one database, so joins and transactions still work); shard only when one machine truly can't handle the writes or data.

**Concepts used:** [01 §16](01_DB_Fundamentals.md#16-partitioning-and-sharding)

---

### 7.2 Choosing a shard key for a multi-tenant SaaS app ⭐⭐

**Answer:** usually **`tenant_id`** (or a hash of it):
- Almost every query is scoped to one tenant → single-shard queries.
- A tenant's data stays together → joins and transactions within a tenant still work.
- Natural isolation and per-tenant operations (export, delete, move).

**Watch out for:**
- **Whale tenants:** one huge customer overloads its shard → move big tenants to dedicated shards (directory-based placement), or use a compound key `(tenant_id, entity_id)` to split their data.
- Cross-tenant analytics → a separate warehouse.
- Make `tenant_id` the leading column of every index and primary key.

**Concepts used:** [01 §16.5](01_DB_Fundamentals.md#16-partitioning-and-sharding), [01 §22.7](01_DB_Fundamentals.md#22-applied-design)

---

### 7.3 One shard is a hot spot ⭐⭐

**Investigate:** is it hot because of **data volume** (uneven distribution) or **traffic** (a few hot keys)?

**Likely causes:** monotonically increasing shard key (all inserts go to the "latest" shard); low-cardinality key; a celebrity/whale entity; time-based range sharding with all current traffic on one range.

**Fixes:**
- Hash the shard key, or prefix it with a well-distributed field.
- Split or move the hot range/tenant to its own shard; rebalance.
- For hot entities: add a suffix (`celebrity_id#0..9`) to spread writes, aggregate on read; cache hot reads.
- MongoDB: `refineCollectionShardKey` / `reshardCollection`; Redis Cluster: spread hot keys with copies or local caching.

**Concepts used:** [01 §16.6](01_DB_Fundamentals.md#16-partitioning-and-sharding), [03 §9](03_NoSQL_MongoDB.md#9-sharding)

---

# 8. Messaging and Pipelines

### 8.1 A consumer keeps crashing on the same message (poison message) ⭐⭐⭐

**Stabilise:** move the message out of the main flow so other messages can proceed (skip/park it); scale back up consumers.

**Investigate:** consumer logs and the exact payload; was there a recent producer deploy (schema change)? Is the failure deterministic (bad data) or transient (dependency down)?

**Fixes:**
- **Bounded retries with exponential backoff**, then a **dead letter queue** (SQS redrive policy, RabbitMQ DLX, Kafka retry + DLQ topics, Redis Streams delivery count + DLQ stream).
- Classify errors: validation errors → DLQ immediately; timeouts → retry.
- Alert on DLQ size; fix the bug/data; replay from the DLQ.
- Prevent: schema registry with compatibility checks, producer-side validation, tolerant consumers (ignore unknown fields), contract tests between services.

**Concepts used:** [01 §19.5](01_DB_Fundamentals.md#19-data-pipelines-and-data-integrity), [04 §6.3](04_Redis.md#6-messaging)

---

### 8.2 Events are processed twice or out of order ⭐⭐⭐

**Duplicates** are normal with at-least-once delivery (consumer crashed after processing but before acknowledging; producer retried). **Fix:** idempotent consumers: a `processed_events(event_id)` table with a unique constraint written in the same transaction as the side effect, or naturally idempotent operations (`SET status = 'SHIPPED'` instead of increments).

**Out of order:** messages for the same entity on different partitions/consumers, or retries delaying older messages. **Fixes:**
- Partition by entity key (Kafka key = `order_id`) so one entity's events stay ordered.
- Include a **version / sequence number** per entity; ignore events older than the stored version.
- Design state machines that reject invalid transitions (`DELIVERED` can't go back to `SHIPPED`).

**Concepts used:** [01 §19.4](01_DB_Fundamentals.md#19-data-pipelines-and-data-integrity)

---

### 8.3 Keeping the database and the message queue in sync ⭐⭐⭐

**Problem:** the dual write: `INSERT order` succeeds, `publish(order.created)` fails (or vice versa).

**Fix: transactional outbox:**
1. In the same database transaction, insert the order **and** an `outbox` row describing the event.
2. A relay (poller or CDC like Debezium) reads the outbox and publishes to Kafka/RabbitMQ, marking rows as sent.
3. Consumers are idempotent because the relay can publish a message more than once.

Alternative: pure CDC on the business tables (publish row changes directly), at the cost of exposing internal schema as events.

**Concepts used:** [01 §19.1–19.3](01_DB_Fundamentals.md#19-data-pipelines-and-data-integrity)

---

# 9. Reliability and Recovery

### 9.1 Someone ran a DELETE without WHERE in production ⭐

**Immediately:**
1. Stop further damage: stop the job/script; if the app is now misbehaving, consider maintenance mode.
2. **Don't** restore over production in a panic.

**Recover:**
- If still in an open transaction: `ROLLBACK`.
- Otherwise, **point-in-time recovery** to a **new** instance at the moment just before the delete (PostgreSQL WAL archive / MySQL binlog replay / MongoDB oplog / managed-service PITR).
- Export the deleted rows from the restored copy and re-insert them into production (handling rows changed since), or switch over if that's simpler.
- A **delayed replica** (e.g. 1 hour behind) can provide the data faster.
- MySQL: tools can generate reverse SQL from row-based binlog events.

**Prevent:**
- No routine direct write access to production; changes via reviewed scripts/migrations.
- Safe SQL habits: `BEGIN;` → run → check rows affected → `COMMIT`; MySQL `--safe-updates` (`sql_safe_updates`).
- Least-privilege database users; soft deletes for important tables; audit logs.
- Tested backups with PITR and known RPO/RTO.

**Concepts used:** [01 §14.6](01_DB_Fundamentals.md#14-recovery-and-durability)

---

### 9.2 Restore the database to a point in time ⭐⭐

**Requirements (must be set up in advance):** a base backup (physical) + continuous archiving of the change log (PostgreSQL WAL, MySQL binlog, MongoDB oplog), retained for the recovery window.

**Steps:**
1. Identify the target time (just before the incident) from logs/audit trails.
2. Provision a **new** instance; restore the latest base backup taken before that time.
3. Replay logs up to the target (`recovery_target_time` in PostgreSQL; `mysqlbinlog --stop-datetime ... | mysql`; `mongorestore --oplogReplay --oplogLimit`).
4. Validate the data.
5. Either switch the application to it, or copy the needed data back to production.

Managed services (RDS, Cloud SQL, Atlas) automate this to a chosen second within the retention period.

---

### 9.3 Design a backup strategy for a production system ⭐⭐

**Start from requirements:**
- **RPO** (max acceptable data loss): e.g. 5 minutes.
- **RTO** (max acceptable downtime): e.g. 1 hour.
- Compliance retention (e.g. 7 years for financial records).

**A typical answer:**
- Daily **physical** full backups (snapshots / pgBackRest / XtraBackup) + **continuous log archiving** for PITR (meets a minutes-level RPO).
- Weekly **logical** dumps for portability and partial restores.
- Store backups **off the primary host**, in another region and ideally another account, **encrypted**, with immutability (object lock) against ransomware.
- Retention policy (e.g. 35 days of PITR, monthly backups for a year).
- **Automated restore tests** (restore weekly to a scratch instance and run sanity checks); measure actual restore time against RTO.
- Replicas and multi-AZ for availability, **not** as backups (they replicate deletes instantly).
- Runbooks for common recovery scenarios.

**Concepts used:** [01 §14.6](01_DB_Fundamentals.md#14-recovery-and-durability)
