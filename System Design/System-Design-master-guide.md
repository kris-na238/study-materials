# System Design Mastery Guide — 20 Core Topics

A structured reference covering the 20 most important system design topics, each with 10+ frequently asked interview questions.

---

## 1. Scalability Fundamentals

1. What's the difference between vertical and horizontal scaling? When would you choose one over the other?
2. What does it mean for a service to be "stateless," and why does it matter for scaling?
3. How do you scale a system that currently runs on a single server?
4. What are the bottlenecks that typically emerge first when scaling a web application?
5. What is the difference between scaling reads vs scaling writes?
6. How does statelessness help with horizontal scaling?
7. What role does session management play in a scaled, stateless architecture?
8. What is vertical scaling's ceiling, and why can't you scale vertically forever?
9. How do you design a system to auto-scale based on traffic?
10. What tradeoffs come with horizontal scaling (e.g., data consistency, coordination overhead)?

---

## 2. Load Balancing

1. What is a load balancer, and why is it needed?
2. What's the difference between Layer 4 and Layer 7 load balancing?
3. Explain round robin, least connections, and IP hash load balancing algorithms.
4. How does a load balancer handle a backend server going down?
5. What is the difference between a hardware and software load balancer?
6. How would you design a load balancer for a global, multi-region application?
7. What is DNS-based load balancing, and how does it differ from a dedicated load balancer?
8. How do sticky sessions work, and when are they necessary?
9. What is health checking, and how does a load balancer use it?
10. How do you avoid a load balancer becoming a single point of failure?

---

## 3. Caching

1. What is caching, and why does it improve system performance?
2. What's the difference between client-side, CDN, and server-side caching?
3. Explain cache-aside, write-through, and write-back caching strategies.
4. How do you handle cache invalidation? What is the hardest part of cache invalidation?
5. What is a cache stampede, and how do you prevent it?
6. What eviction policies exist (LRU, LFU, FIFO), and when would you use each?
7. How would you design a distributed cache (e.g., Redis cluster)?
8. What is the difference between Redis and Memcached?
9. How do you decide what data is a good candidate for caching?
10. What happens when a cache node fails in a distributed cache setup?

---

## 4. Databases: SQL vs NoSQL

1. What are the key differences between SQL and NoSQL databases?
2. When would you choose a NoSQL database over a relational one?
3. What are the different types of NoSQL databases (key-value, document, column-family, graph)?
4. How does indexing improve query performance, and what's the tradeoff?
5. What is a composite index, and when would you use one?
6. What is denormalization, and why is it common in NoSQL schema design?
7. How do ACID properties apply differently in SQL vs NoSQL systems?
8. What is a document database good at that a relational database struggles with?
9. How would you model a many-to-many relationship in a NoSQL database?
10. What factors would push you toward a hybrid (polyglot persistence) approach?

---

## 5. Database Scaling: Replication & Sharding

1. What is database replication, and what problem does it solve?
2. What's the difference between master-slave and master-master replication?
3. What is database sharding, and why is it needed at scale?
4. What are common sharding strategies (range-based, hash-based, geo-based)?
5. What challenges arise from sharding (e.g., cross-shard joins, rebalancing)?
6. How do you handle a "hot shard" problem?
7. What is replication lag, and how does it affect consistency?
8. How would you migrate from a single database to a sharded architecture with minimal downtime?
9. What is the difference between synchronous and asynchronous replication?
10. How do you choose a shard key for a large-scale application?

---

## 6. CAP Theorem & Consistency Models

1. What does the CAP theorem state, and why can't a distributed system have all three?
2. Give a real-world example of a system that favors availability over consistency.
3. What's the difference between strong consistency and eventual consistency?
4. What is a partition in the context of CAP, and can it really be avoided?
5. How does CAP theorem relate to PACELC?
6. What is quorum-based consistency (read/write quorums)?
7. How do systems like DynamoDB achieve eventual consistency, and what are the implications?
8. What is "read-your-writes" consistency, and why might a user expect it?
9. How would you design a system that needs strong consistency for financial transactions?
10. What tradeoffs would you explain to a stakeholder when choosing AP over CP?

---

## 7. API Design & REST

