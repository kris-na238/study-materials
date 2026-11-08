# DB Revision Cheat Sheet

One-line answers and comparison tables condensed from files 01–06. Use it **the day before an interview**. If any line doesn't make sense, go back to the linked section.

[01 Fundamentals](01_DB_Fundamentals.md) · [02 SQL Databases](02_SQL_Databases.md) · [03 MongoDB](03_NoSQL_MongoDB.md) · [04 Redis](04_Redis.md) · [05 Query Practice](05_Query_Practice.md) · [06 Scenarios](06_Scenario_Questions.md)

---

## 1. One-Line Definitions

**Design**
- **Normalization:** store each fact once; 1NF atomic values, 2NF no partial dependency, 3NF no transitive dependency, BCNF every determinant is a super key.
- **Denormalization:** deliberate duplication for faster reads; you must keep the copies in sync.
- **Candidate key:** minimal set of columns that uniquely identifies a row. **Primary key:** the chosen one (not NULL, one per table).
- **Surrogate key:** artificial ID; use it as PK plus a UNIQUE constraint on the natural key.
- **UUID v7 / ULID:** globally unique *and* time-ordered, so inserts stay sequential.

**Storage and indexing**
- **Page:** fixed-size block (PostgreSQL 8KB, InnoDB 16KB); the unit of I/O and caching.
- **Buffer pool:** RAM cache of pages; hot data in RAM = fast queries.
- **B+ tree:** wide, shallow, balanced tree with data in linked leaves; 3–4 page reads for 100M rows; supports ranges and sorting.
- **LSM tree:** memtable + immutable sorted files + compaction; fast writes, more work on reads.
- **Clustered index:** table rows stored in index order (InnoDB PK). **Secondary index:** separate structure pointing to rows.
- **Covering index:** contains every column the query needs → index-only scan.
- **Leftmost-prefix rule:** index `(a, b, c)` serves `a`, `a,b`, `a,b,c`, not `b` alone; columns after a range can't be used to seek.
- **ESR rule:** compound index order = Equality → Sort → Range.
- **Full-text search:** tokenize + stem into an inverted index; PG `tsvector @@ tsquery` + GIN + `ts_rank`, MySQL `FULLTEXT` + `MATCH ... AGAINST`; typos/synonyms/facets → Elasticsearch fed by CDC.
- **Selectivity:** fraction of rows matched; indexes help only selective conditions.

**Transactions**
- **ACID:** Atomic (all or nothing), Consistent (constraints hold), Isolated (no interference), Durable (survives crashes).
- **WAL:** log the change before modifying data pages; flush the log at commit; replay after a crash.
- **MVCC:** keep row versions so readers see a snapshot; readers don't block writers.
- **2PL:** acquire all locks before releasing any; strict 2PL holds write locks until commit.
- **Deadlock:** circular wait; detected via wait-for graph, one transaction aborted; prevent with consistent lock order.
- **Optimistic locking:** version column checked at update; **pessimistic:** `SELECT ... FOR UPDATE`.
- **Dirty write:** overwriting another transaction's uncommitted write; no isolation level allows it (row write locks are held to commit).
- **Write skew:** two transactions read the same data and update different rows, breaking an invariant; only Serializable (or explicit locks) prevents it.

**Distributed**
- **CAP:** during a partition, choose consistency (refuse) or availability (answer, maybe stale).
- **PACELC:** else (no partition), choose latency or consistency.
- **Eventual consistency:** replicas converge if writes stop.
- **Quorum:** R + W > N → reads overlap the latest write.
- **2PC:** prepare + commit across participants; atomic but blocking.
- **Saga:** local transactions + compensating actions; eventual consistency.
- **Consistent hashing:** keys and nodes on a ring; adding a node moves ~1/N of keys.
- **Hot / warm / cold standby:** running + replicating (seconds RTO) / provisioned, periodic log shipping (minutes) / backups + runbook (hours).
- **Split-brain:** two primaries accept writes; prevent with majority elections + fencing.
- **Idempotency:** doing it twice = doing it once; idempotency key + unique constraint.
- **Outbox pattern:** write the event to an outbox table in the same transaction; relay publishes it.
- **Retention policy vs legal hold:** routine "delete after N" rule vs case-specific "preserve" order; the hold wins, and purge jobs must check it.
- **Poison message:** always fails processing; bounded retries → dead letter queue → fix → replay.

