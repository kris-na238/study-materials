# Redis and Caching

Each section explains **what it is, why it matters, how it works with real examples**, followed by **interview questions with model answers**.

General concepts (replication, sharding, consistency, idempotency) are explained in [01_DB_Fundamentals.md](01_DB_Fundamentals.md). This file covers **Redis itself and caching patterns**, which is how Redis shows up in most backend and system design interviews.

**How to revise with this document**
- First pass: run the commands in `redis-cli` (Docker: `docker run -p 6379:6379 redis`).
- Revision passes: read only the **Interview Questions**.
- Priority: ⭐ must-know · ⭐⭐ commonly asked · ⭐⭐⭐ senior / deep dive

> Version notes reflect **Redis up to 8.x** and **Valkey** (the open-source fork). Version-specific features are tagged.

**Running example: ShopKart** uses Redis for: sessions, product page caching, carts, rate limiting the login API, flash-sale stock counters, leaderboards of top sellers, and background job queues.

---

## Table of Contents

[1. Fundamentals](#1-fundamentals) · [2. Data Structures](#2-data-structures) · [3. Expiry and Eviction](#3-expiry-and-eviction) · [4. Persistence](#4-persistence) · [5. Transactions and Atomicity](#5-transactions-and-atomicity) · [6. Messaging](#6-messaging) · [7. Replication and High Availability](#7-replication-and-high-availability) · [8. Redis Cluster](#8-redis-cluster) · [9. Caching Patterns](#9-caching-patterns) · [10. Cache Consistency](#10-cache-consistency) · [11. Cache Problems](#11-cache-problems) · [12. Use-Case Designs](#12-use-case-designs) · [13. Operations and Troubleshooting](#13-operations-and-troubleshooting)

---

## 1. Fundamentals

**Priority:** ⭐ must-know

**What it is:** Redis (REmote DIctionary Server) is an **in-memory data structure store**. Keys map to values that can be strings, lists, hashes, sets, sorted sets, streams and more. It's used as a **cache**, a **fast primary store for ephemeral data** (sessions, rate limits, leaderboards), and a **message broker**.

**Why it exists:** Reading from a relational database involves disk pages, query planning and often joins: milliseconds. Reading a key from Redis's RAM takes **microseconds**, and a single Redis instance can serve **100,000+ operations per second**. Putting Redis in front of the database absorbs most read traffic and protects the database during spikes.

**How it works**

### 1.1 Why Redis is fast ⭐

1. **In-memory:** all data lives in RAM; no disk reads on the request path.
2. **Single-threaded command execution:** one thread executes commands one at a time, so there are **no locks, no context switches, no race conditions** between commands. Each command is atomic.
3. **I/O multiplexing:** one thread handles thousands of client connections using `epoll`/`kqueue`: it waits for whichever sockets are ready instead of one thread per connection.
4. **Efficient data structures:** compact encodings for small values (listpack, intset) and well-chosen algorithms (skip lists, hash tables with incremental rehashing).
5. **Simple protocol (RESP)** that's cheap to parse.

```
           ┌────────────────────────────────────────────┐
clients ──►│ event loop (epoll): which sockets are ready?│
clients ──►│   read command → execute → write reply     │  one command at a time
clients ──►│   (next ready socket)                      │
           └────────────────────────────────────────────┘
```

**The bottleneck is usually network and memory, not CPU.** For the CPU-bound part, scale by running several Redis instances (Cluster).

**Multi-threaded I/O (Redis 6+):** reading requests from sockets and writing replies can use extra threads (`io-threads`), but **command execution stays single-threaded**, so atomicity guarantees are unchanged. Background threads also handle some slow work: freeing big keys (`UNLINK`), fsync for AOF, closing files.

**Consequence of single-threading:** one slow command (`KEYS *` on 10 million keys, `SMEMBERS` on a huge set, a long Lua script) **blocks every other client**. Most Redis production incidents come from this.

### 1.2 Redis vs Memcached ⭐

| | Redis | Memcached |
|---|---|---|
| Data types | Strings, lists, hashes, sets, sorted sets, streams, bitmaps, HLL, geo, JSON (8.0+) | Strings only |
| Persistence | RDB snapshots, AOF | None |
| Replication / HA | Replicas, Sentinel, Cluster | None built in |
| Threads | Single-threaded execution (+ I/O threads) | Multi-threaded |
| Atomic operations | Rich (INCR, Lua scripts, transactions) | Basic (incr/decr, CAS) |
| Pub/Sub, streams | Yes | No |
| Max value size | 512MB | 1MB default |
| Eviction | 8 policies | LRU |
| Use | Cache + data structures + messaging | Simple, very large, multi-threaded cache |

Typical answer: **Redis**, unless you need only a simple string cache and want multi-threaded scaling on one big machine.

### 1.3 Licensing (Redis vs Valkey) ⭐⭐⭐

In March 2024 Redis changed from the open-source BSD licence to source-available licences (RSALv2/SSPLv1). The Linux Foundation forked the last BSD version as **Valkey**, backed by AWS, Google and others; many cloud services moved to it (e.g. ElastiCache for Valkey). In 2025, **Redis 8** added the OSI-approved **AGPLv3** as a licence option. For interviews, it's enough to know both exist and are largely command-compatible.

### Interview Questions

**Q1. Why is Redis so fast even though it's single-threaded?** ⭐
Data is in memory, commands are simple operations on efficient data structures, and one thread with I/O multiplexing serves thousands of connections without locking or context switching. The bottleneck is usually network or memory, not CPU. Single-threaded execution also makes every command atomic.

**Q2. If Redis is single-threaded, how does it handle many clients?** ⭐
Through an event loop with I/O multiplexing (epoll/kqueue): it waits for whichever sockets have data, reads and executes each command quickly, writes the reply, and moves on. Since each command takes microseconds, thousands of clients are served concurrently. Redis 6+ also offloads socket reading/writing to I/O threads, while command execution stays on the main thread.

**Q3. What's the downside of the single-threaded model?** ⭐
One slow command blocks everyone: `KEYS *`, deleting a huge key with `DEL`, `SMEMBERS`/`HGETALL` on very large collections, long Lua scripts. Use `SCAN`, `UNLINK`, paginated reads and short scripts, and keep big keys out. To use more cores, run multiple instances or Redis Cluster.

**Q4. Redis or Memcached?** ⭐
Redis in most cases: it offers rich data structures, persistence, replication and failover, Lua scripting and pub/sub/streams. Memcached is a simpler multi-threaded string-only cache with no persistence, which can be fine for a pure, very large look-aside cache.

---

## 2. Data Structures

**Priority:** ⭐ must-know (knowing *which structure fits which problem* is what interviewers test)

### 2.1 Strings

Binary-safe values up to **512MB**: text, numbers, JSON, serialized objects, images.

```bash
SET product:100 '{"name":"Galaxy S26","price":79999}' EX 300   # cache with 5-min TTL
GET product:100
SET lock:order:501 "worker-7" NX PX 30000                      # set only if not exists (locks)
INCR page:views:100                                            # atomic counter → 1, 2, 3...
INCRBY stock:flashsale:100 -1                                  # atomic decrement
MSET k1 v1 k2 v2 / MGET k1 k2                                  # batch
GETDEL otp:42                                                  # get and delete (one-time use)
```
Internally a **SDS** (Simple Dynamic String): stores its length (O(1) `STRLEN`, binary-safe) and pre-allocates space for appends. Small integers are stored as actual integers (`int` encoding); short strings use a compact `embstr` encoding.

**Use for:** caching objects, counters, flags, locks, OTPs, rate-limit counters.

### 2.2 Lists

Ordered sequences, fast push/pop at both ends (O(1)); access by index is O(n).

```bash
LPUSH notifications:42 "Order shipped"     # push left
LRANGE notifications:42 0 9                # latest 10
LTRIM notifications:42 0 99                # keep only the latest 100 (capped list)
RPUSH jobs "job1" / LPOP jobs              # queue (FIFO)
BRPOP jobs 5                               # blocking pop: wait up to 5s for a job
LMOVE jobs jobs:processing LEFT RIGHT      # reliable queue: move atomically to a processing list
```
Encoding: a **quicklist** (linked list of compact **listpack** nodes).

**Use for:** simple queues, recent-activity feeds, capped logs.

### 2.3 Hashes

A map of field → value inside one key, like a small object.

```bash
HSET user:42 name "Asha" city "Pune" cartCount 3
HGET user:42 name
HINCRBY user:42 cartCount 1
HGETALL user:42                           # careful on huge hashes
HSET cart:42 product:100 1 product:205 2  # cart: productId → qty
HEXPIRE cart:42 3600 FIELDS 1 product:205 # per-field TTL (Redis 7.4+)
```
Encoding: **listpack** when small (≤ 128 fields and values ≤ 64 bytes by default), otherwise a hash table. Small hashes are very memory-efficient, so storing an object as a hash often uses less memory than many separate string keys.

**Hash vs JSON string:** a hash lets you read/update individual fields (`HINCRBY` a counter without rewriting the whole object). A JSON string is simpler when you always read/write the whole object. (Redis 8 / Redis Stack also has a native JSON type with path updates.)

**Use for:** objects (user session data, product summaries), carts, per-entity counters.

### 2.4 Sets

Unordered collections of **unique** strings, with O(1) add/remove/membership and server-side set algebra.

```bash
SADD product:100:viewers user:1 user:2 user:3
SISMEMBER product:100:viewers user:2           # 1
SCARD product:100:viewers                      # count
SINTER user:1:friends user:2:friends           # mutual friends
SDIFF  user:1:follows user:2:follows
SRANDMEMBER giveaway:entrants 3                # pick 3 random winners
SPOP giveaway:entrants                         # remove and return random
```
Encoding: **intset** (sorted integer array) for small all-integer sets, **listpack** for small sets (7.2+), otherwise a hash table.

**Use for:** unique visitors (small scale), tags, mutual friends, "has the user already liked this?", deduplication.

### 2.5 Sorted sets (ZSET) ⭐

Unique members, each with a **score**; kept sorted by score. Range by rank or score in O(log n + m).

```bash
ZADD leaderboard:sellers 15000 "seller:1" 22000 "seller:2" 9000 "seller:3"
ZINCRBY leaderboard:sellers 500 "seller:3"             # add to a score atomically
ZREVRANGE leaderboard:sellers 0 9 WITHSCORES           # top 10 (ZRANGE ... REV in 6.2+)
ZREVRANK leaderboard:sellers "seller:3"                # this seller's rank
ZRANGEBYSCORE delayed:jobs 0 1762600000                # all jobs due by this timestamp
ZREMRANGEBYSCORE ratelimit:42 0 1762599940000          # drop entries older than a window
```

**Internals (frequently asked):** a **skip list + hash table**.
- **Hash table:** member → score, for O(1) `ZSCORE` and existence checks.
- **Skip list:** members sorted by score for O(log n) insert/delete and range queries.
- Small sorted sets (≤ 128 members, short values) use a **listpack** instead.

```
Level 3:  1 ─────────────────────────────► 9
Level 2:  1 ─────────► 4 ────────────────► 9
Level 1:  1 ──► 2 ──► 4 ──► 5 ──► 7 ──► 9     (each node randomly promoted to higher levels)
```

**Why a skip list rather than a balanced tree (red-black / AVL)?**
- Simpler to implement and debug (no rotations).
- Range queries are natural: find the start, then walk level 1 forward.
- Comparable average performance (O(log n)), tunable memory per node.
- Antirez (Redis's creator) has said: easier to implement and reason about, and range operations are fast.

**Use for:** leaderboards, priority queues, delayed jobs (score = run-at timestamp), rate limiting windows, "trending" items, time-ordered indexes.

### 2.6 Bitmaps ⭐⭐

Not a separate type: bit operations on a string. Up to 2³² bits (512MB).

```bash
SETBIT active:2026-11-08 42 1          # user 42 was active today
GETBIT active:2026-11-08 42            # 1
BITCOUNT active:2026-11-08             # daily active users
BITOP AND active:both active:2026-11-07 active:2026-11-08   # active on both days
```
**10 million users = 10 million bits ≈ 1.2MB** per day. Requires integer, reasonably dense user IDs.

**Use for:** daily active users, feature flags per user, attendance/check-in streaks, bloom-filter-like membership.

### 2.7 HyperLogLog ⭐⭐

Probabilistic **distinct count** in a fixed **12KB** per key, with ~**0.81%** standard error, no matter how many items.

```bash
PFADD visitors:product:100 user:1 user:2 user:1
PFCOUNT visitors:product:100                           # ≈ 2
PFMERGE visitors:week visitors:mon visitors:tue ...    # union of days
```
You can't list members or remove one; you only get the approximate count.

**Use for:** unique visitors per page, unique search queries, anything where "about 1.2 million" is good enough.

**Choosing a unique-count structure:**

| | Set | Bitmap | HyperLogLog |
|---|---|---|---|
| Exact? | Yes | Yes | ~0.81% error |
| Memory for 10M users | ~hundreds of MB | ~1.2MB | 12KB |
| List members? | Yes | Yes (by bit position) | No |
| Needs integer IDs | No | Yes | No |

### 2.8 Geospatial ⭐⭐

Stored in a sorted set with a geohash as the score.
```bash
GEOADD stores 73.8567 18.5204 "store:pune" 72.8777 19.0760 "store:mumbai"
GEOSEARCH stores FROMLONLAT 73.85 18.52 BYRADIUS 10 km ASC WITHDIST
GEODIST stores store:pune store:mumbai km
```
**Use for:** nearby stores/drivers/restaurants.

### 2.9 Streams ⭐⭐

An append-only log with IDs, consumer groups and acknowledgements, like a lightweight Kafka. Covered in §6.

### 2.10 Choosing a structure

| Problem | Structure |
|---|---|
| Cache an object | String (JSON) or Hash |
| Counter / rate limit | String with `INCR` + TTL |
| Session | Hash or String + TTL |
| Shopping cart | Hash (productId → qty) |
| Leaderboard / ranking | Sorted set |
| Recent N items | List with `LTRIM`, or capped sorted set |
| Unique members, set algebra | Set |
| Approximate unique count | HyperLogLog |
| Per-user boolean for millions of users | Bitmap |
| Nearby search | Geo |
| Job queue with acknowledgements | Stream (or List with `LMOVE`) |
| Delayed / scheduled jobs | Sorted set (score = timestamp) |

### Interview Questions

**Q1. Which Redis data structure would you use for a leaderboard, and why?** ⭐
A sorted set: members are players and scores are points. `ZINCRBY` updates a score atomically, `ZREVRANGE 0 9` returns the top 10 and `ZREVRANK` returns a player's rank, all in O(log n). It stays sorted automatically.

**Q2. How are sorted sets implemented and why a skip list?** ⭐⭐
A hash table (member → score) for O(1) lookups plus a skip list ordered by score for O(log n) inserts, deletes, rank and range queries; small sets use a compact listpack. Skip lists were chosen over balanced trees because they're simpler to implement, naturally support range scans by walking the bottom level, and perform comparably.

**Q3. How would you count unique visitors to a page?** ⭐⭐
Exact and small scale: a Set (`SADD` then `SCARD`). Large scale with approximate results: HyperLogLog (`PFADD`/`PFCOUNT`), 12KB per counter with ~0.81% error, mergeable across days. With dense integer user IDs: a Bitmap (`SETBIT`/`BITCOUNT`), exact and compact.

**Q4. Should you store an object as a JSON string or a hash?** ⭐⭐
A JSON string if you always read and write the whole object and it's not huge. A hash if you need to read or update individual fields (e.g. increment one counter) without rewriting the whole object, and small hashes are memory efficient due to listpack encoding. Native RedisJSON allows path updates on nested JSON if available.

**Q5. How would you implement a delayed job queue in Redis?** ⭐⭐
A sorted set with the run-at timestamp as the score. Producers `ZADD delayed now+delay jobId`. A poller repeatedly takes due jobs (`ZRANGEBYSCORE delayed 0 now LIMIT 0 100`) and atomically removes and enqueues them (a Lua script or `ZREM` + check its return value so only one worker claims each job), then pushes them to a ready list or stream.

---

## 3. Expiry and Eviction

**Priority:** ⭐ must-know

### 3.1 TTL commands

```bash
SET session:abc '{...}' EX 1800        # set with TTL (seconds); PX for ms
EXPIRE cart:42 86400                   # set TTL on an existing key
EXPIREAT promo:diwali 1763000000       # expire at a Unix timestamp
TTL cart:42                            # seconds left; -1 = no expiry; -2 = key doesn't exist
PERSIST cart:42                        # remove TTL
SET k v KEEPTTL                        # overwrite value but keep existing TTL
```
⚠️ A plain `SET` on an existing key **removes its TTL**. Use `KEEPTTL` or set the TTL again.

### 3.2 How expired keys are removed ⭐⭐

Redis combines two strategies:

1. **Lazy (passive) expiry:** when a client accesses a key, Redis checks its TTL; if expired, it deletes it and returns nil.
2. **Active expiry:** about 10 times per second, Redis samples a set of keys that have TTLs (20 by default) and deletes the expired ones. If more than 25% of the sample was expired, it repeats immediately (bounded by a time limit so it doesn't block).

Why both: lazy alone would leave never-accessed expired keys consuming memory forever; scanning all keys actively would be too expensive. So expired keys may linger briefly, but they're never *returned*.

Replicas don't expire keys on their own: the primary sends explicit `DEL`s to replicas (replicas do hide logically expired keys from reads).

### 3.3 Eviction (when memory is full) ⭐

`maxmemory` sets the memory limit. When it's reached, `maxmemory-policy` decides what happens on new writes:

| Policy | Evicts | Use when |
|---|---|---|
| `noeviction` (**default**) | Nothing; writes return **OOM errors** | Redis is a primary store and data must not vanish |
| `allkeys-lru` | Least recently used, any key | **General-purpose cache (most common choice)** |
| `allkeys-lfu` | Least frequently used, any key | Cache with stable hot keys (popular products) |
| `allkeys-random` | Random key | Uniform access patterns |
| `volatile-lru` | LRU among keys **with a TTL** | Mixed: cache keys have TTLs, persistent keys don't |
| `volatile-lfu` | LFU among keys with a TTL | Same, frequency-based |
| `volatile-random` | Random key with a TTL | |
| `volatile-ttl` | Key with the **shortest remaining TTL** | Prefer evicting data that's about to expire anyway |

⚠️ With a `volatile-*` policy and no keys having TTLs, nothing can be evicted, so it behaves like `noeviction`.

Real life: ShopKart's cache instance runs `allkeys-lru` with `maxmemory 6gb` on an 8GB machine. The separate instance holding sessions and rate limits runs `noeviction`, because silently losing a session logs the user out.

### 3.4 How LRU and LFU are implemented ⭐⭐⭐

**Approximated LRU:** a true LRU needs a linked list of all keys, updated on every access (memory and CPU cost). Instead, each key stores a 24-bit "last access" clock. On eviction, Redis **samples** `maxmemory-samples` keys (default 5), keeps a small pool of the best candidates, and evicts the oldest. With 10 samples it's very close to true LRU.

**LFU (Redis 4+):** the same 24 bits store an 8-bit **logarithmic access counter** (a Morris counter: increments become less likely as the count grows, so 8 bits can represent millions of accesses) plus a 16-bit timestamp used to **decay** the counter over time, so keys that were popular last week but not now can be evicted. Tunable via `lfu-log-factor` and `lfu-decay-time`.

**LRU vs LFU:** LRU can evict a genuinely hot key after a one-off scan of many cold keys pushes it out. LFU protects consistently popular keys but adapts more slowly to new trends.

### Interview Questions

**Q1. How does Redis delete expired keys?** ⭐⭐
Lazily, when an expired key is accessed, and actively, through a background cycle that ~10 times a second samples keys with TTLs and deletes expired ones, repeating while more than 25% of a sample is expired. So expired keys are never returned, but may use memory briefly.

**Q2. What happens when Redis runs out of memory?** ⭐
It depends on `maxmemory-policy`. With the default `noeviction`, writes fail with OOM errors (reads still work). With an eviction policy, Redis removes keys according to it: all keys or only keys with TTLs, chosen by approximate LRU, LFU, random, or shortest TTL. For caches, `allkeys-lru` or `allkeys-lfu` is typical.

**Q3. LRU or LFU for a product cache?** ⭐⭐
LFU if there's a stable set of very popular products, since it won't evict them just because a burst of rarely viewed products was accessed recently. LRU if popularity shifts quickly (news, trending items) and recency is the best signal. Both are approximated in Redis via sampling.

**Q4. Why doesn't Redis implement exact LRU?**
Exact LRU needs a doubly linked list across all keys updated on every access, costing extra pointers per key and work on every read. Redis instead stores a small access clock per key and samples a few keys at eviction time, getting near-LRU behaviour cheaply.

---

## 4. Persistence

**Priority:** ⭐⭐ commonly asked

**What it is:** Redis keeps data in memory, but can save it to disk so it survives restarts. Two mechanisms, usable together.

### 4.1 RDB snapshots

A compact binary dump of the entire dataset at a point in time.

```
save 3600 1 300 100 60 10000    # snapshot if ≥1 change in 1h, ≥100 in 5min, or ≥10000 in 1min
BGSAVE                          # manual background snapshot
```
**How BGSAVE works:** Redis calls `fork()`. The child process writes the snapshot while the parent keeps serving clients. Thanks to **copy-on-write**, parent and child initially share memory pages; a page is copied only when the parent modifies it.

| Pros | Cons |
|---|---|
| Compact single file, great for backups and moving data | **Data since the last snapshot is lost** on a crash (minutes) |
| Fast restart (load a binary file) | `fork()` on a large dataset can pause Redis (tens to hundreds of ms per GB of page tables) |
| Minimal impact during normal operation | Copy-on-write can **double memory** under heavy writes during a save |

### 4.2 AOF (Append-Only File)

Logs every write command; on restart, Redis replays it.

```
appendonly yes
appendfsync everysec          # always | everysec (default) | no
```

| `appendfsync` | Durability | Performance |
|---|---|---|
| `always` | fsync after every write: lose at most one command | Slowest |
| `everysec` (default) | fsync once per second: lose up to ~1 second | Good balance |
| `no` | Let the OS flush (~30s on Linux) | Fastest, least durable |

**AOF rewrite:** the log grows forever (100 `INCR`s = 100 lines), so Redis periodically rewrites it into the minimal set of commands (`SET counter 100`), in a forked child (`BGREWRITEAOF`, automatic based on growth). Redis 7 uses a **multi-part AOF** (a base file plus incremental files).

### 4.3 Hybrid (recommended)

`aof-use-rdb-preamble yes` (default): the AOF rewrite starts with an RDB snapshot followed by AOF commands since then, so you get fast loading *and* ≤ 1s of loss. Many production setups enable AOF + periodic RDB snapshots for backups.

### 4.4 Which to choose

| Use | Persistence |
|---|---|
| Pure cache (data can be rebuilt from the database) | None, or RDB for faster warm restarts |
| Sessions, rate limits, queues (losing a few seconds is OK) | AOF `everysec` (+ RDB for backups) |
| Primary store where loss is unacceptable | AOF `always` + replicas, and question whether Redis is the right primary store |

Even with AOF `always`, Redis isn't a replacement for a relational database's durability guarantees across failover: replication is asynchronous (§7).

### Interview Questions

**Q1. RDB vs AOF?** ⭐⭐
RDB takes periodic point-in-time snapshots: compact, fast to restore, great for backups, but you lose everything since the last snapshot. AOF logs every write and replays it: much less data loss (≤ 1s with `everysec`), but larger files and slower restarts. The hybrid mode (AOF with an RDB preamble) combines fast loading with low loss and is the common production choice.

**Q2. How does Redis take a snapshot without blocking clients?** ⭐⭐
It forks a child process that writes the dataset to disk while the parent keeps serving requests. Copy-on-write means memory pages are shared until the parent modifies them. Watch out for fork latency on large instances and memory growth from copied pages under heavy writes; leave memory headroom.

**Q3. How much data can you lose with Redis?** ⭐⭐
With RDB only: everything since the last snapshot. With AOF `everysec`: about one second. With AOF `always`: at most the last command, but at a throughput cost. Plus, with asynchronous replication, writes acknowledged by the primary but not yet replicated can be lost if a replica is promoted.

---

## 5. Transactions and Atomicity

**Priority:** ⭐⭐ commonly asked

### 5.1 Single commands are atomic

Because commands execute one at a time, `INCR`, `SET NX`, `ZINCRBY`, `LMOVE`, `GETDEL` etc. are atomic. Most race conditions can be solved by choosing the right single command.

### 5.2 MULTI / EXEC

Queue several commands and execute them together, with no other client's command running in between.

```bash
MULTI
DECRBY stock:100 1
LPUSH orders:pending 501
EXEC          # runs both, returns both results
# DISCARD     # cancels the queued commands
```

**No rollback** (very common interview question):
- **Errors before EXEC** (wrong syntax, unknown command): the whole transaction is aborted (`EXECABORT`).
- **Errors during EXEC** (e.g. `INCR` on a key holding a list): that command fails, **the others still run**. Nothing is rolled back.

Redis's reasoning: runtime errors are programming errors (wrong type) that should be caught in development, and skipping rollback keeps Redis simple and fast.

Also: you can't read a value inside MULTI and use it to decide what to write next, because commands are only queued. That's what `WATCH` or Lua are for.

### 5.3 WATCH: optimistic locking

```bash
WATCH wallet:42                    # abort EXEC if this key changes before EXEC
GET wallet:42                      # → 500 (read outside the transaction)
# app checks 500 >= 200
MULTI
DECRBY wallet:42 200
EXEC                               # returns nil (aborted) if wallet:42 was modified since WATCH → retry
```
This is **check-and-set (CAS)**. Under heavy contention on the same key, retries pile up; a Lua script is better.

### 5.4 Lua scripts and Redis Functions ⭐

A Lua script runs **atomically** on the server: no other command runs while it executes. It can read, decide and write in one step, in one round trip.

```lua
-- Atomic "decrement stock only if enough is available"
-- KEYS[1] = stock key, ARGV[1] = quantity
local stock = tonumber(redis.call('GET', KEYS[1]) or '0')
local qty = tonumber(ARGV[1])
if stock >= qty then
  return redis.call('DECRBY', KEYS[1], qty)
else
  return -1
end
```
```bash
EVAL "<script>" 1 stock:100 2
SCRIPT LOAD "<script>"   # → sha; then EVALSHA <sha> 1 stock:100 2 (avoid resending the script)
```
- Pass every key the script touches in `KEYS` (required for Redis Cluster, so it can check they're in the same slot).
- **Keep scripts short:** a slow script blocks the whole server (`busy-reply-threshold`, formerly `lua-time-limit`, defaults to 5s; after that, Redis only accepts `SCRIPT KILL` or `SHUTDOWN NOSAVE`).
- **Redis Functions (7.0+):** named, persistent libraries of Lua functions loaded with `FUNCTION LOAD` and called with `FCALL`; they survive restarts and replicate, replacing ad-hoc `EVAL` for shared logic.

### 5.5 Pipelining

Send many commands without waiting for each reply, then read all replies. It cuts **network round trips**, not atomicity: other clients' commands can interleave.

```
Without pipelining: 1000 commands × 1ms RTT = 1 second
With pipelining:    1000 commands in ~1–2 round trips = a few ms
```

| | Pipelining | MULTI/EXEC | Lua script |
|---|---|---|---|
| Purpose | Fewer round trips | Run a batch without interleaving | Atomic read-decide-write |
| Atomic | No | Yes (isolated, no rollback) | Yes |
| Can use intermediate results | No | No | **Yes** |
| Round trips | 1 batch | 1 batch (with pipelining) | 1 |

### Interview Questions

**Q1. Are Redis transactions ACID?** ⭐⭐
Partially. MULTI/EXEC gives isolation (no other command runs in between) and all queued commands are executed, but there's no rollback: if one command fails at runtime, the others still apply. Durability depends on persistence settings (and async replication). Consistency is up to your logic. For conditional logic, use WATCH (optimistic) or Lua scripts.

**Q2. What does WATCH do?** ⭐⭐
It makes a following EXEC fail (return nil) if any watched key was modified after WATCH, implementing optimistic check-and-set: read the value, decide, then MULTI/EXEC the write; if someone else changed it, retry.

**Q3. MULTI/EXEC vs Lua script vs pipelining?** ⭐⭐
Pipelining only batches commands to save round trips; it's not atomic. MULTI/EXEC runs queued commands together without interleaving, but you can't branch on intermediate results. Lua scripts run atomically on the server and can read, compute and write, so they're the tool for logic like "decrement only if enough stock" or "acquire a rate-limit slot".

**Q4. What's the danger of Lua scripts?**
They block the single thread while running. A slow or looping script freezes Redis for every client. Keep them small and O(small n), declare all keys, and avoid scripts that scan large collections.

---

## 6. Messaging

**Priority:** ⭐⭐ commonly asked

### 6.1 Pub/Sub

```bash
SUBSCRIBE order-events                 # subscriber
PUBLISH order-events '{"orderId":501,"status":"SHIPPED"}'   # returns number of receivers
PSUBSCRIBE order-*                     # pattern subscribe
```
- **Fire-and-forget, at-most-once:** messages go only to clients connected *right now*. If a subscriber is offline or slow (output buffer limit), it misses messages; nothing is stored.
- Use for: real-time notifications where loss is acceptable (live dashboards, WebSocket fan-out between app servers, cache invalidation broadcasts).
- In Redis Cluster, classic pub/sub broadcasts every message to all nodes; **sharded pub/sub** (`SPUBLISH`/`SSUBSCRIBE`, 7.0+) scopes channels to slots.

### 6.2 Lists as queues

`LPUSH` + `BRPOP` is a simple queue, but if a worker pops a job and crashes, the job is lost. The **reliable queue** pattern uses `LMOVE`/`BLMOVE` to atomically move the job to a per-worker processing list, removes it after finishing, and a reaper re-queues jobs stuck in processing lists. (This is what libraries like BullMQ, Sidekiq and RQ build on.)

### 6.3 Streams ⭐⭐

An append-only log: entries with auto-generated IDs (`<ms timestamp>-<sequence>`), persisted, replayable, with **consumer groups**, like a lightweight Kafka inside Redis.

```bash
# Producer
XADD orders:stream MAXLEN ~ 100000 * orderId 501 status PLACED
#                  ^ cap length (approximate, efficient)   ^ auto ID

# Create a consumer group (reads from the beginning: 0, or only new: $)
XGROUP CREATE orders:stream warehouse-svc $ MKSTREAM

# Consumer "worker-1" reads new messages for the group
XREADGROUP GROUP warehouse-svc worker-1 COUNT 10 BLOCK 5000 STREAMS orders:stream >

# After processing successfully
XACK orders:stream warehouse-svc 1762600000000-0
```

**How consumer groups work:**
- Each message is delivered to **one** consumer in the group (load balancing); different groups each get every message (fan-out to services).
- Delivered but unacknowledged messages sit in the group's **Pending Entries List (PEL)** with a delivery count.
- If a consumer crashes, another consumer can take over its pending messages: `XPENDING` to inspect, `XCLAIM` / `XAUTOCLAIM` (6.2+) to claim messages idle longer than N ms.
- This gives **at-least-once** delivery, so consumers must be idempotent.

**Handling poison messages in Streams** ⭐⭐⭐: a message that always fails stays in the PEL and gets reclaimed and retried forever. Pattern:
```
1. XAUTOCLAIM messages idle > 60s
2. Check the delivery count (returned by XPENDING)
3. If count > 5 → XADD it to orders:stream:dlq with the error, then XACK it in the main stream
4. Alert on the DLQ; fix and replay later
```

### 6.4 Redis Streams vs Kafka

| | Redis Streams | Kafka |
|---|---|---|
| Storage | In memory (bounded by RAM; trim with MAXLEN/MINID) | On disk, retained for days/forever |
| Throughput / scale | Very fast, limited by one node's memory per stream | Massive, partitions spread across brokers |
| Ordering | Per stream | Per partition |
| Consumer groups | Yes, with per-message acks | Yes, with offset commits per partition |
| Replay | Yes (within retention) | Yes, long-term |
| Ops complexity | Low (you probably run Redis already) | Higher |
| Use | Lightweight job/event queues, moderate volume | Event backbone, analytics pipelines, CDC, high volume |

### Interview Questions

**Q1. Redis Pub/Sub vs Streams?** ⭐⭐
Pub/Sub is fire-and-forget: messages go only to currently connected subscribers and are never stored, so offline consumers miss them (at-most-once). Streams persist messages, support consumer groups with acknowledgements, pending lists and reclaiming from crashed consumers, and allow replay (at-least-once). Use Pub/Sub for ephemeral real-time notifications, Streams for work queues and events that must not be lost.

**Q2. How do consumer groups handle a crashed consumer?** ⭐⭐
Messages delivered to a consumer stay in the group's pending entries list until acknowledged with XACK. Another consumer can find long-idle pending messages with XPENDING and take them over with XCLAIM or XAUTOCLAIM, which also increments their delivery count.

**Q3. How would you handle a message that keeps failing in a Redis Stream?** ⭐⭐⭐
Track the delivery count from the pending entries list. After N failed deliveries, copy the message (with error details) to a dead-letter stream and XACK it in the original stream so it stops blocking. Alert on the dead-letter stream, fix the bug or data, and replay. Keep consumers idempotent since redelivery is normal.

**Q4. When would you choose Kafka over Redis Streams?** ⭐⭐
When you need very high throughput, long retention on disk, many partitions spread across brokers, a large ecosystem (Kafka Connect, stream processing), or the stream is the backbone of many services. Redis Streams fit moderate volumes, short retention and teams that want minimal extra infrastructure.

---

## 7. Replication and High Availability

**Priority:** ⭐⭐ commonly asked

### 7.1 Replication

```
REPLICAOF primary-host 6379        # on the replica
```
- **Asynchronous:** the primary acknowledges writes immediately, then streams them to replicas.
- **Full sync:** a new replica gets an RDB snapshot from the primary (via fork, or diskless directly over the socket), loads it, then receives the buffered command stream.
- **Partial resync (`PSYNC`):** each primary has a **replication ID** and an **offset**. The primary keeps a circular **replication backlog** (`repl-backlog-size`, 1MB by default; usually raise it). If a replica disconnects briefly and its offset is still in the backlog, it gets only the missing bytes instead of a full resync. If not, it does a full sync again.
- Replicas are read-only by default and can serve reads (stale by the replication lag).

### 7.2 Sentinel

Provides **automatic failover** for a primary-replica setup without Cluster.

```
   Sentinel 1     Sentinel 2     Sentinel 3        (at least 3, on separate machines)
        \             |              /
         └──── monitor primary + replicas ────┘

1. A Sentinel can't reach the primary  → marks it "subjectively down" (SDOWN)
2. Enough Sentinels agree (≥ quorum)   → "objectively down" (ODOWN)
3. Sentinels elect a leader Sentinel (needs a majority of Sentinels)
4. Leader promotes the best replica, reconfigures the others, tells clients
```
Clients ask Sentinel "who is the primary for `mymaster`?" (Sentinel-aware client libraries do this automatically).

### 7.3 Data loss and split-brain ⭐⭐⭐

Because replication is asynchronous, **acknowledged writes can be lost** on failover:
```
1. Client writes to primary → primary says OK
2. Primary crashes before sending it to the replica
3. Replica is promoted → that write is gone
```

**Split-brain:** the old primary is cut off from Sentinels but still reachable by some clients, so it keeps accepting writes, which are discarded when it rejoins as a replica.

**Mitigations:**
```
min-replicas-to-write 1       # primary refuses writes unless ≥1 replica is connected...
min-replicas-max-lag 10       # ...with lag ≤ 10s
```
This bounds the window of lost writes (an isolated primary stops accepting writes after ~10s). `WAIT numreplicas timeout` makes a client wait until N replicas acknowledged its writes, which reduces but doesn't eliminate loss (Redis is still not strongly consistent). `WAITAOF` (7.2+) waits for writes to be fsynced to AOF locally and/or on replicas.

**Bottom line:** Redis HA favours availability. Don't use it as the only store for data you can't afford to lose (payments, orders).

### Interview Questions

**Q1. How does Redis replication work?** ⭐⭐
Asynchronously. A new replica receives an RDB snapshot and then a stream of write commands. Each primary has a replication ID and offset, and keeps a backlog buffer; after a short disconnect, the replica resumes with a partial resync (PSYNC) from its offset if it's still in the backlog, otherwise it does a full resync.

**Q2. What is Redis Sentinel?** ⭐⭐
A distributed monitoring system for primary-replica Redis. Multiple Sentinels watch the primary; when a quorum agrees it's down, a majority-elected leader Sentinel promotes a replica, reconfigures the others, and informs clients through service discovery. Use at least three Sentinels on independent machines.

**Q3. Can Redis lose acknowledged writes?** ⭐⭐⭐
Yes. Replication is asynchronous, so a write acknowledged by the primary can be lost if the primary fails before replicating it and a replica is promoted; a partitioned old primary can also accept writes that are discarded later. `min-replicas-to-write`/`min-replicas-max-lag` limit the window and `WAIT` lets a client wait for replica acknowledgements, but Redis doesn't provide strong consistency.

**Q4. Sentinel or Cluster?** ⭐⭐
Sentinel gives high availability for a single dataset that fits on one primary. Cluster gives both HA and horizontal scaling by sharding data across multiple primaries, at the cost of multi-key operation restrictions. Use Sentinel when one node's memory and CPU are enough; Cluster when they're not.

---

## 8. Redis Cluster

**Priority:** ⭐⭐ commonly asked

**What it is:** Redis's built-in sharding with automatic failover. Data is split across multiple primaries, each with replicas.

### 8.1 Hash slots

```
slot = CRC16(key) mod 16384

Primary A: slots 0–5460        (+ replica A1)
Primary B: slots 5461–10922    (+ replica B1)
Primary C: slots 10923–16383   (+ replica C1)
```
- The key space is divided into **16,384 slots**; each primary owns a subset.
- Adding a node = moving some slots (and their keys) to it; no rehashing of everything.
- **Why 16,384?** Nodes exchange heartbeat messages containing a bitmap of the slots they own: 16,384 bits = 2KB per message, small enough to gossip frequently. Redis Cluster targets at most ~1,000 primaries, so 16,384 slots are enough for fine-grained distribution.

### 8.2 Hash tags

Only the part of the key inside `{...}` is hashed:
```bash
user:{42}:profile
user:{42}:cart        # same slot as above → multi-key ops allowed
```

### 8.3 Client routing: MOVED and ASK

```
Client → Node A: GET product:100
Node A: -MOVED 7438 10.0.0.2:6379     # "slot 7438 lives on B, permanently; update your map"
Client → Node B: GET product:100      # cluster-aware clients cache the slot map, so this is rare

During slot migration from B to C:
Node B: -ASK 7438 10.0.0.3:6379       # "this particular key already moved; ask C this one time"
Client → Node C: ASKING; GET product:100   # don't update the map: migration isn't finished
```

### 8.4 Cluster bus, failure detection and failover

Nodes talk over a separate **cluster bus** (port + 10000) using a **gossip protocol**: they ping each other, share views of which nodes seem down, and when a majority of primaries agree a primary has failed, its replicas hold an election and one is promoted. No Sentinel needed.

### 8.5 Limitations ⭐

- **Multi-key commands** (`MGET`, `SUNION`, transactions, Lua scripts) only work if all keys are in the **same slot** (otherwise `CROSSSLOT` error). Design keys with hash tags for data that's used together.
- Only database 0 (`SELECT` isn't supported).
- Hot keys don't spread: a single key always lives on one node.
- Same async replication data-loss caveats as Sentinel.
- Over-using one hash tag defeats sharding (everything in one slot).

### Interview Questions

**Q1. How does Redis Cluster distribute data?** ⭐⭐
Each key maps to one of 16,384 hash slots via CRC16(key) mod 16384, and each primary owns a range of slots. Clients cache the slot-to-node map and send commands directly to the right node; a MOVED redirect updates the map when slots change. Adding nodes moves slots, not individual keys by rehashing.

**Q2. What are hash tags and why are they needed?** ⭐⭐
If a key contains `{...}`, only that substring is hashed, so keys like `cart:{42}` and `profile:{42}` land in the same slot. This is needed because multi-key commands, MULTI/EXEC and Lua scripts in Cluster only work when all keys are in one slot.

**Q3. MOVED vs ASK?** ⭐⭐⭐
MOVED means the slot permanently belongs to another node; the client should update its slot map and retry there. ASK happens during a slot migration: this particular key has already moved, so the client should send `ASKING` plus the command to the target node once, without updating its map, since the migration isn't complete.

**Q4. Why 16,384 slots?** ⭐⭐⭐
Nodes include a bitmap of their slots in every heartbeat; 16,384 bits is 2KB, keeping gossip messages small, while still giving enough granularity for clusters up to about 1,000 primaries.

---

## 9. Caching Patterns

**Priority:** ⭐ must-know

**What it is:** Strategies for how the application reads from and writes to the cache and the database.

**Why caching:** ShopKart's product page gets 50,000 requests/second during a sale. The database can handle maybe 5,000 such queries per second. A cache with a 95% hit rate means only 2,500 queries/sec reach the database.

### 9.1 Cache-aside (lazy loading) ⭐

The **application** manages the cache. The most common pattern.

```javascript
async function getProduct(id) {
  const cached = await redis.get(`product:${id}`);
  if (cached) return JSON.parse(cached);                     // cache hit

  const product = await db.products.findById(id);           // cache miss → database
  if (product) {
    await redis.set(`product:${id}`, JSON.stringify(product), { EX: 600 });
  }
  return product;
}

async function updateProduct(id, data) {
  await db.products.update(id, data);                       // 1. update the database
  await redis.del(`product:${id}`);                         // 2. delete (invalidate) the cache
}
```

| Pros | Cons |
|---|---|
| Only requested data is cached | First request for each key is slow (miss) |
| Cache failure isn't fatal: fall back to the database | Possible stale data (see §10) |
| Simple, works with any database | Cache logic spread through application code |

### 9.2 Read-through

The **cache layer** (a library or the cache provider) loads from the database on a miss; the application only talks to the cache. Same behaviour as cache-aside, but encapsulated (e.g. a caching library with a loader function, NCache, DAX for DynamoDB).

### 9.3 Write-through

Writes go to the cache, and the cache layer **synchronously** writes to the database.

| Pros | Cons |
|---|---|
| Cache always fresh for written data | Every write pays both latencies |
| Reads after writes always hit | Caches data that may never be read |

### 9.4 Write-behind (write-back)

Writes go to the cache only; the cache layer flushes them to the database **asynchronously**, often batched.

| Pros | Cons |
|---|---|
| Very fast writes; absorbs write spikes; batching reduces database load | **Data loss** if the cache fails before flushing |
| | Database is temporarily behind; complex consistency |

Real life: view counters or "likes": increment in Redis instantly, flush aggregated counts to the database every minute. Losing a few seconds of view counts is acceptable; losing payments isn't.

### 9.5 Write-around

Writes go straight to the database, and the cache is filled only on reads (combined with cache-aside). Good for data written once and rarely read immediately (logs, archives).

### 9.6 Summary

| Pattern | Read path | Write path | Best for |
|---|---|---|---|
| Cache-aside | App checks cache, loads DB on miss | App writes DB, deletes cache | **General purpose, most common** |
| Read-through | Cache loads DB on miss | — | Cleaner code with a caching layer |
| Write-through | From cache | Cache + DB synchronously | Read-after-write heavy, need fresh cache |
| Write-behind | From cache | Cache now, DB later | Write-heavy, loss-tolerant (counters, metrics) |
| Write-around | Cache-aside | DB only | Write-once, rarely re-read data |

### 9.7 TTLs and cache warming

**TTL choice** balances freshness vs hit rate:
- Product details: 5–30 minutes (with invalidation on update).
- Configuration / feature flags: 30–60 seconds.
- Expensive aggregated data (homepage "top sellers"): refresh in the background every few minutes.
- Always set a TTL on cache entries, even with explicit invalidation, as a safety net against missed invalidations.

**Cache warming:** pre-load hot keys before traffic arrives: after a deploy, after a cache restart, or before a flash sale (load the sale products at 11:55 for a 12:00 sale). Otherwise the first wave of traffic all misses and hits the database (a cold-start stampede).

**What to cache:** data that's read often, expensive to compute, and tolerates slight staleness. **What not to cache:** rapidly changing data where staleness is harmful (stock during checkout, account balances) unless you design for it.

### Interview Questions

**Q1. Explain cache-aside.** ⭐
The application checks the cache first; on a hit it returns the cached value; on a miss it reads the database, stores the result in the cache with a TTL and returns it. On updates, it writes the database and deletes the cache key so the next read reloads fresh data. It's the most common pattern because it's simple and tolerates cache failures.

**Q2. Write-through vs write-behind?** ⭐
Write-through writes to the cache and synchronously to the database, keeping both consistent at the cost of write latency. Write-behind writes to the cache and flushes to the database asynchronously, giving fast writes and batching but risking data loss if the cache fails before flushing. Write-behind suits counters/analytics; write-through suits data that's read right after it's written.

**Q3. How do you choose a TTL?** ⭐⭐
Based on how stale the data may be and how expensive a miss is. Frequently changing or correctness-sensitive data gets short TTLs (or isn't cached); stable data gets longer ones. Combine TTLs with explicit invalidation on writes, add random jitter to avoid mass expiry, and always set some TTL as a safety net.

---

## 10. Cache Consistency

**Priority:** ⭐ must-know (a favourite system design follow-up)

**The problem:** the database and the cache are two separate systems with no shared transaction, so they can disagree.

### 10.1 Update the cache or delete it? ⭐

**Option A: update DB, then update cache**
```
T1: update DB price = 100
T2: update DB price = 200
T2: update cache = 200
T1: update cache = 100        ← cache says 100, DB says 200 → stale until TTL
```
Concurrent writers can apply cache updates in a different order from the database updates. It also wastes work computing cache values that may never be read.

**Option B: update DB, then delete cache** ✅ (preferred)
The next read reloads the latest value from the database. Deletes are idempotent and order doesn't matter (two deletes = one delete).

**Why not delete the cache *before* updating the DB?**
```
T1 (writer): delete cache
T2 (reader): cache miss → reads OLD value from DB → puts OLD value in cache
T1 (writer): updates DB
→ cache holds the old value until TTL
```

### 10.2 The remaining race with "update DB then delete cache" ⭐⭐

Rare, but possible:
```
1. Cache is empty for key X
2. Reader: cache miss, reads DB → gets OLD value (v1)
3. Writer: updates DB to v2
4. Writer: deletes cache (nothing to delete)
5. Reader: writes OLD value v1 into the cache   ← stale
```
It requires the reader's DB read to happen before the write but its cache write after the delete: unlikely, because cache writes are much faster than DB writes, but possible under load or GC pauses.

**Mitigations:**
1. **Short TTLs** bound how long staleness lasts.
2. **Delayed double delete:** delete the cache, update the DB, then delete the cache **again** after a short delay (e.g. 500ms, longer than a typical read-then-cache cycle), via a delayed job.
3. **Versioned writes:** store a version with the cached value and only overwrite if newer (Lua script compares versions).
4. **Lease / "set only if absent since miss" tokens:** Facebook's memcache paper: on a miss the reader gets a lease token; a delete invalidates outstanding leases, so the stale `set` in step 5 is rejected.
5. **CDC-driven invalidation (most robust):** the app only writes the DB; a CDC pipeline (Debezium reading the binlog/WAL) emits change events, and a consumer deletes the corresponding cache keys. Invalidation happens after commit, can't be skipped by a crashed app server, and retries on failure.

### 10.3 What if the cache delete fails?

The DB update succeeded, but the network call to delete the cache failed. Options: retry with backoff; put the delete on a queue to retry asynchronously; rely on TTL as the last line of defence; or use CDC-based invalidation so deletion is driven by the database log.

### 10.4 Is perfect consistency possible?

Not without distributed transactions, which defeat the purpose of a cache. The practical goal is **eventual consistency with a bounded staleness window**, and **not using cached values for critical decisions**: during checkout, read stock and prices from the database (or a strongly consistent source), even if the product page showed cached values.

### Interview Questions

**Q1. Should you update or delete the cache when data changes?** ⭐
Delete (invalidate) it after updating the database. Updating the cache from concurrent writers can apply values out of order and leave stale data, and it wastes effort on values that might not be read. After a delete, the next read loads the current value from the database.

**Q2. Why update the database first and then delete the cache, not the reverse?** ⭐
If you delete first, a concurrent reader can miss, read the old value from the database (not yet updated), and put it back in the cache, which stays stale until TTL. Updating the database first and deleting afterwards leaves only a much rarer race, which TTLs or delayed double deletes handle.

**Q3. How do you keep cache and database consistent at scale?** ⭐⭐
Cache-aside with DB-first-then-delete, TTLs as a safety net, retries for failed deletes, and for robustness, CDC-based invalidation: consume the database's change log and delete affected keys after each commit. Add versioning or delayed double delete where races matter. Accept eventual consistency for displays, and read critical values (stock at checkout, balances) from the source of truth.

**Q4. What is delayed double delete?** ⭐⭐
Delete the cache, update the database, then delete the cache again after a short delay. The second delete removes any stale value a concurrent reader may have written between the first delete and the database commit. The delay should exceed a typical read-and-repopulate time.

---

## 11. Cache Problems

**Priority:** ⭐ must-know (the "three cache problems" are asked very often)

### 11.1 Cache penetration ⭐

**What:** requests for keys that **don't exist** in the cache **or** the database. Every request misses and hits the database. Often malicious: an attacker requests `/products/999999999`, `/products/-1`, random IDs.

**Fixes:**
1. **Cache null results** with a short TTL:
   ```javascript
   if (!product) await redis.set(`product:${id}`, "NULL", { EX: 60 });
   ```
   Downside: many distinct bad keys fill the cache, so keep the TTL short.
2. **Bloom filter** of all valid IDs in front of the cache: if it says "definitely not present", reject immediately without touching the cache or database. (False positives are possible but only let a few bad requests through; there are no false negatives.) Redis Stack/8 has `BF.ADD` / `BF.EXISTS`.
3. **Input validation:** reject malformed IDs (negative, wrong format) at the API layer; rate-limit abusive clients.

### 11.2 Cache breakdown (hot key expiry) ⭐

**What:** one **very hot** key expires (e.g. the flash-sale product page). In the moment before it's rebuilt, thousands of concurrent requests miss and all query the database for the same thing.

**Fixes:**
1. **Mutex / single-flight rebuild:** only one request rebuilds; others wait briefly or serve stale data.
   ```javascript
   async function getHotProduct(id) {
     const v = await redis.get(`product:${id}`);
     if (v) return JSON.parse(v);
     const gotLock = await redis.set(`lock:product:${id}`, "1", { NX: true, PX: 5000 });
     if (gotLock) {
       try {
         const product = await db.products.findById(id);
         await redis.set(`product:${id}`, JSON.stringify(product), { EX: 600 });
         return product;
       } finally {
         await redis.del(`lock:product:${id}`);
       }
     }
     await sleep(50);                      // someone else is rebuilding; retry shortly
     return getHotProduct(id);
   }
   ```
2. **Logical expiry:** store `{ value, expiresAt }` with **no Redis TTL**. When a reader sees it's logically expired, it returns the **stale value** immediately and triggers one background refresh (guarded by a lock). Users never wait; the database sees one query.
3. **Never expire known hot keys;** refresh them proactively with a background job.

### 11.3 Cache avalanche ⭐

**What:** **many keys expire at the same time** (all loaded with the same TTL at startup or by a batch job), or the **cache cluster goes down**. A huge wave of misses hits the database at once and can take it down, and then the whole site.

**Fixes:**
1. **TTL jitter:** add randomness: `EX: 600 + random(0, 120)` so expirations spread out.
2. **High availability for the cache:** replicas, Sentinel/Cluster, multi-AZ.
3. **Multi-level cache:** a small in-process cache (e.g. LRU in each app instance, a few seconds TTL) in front of Redis.
4. **Circuit breakers, rate limiting and load shedding** to protect the database: return a degraded response ("prices loading…") rather than letting it collapse.
5. **Warm the cache** before traffic after a restart.

### 11.4 Cache stampede / thundering herd ⭐⭐

The general case of "many concurrent misses for the same key cause duplicate expensive recomputation" (cache breakdown is one instance).

**Fixes:**
- **Request coalescing / single-flight:** within one app instance, concurrent requests for the same key share one in-flight database call (e.g. Go's `singleflight`, a promise map in Node). Across instances, use a Redis lock as above.
- **Probabilistic early expiration (XFetch):** each reader, as the TTL nears its end, has a small, increasing probability of refreshing the value early. One request refreshes before expiry, so there's never a mass miss.
- **Stale-while-revalidate:** serve the stale value while one request refreshes it (same idea as logical expiry; also an HTTP caching directive).

### 11.5 Cache poisoning ⭐⭐

**What:** wrong, stale or malicious data gets into the cache and is served to many users until it expires.

**Causes and fixes:**

| Cause | Example | Fix |
|---|---|---|
| Race conditions | The stale write in §10.2 | Delete-on-write, versioned values, short TTLs, CDC invalidation |
| Caching error responses | DB timeout → app caches an empty product list for 10 minutes | Don't cache errors, or cache them for seconds only |
| Bad cache keys | Key ignores a parameter (`product:100` cached for currency INR served to a USD user; personalised data cached under a shared key, leaking one user's data to others) | Include every varying input (user/tenant/locale/currency) in the key |
| Unvalidated input in keys or values | User-controlled header ends up in a shared cache entry (web cache poisoning) | Validate input; never cache based on unkeyed inputs |
| Bugs in a deploy | New code writes a malformed shape into the cache | Version the key prefix (`v2:product:100`), so a rollback reads old keys |

Being able to **flush by prefix or version** quickly is an important operational tool.

### Interview Questions

**Q1. What are cache penetration, breakdown and avalanche, and how do you handle each?** ⭐
**Penetration:** requests for keys that don't exist anywhere always miss and hit the database; fix with caching null results briefly, a Bloom filter of valid keys, and input validation. **Breakdown:** a single hot key expires and concurrent requests all rebuild it; fix with a mutex/single-flight rebuild, logical expiry serving stale data while refreshing, or not expiring hot keys. **Avalanche:** many keys expire together or the cache fails, flooding the database; fix with TTL jitter, cache HA, multi-level caching, circuit breakers and warming.

**Q2. How does a Bloom filter prevent cache penetration?** ⭐⭐
It's a compact probabilistic set of all valid keys. Before hitting the cache or database, check the filter: if it says the key definitely doesn't exist, reject immediately. It may occasionally say "maybe" for a non-existent key (false positive), letting a few through, but never says "no" for a key that exists. The filter must be updated when new valid keys are created.

**Q3. What is a cache stampede and how do you prevent it?** ⭐⭐
Many concurrent requests miss the same key at once and all recompute it, overloading the database. Prevent it with request coalescing (one in-flight load per key), a distributed lock so only one instance rebuilds, serving stale values while refreshing (logical expiry / stale-while-revalidate), and probabilistic early refresh before the TTL ends.

**Q4. What is cache poisoning in a backend context?** ⭐⭐
Incorrect data stored in a cache and served to many users: stale values written by a race, cached error responses, cache keys that miss a varying parameter (serving one user's or currency's data to others), or data shaped by unvalidated input. Prevent it with careful key design, not caching errors (or only briefly), invalidation strategies that avoid races, input validation, and versioned key prefixes that make purges and rollbacks easy.

---

## 12. Use-Case Designs

**Priority:** ⭐ must-know (these are classic system design building blocks)

### 12.1 Distributed lock ⭐

**Goal:** only one process performs an action at a time across many servers (e.g. only one worker processes order 501; only one instance runs the nightly job).

**Acquire:**
```bash
SET lock:order:501 <unique-token> NX PX 30000
# NX: only if the key doesn't exist  ·  PX 30000: auto-expire after 30s (if the holder crashes)
# returns OK → you hold the lock; nil → someone else does
```

**Release safely:** only delete the lock if **you** still own it, atomically (a Lua script). A plain `DEL` could delete *someone else's* lock if yours already expired.
```lua
-- KEYS[1] = lock key, ARGV[1] = my token
if redis.call("GET", KEYS[1]) == ARGV[1] then
  return redis.call("DEL", KEYS[1])
else
  return 0
end
```

**Problem: the work takes longer than the TTL.** The lock expires, another process acquires it, and now two are working. Fixes: choose a generous TTL, and **renew** the lock periodically while working (a "watchdog" thread extends the TTL if the token still matches; Redisson does this).

**Redlock** ⭐⭐⭐: acquire the lock on a majority (e.g. 3 of 5) of **independent** Redis primaries within a time limit, to survive a single Redis failure.

**Martin Kleppmann's critique (2016):** no lock with a timeout is safe on its own. A process can pause (GC, VM migration, network delay) after acquiring the lock, the lock expires, another process acquires it, and then the first wakes up and writes, believing it still holds the lock. Redlock also depends on bounded clock drift.

**The fix: fencing tokens.** Each lock grant comes with a monotonically increasing number. The protected resource (e.g. the database) rejects writes with a token older than the newest it has seen:
```sql
UPDATE orders SET status = 'PROCESSED', fence = 34 WHERE id = 501 AND fence < 34;
```

**Practical guidance:**
- Redis locks are fine for **efficiency** (avoiding duplicate work, where an occasional double execution is harmless).
- For **correctness** (money, inventory), don't rely on the lock alone: use database constraints, conditional updates, fencing tokens, or a consensus-based system (ZooKeeper, etcd).

### 12.2 Rate limiter ⭐

**Goal:** e.g. at most 100 requests per minute per user/IP on the login API.

**Fixed window counter** (simplest):
```javascript
const key = `rl:${userId}:${Math.floor(Date.now() / 60000)}`;   // one key per minute
const count = await redis.incr(key);
if (count === 1) await redis.expire(key, 60);                    // (or use a Lua script / SET NX EX)
if (count > 100) return res.status(429).send("Too Many Requests");
```
Problem: **boundary burst**: 100 requests at 12:00:59 and 100 at 12:01:00 = 200 within 2 seconds.

**Sliding window log** (exact):
```lua
-- ZSET of request timestamps. KEYS[1]=key, ARGV: now(ms), window(ms), limit, unique request id
redis.call('ZREMRANGEBYSCORE', KEYS[1], 0, ARGV[1] - ARGV[2])   -- drop old entries
if redis.call('ZCARD', KEYS[1]) < tonumber(ARGV[3]) then
  redis.call('ZADD', KEYS[1], ARGV[1], ARGV[4])
  redis.call('PEXPIRE', KEYS[1], ARGV[2])
  return 1    -- allowed
end
return 0      -- rejected
```
Exact, but stores every request: memory grows with the limit.

**Sliding window counter** (good approximation, cheap): keep counters for the current and previous fixed windows and weight the previous one by how much of it overlaps the sliding window:
```
estimate = current_count + previous_count × (1 − elapsed_fraction_of_current_window)
```

**Token bucket** (allows controlled bursts):
- A bucket holds up to `capacity` tokens and refills at `rate` tokens/second.
- Each request takes a token; no token → rejected.
- Store `{tokens, last_refill}` in a hash and update it in a Lua script (refill based on elapsed time, then try to take a token) so it's atomic.
- Used by AWS API Gateway, Stripe and most public APIs.

**Leaky bucket:** requests enter a queue processed at a constant rate; smooths traffic (more of a shaping/queueing algorithm).

| Algorithm | Accuracy | Memory | Bursts |
|---|---|---|---|
| Fixed window | Burst at window edges | 1 counter | Up to 2× at boundaries |
| Sliding window log | Exact | O(limit) per user | No |
| Sliding window counter | Approximate (good) | 2 counters | Smoothed |
| Token bucket | Exact rate, configurable burst | 2 values | Yes, up to capacity |

Always do the check-and-increment **atomically** (single command or Lua), or concurrent requests can all pass.

### 12.3 Leaderboard

```bash
ZINCRBY leaderboard:2026-11 49999 "seller:7"      # add sale amount
ZREVRANGE leaderboard:2026-11 0 9 WITHSCORES      # top 10
ZREVRANK leaderboard:2026-11 "seller:7"           # my rank (0-based)
ZSCORE leaderboard:2026-11 "seller:7"
ZREVRANGE leaderboard:2026-11 <rank-5> <rank+5>   # people around me
```
- Ties: equal scores sort lexicographically by member; to break ties by time, encode it in the score (e.g. `score × 10^10 + (MAX_TS − ts)`).
- Daily/weekly boards: one key per period with a TTL; `ZUNIONSTORE` to combine.
- SQL alternative: `RANK() OVER (ORDER BY score DESC)`, fine for small or infrequent leaderboards, but recomputing ranks across millions of rows on every request is expensive; Redis does it in O(log n).

### 12.4 Session store

```bash
HSET session:9f2c userId 42 role customer cartId 77
EXPIRE session:9f2c 1800          # 30 min, refreshed on each request (sliding expiry)
```
Stateless app servers share sessions through Redis, so any server can handle any request. Use `noeviction` (or a separate instance) so sessions aren't evicted under memory pressure. (Alternative: JWTs, with Redis used for revocation lists.)

### 12.5 Shopping cart

```bash
HINCRBY cart:42 product:100 1
HDEL cart:42 product:205
HGETALL cart:42
EXPIRE cart:42 2592000            # 30 days for guest carts
```
Persist to the database at checkout (or periodically for logged-in users) since a cart is valuable business data.

### 12.6 Flash sale stock counter

```bash
SET stock:flashsale:100 1000                     # load before the sale
DECR stock:flashsale:100                         # each purchase attempt
# result ≥ 0 → reserved; result < 0 → sold out (INCR back to correct it, or use a Lua check-then-decrement)
```
Redis absorbs tens of thousands of attempts per second; successful reservations go into a queue (Stream) and are written to the database asynchronously, where the real order and payment happen. Reconcile counts afterwards and release reservations that never got paid.

### 12.7 Unique counts, deduplication and idempotency

- **Daily active users:** bitmap per day, or HyperLogLog.
- **Deduplicate events/requests:** `SET dedup:<eventId> 1 NX EX 86400`: returns nil if seen already.
- **Idempotency keys for APIs:** store `idem:<key>` → status/response with `SET NX` when a request starts; a retry with the same key returns the stored response instead of charging twice. (Keep the authoritative record in the database for payments.)

### 12.8 Other common uses

- **Feed / timeline caching:** a list or sorted set of post IDs per user (fan-out on write for normal users, fan-out on read for celebrities).
- **Real-time "who's online":** a set or sorted set with last-seen timestamps.
- **Autocomplete:** sorted sets with lexicographic ranges (`ZRANGEBYLEX`) or RediSearch.
- **Geo search:** nearby drivers/stores with `GEOSEARCH`.
- **Job queues:** BullMQ (Node.js), Sidekiq (Ruby), RQ/Celery (Python) all use Redis.

### Interview Questions

**Q1. How do you implement a distributed lock with Redis?** ⭐
`SET key token NX PX ttl` to acquire (only succeeds if the key doesn't exist, auto-expires if the holder dies), and release with a Lua script that deletes the key only if its value equals your token. Renew the TTL periodically for long tasks. For correctness-critical operations, also use fencing tokens or database constraints, because a paused process can outlive its lock.

**Q2. What's the problem with Redlock?** ⭐⭐⭐
Martin Kleppmann argued that timeout-based locks can't guarantee mutual exclusion on their own: a client can pause (GC, network delay) after acquiring the lock, the lock expires, another client acquires it, and both act. Redlock also assumes bounded clock drift and process pauses. The remedy is fencing tokens checked by the resource, or a consensus system like ZooKeeper/etcd for strict correctness. Redis locks remain fine for efficiency purposes.

**Q3. Design a rate limiter with Redis.** ⭐
Pick an algorithm: a fixed window counter (`INCR` + `EXPIRE` per user per window) is simplest but allows bursts at window boundaries; a sliding window log with a sorted set is exact but memory-heavy; a sliding window counter approximates well cheaply; a token bucket stored in a hash allows controlled bursts. Implement the check-and-update atomically with a Lua script, key by user/IP/API key, return 429 with `Retry-After`, and decide whether to fail open or closed if Redis is down.

**Q4. How would you build a leaderboard for millions of users?** ⭐
A Redis sorted set: `ZINCRBY` on each score change, `ZREVRANGE` for the top N, `ZREVRANK` for a user's rank, all O(log n). Use separate keys per period with TTLs, encode tie-breakers in the score, and persist scores in the database as the source of truth so the leaderboard can be rebuilt. For hundreds of millions of members, shard by region/league or keep exact ranks only for the top tier.

**Q5. How would you handle a flash sale with 1,000 items and 1 million users?** ⭐⭐
Pre-load stock into Redis and decrement atomically (DECR or a Lua check-then-decrement) so only 1,000 attempts succeed; reject the rest immediately. Put successful reservations on a queue and create orders and payments in the database asynchronously with idempotency keys. Rate-limit per user, serve the product page from cache/CDN, and release unpaid reservations after a timeout. The database never sees the million-request spike.

---

## 13. Operations and Troubleshooting

**Priority:** ⭐⭐ commonly asked

### 13.1 Dangerous commands

| Avoid | Why | Use instead |
|---|---|---|
| `KEYS pattern` | O(N) over all keys, blocks the server for seconds on large datasets | `SCAN 0 MATCH pattern COUNT 1000` (iterative, non-blocking; may return duplicates, handle them) |
| `DEL bigkey` | Freeing millions of elements blocks the server | `UNLINK` (frees memory in a background thread); also `lazyfree-lazy-*` settings |
| `FLUSHALL` / `FLUSHDB` | Deletes everything (synchronously by default) | `FLUSHALL ASYNC`; rename/disable in production |
| `HGETALL` / `SMEMBERS` / `LRANGE 0 -1` on huge collections | Huge reply, blocks the server and the network | `HSCAN` / `SSCAN` / ranges in pages |
| `MONITOR` | Streams every command; heavy overhead | Short sessions only, or sampling |

```bash
# Production config: disable or rename dangerous commands (or use ACLs)
rename-command FLUSHALL ""
rename-command KEYS ""
```

### 13.2 Big keys and hot keys ⭐⭐

**Big key:** a single key with a very large value (a 50MB string) or a huge collection (a hash with 5 million fields).

Problems:
- Reading/writing/deleting it blocks the single thread.
- Uneven memory and network across Cluster nodes (one node fills up).
- Slow replication and failover; network saturation.

Detect: `redis-cli --bigkeys` (largest per type), `redis-cli --memkeys`, `MEMORY USAGE key`, offline RDB analysis tools.
Fix: split into smaller keys (shard a big hash by `hash(field) % 100` into `cart:{42}:0..99`), store large blobs elsewhere (S3) and keep only references, paginate, delete with `UNLINK`.

**Hot key:** a single key receiving a disproportionate share of traffic (the flash-sale product, a celebrity's profile).

Problems: in Cluster, all traffic for that key hits one node, which saturates its CPU/network while others idle.

Detect: `redis-cli --hotkeys` (requires an LFU eviction policy), `MONITOR` sampling, client-side metrics, cloud provider key analytics.
Fix:
- **Local in-process cache** in each app instance for a few seconds (absorbs most reads).
- **Replicate the key:** store copies `product:100:copy1..N` across slots and read a random copy.
- **Read from replicas** for that data.
- For hot **write** counters: split into N sub-counters and sum them on read.

### 13.3 Monitoring

```bash
INFO                    # memory, clients, persistence, replication, stats (hits/misses), keyspace
INFO memory             # used_memory, used_memory_rss, mem_fragmentation_ratio
INFO stats              # keyspace_hits, keyspace_misses → hit ratio; evicted_keys; expired_keys
SLOWLOG GET 10          # commands slower than slowlog-log-slower-than (10ms default)
LATENCY DOCTOR          # human-readable latency analysis
MEMORY DOCTOR
CLIENT LIST             # connected clients, output buffers
```

Key metrics to alert on:
- **Memory usage vs maxmemory**, and **evicted_keys** (unexpected evictions = undersized).
- **Cache hit ratio** = hits / (hits + misses).
- **Latency** (p99), **slowlog** entries.
- **Connected clients** and **blocked clients**.
- **Replication lag** / link status.
- **`mem_fragmentation_ratio`** (RSS / used memory): ≫ 1.5 means fragmentation (`activedefrag yes` helps); < 1 means the OS is **swapping**, which is very bad for Redis.
- **Fork time** (`latest_fork_usec`) for persistence.

### 13.4 Memory optimization

- Set TTLs on everything that's a cache.
- Use **hashes for small objects** (listpack encoding is compact) instead of many separate keys.
- Short key names in very large keyspaces (`u:42:s` vs `user:42:session`) matter at hundreds of millions of keys.
- Compress large values on the client (gzip/snappy/msgpack).
- Avoid storing what you can recompute cheaply.
- Leave headroom: RDB/AOF rewrite forks can use extra memory via copy-on-write; set `maxmemory` to ~70–80% of RAM.

### 13.5 Security

- Never expose Redis to the internet. **Protected mode** (on by default) refuses external connections when no password is set; thousands of exposed instances have been compromised (attackers write SSH keys or cron jobs via `CONFIG SET dir` + `SAVE`).
- **ACLs (Redis 6+):** per-user permissions: `ACL SETUSER app on >password ~cache:* +get +set +del` (limit keys and commands).
- Require auth, enable **TLS** (6.0+), bind to private interfaces, run as a non-root user, disable/rename `CONFIG`, `FLUSHALL`, `KEYS`, `DEBUG`.

### 13.6 Client-side caching ⭐⭐⭐

Redis 6+ supports **server-assisted client-side caching** (`CLIENT TRACKING`): the client caches values locally, and Redis sends invalidation messages when those keys change. You get in-process speed for hot keys without manual invalidation logic.

### Interview Questions

**Q1. Why shouldn't you use KEYS in production?** ⭐
It scans the entire keyspace in one blocking O(N) operation; with millions of keys it stalls Redis for seconds, and every other client waits. Use SCAN, which iterates with a cursor in small steps, or maintain your own index (a set of related keys).

**Q2. What is a big key, why is it a problem, and how do you fix it?** ⭐⭐
A key with a very large value or collection. Operations on it (reads, deletes, expirations, migrations) block the single thread and saturate the network, and it unbalances memory in a cluster. Detect it with `--bigkeys`/`MEMORY USAGE`, split it into smaller keys, store big blobs outside Redis, paginate access, and delete with UNLINK.

**Q3. How do you handle a hot key?** ⭐⭐
Find it (`--hotkeys` with LFU, client metrics), then spread or absorb the load: a short-lived in-process cache in each app instance, multiple copies of the key across slots with random reads, reading from replicas, or splitting a hot counter into sub-counters summed on read.

**Q4. DEL vs UNLINK?** ⭐⭐
DEL frees the memory synchronously, which for a huge collection can block Redis. UNLINK removes the key from the keyspace immediately and frees the memory in a background thread, so it doesn't block. Use UNLINK for potentially large keys (and enable lazyfree options for evictions and expirations).

**Q5. Redis latency suddenly spiked. How do you investigate?** ⭐⭐
Check SLOWLOG for slow commands (KEYS, big HGETALL, long Lua), `LATENCY DOCTOR`, `INFO` for memory/fragmentation, evictions and swapping, `latest_fork_usec` (RDB/AOF rewrite forks on a big dataset), client count and output buffers, big/hot keys, and network or CPU saturation on the host. Then fix the specific cause: replace blocking commands, split big keys, tune persistence, add capacity.