1. What makes an API "RESTful"? What are the core REST constraints?
2. How do you design idempotent APIs, and why does idempotency matter?
3. What's the difference between PUT and PATCH?
4. How would you design pagination for an API returning millions of records?
5. What is API versioning, and what strategies exist for it?
6. How do you handle rate limiting at the API layer?
7. What's the difference between REST and gRPC, and when would you choose gRPC?
8. How do you design error handling and status codes consistently across an API?
9. What is HATEOAS, and is it practical in real systems?
10. How would you secure a public-facing API?

---

## 8. Message Queues & Async Processing

1. Why would you introduce a message queue into a system design?
2. What's the difference between Kafka and RabbitMQ?
3. Explain the difference between pub-sub and point-to-point messaging.
4. How do message queues help with decoupling services?
5. What happens if a consumer fails to process a message — how do you handle retries and dead-letter queues?
6. How does Kafka achieve high throughput compared to traditional queues?
7. What is message ordering, and how do you guarantee it in a distributed queue?
8. What is "at-least-once" vs "exactly-once" vs "at-most-once" delivery?
9. How would you design a system to process events asynchronously at scale (e.g., order processing)?
10. What are the tradeoffs of introducing async processing (e.g., added complexity, eventual consistency)?

---

## 9. Microservices vs Monolithic Architecture

1. What are the pros and cons of a monolithic architecture?
2. What are the pros and cons of microservices?
3. When would you recommend starting with a monolith over microservices?
4. How do microservices communicate with each other (sync vs async)?
5. What is service discovery, and why is it needed in a microservices setup?
6. How do you handle data consistency across microservices without shared databases?
7. What is the "database per service" pattern, and what problems does it introduce?
8. How would you migrate a monolith to microservices incrementally?
9. What is an API gateway, and what role does it play in a microservices architecture?
10. What are the operational costs (monitoring, deployment, debugging) of microservices vs monoliths?

---

## 10. Consistent Hashing

1. What problem does consistent hashing solve that simple modulo hashing doesn't?
2. How does consistent hashing minimize data movement when nodes are added or removed?
3. What are virtual nodes, and why are they used in consistent hashing?
4. How is consistent hashing used in distributed caches like Redis Cluster?
5. How does consistent hashing help with load balancing across shards?
6. What happens to data when a new node joins a consistent hashing ring?
7. How would you handle uneven load distribution in a consistent hashing setup?
8. What systems in the real world use consistent hashing (e.g., DynamoDB, Cassandra)?
9. What is the relationship between consistent hashing and sharding?
10. How do you handle hot spots even with consistent hashing in place?

---

## 11. Rate Limiting

1. Why is rate limiting important for APIs and services?
2. Explain the token bucket algorithm.
3. Explain the leaky bucket algorithm, and how it differs from token bucket.
4. What is a sliding window rate limiter, and why is it more accurate than fixed window?
5. How would you implement rate limiting in a distributed system (multiple servers)?
6. Where should rate limiting be enforced — client, API gateway, or service level?
7. How do you rate limit per-user vs per-IP vs globally?
8. What happens when a rate limit is exceeded — how should the system respond?
9. How would you design a rate limiter that scales to millions of users?
10. What data store would you use to track rate limit counters, and why?

---

## 12. Distributed Consensus (Paxos/Raft)

1. What problem does a consensus algorithm solve in distributed systems?
2. What is leader election, and why is it necessary?
3. How does Raft achieve consensus at a high level?
4. What is a quorum, and why is majority agreement important?
5. How does Paxos differ from Raft in approach and complexity?
6. What happens during a network partition in a Raft-based system?
7. How does a distributed database use consensus to ensure data consistency?
8. What is split-brain, and how does consensus help prevent it?
9. Why is consensus considered expensive, and how does it affect system latency?
10. Can you name real systems that use Raft or Paxos (e.g., etcd, Zookeeper)?

---

## 13. Distributed Locks & Idempotency

1. Why would you need a distributed lock in a system with multiple servers?
2. How would you implement a distributed lock using Redis?
3. What is the "Redlock" algorithm, and what problem does it address?
4. What happens if a process holding a lock crashes before releasing it?
5. How do lock timeouts/TTLs prevent deadlocks in distributed locking?
6. What is idempotency, and why does it matter for retries in distributed systems?
7. How would you design an idempotent payment API?
8. What's the difference between optimistic and pessimistic locking?
9. How do idempotency keys work in practice?
10. What are the risks of using distributed locks too liberally in a system?

---

## 14. Fault Tolerance & Resilience

