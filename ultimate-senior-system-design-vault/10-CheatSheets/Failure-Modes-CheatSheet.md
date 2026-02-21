# Failure Modes Cheat Sheet

> A quick reference for common distributed system failure modes and mitigations.

---

## The Failure Mode Taxonomy

```
Failures
├── Hardware failures (disk, memory, NIC, power)
├── Software failures (bugs, memory leaks, deadlocks)
├── Network failures (packet loss, partition, high latency)
├── Dependency failures (DB down, third-party API unavailable)
├── Human errors (bad deploys, misconfigs, accidental deletes)
└── Cascading failures (overload propagation, retry storms)
```

---

## Common Failure Modes

### 1. Single Point of Failure (SPOF)

**Definition**: Any single component whose failure causes system failure.

**Examples**:
- Single database server
- Single load balancer
- Single DNS server
- Single availability zone deployment

**Mitigations**:
- Redundancy (N+1 minimum)
- Multi-AZ deployment
- Active-active or active-passive failover
- Health checks with automatic failover

---

### 2. Cascading Failure

**Definition**: Failure in one service causes overload in others, leading to chain collapse.

**Example**: Payment service is slow → order service retries → payment service gets more load → full outage.

**Mitigations**:
- Circuit breakers (stop calling failing service)
- Bulkhead pattern (isolate resources per service)
- Timeout and retry with exponential backoff + jitter
- Load shedding (reject requests when overloaded)
- Graceful degradation (return cached/partial data)

---

### 3. Network Partition

**Definition**: Nodes cannot communicate, causing split-brain or partial availability.

**Examples**:
- Broken cable between data centers
- Misconfigured firewall
- Network congestion causing packet drop

**Mitigations**:
- Design for partition tolerance (CAP-aware)
- Quorum reads/writes
- Fencing tokens to prevent split-brain writes
- STONITH (Shoot the Other Node in the Head) for clusters

---

### 4. Memory Leak

**Definition**: Application continuously allocates memory without releasing, eventually causing OOM.

**Indicators**: Gradual memory growth, eventually crashes.

**Mitigations**:
- Code review (close resources in finally blocks, use try-with-resources)
- Heap profiling (JVM: jmap, jstat; Python: tracemalloc)
- Memory limits + automatic restarts (Kubernetes `resources.limits.memory`)
- Canary deployments to catch before full rollout

---

### 5. Thundering Herd

**Definition**: Many clients simultaneously wake up and hammer the same resource.

**Examples**:
- Cache expiration causing all clients to hit DB
- All cron jobs starting at the same second
- Reconnect storm after network blip

**Mitigations**:
- Cache stampede prevention (mutex, probabilistic expiration)
- Jitter on retry and TTL
- Staggered cron jobs
- Exponential backoff on reconnect

---

### 6. Deadlock

**Definition**: Two or more processes wait for each other indefinitely.

**Classic scenario**: Thread A holds lock 1, wants lock 2. Thread B holds lock 2, wants lock 1.

**Mitigations**:
- Lock ordering (always acquire locks in the same order)
- Lock timeout (fail after N ms)
- Deadlock detection (DB-level: PostgreSQL detects and kills one transaction)
- Prefer optimistic locking for low-contention workloads

---

### 7. Split Brain

**Definition**: Two nodes both believe they are the primary/leader and both accept writes, causing data divergence.

**Mitigations**:
- Quorum-based systems (need majority to be leader)
- Fencing tokens (each leader gets monotonically increasing token; old leader's writes rejected)
- Epoch-based leases

---

### 8. Clock Drift

**Definition**: System clocks diverge across nodes, causing ordering and TTL issues.

**Impacts**: Event ordering, certificate expiration, distributed lock reliability.

**Mitigations**:
- NTP synchronization (typically <1ms drift)
- Use logical clocks (Lamport timestamps, vector clocks) for ordering
- Google TrueTime (bounded uncertainty with atomic clocks)
- Don't rely on wall clock for strict ordering

---

### 9. Hot Partition / Hot Key

**Definition**: A single shard/key receives disproportionate traffic.

**Examples**:
- Viral tweet hits same shard
- All writes to partition "2024-01-01" in time-series DB
- Popular product in e-commerce

**Mitigations**:
- Salting keys (add random prefix)
- Read replicas for hot keys
- Local caching at application tier
- Adaptive load balancing (detect and reroute hot keys)

---

### 10. Configuration Drift

**Definition**: Production config diverges from expected state through manual changes.

**Mitigations**:
- Infrastructure as Code (Terraform, Pulumi)
- Immutable infrastructure
- Config management (Ansible, Chef)
- Config auditing and alerts on manual change

---

## Failure Detection Mechanisms

| Mechanism | Description |
|-----------|-------------|
| Health checks (HTTP) | Ping endpoint, check status code |
| TCP health check | Can open TCP connection? |
| Heartbeat | Node periodically sends "I'm alive" |
| Gossip protocol | Nodes propagate health info peer-to-peer |
| Watchdog | External process restarts crashed service |
| Circuit breaker | Track error rate, open circuit on threshold |

---

## Recovery Patterns

| Pattern | Description |
|---------|-------------|
| Retry with backoff | Retry transient failures with exponential delay |
| Circuit breaker | Stop retrying when failure rate is high |
| Fallback | Return cached/default when service unavailable |
| Timeout | Don't wait forever; fail fast |
| Bulkhead | Isolate failures to one pool |
| Compensating transaction | Undo committed work on downstream failure |

---

## See Also
- [Circuit Breaker Pattern](../03-System-Design-Patterns/Circuit-Breaker-Pattern.md)
- [Bulkhead Pattern](../03-System-Design-Patterns/Bulkhead-Pattern.md)
- [Availability vs Durability](../01-Core-Fundamentals/Availability-vs-Durability.md)