**Redis**
- **Why fast:** in-memory + single-threaded execution + I/O multiplexing + efficient structures.
- **Expiry:** lazy (on access) + active sampling (~10×/s).
- **Eviction:** `maxmemory-policy`; default `noeviction`; caches usually `allkeys-lru`/`allkeys-lfu`.
- **RDB vs AOF:** snapshots (compact, lose minutes) vs command log (≤ 1s loss with `everysec`); hybrid is common.
- **MULTI/EXEC:** isolated batch, **no rollback**; **WATCH:** optimistic CAS; **Lua:** atomic read-decide-write.
- **Cluster:** 16,384 hash slots, `CRC16(key) mod 16384`; hash tags `{}` for multi-key ops.

**MongoDB**
- **Embed vs reference:** embed what's read together and bounded; reference what's large, shared or unbounded.
- **Single-document writes are atomic;** multi-document transactions exist (4.0+) with overhead.
- **`$elemMatch`:** all conditions on the same array element.
- **`w: "majority"`:** write survives failover (default since 5.0).
- **Shard key:** high cardinality, even distribution, non-monotonic, used in queries.

---

## 2. Isolation Levels vs Anomalies

| Level | Dirty read | Non-repeatable read | Phantom | Lost update | Write skew |
|---|---|---|---|---|---|
| Read Uncommitted | ⚠️ | ⚠️ | ⚠️ | ⚠️ | ⚠️ |
| Read Committed | ✅ | ⚠️ | ⚠️ | ⚠️ | ⚠️ |
| Repeatable Read | ✅ | ✅ | ⚠️ standard / ✅ PostgreSQL, mostly InnoDB | PostgreSQL ✅ (aborts) / MySQL ⚠️ | ⚠️ |
| Serializable | ✅ | ✅ | ✅ | ✅ | ✅ |

⚠️ = possible · ✅ = prevented

**Defaults:** PostgreSQL = Read Committed · MySQL InnoDB = Repeatable Read · MongoDB = read concern `local` (transactions: snapshot)

**Fix anomalies without changing level:** atomic `UPDATE ... SET x = x - 1 WHERE x >= 1`, `SELECT ... FOR UPDATE`, version columns, constraints.

---

## 3. Index Types: When to Use Which

| Index | Use for | Database |
|---|---|---|
| B-tree / B+ tree | Default: `=`, ranges, `ORDER BY`, prefix `LIKE 'abc%'` | All |
| Composite | Multi-column filters + sort (leftmost prefix, ESR) | All |
| Covering / `INCLUDE` | Index-only scans for hot queries | All (`INCLUDE`: PostgreSQL) |
| Partial | Rare values (`status='PENDING'`), unique among active rows | PostgreSQL, MongoDB |
| Expression / functional | `LOWER(email)`, JSON fields | PostgreSQL, MySQL 8.0.13+ |
| Hash / hashed | Equality only | PostgreSQL, MongoDB (hashed, for sharding) |
| GIN | JSONB containment, arrays, full-text, trigram `LIKE '%x%'` | PostgreSQL |
| GiST / SP-GiST | Geospatial, ranges, exclusion constraints, nearest-neighbour | PostgreSQL |
| BRIN | Huge append-only, time-ordered tables | PostgreSQL |
| FULLTEXT | Word search | MySQL |
| Multi-valued | JSON arrays | MySQL 8.0.17+ |
| Invisible / hidden | Test dropping an index safely | MySQL / MongoDB |
| Multikey | Array fields | MongoDB |
| TTL | Auto-expire documents | MongoDB |
| Wildcard | Dynamic field names | MongoDB |
| 2dsphere | Geo queries | MongoDB |
| HNSW / IVFFlat | Vector similarity | pgvector |

