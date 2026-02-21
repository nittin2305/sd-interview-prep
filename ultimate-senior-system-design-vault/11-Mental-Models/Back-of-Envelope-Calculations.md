# Back-of-Envelope Calculations

> A step-by-step guide for performing quick estimates in system design interviews.

---

## Why Back-of-Envelope Matters

Interviewers use estimation to assess:
1. Do you understand scale and order of magnitude?
2. Can you identify where the bottlenecks are?
3. Are your technology choices appropriate for the scale?

**You don't need exact numbers — you need the right order of magnitude.**

---

## The Estimation Process

### Step 1: Clarify Assumptions
Always state what you're assuming before calculating.

> "I'll assume 100M daily active users, each making 2 reads and 1 write per day."

### Step 2: Calculate Average QPS
```
Average QPS = (Users × Requests/day) / Seconds/day
Seconds/day = 86,400 ≈ 100,000 (round up for simplicity)
```

### Step 3: Estimate Peak QPS
```
Peak QPS ≈ Average QPS × 3 (typical peak-to-average ratio)
```

### Step 4: Calculate Storage
```
Daily new data = new objects/day × object size
Monthly storage = daily × 30
5-year storage = daily × 365 × 5
```

### Step 5: Estimate Bandwidth
```
Bandwidth = QPS × average request/response size
```

---

## Common Constants to Memorize

| Constant | Value |
|---------|-------|
| Seconds in a day | 86,400 ≈ 100K |
| Seconds in a month | 2.6M ≈ 3M |
| Seconds in a year | 31.5M ≈ 30M |
| 1 million | 10⁶ |
| 1 billion | 10⁹ |
| 1 trillion | 10¹² |
| 1 KB | 10³ bytes (or 2¹⁰ = 1,024) |
| 1 MB | 10⁶ bytes |
| 1 GB | 10⁹ bytes |
| 1 TB | 10¹² bytes |
| 1 PB | 10¹⁵ bytes |

---

## Worked Examples

### Example 1: Design Twitter (scale estimation)
```
Assumptions:
- 500M DAU
- Each user reads 100 tweets/day, writes 1 tweet/day
- Each tweet = 300 bytes

Read QPS  = (500M × 100) / 100K  = 500,000 reads/sec
Write QPS = (500M × 1)   / 100K  = 5,000 writes/sec

Peak read QPS  = 500K × 3 = 1.5M reads/sec
Peak write QPS = 5K  × 3  = 15K writes/sec

Storage:
New tweets/day = 500M × 1    = 500M tweets/day
Storage/day    = 500M × 300B = 150 GB/day
Storage/year   = 150 GB × 365 ≈ 55 TB/year
5-year storage = 55 × 5 = 275 TB (text only)
```

**Conclusion**: Need heavily cached read path (1.5M reads/sec is a lot). Writes are manageable. DB sharding needed at 275TB.

---

### Example 2: Design YouTube (video storage)
```
Assumptions:
- 500 hours of video uploaded per minute
- 1 minute of video = 300 MB (1080p)
- Videos transcoded into 5 resolutions (0.3x, 0.5x, 1x, 1.5x, 2x original size)
  → Multiplier ≈ 2.3x

Upload rate:
- 500 hours/min = 500 × 60 min of content/min
- = 30,000 min/min
- Storage/min = 30,000 × 300 MB = 9,000 GB/min = 9 TB/min
- With transcoding: 9 TB × 2.3 ≈ 20 TB/min
- Daily: 20 TB × 60 × 24 = 28,800 TB/day ≈ 29 PB/day
```

**Conclusion**: Massive storage, needs object store (S3-class), aggressive CDN caching for popular videos.

---

### Example 3: Design a URL Shortener
```
Assumptions:
- 100M URLs created per day
- 10B redirects per day (100x read:write ratio)
- Short URLs stored for 5 years

Write QPS = 100M / 100K = 1,000 writes/sec
Read QPS  = 10B  / 100K = 100,000 reads/sec

Storage per URL:
- Short URL (7 chars) = 7 bytes
- Long URL (avg 200 chars) = 200 bytes
- Created date = 8 bytes
- Total ≈ 500 bytes

Daily new storage = 100M × 500 bytes = 50 GB/day
5-year storage = 50 GB × 365 × 5 ≈ 90 TB
```

**Conclusion**: 100K reads/sec → heavy caching needed. 90 TB over 5 years is manageable with a single sharded DB + cache.

---

### Example 4: Design a Chat System
```
Assumptions:
- 500M DAU
- Each user sends 20 messages/day
- Average message = 100 bytes
- Messages retained 5 years

Write QPS = (500M × 20) / 100K = 100,000 msg/sec
Storage/day = 500M × 20 × 100B = 1 TB/day
5-year storage = 1 TB × 365 × 5 = 1.8 PB
```

**Conclusion**: Write-heavy (100K writes/sec) → Cassandra or HBase. 1.8 PB → need distributed storage with tiering.

---

## Validation Rule of Thumbs

After calculating, sanity check with these:

| Signal | Meaning |
|--------|---------|
| QPS > 1M | Need CDN + aggressive caching |
| QPS > 100K reads | Multiple read replicas or caching mandatory |
| QPS > 10K writes | Consider sharding |
| Storage > 1 TB | Consider partitioning/sharding |
| Storage > 1 PB | Need object storage + data tiering |
| Bandwidth > 10 Gbps | Consider CDN or P2P distribution |

---

## The "Is This Feasible?" Quick Check

Single-server limits (modern hardware):

| Resource | Limit |
|----------|-------|
| CPU | ~100K simple requests/sec |
| Memory | 128–512 GB |
| Network (10G NIC) | ~10 Gbps = ~1 GB/sec |
| SSD throughput (random reads) | ~500K IOPS |
| PostgreSQL | ~10K transactions/sec (write-heavy) |
| Redis | ~1M commands/sec |

---

## See Also
- [Design Numbers CheatSheet](../10-CheatSheets/Design-Numbers-CheatSheet.md)
- [Interview Tips and Frameworks](../09-Interview-Question-Bank/Interview-Tips-and-Frameworks.md)
- [Scale Ladders](./Scale-Ladders.md)
