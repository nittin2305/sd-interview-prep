# Interview Tips and Frameworks

> Proven strategies for acing senior system design and engineering interviews.

---

## The 45-Minute System Design Interview Playbook

### Minute 0–5: Clarify Requirements
- Ask about **scale**: users, QPS, storage
- Ask about **functional boundaries**: what's in scope vs out
- Ask about **constraints**: latency SLA, consistency needs, budget
- Ask about **read/write ratio**

> "Before I dive in, let me clarify a few things to make sure I'm designing the right system..."

**Do NOT** jump into solution before you understand the problem.

---

### Minute 5–10: High-Level Design
- Draw the **basic blocks**: clients, API gateway, services, databases, caches
- Identify primary **data flows** for the 1-2 most important use cases
- Propose **storage choices** and justify them briefly

> "At a high level, I'd start with these components..."

---

### Minute 10–25: Deep Dive (Core Components)
- Pick the **hardest/most interesting** component and go deep
- Discuss **data models**, schema, indexing strategy
- Cover **scalability**: how does each component scale?
- Address **reliability**: how does it handle failure?

---

### Minute 25–35: Identify Bottlenecks & Optimizations
- **Where is the hotspot?** (DB, network, single service)
- Add **caching layers** where appropriate
- Discuss **sharding** or **read replicas** for databases
- Consider **CDN** for static/media content

---

### Minute 35–45: Edge Cases & Trade-offs
- What happens during **peak load**?
- How do you handle **cascading failures**?
- Summarize key **trade-offs** you made and why
- Ask if the interviewer wants to explore any area deeper

---

## How to Estimate (Back-of-Envelope)

Always show your work clearly.

### Common Reference Numbers
| Metric | Value |
|--------|-------|
| L1 cache | 1 ns |
| L2 cache | 10 ns |
| RAM | 100 ns |
| SSD | 100 μs |
| HDD | 10 ms |
| Network same DC | 0.5 ms |
| Network cross-region | 150 ms |
| Disk read 1MB | 1 ms |
| Network read 1MB | 10 ms |
| TCP packet round trip | 100 ms |

### Traffic Estimation Template
```
Daily Active Users (DAU) = 100M
Write QPS = (DAU × writes/user/day) / 86,400
           = (100M × 2) / 86,400 ≈ 2,300 QPS

Read QPS  = Write QPS × read:write ratio
           = 2,300 × 10 = 23,000 QPS

Peak QPS  = Avg QPS × 3 (rule of thumb)
           = 69,000 QPS
```

### Storage Estimation Template
```
Object size = 500 bytes
Daily new objects = DAU × objects/user/day = 100M × 2 = 200M
Daily storage = 200M × 500 bytes = 100 GB/day
5-year storage = 100 GB × 365 × 5 ≈ 180 TB
```

---

## Communication Tips

### Use Signposting
Tell the interviewer what you're about to do:
> "I'll start with the API design, then move to the data model, and then talk about scaling."

### Think Aloud
Narrate your thought process:
> "I'm considering Cassandra here because... but the trade-off is..."

### Ask for Feedback
> "Does this make sense? Would you like me to go deeper on the caching layer?"

### Handle Uncertainty Gracefully
> "I'm not 100% sure of the exact implementation, but the approach I'd take is..."

---

## Common Mistakes & How to Avoid Them

| Mistake | Fix |
|---------|-----|
| Starting coding immediately | Always clarify requirements first |
| Over-engineering from the start | Start simple, evolve |
| Ignoring non-functional requirements | Explicitly address latency, availability |
| Solutioning in a silo | Engage with interviewer, make it a conversation |
| Using buzzwords without depth | Only mention tech you can explain in detail |
| Getting stuck silently | Say "let me think through this" aloud |
| Not estimating scale | Always do back-of-envelope math |
| Forgetting failure scenarios | Discuss what happens when components fail |

---

## Technology Choice Framework

When choosing technology, use this decision template:

```
1. What are the access patterns? (read-heavy? write-heavy? point lookups? range scans?)
2. What consistency level do we need?
3. What is the expected data volume?
4. Do we need ACID transactions?
5. Is the schema fixed or flexible?
6. What's the team's familiarity?
```

---

## Trade-off Language

Practice these phrases for expressing trade-offs:

- "We gain X but at the cost of Y..."
- "This optimizes for reads at the expense of write throughput..."
- "Given the requirements, I'd prioritize availability over consistency here because..."
- "We could use X for simplicity, but if scale requires it, we'd move to Y..."

---

## Green Flags (What Impresses Interviewers)

✅ Asking clarifying questions before diving in  
✅ Proposing simple solution first, then iterating  
✅ Quantifying everything (QPS, storage, latency)  
✅ Proactively identifying single points of failure  
✅ Discussing monitoring and alerting  
✅ Referencing real-world systems as analogies  
✅ Knowing the limits of the technologies you propose  
✅ Making trade-offs explicit and justified  

---

## Red Flags

❌ Jumping to code before architecture  
❌ Designing without knowing requirements  
❌ Mentioning tech you can't explain  
❌ Ignoring failure modes  
❌ Silent for more than 60 seconds  
❌ Refusing to reconsider when challenged  

---

## See Also
- [Design Question Bank](./Design-Question-Bank.md)
- [Fundamental Questions](./Fundamental-Questions.md)
- [Back-of-Envelope Calculations](../11-Mental-Models/Back-of-Envelope-Calculations.md)