**Index not used when:** function/cast on column · leading `%` · wrong composite order · low selectivity · stale stats · `OR` across columns.

---

## 4. Comparison Tables

### MySQL vs PostgreSQL

| | MySQL InnoDB | PostgreSQL |
|---|---|---|
| Connections | Thread each | Process each (needs PgBouncer) |
| Table storage | Clustered on PK | Heap + separate indexes |
| Secondary index stores | PK value | Physical location (ctid) |
| MVCC | Undo log, in-place update, purge | New row version per update, VACUUM |
| Default isolation | Repeatable Read (gap locks) | Read Committed |
| Serializable | Lock-based | SSI (aborts + retry) |
| Transactional DDL | No | Yes |
| JSON | JSON + generated-column indexes | JSONB + GIN |
| Full-text search | `FULLTEXT`, `MATCH ... AGAINST`, no stemming | `tsvector`/`tsquery` + GIN, stemming, `ts_rank`, `pg_trgm` for typos |
| Extras | Invisible indexes, Vitess | Partial/expression indexes, `RETURNING`, `DISTINCT ON`, extensions |
| Upsert | `ON DUPLICATE KEY UPDATE` | `ON CONFLICT`, `MERGE` (15+) |
| Logs | Redo log + binlog | WAL |

### SQL vs NoSQL

| | Relational | Document | Key-value | Wide-column | Graph |
|---|---|---|---|---|---|
| Example | PostgreSQL, MySQL | MongoDB | Redis, DynamoDB | Cassandra | Neo4j |
| Best for | Relations, transactions, integrity | Self-contained, varied records | Fast lookups by key | Massive writes, known queries | Relationship traversal |
| Schema | Enforced | Flexible (optional validation) | None | Per-table, query-driven | Flexible |
| Scaling | Vertical + replicas (+ sharding tools) | Built-in sharding | Built-in | Built-in, leaderless | Mostly vertical |

### MongoDB vs PostgreSQL JSONB

| | MongoDB | PostgreSQL JSONB |
|---|---|---|
| Model | Documents everywhere | Tables + flexible JSON column |
| Partial update | In-place operators (`$set`, `$inc`, `$push`) | Rewrites the whole JSONB value |
| Joins | `$lookup` (not its strength) | Native |
| Transactions | Single-doc atomic; multi-doc with overhead | Full ACID |
| Constraints | Unique, validation, no FKs | FK, CHECK, exclusion |
| Scale-out / HA | Built in | Citus / Patroni / managed |

### B+ tree vs LSM tree

| | B+ tree | LSM tree |
|---|---|---|
| Writes | In place (random I/O) | Append + compaction (sequential) |
| Reads | Fast, predictable | May check several files (Bloom filters) |
| Used by | PostgreSQL, InnoDB, MongoDB | Cassandra, RocksDB, ScyllaDB |

### Replication logs

| Database | Crash recovery | Replication / CDC |
|---|---|---|
| PostgreSQL | WAL | WAL (physical streaming, logical decoding) |
| MySQL | Redo log | Binlog |
| MongoDB | Journal | Oplog (change streams) |
| Redis | AOF / RDB | Replication stream + backlog |

### Redis vs Memcached

| | Redis | Memcached |
|---|---|---|
| Types | Many structures | Strings |
| Persistence / replication | Yes | No |
| Threads | Single-threaded execution (+ I/O threads) | Multi-threaded |
| Extras | Lua, transactions, pub/sub, streams | — |

### Redis Pub/Sub vs Streams vs Kafka

