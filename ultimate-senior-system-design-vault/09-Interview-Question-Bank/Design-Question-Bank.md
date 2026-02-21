# System Design Question Bank

> Practice questions with structured answer frameworks for common senior-engineer interviews.

---

## Format: RADIO Framework

For each design question, structure your answer using:

- **R**equirements — functional + non-functional
- **A**PI Design — define the contract
- **D**ata Model — entities, storage choices
- **I**nfrastructure — components, architecture diagram
- **O**ptimizations — scale, bottlenecks, edge cases

---

## Classic Designs

### Design a URL Shortener (TinyURL / Bitly)

**Functional Requirements**:
- Shorten a long URL to a 7-character alias
- Redirect short URL to original
- Custom aliases (optional)
- Analytics (click count, geography)

**Non-functional Requirements**:
- 100M URLs created/day, 10B redirects/day
- 99.99% availability for redirects
- Low latency (<10ms P99 for redirects)

**Key Decisions**:
| Decision | Choice | Why |
|----------|--------|-----|
| ID generation | Base62 of auto-increment or hash | Predictable length, no collisions |
| Storage | MySQL + Redis cache | Relational for writes, cache for reads |
| Redirect type | 301 vs 302 | 302 preserves analytics |

**Deep dives**: Bloom filter for short URL availability check, consistent hashing for cache nodes, analytics using Kafka + ClickHouse.

---

### Design a Chat System (WhatsApp / Slack)

**Functional Requirements**:
- 1:1 and group messaging
- Online presence indicators
- Message delivery receipts (sent, delivered, read)
- Push notifications

**Non-functional Requirements**:
- 500M DAU, 60B messages/day
- Messages delivered in <100ms
- Message history stored 5 years

**Key Decisions**:
| Decision | Choice | Why |
|----------|--------|-----|
| Protocol | WebSocket | Full-duplex, low overhead |
| Message storage | Cassandra | Write-heavy, horizontal scale |
| Fan-out | Pull model for large groups | Avoid write amplification |
| Notifications | APNs/FCM via SNS | Platform-native delivery |

**Deep dives**: Message ID ordering (Snowflake), end-to-end encryption (Signal Protocol), offline message queuing.

---

### Design a Social Media Feed (Twitter / Instagram)

**Functional Requirements**:
- Post tweets/photos
- Follow users
- View personalized timeline
- Like, comment, retweet

**Non-functional Requirements**:
- 500M DAU, 500M tweets/day
- Timeline loads in <200ms
- Support celebrity users (100M followers)

**Fan-out Strategies**:

| Approach | Pros | Cons |
|----------|------|------|
| Fan-out on write (push) | Fast reads | Slow writes for celebrities |
| Fan-out on read (pull) | Simple writes | Slow reads, no pre-compute |
| Hybrid | Best of both | Complex logic |

**Hybrid solution**: Push for regular users, pull for celebrity posts. Merge at read time.

**Deep dives**: Ranking algorithm (ML-based), real-time vs eventual feed, CDN for media, global distribution.

---

### Design a Ride-Sharing Service (Uber / Lyft)

**Functional Requirements**:
- Riders request rides with pickup/destination
- Match to nearby drivers
- Real-time location tracking
- Pricing and payment

**Non-functional Requirements**:
- 10M rides/day
- Match driver within 5 seconds
- Location updates every 4 seconds

**Key Decisions**:
| Decision | Choice | Why |
|----------|--------|-----|
| Location storage | Redis Geospatial | O(log N) radius search |
| Matching | Proximity + scoring | Balance ETA + driver rating |
| Location streaming | WebSocket + Kafka | Real-time with persistence |
| Surge pricing | Lambda + stream | React to supply/demand ratio |

**Deep dives**: Geohash partitioning, ETA prediction (ML), driver dispatch optimization, payment idempotency.

---

### Design a Video Streaming Platform (YouTube / Netflix)

**Functional Requirements**:
- Upload videos
- Stream videos globally
- Search and recommendations
- Comments and likes

**Non-functional Requirements**:
- 500 hours of video uploaded per minute
- 1B hours streamed per day
- <2s stream start time globally

**Key Decisions**:
| Decision | Choice | Why |
|----------|--------|-----|
| Video encoding | Transcoding pipeline | Multiple resolutions/codecs |
| Storage | Object store (S3) + CDN | Cost-effective, global edge |
| Streaming protocol | HLS / DASH with ABR | Adaptive bitrate |
| Metadata | MySQL + Elasticsearch | ACID + full-text search |

**Deep dives**: Adaptive bitrate streaming, CDN cache hit rate optimization, video deduplication (perceptual hashing), recommendation engine.

---

### Design a Distributed Cache (Redis / Memcached)

**Functional Requirements**:
- GET, SET, DELETE operations
- TTL support
- Eviction policies

**Non-functional Requirements**:
- Sub-millisecond P99 latency
- 99.99% availability
- Horizontal scalability to 100TB

**Key Decisions**:
| Decision | Choice | Why |
|----------|--------|-----|
| Data structure | Hash map + doubly linked list | O(1) LRU |
| Distribution | Consistent hashing | Minimize remapping |
| Replication | Leader-follower | High read throughput |
| Persistence | RDB + AOF (optional) | Trade durability for speed |

**Deep dives**: Eviction policies (LRU vs LFU vs FIFO), hot key problem, cache stampede prevention.

---

### Design a Notification System

**Functional Requirements**:
- Send push, email, SMS notifications
- Templates with variable substitution
- Delivery tracking
- User preferences (opt-out)

**Non-functional Requirements**:
- 10M notifications/day
- Delivery within 30 seconds
- At-least-once guarantee

**Architecture**:
```
API → Notification Service → Kafka
                              ├── Email Worker → SES
                              ├── Push Worker → FCM/APNs
                              └── SMS Worker → Twilio
```

**Deep dives**: Rate limiting per user, retry with exponential backoff, idempotency, user preference enforcement.

---

### Design a Search Engine (Google / Elasticsearch)

**Functional Requirements**:
- Crawl and index web pages
- Full-text search with ranking
- Autocomplete / query suggestions

**Non-functional Requirements**:
- 10B web pages indexed
- <100ms query response
- Crawl 1B new pages/day

**Components**:
1. **Web Crawler** — distributed, politeness policy, deduplication
2. **Inverted Index** — term → [docID, frequency, positions]
3. **Ranking** — TF-IDF × PageRank × freshness × user signals
4. **Serving** — sharded index, scatter-gather query

**Deep dives**: Crawler politeness (robots.txt, crawl delay), near-duplicate detection (SimHash), index compression, query expansion.

---

## See Also
- [Fundamental Questions](./Fundamental-Questions.md)
- [Behavioral Questions](./Behavioral-Questions.md)
- [Interview Tips and Frameworks](./Interview-Tips-and-Frameworks.md)
