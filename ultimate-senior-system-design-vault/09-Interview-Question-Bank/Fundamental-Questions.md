# Fundamental System Design Questions

> Senior-level questions that test depth of understanding on core distributed systems concepts.

---

## Networking & Protocols

### Q1: What happens when you type a URL in a browser?
**Expected depth**: DNS lookup → TCP handshake → TLS handshake → HTTP request → server processing → HTML parsing → resource loading → render.

**Senior follow-ups**:
- How does DNS caching affect performance?
- What is HTTP/2 multiplexing vs HTTP/1.1 pipelining?
- How does QUIC (HTTP/3) improve latency?

---

### Q2: Explain TCP vs UDP. When would you choose UDP?
| TCP | UDP |
|-----|-----|
| Connection-oriented | Connectionless |
| Reliable, ordered delivery | Best-effort, no guarantee |
| Flow & congestion control | Minimal overhead |
| Higher latency | Lower latency |

**Use UDP for**: live video/audio streaming, DNS, gaming, IoT telemetry where occasional loss is acceptable.

---

### Q3: What is the difference between synchronous and asynchronous communication?

| Sync | Async |
|------|-------|
| Caller blocks until response | Caller continues after sending |
| Tight coupling | Loose coupling |
| Simpler error handling | Complex retry/dead-letter logic |
| REST, gRPC | Kafka, SQS, RabbitMQ |

---

## Consistency & Availability

### Q4: Explain eventual consistency with a real-world example.

**Definition**: All replicas will converge to the same value given no new updates.

**Example**: Amazon shopping cart — items added from two devices may temporarily diverge but eventually merge.

**Interview answer structure**:
1. Define eventual consistency
2. Contrast with strong consistency
3. Real example (DNS propagation, social media likes)
4. Techniques to achieve it (vector clocks, CRDTs, last-write-wins)

---

### Q5: What is the difference between consistency in CAP and ACID?

| Context | Consistency means |
|---------|-----------------|
| CAP Theorem | Every read returns the most recent write |
| ACID | Database remains in a valid state after transactions |

They are completely different concepts! CAP consistency = linearizability. ACID consistency = referential integrity / business rules.

---

### Q6: How would you achieve strong consistency in a distributed system?

Options:
- **Single master writes** — only one node accepts writes
- **Quorum reads/writes** — write to W nodes, read from R nodes where W+R > N
- **Two-phase commit (2PC)** — coordinator + participants with prepare/commit phases
- **Raft/Paxos** — consensus algorithms

Trade-off: higher latency, lower availability.

---

## Caching

### Q7: What are the main cache invalidation strategies?

1. **TTL (Time-to-Live)** — cache expires after fixed time. Simple but may serve stale data.
2. **Write-through** — update cache on every write. Consistent but adds write latency.
3. **Write-behind (Write-back)** — write to cache, async persist to DB. Fast writes but risk data loss.
4. **Cache-aside (Lazy loading)** — app checks cache, on miss loads from DB and populates cache.
5. **Event-driven invalidation** — publish event on write, subscribers invalidate their caches.

**Senior insight**: Cache invalidation is one of the two hardest problems in CS. Discuss versioned keys, conditional GETs (ETags), and staggered TTLs.

---

### Q8: How do you handle cache stampede (thundering herd)?

**Problem**: Cache key expires, thousands of requests simultaneously hit the DB.

**Solutions**:
1. **Mutex/lock** — first request acquires lock, others wait or return stale
2. **Probabilistic early expiration** — randomly expire before TTL to spread refreshes
3. **Background refresh** — async update before TTL expires
4. **Jitter on TTL** — add random offset to expiration time

---

## Databases

### Q9: What is an N+1 query problem and how do you solve it?

**Problem**: Loading a list of N entities then executing a separate query for each entity's related data → N+1 total queries.

**Solutions**:
- **Eager loading** (SQL JOIN or ORM `include`)
- **DataLoader pattern** — batch and cache per request cycle (GraphQL)
- **Denormalization** — store related data together

---

### Q10: Explain MVCC (Multi-Version Concurrency Control).

**Concept**: Instead of locking, keep multiple versions of each row. Each transaction sees a consistent snapshot.

- **Read** never blocks write
- **Write** never blocks read
- **Used by**: PostgreSQL, MySQL InnoDB, CockroachDB

**Interview angles**: How does PostgreSQL vacuum work? What is a transaction isolation level? (Read Committed, Repeatable Read, Serializable)

---

## Load Balancing

### Q11: Compare load balancing algorithms.

| Algorithm | Best for |
|-----------|---------|
| Round Robin | Equal-weight, stateless requests |
| Weighted Round Robin | Heterogeneous server capacity |
| Least Connections | Variable-length requests |
| Least Response Time | Latency-sensitive workloads |
| IP Hash | Session affinity (sticky sessions) |
| Random | Simple, surprisingly effective |
| Resource-Based | CPU/memory-aware routing |

**Senior insight**: Consistent hashing is used for cache routing (Memcached, CDNs) to minimize key remapping when nodes change.

---

### Q12: What is the difference between L4 and L7 load balancers?

| L4 (Transport Layer) | L7 (Application Layer) |
|---------------------|----------------------|
| Routes by IP + TCP/UDP port | Routes by HTTP headers, URL, cookies |
| Faster (no content inspection) | Smarter routing, SSL termination |
| No HTTP awareness | Can do A/B testing, canary routing |
| AWS NLB | AWS ALB, Nginx, HAProxy |

---

## Message Queues

### Q13: What is the difference between a message queue and a message bus (pub/sub)?

| Message Queue | Pub/Sub |
|--------------|---------|
| Point-to-point | Broadcast to multiple subscribers |
| Message consumed by one consumer | Message delivered to all subscribers |
| SQS, RabbitMQ | SNS, Kafka topics, Redis Pub/Sub |
| Work distribution | Event notification |

---

### Q14: How do you ensure exactly-once message processing?

**The challenge**: Networks are unreliable — messages may be duplicated, reordered, or lost.

Levels of delivery:
1. **At-most-once** — fire and forget, possible loss
2. **At-least-once** — retry on failure, possible duplicates
3. **Exactly-once** — hardest; requires idempotency + transactional messaging

**Practical approaches**:
- Store processed message IDs (idempotency keys)
- Use Kafka transactions + consumer group offsets
- Outbox pattern: write event to DB in same transaction as business change

---

## Security

### Q15: How does OAuth 2.0 work?

**Roles**: Resource Owner (user), Client (app), Authorization Server, Resource Server

**Authorization Code Flow**:
1. Client redirects user to Authorization Server
2. User authenticates and grants permission
3. Auth Server returns authorization code
4. Client exchanges code for access token (server-side)
5. Client uses access token to call Resource Server

**PKCE** (Proof Key for Code Exchange): prevents code interception for public clients (mobile/SPA).

---

## See Also
- [Interview Tips and Frameworks](./Interview-Tips-and-Frameworks.md)
- [Design Question Bank](./Design-Question-Bank.md)
- [Behavioral Questions](./Behavioral-Questions.md)