| | Pub/Sub | Streams | Kafka |
|---|---|---|---|
| Stored | No | Yes (RAM, trimmed) | Yes (disk, long retention) |
| Delivery | At-most-once | At-least-once (ack) | At-least-once (offsets) |
| Consumer groups | No | Yes | Yes |
| Scale | Moderate | Moderate | Massive |

### Cache patterns

| Pattern | Write path | Risk | Use |
|---|---|---|---|
| Cache-aside | Write DB, **delete** cache | Brief staleness, races | Default |
| Read-through | — (cache loads on miss) | Same as cache-aside | Caching library |
| Write-through | Cache + DB synchronously | Slower writes | Read-after-write heavy |
| Write-behind | Cache now, DB async | Data loss on cache failure | Counters, metrics |
| Write-around | DB only | First read misses | Write-once data |

### Cache problems

| Problem | Cause | Fix |
|---|---|---|
| Penetration | Keys that exist nowhere | Cache nulls (short TTL), Bloom filter, validate input |
| Breakdown | One hot key expires | Mutex/single-flight, logical expiry, never-expire hot keys |
| Avalanche | Many keys expire together / cache down | TTL jitter, HA, multi-level cache, circuit breaker |
| Stampede | Many concurrent misses on a key | Request coalescing, early refresh, stale-while-revalidate |
| Poisoning | Wrong/stale/malicious data cached | Good key design, don't cache errors, versioned keys |

### Join algorithms

| | Nested loop | Hash join | Merge join |
|---|---|---|---|
| Best for | Small outer + indexed inner | Large unsorted equality joins | Pre-sorted inputs |

### Sharding strategies

| Strategy | Pro | Con |
|---|---|---|
| Range | Range queries on one shard | Hot spots |
| Hash | Even distribution | Range queries hit all shards |
| Consistent hashing | Minimal data movement | Complexity |
| Directory | Flexible placement | Lookup service bottleneck |

---

## 5. SQL Patterns Quick Reference

```sql
-- Anti-join: rows in A without a match in B
SELECT * FROM a WHERE NOT EXISTS (SELECT 1 FROM b WHERE b.a_id = a.id);

-- Nth highest distinct value
SELECT MAX(salary) FROM (SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) r FROM emp) t WHERE r = :n;

-- Top N per group / latest per group
SELECT * FROM (SELECT t.*, ROW_NUMBER() OVER (PARTITION BY grp ORDER BY val DESC) rn FROM t) x WHERE rn <= :n;

-- Running total
SUM(x) OVER (PARTITION BY g ORDER BY d ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)

-- Growth vs previous period
(x - LAG(x) OVER (ORDER BY period)) * 100.0 / NULLIF(LAG(x) OVER (ORDER BY period), 0)

-- Gaps and islands (consecutive days)
d - ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY d)::int AS grp   -- then GROUP BY grp

-- New group when a condition happens (sessionization)
SUM(CASE WHEN gap > interval '30 min' THEN 1 ELSE 0 END) OVER (PARTITION BY user_id ORDER BY ts)

-- Conditional aggregation / pivot
SUM(CASE WHEN status = 'X' THEN 1 ELSE 0 END)          -- PostgreSQL: COUNT(*) FILTER (WHERE status = 'X')

-- Delete duplicates keeping the smallest id
DELETE FROM t WHERE id IN (SELECT id FROM (SELECT id, ROW_NUMBER() OVER (PARTITION BY k ORDER BY id) rn FROM t) x WHERE rn > 1);

-- Recursive hierarchy
WITH RECURSIVE r AS (SELECT * FROM emp WHERE id = :root
                     UNION ALL SELECT e.* FROM emp e JOIN r ON e.manager_id = r.id)
SELECT * FROM r;

-- Relational division ("bought all")
GROUP BY user_id HAVING COUNT(DISTINCT product_id) = (SELECT COUNT(*) FROM products WHERE category = :c)

-- Fill missing dates
generate_series(:start, :end, interval '1 day') d LEFT JOIN facts f ON f.day = d

-- Median
PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY x)

-- Upsert
INSERT ... ON CONFLICT (k) DO UPDATE SET v = EXCLUDED.v;        -- PostgreSQL
INSERT ... ON DUPLICATE KEY UPDATE v = VALUES(v);                -- MySQL (or row alias, 8.0.19+)

-- Keyset pagination
WHERE (created_at, id) < (:last_created_at, :last_id) ORDER BY created_at DESC, id DESC LIMIT 20
```

