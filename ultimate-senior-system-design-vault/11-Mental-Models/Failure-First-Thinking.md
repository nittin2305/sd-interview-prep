# Failure-First Thinking

> A mental model for building reliable systems by designing for failure from the start.

---

## The Core Principle

**Assume everything will fail. Design for it.**

> "Anything that can fail, will fail. The question is whether it fails gracefully or catastrophically."

The difference between a junior and senior engineer's design often comes down to one thing: **the senior engineer has seen production.**

---

## The Failure-First Design Questions

For every component you add to a design, ask:

1. **What happens when this component fails?**
2. **How do other components behave when this one is unavailable?**
3. **How do we detect the failure?**
4. **How do we recover from it?**
5. **Can we prevent it from cascading?**

---

## The Five Categories of Failure

### 1. Hardware Failures
**Examples**: Disk crash, memory error, NIC failure, power supply, data center fire

**Design for it**:
- RAID for disk redundancy
- ECC memory
- Redundant power supplies
- Multi-AZ / multi-region deployment
- Assume disk = ephemeral (12-factor apps)

---

### 2. Software Failures
**Examples**: Memory leak, unhandled exception, infinite loop, OOM kill, deadlock

**Design for it**:
- Health checks (liveness + readiness probes)
- Memory and CPU limits
- Auto-restart on failure (systemd, Kubernetes)
- Graceful shutdown handlers
- Canary deployments

---

### 3. Dependency Failures
**Examples**: Database is down, third-party API returns 500, queue is unavailable

**Design for it**:
- Circuit breakers
- Fallback responses (cached data, default values)
- Retry with exponential backoff + jitter
- Timeouts on all external calls
- Bulkhead pattern (don't let one bad dependency kill everything)

---

### 4. Network Failures
**Examples**: Packet loss, network partition, high latency, connection pool exhaustion

**Design for it**:
- Idempotent operations (safe to retry)
- Connection pooling with limits
- Health-based routing (route around unhealthy nodes)
- Keep-alives and reconnection logic

---

### 5. Human Errors
**Examples**: Bad deployment, misconfiguration, accidental delete, runaway query

**Design for it**:
- Automated deployments (no manual SSH)
- Feature flags (turn off bad features without deploy)
- Soft deletes (mark as deleted, don't actually delete)
- Point-in-time recovery for databases
- Change management process

---

## The Reliability Hierarchy

```
Level 5: Antifragile    — System gets stronger from failures (Chaos Engineering)
Level 4: Self-healing   — System detects and recovers automatically
Level 3: Fault-tolerant — System continues operating with failures
Level 2: Fault-resilient — System degrades gracefully
Level 1: Fault-fragile  — System fails completely on first failure
```

Most production systems should target Level 3-4.

---

## Chaos Engineering

**Definition**: Intentionally introducing failures in production (or production-like environments) to test system resilience.

**Examples**:
- Netflix Chaos Monkey: randomly terminates EC2 instances
- Chaos Toolkit: inject latency, kill processes, drain DNS
- AWS Fault Injection Simulator: cloud-native chaos

**Process**:
1. Define steady state (normal metrics)
2. Hypothesize: "If X fails, system should continue working"
3. Introduce failure
4. Compare metrics to steady state
5. Fix any gaps found

**Start small**: Test in staging before production. Gamedays before full automation.

---

## Failure Mode and Effects Analysis (FMEA)

A systematic approach to identifying failure modes:

| Component | Failure Mode | Effect | Severity (1-5) | Probability (1-5) | Risk Score | Mitigation |
|-----------|-------------|--------|---------------|------------------|------------|-----------|
| Redis | Node crash | Cache miss storm | 4 | 2 | 8 | Sentinel/Cluster + DB retry |
| Payment API | Timeout | Order stuck in pending | 5 | 2 | 10 | Idempotency key + timeout job |
| DB Primary | Disk full | All writes fail | 5 | 1 | 5 | Disk alert at 70%, auto-expand |

**Risk Score** = Severity × Probability. Focus on highest risk scores first.

---

## The Pre-mortem Technique

Before launching, run a pre-mortem:

> "Imagine it's 6 months from now, and this system has failed catastrophically. What went wrong?"

Brainstorm all the ways it could fail, then build mitigations before launch. This is far cheaper than learning in production.

---

## SLO / SLA / SLI Vocabulary

| Term | Definition | Example |
|------|-----------|---------|
| SLI (Indicator) | What you measure | P99 latency, error rate, availability % |
| SLO (Objective) | Target you set | 99.9% requests < 200ms |
| SLA (Agreement) | Contract with consequences | 99.9% uptime or 10% credit |
| Error Budget | 1 - SLO | 0.1% of requests can fail/month |

**Error budget principle**: When error budget is exhausted, stop feature work and focus on reliability.

---

## See Also
- [Failure Modes CheatSheet](../10-CheatSheets/Failure-Modes-CheatSheet.md)
- [Circuit Breaker Pattern](../03-System-Design-Patterns/Circuit-Breaker-Pattern.md)
- [Availability vs Durability](../01-Core-Fundamentals/Availability-vs-Durability.md)
- [Alerting and Monitoring](../07-Observability/Alerting-and-Monitoring.md)
