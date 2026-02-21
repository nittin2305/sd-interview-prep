# Consistency Models

> **References:** [DDIA Ch 5 & 9](https://dataintensive.net/) | [Jepsen](https://jepsen.io/consistency) | [AWS DynamoDB Consistency](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadConsistency.html)

---

## Consistency Spectrum

```mermaid
graph LR
    SC[Strong / Linearizable] --> SEQ[Sequential]
    SEQ --> CC[Causal]
    CC --> RYW[Read-Your-Writes]
    RYW --> MT[Monotonic Read]
    MT --> EC[Eventual Consistency]

    style SC fill:#d32f2f,color:#fff
    style EC fill:#388e3c,color:#fff
```

Higher on the left = safer but slower. Higher on the right = faster but more stale.

---

## Consistency Models Explained

### 1. Linearizability (Strong Consistency)
- Every operation appears to take effect instantaneously at some point between invocation and completion
- All nodes see writes in the same order
- **AWS:** Aurora within region, RDS Multi-AZ, DynamoDB `ConsistentRead=true`
- **Cost:** Requires consensus → high latency, low availability under partitions

### 2. Sequential Consistency
- All nodes see operations in the same order, but not necessarily "real-time"
- Weaker than linearizability — timing doesn't need to match wall clock
- **AWS:** Not commonly advertised; most "strong" DBs provide linearizability

### 3. Causal Consistency
- Operations that are causally related are seen in order by all nodes
- Concurrent operations may be seen in different orders
- **Use case:** Commenting systems (reply must appear after the post it replies to)
- **AWS:** DynamoDB Transactions, DAX

### 4. Read-Your-Writes (Session Consistency)
- After a write, the same client always sees that write in subsequent reads
- Other clients may still see stale data
- **AWS:** DynamoDB Global Tables with session affinity, ElastiCache with sticky routing

### 5. Monotonic Read Consistency
- Once a client reads a value, subsequent reads never return older values
- **AWS:** DynamoDB global secondary indexes with eventual consistency

### 6. Eventual Consistency
- Given enough time with no new writes, all replicas converge to the same value
- **AWS:** DynamoDB default, S3 object listings, Route 53 DNS propagation

---

## ACID vs BASE

| Property | ACID | BASE |
|----------|------|------|
| **A** | Atomicity | Basically Available |
| **C** | Consistency | Soft state |
| **I** | Isolation | Eventually consistent |
| **D** | Durability | |
| **Typical DB** | PostgreSQL, MySQL, Oracle | Cassandra, DynamoDB, Riak |
| **Trade-off** | Strong guarantees, lower throughput | High throughput, eventual correctness |

---

## AWS Consistency Mapping

| Service | Model | Details |
|---------|-------|---------|
| Aurora (single region) | Linearizable | Writes go to single writer; replicas lag <10ms |
| RDS Read Replica | Eventual | Async replication; replica lag varies |
| DynamoDB default | Eventual | Any of 3 replicas can serve read |
| DynamoDB ConsistentRead | Linearizable (per-item) | Leader-only read, 2× cost |
| DynamoDB Transactions | Serializable | Across items, 2× WCU/RCU |
| ElastiCache Redis | Linearizable (primary) | Reads from replica = eventual |
| S3 (2020+) | Strong read-after-write | Strongly consistent for GET/LIST after PUT |
| SQS | At-least-once | Duplicate message possible |

---

## Java Example — Read-Your-Writes with Spring

```java
@Service
public class UserProfileService {

    private final DynamoDbClient dynamoDb;
    // Track versions to implement read-your-writes
    private final Map<String, Long> sessionVersions = new ConcurrentHashMap<>();

    public void updateProfile(String userId, UserProfile profile) {
        long version = System.currentTimeMillis();
        
        PutItemRequest request = PutItemRequest.builder()
            .tableName("UserProfiles")
            .item(Map.of(
                "userId", AttributeValue.builder().s(userId).build(),
                "version", AttributeValue.builder().n(String.valueOf(version)).build(),
                "data", AttributeValue.builder().s(toJson(profile)).build()
            ))
            .build();
        
        dynamoDb.putItem(request);
        // Remember the version we just wrote for this session
        sessionVersions.put(userId, version);
    }

    public UserProfile getProfile(String userId) {
        Long requiredVersion = sessionVersions.get(userId);
        
        GetItemRequest request = GetItemRequest.builder()
            .tableName("UserProfiles")
            .key(Map.of("userId", AttributeValue.builder().s(userId).build()))
            // Use consistent read if we just wrote (read-your-writes guarantee)
            .consistentRead(requiredVersion != null)
            .build();
        
        var item = dynamoDb.getItem(request).item();
        long readVersion = Long.parseLong(item.get("version").n());
        
        // If stale read, retry with consistent read
        if (requiredVersion != null && readVersion < requiredVersion) {
            return getProfileWithConsistentRead(userId);
        }
        
        return fromJson(item.get("data").s(), UserProfile.class);
    }
    
    private UserProfile getProfileWithConsistentRead(String userId) {
        GetItemRequest request = GetItemRequest.builder()
            .tableName("UserProfiles")
            .key(Map.of("userId", AttributeValue.builder().s(userId).build()))
            .consistentRead(true)
            .build();
        return fromJson(dynamoDb.getItem(request).item().get("data").s(), UserProfile.class);
    }
}
```

---

## When NOT to Use Strong Consistency

1. **High-traffic read-heavy systems** — linearizable reads require leader contact → bottleneck
2. **Cross-region** — strong consistency across regions requires synchronous replication → 100ms+ latency
3. **Analytics / reporting** — slightly stale data is acceptable; use eventual consistency for scale
4. **User feeds / timelines** — causal consistency is sufficient; users rarely notice ordering differences

---

## Tradeoffs Table

| Model | Latency | Throughput | Complexity | Staleness |
|-------|---------|------------|------------|-----------|
| Linearizable | High | Low | High | None |
| Sequential | Medium | Medium | Medium | None (order only) |
| Causal | Low-Medium | Medium-High | Medium | Possible |
| Read-Your-Writes | Low | High | Low | Possible for others |
| Eventual | Lowest | Highest | Lowest | Yes |

---

## Interview Q&A

**Q1: What's the difference between linearizability and serializability?**
> Linearizability is a property of single-object operations — reads/writes to a single register appear instantaneous. Serializability is about multi-object transactions — the effect is as if transactions executed serially. A database can be serializable but not linearizable if it doesn't respect real-time ordering.

**Q2: How does DynamoDB achieve strong consistency if it's an AP system?**
> DynamoDB stores 3 replicas. By default, reads go to any replica (AP). With `ConsistentRead=true`, DynamoDB routes the read to the leader replica which has the most up-to-date data. This is CP behavior for individual item reads, but not globally linearizable across transactions without DynamoDB Transactions.

**Q3: When would you choose eventual consistency and how do you handle stale reads?**
> For user feeds or product listings where slight staleness is acceptable. Handle stale reads by: (1) showing a "loading" state and retrying, (2) using version tokens to detect staleness, (3) using read-your-writes at the session level so the user who just wrote always sees their write.