**Traps:** `NOT IN` with NULLs · filter on right table of `LEFT JOIN` in `WHERE` · integer division (`100.0`) · default `RANGE` window frame with duplicates · `BETWEEN` on timestamps · fan-out double counting after joins.

---

## 6. Commands and Tools Quick Reference

### EXPLAIN

| | Command | Red flags |
|---|---|---|
| MySQL | `EXPLAIN` / `EXPLAIN ANALYZE` / `FORMAT=TREE` | `type = ALL`, `key = NULL`, `Using filesort`, `Using temporary`, huge `rows` |
| PostgreSQL | `EXPLAIN (ANALYZE, BUFFERS)` | `Seq Scan` on big tables, estimated ≪ actual rows, `Rows Removed by Filter` large, sort spilling to disk, high `loops` |
| MongoDB | `.explain("executionStats")` | `COLLSCAN`, `SORT` stage, keys/docs examined ≫ returned |

MySQL `type` best → worst: `const` → `eq_ref` → `ref` → `range` → `index` → `ALL`.

### Monitoring

| | Slow queries | Running now | Kill |
|---|---|---|---|
| MySQL | Slow query log + pt-query-digest, `sys.statement_analysis` | `SHOW PROCESSLIST`, `information_schema.innodb_trx` | `KILL <id>` |
| PostgreSQL | `pg_stat_statements`, `log_min_duration_statement`, auto_explain | `pg_stat_activity`, `pg_blocking_pids()` | `pg_cancel_backend` / `pg_terminate_backend` |
| MongoDB | Profiler (`setProfilingLevel`), mongod log | `db.currentOp()` | `db.killOp()` |
| Redis | `SLOWLOG GET` | `CLIENT LIST`, `INFO` | `CLIENT KILL` |

### Key settings

| | Memory | Durability | Safety nets |
|---|---|---|---|
| MySQL | `innodb_buffer_pool_size` (50–75% RAM) | `innodb_flush_log_at_trx_commit=1`, `sync_binlog=1` | `lock_wait_timeout`, strict `sql_mode` |
| PostgreSQL | `shared_buffers` (~25%), `work_mem` (per sort!), `effective_cache_size` | `synchronous_commit` | `statement_timeout`, `lock_timeout`, `idle_in_transaction_session_timeout` |
| MongoDB | WiredTiger cache (~50% of RAM−1GB) | `w: "majority"`, `j: true` | `maxTimeMS` |
| Redis | `maxmemory` + `maxmemory-policy` | `appendonly yes`, `appendfsync everysec` | `min-replicas-to-write` |

### Safe online changes

| | Index | Column | Big cleanup |
|---|---|---|---|
| MySQL | `ALGORITHM=INPLACE, LOCK=NONE`, gh-ost | `ALGORITHM=INSTANT` | Batched deletes, drop partitions |
| PostgreSQL | `CREATE INDEX CONCURRENTLY` | Constant default is instant (11+); `NOT VALID` + `VALIDATE` for constraints | Batched deletes, drop partitions, pg_repack |
| MongoDB | Non-blocking builds (4.2+) | Schema versioning + lazy migration | TTL index, batched `deleteMany` |

### Dangerous commands → alternatives

| Avoid | Use |
|---|---|
| Redis `KEYS *` | `SCAN` |
| Redis `DEL bigkey` | `UNLINK` |
| Redis `HGETALL`/`SMEMBERS` on huge keys | `HSCAN` / `SSCAN` |
| MySQL `REPLACE INTO` | `INSERT ... ON DUPLICATE KEY UPDATE` |
| PostgreSQL `VACUUM FULL` on busy tables | `pg_repack` |
| Giant `DELETE` | Batches / drop partitions |
| `NOT IN (subquery)` | `NOT EXISTS` |
| `OFFSET` deep pagination | Keyset pagination |

