# Scale Ladders — Evolving Architecture as You Grow

> A mental model for thinking about architecture at different scales, and knowing when to upgrade.

---

## The Scale Ladder

Many senior engineers think in terms of "scale ladders" — each rung represents an order-of-magnitude growth requiring architectural changes.

---

## User Scale Ladder

### Rung 1: 1 – 1,000 Users
**Architecture**: Single server + single database

```
Browser → Single VM (App + DB)
```

- Monolith application
- Single PostgreSQL/MySQL instance
- No caching needed
- Deployments: SSH + bash script

**Don't** add microservices here. Keep it simple.

---

### Rung 2: 1K – 100K Users
**Architecture**: Separate app servers + database

```
Browser → Load Balancer → App Servers (2-3)
                          ↓
                       Primary DB + Read Replica
                          ↓
                       CDN (static assets)
```

- Separate app tier from DB tier
- Add a read replica for read-heavy workloads
- Introduce Redis for session storage and caching
- CDN for static assets
- DNS round robin or L7 load balancer

---

### Rung 3: 100K – 10M Users
**Architecture**: Horizontal scaling + dedicated services

```
Browser → CDN → Load Balancer → Stateless App Servers (auto-scaled)
                                    ↓
                         Cache Layer (Redis Cluster)
                                    ↓
                    Primary DB + Multiple Read Replicas
                                    ↓
                    Message Queue (Kafka/SQS) → Workers
```

- Stateless app servers (sessions in Redis)
- Cache layer in front of DB
- Async processing for non-critical paths
- Monitoring, alerting, on-call rotation
- Consider vertical sharding (separate DBs per domain)

---

### Rung 4: 10M – 100M Users
**Architecture**: Sharding, microservices, multi-region

```
Browser → GeoDNS → Regional Load Balancers
                        ↓
                   API Gateway
                        ↓
              ┌─────────────────────┐
              │  Microservices      │
              │  User | Order | Pay │
              └─────────────────────┘
                        ↓
              ┌─────────────────────┐
              │  Sharded DBs        │
              │  Cache Cluster      │
              │  Kafka              │
              └─────────────────────┘
```

- Horizontal DB sharding
- Microservices for independent scaling
- Multi-AZ deployments
- Global CDN
- Data replication across regions
- Service discovery (Consul, Kubernetes DNS)

---

### Rung 5: 100M+ Users (Hyperscale)
**Architecture**: Custom infrastructure, global distribution

- Custom distributed databases
- Multi-region active-active
- Edge computing for ultra-low latency
- Custom CDN or private backbone
- ML-driven autoscaling and traffic management
- Dedicated teams per service

---

## Traffic Scale Ladder

| QPS | Architecture |
|-----|-------------|
| < 100 | Single server |
| 100–1K | App server + DB with read replica |
| 1K–10K | Auto-scaling + caching |
| 10K–100K | Sharding + CDN + async processing |
| 100K–1M | Multi-region + custom data structures |
| > 1M | Hyperscale: custom hardware, protocol optimization |

---

## Storage Scale Ladder

| Data Volume | Storage Approach |
|------------|-----------------|
| < 100 GB | Single database instance |
| 100 GB – 1 TB | Vertical scaling, read replicas |
| 1 TB – 10 TB | Sharding by entity ID or date |
| 10 TB – 1 PB | Columnar storage for analytics, object storage |
| > 1 PB | Data lake (S3 + Athena/Spark), distributed file system |

---

## The Two-Pizza Rule for Services

Jeff Bezos: "If a team can't be fed by two pizzas, it's too big."

Apply the same logic to microservices:
- A service should be small enough for a 2-pizza team to own
- A service that requires coordination between 5 teams is too big to decompose
- Start with a monolith, split when team boundaries require it

---

## When to Shard

Shard when:
- Single DB can't handle write throughput
- Data volume exceeds single machine capacity
- You need geographic data locality

Don't shard when:
- Read replicas solve the problem
- You need cross-entity transactions
- Application is still small

---

## The 10x Scaling Rule

When your load is expected to 10x:
1. Identify your current bottleneck (CPU? DB? Network?)
2. Determine which tier fails first
3. Scale that tier before the others
4. Validate with load tests

---

## See Also
- [Scalability Principles](../01-Core-Fundamentals/Scalability-Principles.md)
- [Sharding and Partitioning](../02-Database-Deep-Dive/Sharding-and-Partitioning.md)
- [Back-of-Envelope Calculations](./Back-of-Envelope-Calculations.md)