1. What is a circuit breaker pattern, and what problem does it solve?
2. How does a circuit breaker transition between closed, open, and half-open states?
3. What is the retry-with-exponential-backoff pattern, and why is jitter added?
4. What is a bulkhead pattern, and how does it prevent cascading failures?
5. How would you design a system to gracefully degrade under high load?
6. What is a timeout, and why is setting the right timeout value tricky?
7. How do you prevent a single failing service from taking down an entire system?
8. What is chaos engineering, and why do companies practice it?
9. How would you design health checks for services in production?
10. What's the difference between fault tolerance and high availability?

---

## 15. Content Delivery Networks (CDN)

1. What is a CDN, and how does it improve performance?
2. How does a CDN decide which edge server serves a given user?
3. What's the difference between a push CDN and a pull CDN?
4. How do CDNs handle cache invalidation for dynamic content?
5. What types of content are best suited for CDN caching?
6. How does a CDN help mitigate DDoS attacks?
7. What is edge computing, and how does it relate to CDNs?
8. How would you configure cache headers (TTL, cache-control) for CDN content?
9. What happens on a CDN cache miss?
10. How do CDNs handle SSL/TLS termination?

---

## 16. Search Systems

1. What is an inverted index, and why is it fundamental to search engines?
2. How does Elasticsearch achieve fast full-text search at scale?
3. What is the difference between a search index and a traditional database index?
4. How would you design autocomplete/typeahead search?
5. How do you handle relevance ranking in search results?
6. What is sharding in Elasticsearch, and how does it affect search performance?
7. How would you keep a search index in sync with a primary database?
8. What is tokenization, and why does it matter for search quality?
9. How would you design a search system that handles typos and fuzzy matching?
10. What are the tradeoffs of near-real-time search indexing?

---

## 17. Notification Systems

1. How would you design a notification system supporting email, SMS, and push?
2. How do you handle retries for failed notification delivery?
3. How would you prevent duplicate notifications from being sent to the same user?
4. How do you design a system to respect user notification preferences at scale?
5. What role do message queues play in a notification system?
6. How would you handle rate limiting for outbound notifications (e.g., to avoid spamming)?
7. How would you design a system for scheduled/delayed notifications?
8. How do third-party push notification services (APNs, FCM) fit into the architecture?
9. How would you track notification delivery and open/click analytics at scale?
10. What happens when a notification provider (e.g., Twilio) is down — how do you handle failover?

---

## 18. Security in System Design

1. What's the difference between authentication and authorization?
2. How does OAuth 2.0 work at a high level?
3. What is a JWT, and what are its advantages and risks?
4. How would you securely store passwords in a database?
5. What is an API gateway's role in securing microservices?
6. How do you prevent replay attacks in an authentication system?
7. What is the difference between symmetric and asymmetric encryption, and where is each used?
8. How would you design rate limiting and throttling to prevent abuse or brute-force attacks?
9. What is CSRF, and how do you protect against it in a web application?
10. How would you design a system to handle secure token refresh and revocation?

---

## 19. Observability: Logging, Monitoring, Tracing

1. Why is observability important in distributed systems?
2. What's the difference between logs, metrics, and traces?
3. How would you design a centralized logging system for microservices?
4. What is distributed tracing, and how does it help debug latency issues?
5. What is the role of a correlation/trace ID across services?
6. How would you set up alerting thresholds for system health metrics?
7. What is the difference between monitoring and observability?
8. How would you design a dashboard to detect anomalies in real time?
9. What tools are commonly used for observability (Prometheus, Grafana, Jaeger, ELK)?
10. How do you balance the cost of extensive logging/tracing against system performance?

---

## 20. Real-World System Design Case Studies

1. How would you design a URL shortener (like TinyURL)?
2. How would you design a rate limiter as a standalone service?
3. How would you design a chat application (like WhatsApp)?
4. How would you design a social media news feed (like Twitter/Instagram)?
5. How would you design a video streaming service (like YouTube/Netflix)?
6. How would you design a ride-sharing service (like Uber)?
7. How would you design a payment processing system?
8. How would you design a distributed file storage system (like Dropbox/S3)?
9. How would you design an e-commerce checkout and inventory system?
10. How would you design a web crawler at scale?

---

*Use each topic as a study block: understand the concept, then practice answering the questions out loud or in writing before moving to the next.*
