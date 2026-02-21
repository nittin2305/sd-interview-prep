# System Design Numbers Cheat Sheet

> Key numbers every senior engineer should know for back-of-envelope estimations.

---

## Latency Reference Numbers (Jeff Dean's Numbers)

| Operation | Latency |
|-----------|---------|
| L1 cache reference | 0.5 ns |
| Branch mispredict | 5 ns |
| L2 cache reference | 7 ns |
| Mutex lock/unlock | 25 ns |
| Main memory reference | 100 ns |
| Compress 1KB with Snappy | 3 μs |
| Send 1KB over 1 Gbps network | 10 μs |
| Read 4KB randomly from SSD | 150 μs |
| Read 1MB sequentially from memory | 250 μs |
| Round trip within same datacenter | 500 μs |
| Read 1MB sequentially from SSD | 1 ms |
| HDD seek | 10 ms |
| Read 1MB sequentially from HDD | 20 ms |
| Send packet US → Europe → US | 150 ms |
| DNS lookup | 20–100 ms |
| TLS handshake | 100–300 ms |

**Key insight**: Memory is 1000x faster than disk. Same-DC network is 1000x faster than cross-continental.

---

## Scale Reference Points

| Metric | Value |
|--------|-------|
| 1 day in seconds | 86,400 ≈ 100K |
| 1 million requests/day | ~12 requests/second |
| 100 million requests/day | ~1,200 requests/second |
| 1 billion requests/day | ~12,000 requests/second |
| Twitter peak QPS | ~150,000 |
| Google search QPS | ~100,000 |
| Netflix peak bandwidth | 600 Gbps |

---

## Storage Reference Points

| Unit | Size |
|------|------|
| ASCII char | 1 byte |
| Unicode char | 2–4 bytes |
| Int32 | 4 bytes |
| Int64 / Double | 8 bytes |
| UUID | 16 bytes |
| SHA-256 hash | 32 bytes |
| Typical tweet | 200–500 bytes |
| Typical user row | 1 KB |
| JPEG thumbnail | 10–50 KB |
| Web page | 100 KB |
| 1 minute audio (128kbps) | 1 MB |
| 1 minute HD video | 100–200 MB |
| 1 hour HD video | 6–12 GB |

---

## Bandwidth Reference Points

| Technology | Bandwidth |
|------------|-----------|
| Gigabit Ethernet | 125 MB/s |
| 10 Gigabit Ethernet | 1.25 GB/s |
| SSD (sequential) | 500 MB/s – 3 GB/s |
| HDD (sequential) | 100–200 MB/s |
| PCIe NVMe SSD | 5–7 GB/s |
| Memory bandwidth | 50–100 GB/s |
| AWS EC2 to S3 | 10+ Gbps (instance type dependent) |

---

## Availability and Reliability Numbers

| Availability | Downtime per year | Downtime per month |
|-------------|------------------|-------------------|
| 99% | 3.65 days | 7.3 hours |
| 99.9% | 8.76 hours | 43.8 minutes |
| 99.95% | 4.38 hours | 21.9 minutes |
| 99.99% | 52.6 minutes | 4.4 minutes |
| 99.999% | 5.26 minutes | 26 seconds |

---

## Database Throughput Reference

| Database | Typical QPS (single node) |
|----------|--------------------------|
| PostgreSQL | 10K–100K reads, 5K–20K writes |
| MySQL | 10K–100K reads, 5K–20K writes |
| Cassandra (single node) | 50K–100K reads/writes |
| Redis (single node) | 100K–1M commands/sec |
| DynamoDB | Scales linearly with capacity units |
| MongoDB | 10K–50K operations/sec |

---

## Network Packet Sizes

| Protocol | Header Size |
|----------|------------|
| Ethernet | 14 bytes |
| IP | 20 bytes |
| TCP | 20 bytes |
| UDP | 8 bytes |
| HTTP/1.1 headers | 200–1000 bytes |
| HTTP/2 headers (compressed) | 20–100 bytes |

---

## Cache Hit Rate Impact

| Hit Rate | Requests hitting DB (out of 1000 QPS) |
|----------|--------------------------------------|
| 80% | 200 QPS |
| 90% | 100 QPS |
| 95% | 50 QPS |
| 99% | 10 QPS |

**Rule of thumb**: 10x reduction in DB load for each 10% improvement in cache hit rate.

---

## Quick Estimation Templates

### Web service capacity planning
```
Assume:
- 10M DAU
- Each user makes 10 requests/day

QPS = (10M × 10) / 86,400 ≈ 1,160 QPS avg
Peak QPS = 1,160 × 3 ≈ 3,500 QPS (3x peak factor)
```

### Storage for user-generated content
```
Assume:
- 1M new images/day
- Average image = 1 MB
- 5-year retention

Daily storage = 1M × 1 MB = 1 TB/day
5-year storage = 1 TB × 365 × 5 = 1.8 PB
With replication (3x) = 5.4 PB
```

### Message queue throughput
```
Assume:
- 10M messages/day
- Average message = 1 KB

QPS = 10M / 86,400 ≈ 116 msg/sec
Bandwidth = 116 msg/sec × 1 KB = 116 KB/sec ≈ 1 Mbps (negligible)
```

---

## See Also
- [Back-of-Envelope Calculations](../11-Mental-Models/Back-of-Envelope-Calculations.md)
- [Interview Tips and Frameworks](../09-Interview-Question-Bank/Interview-Tips-and-Frameworks.md)
