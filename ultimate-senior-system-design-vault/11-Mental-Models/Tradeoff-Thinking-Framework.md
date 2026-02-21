# Tradeoff Thinking Framework

> A structured mental model for making and communicating technical trade-offs in system design.

---

## The Core Principle

**Every design decision is a trade-off. There are no universally correct answers — only context-appropriate ones.**

The goal in an interview is not to find the "right" answer, but to demonstrate that you understand the forces at play and can make reasoned, contextual decisions.

---

## The RAPES Framework for Trade-offs

| Letter | Dimension | Questions to Ask |
|--------|-----------|-----------------|
| **R**eliability | How does this fail? | What's the SLA? What's the impact of failure? |
| **A**vailability | Is it always accessible? | Active-active vs active-passive? |
| **P**erformance | How fast is it? | Latency vs throughput? P50 vs P99? |
| **E**fficacy | Does it do the job? | Does it meet functional requirements? |
| **S**calability | Can it grow? | Horizontal vs vertical? Stateless? |

Also consider: **Cost**, **Complexity**, **Security**, **Maintainability**, **Team familiarity**.

---

## The Classic Trade-off Triangle

```
          Consistency
             /\
            /  \
           /    \
          / You  \
         / can   \
        / pick 2  \
       /___________\
   Availability    Partition Tolerance
```

**CAP Theorem**: In the presence of a network partition, choose either Consistency or Availability.

In practice:
- **CP systems**: HBase, ZooKeeper, Redis Cluster (when configured)
- **AP systems**: Cassandra, CouchDB, DynamoDB (default)
- **CA systems**: Traditional RDBMS (not partition-tolerant at scale)

---

## The Read-Write Trade-off

Most design decisions can be framed as optimizing for reads OR writes:

| Optimization | Techniques | Cost |
|-------------|-----------|------|
| Optimize reads | Denormalization, caching, read replicas, CDN | More storage, stale data risk, write amplification |
| Optimize writes | Normalization, write-ahead log, batch writes | More complex reads, higher read latency |

**Key question**: What is your read:write ratio?
- 100:1 → heavy caching, read replicas, CDN
- 1:100 → LSM trees, write batching, async processing

---

## The Consistency-Latency Trade-off

```
Stronger Consistency → Higher Latency → Lower Availability

Weaker Consistency → Lower Latency → Higher Availability
```

| Level | Mechanism | Latency Impact |
|-------|-----------|---------------|
| Linearizability | Single-node or Paxos quorum | High |
| Sequential consistency | Leader-based replication | Medium-High |
| Causal consistency | Vector clocks | Medium |
| Eventual consistency | Async replication | Low |

---

## The Scalability Trade-off

| Approach | Pros | Cons |
|----------|------|------|
| Vertical scaling | Simple, no code change | Limited ceiling, single point of failure |
| Horizontal scaling | Unlimited scale | Requires stateless services, distributed complexity |
| Sharding | Scale writes + storage | Cross-shard queries, rebalancing complexity |
| Caching | Reduce DB load, latency | Stale data, cache invalidation complexity |
| Async processing | Decouple, buffer bursts | Eventual results, harder debugging |

---

## Normalizing Trade-offs for Communication

Use this template when explaining trade-offs:

> "If we choose **[Option A]**, we get **[Benefit X]** but we sacrifice **[Cost Y]**. Given our requirement of **[Context Z]**, I'd lean towards Option A because..."

**Example**:
> "If we choose Cassandra over PostgreSQL, we get horizontal scale and high write throughput, but we lose ACID transactions and complex joins. Given that this is a time-series metrics system with high write volume and no relational queries, Cassandra is the better fit."

---

## The Build vs Buy vs OSS Decision

| Factor | Build | Buy (SaaS) | Open Source |
|--------|-------|-----------|-------------|
| Customization | Full | Limited | Medium |
| Time to market | Slow | Fast | Medium |
| Ongoing cost | Engineering time | Subscription | Ops overhead |
| Control | Full | Limited | Full |
| Risk | High | Vendor lock-in | Community dependency |

**Rule of thumb**: Buy for non-differentiating capabilities (email, payments, auth). Build for core business logic.

---

## Prioritization Framework (MoSCoW)

For scope decisions in design:

| Category | Meaning |
|----------|---------|
| **M**ust have | Core functionality; system fails without it |
| **S**hould have | Important but not critical for MVP |
| **C**ould have | Nice to have; defer if time-constrained |
| **W**on't have (this time) | Explicitly out of scope for now |

---

## Trade-off Decision Log Template

When documenting decisions:

```markdown
## Decision: [Short title]

**Context**: What problem are we solving?

**Options Considered**:
1. Option A - pros/cons
2. Option B - pros/cons
3. Option C - pros/cons

**Decision**: We chose Option B.

**Rationale**: Given [requirements], Option B provides [benefit] while
acceptably compromising on [trade-off].

**Consequences**:
- We will need to [follow-up action]
- We accept [limitation]
- We revisit if [trigger condition]
```

---

## Common Trade-off Phrases for Interviews

- "This optimizes for [X] at the cost of [Y]..."
- "Given the requirement for [Z], I'd accept the [trade-off]..."
- "We can revisit this if the scale grows beyond [threshold]..."
- "This is a reversible decision — if it doesn't work, we can..."
- "The simpler solution first — we premature-optimize when we have evidence..."

---

## See Also
- [CAP Theorem](../01-Core-Fundamentals/CAP-Theorem-and-Applying-it.md)
- [Database Tradeoffs](../02-Database-Deep-Dive/Database-Tradeoffs.md)
- [Scalability Principles](../01-Core-Fundamentals/Scalability-Principles.md)