---

## 7. Scenario Answer Outlines

**Universal structure:** clarify → stabilise → investigate with evidence → causes → fix short/long term → prevent.

| Scenario | 3-line answer |
|---|---|
| Query suddenly slow | EXPLAIN ANALYZE, compare estimated vs actual rows → stale stats, data growth, missing/unused index, bloat, locks → ANALYZE, index, rewrite, cache, monitor |
| N+1 queries | Query logs/APM show repeated queries → lazy loading in a loop → eager load / `IN (...)` batch / DataLoader |
| Deep pagination slow | OFFSET reads and discards rows → keyset pagination on an indexed `(sort_col, id)` |
| Double booking / overselling | Check-then-act race → atomic conditional UPDATE or unique constraint → plus holds with expiry, idempotency |
| Lost update on balance | Read-modify-write in app → `SET balance = balance - x WHERE balance >= x`, FOR UPDATE, or version → ledger design |
| Deadlocks | Read deadlock log → inconsistent lock order, missing index, gap locks → order locks, index, short transactions, retry |
| Double payment | Retry of a succeeded request → idempotency key + unique constraint → return stored result |
| Too many connections | instances × pool size → smaller pools + PgBouncer/RDS Proxy → size from DB cores |
| Pool exhausted | Leaks, long transactions, slow queries → `finally`, no network calls in transactions, timeouts |
| DB CPU 100% | Kill runaway, roll back deploy → top queries by total time → index, cache, replicas |
| Stale read after update | Replica lag (or stale cache) → read-your-writes routing, wait for LSN/GTID, invalidate cache |
| Replication lag growing | Big transactions, weak replica, long replica queries, no PKs → batch writes, parallel apply, separate analytics replica |
| Data lost after failover | Async replication → sync/semi-sync, `w: majority`, fencing |
| Cache/DB out of sync | Update-instead-of-delete, delete-before-write, failed deletes → DB-then-delete, TTLs, CDC invalidation |
| Hot key expiry crashed DB | Cache breakdown → single-flight lock, logical expiry, jitter, circuit breaker |
| Redis memory full | No TTLs, big keys, wrong policy → TTLs, `allkeys-lru`, split keys, separate instances |
| Redis latency spike | SLOWLOG, LATENCY DOCTOR → blocking commands, fork, swap, hot keys → SCAN/UNLINK, split, tune persistence |
| Index/column on huge table | Online DDL (`CONCURRENTLY`, `INSTANT`, gh-ost) + lock timeout + batched backfill |
| Rename column | Expand → dual write → backfill → switch reads → stop old writes → drop |
| Purge old data | Batched deletes now, time partitions + drop later |
| Migrate databases | Snapshot + CDC sync → validate (counts, checksums, shadow reads) → brief freeze → cutover → reverse replication for rollback |
| Partition or shard? | Partition for time-based pruning/cleanup; shard only when writes/data exceed one server |
| SaaS shard key | `tenant_id` (hashed) → isolate whale tenants |
| Hot shard | Monotonic or low-cardinality key, whale entity → hash/prefix key, split/move, suffix hot keys |
| Poison message | Bounded retries with backoff → DLQ → alert, fix, replay; schema registry |
| Duplicate / out-of-order events | Idempotent consumers (processed-events table) → partition by entity key, version numbers |
| DB + queue sync | Transactional outbox (or CDC) + idempotent consumers |
| DELETE without WHERE | Stop the damage → PITR to a new instance → copy rows back → restrict access, safe-update habits |
| Backup strategy | RPO/RTO → full physical + continuous log archiving (PITR) → off-site, encrypted, immutable → automated restore tests |
