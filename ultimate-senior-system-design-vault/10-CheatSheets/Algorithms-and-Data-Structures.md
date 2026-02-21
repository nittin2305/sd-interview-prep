# Algorithm & Data Structure Selection Guide

> Quick reference for choosing the right algorithm or data structure in system design and coding interviews.

---

## Data Structure Decision Tree

```
Need fast lookup by key?
├── Yes → HashMap / HashSet O(1) avg
│   ├── Need ordering? → TreeMap / TreeSet O(log n)
│   └── Need concurrent? → ConcurrentHashMap
└── No → consider below

Need ordered sequence?
├── Yes, random access → Array / ArrayList O(1) access
├── Yes, frequent insert/delete → LinkedList O(1) insert, O(n) search
└── Yes, sorted order maintained → Balanced BST (TreeMap)

Need LIFO? → Stack
Need FIFO? → Queue / Deque
Need priority ordering? → Heap / PriorityQueue O(log n)
Need range queries? → Segment Tree / BIT (Fenwick Tree)
Need prefix lookups? → Trie O(L) where L = key length
Need graph? → Adjacency List (sparse) or Matrix (dense)
```

---

## Complexity Reference

| Data Structure | Access | Search | Insert | Delete | Space |
|---------------|--------|--------|--------|--------|-------|
| Array | O(1) | O(n) | O(n) | O(n) | O(n) |
| LinkedList | O(n) | O(n) | O(1) | O(1) | O(n) |
| Stack | O(n) | O(n) | O(1) | O(1) | O(n) |
| Queue | O(n) | O(n) | O(1) | O(1) | O(n) |
| HashMap | - | O(1) avg | O(1) avg | O(1) avg | O(n) |
| Heap | O(n) | O(n) | O(log n) | O(log n) | O(n) |
| BST (balanced) | O(log n) | O(log n) | O(log n) | O(log n) | O(n) |
| Trie | - | O(L) | O(L) | O(L) | O(n×L) |

---

## Algorithm Selection by Problem Type

### Searching
| Problem | Algorithm | Complexity |
|---------|-----------|------------|
| Sorted array lookup | Binary Search | O(log n) |
| Unsorted lookup | Linear scan | O(n) |
| Graph path (unweighted) | BFS | O(V+E) |
| Graph path (weighted) | Dijkstra | O((V+E) log V) |
| Graph shortest path (negative edges) | Bellman-Ford | O(VE) |
| All pairs shortest path | Floyd-Warshall | O(V³) |

### Sorting
| Algorithm | Average | Worst | Space | Stable? |
|-----------|---------|-------|-------|---------|
| QuickSort | O(n log n) | O(n²) | O(log n) | No |
| MergeSort | O(n log n) | O(n log n) | O(n) | Yes |
| HeapSort | O(n log n) | O(n log n) | O(1) | No |
| TimSort (Python/Java) | O(n log n) | O(n log n) | O(n) | Yes |
| CountingSort | O(n+k) | O(n+k) | O(k) | Yes |
| RadixSort | O(nk) | O(nk) | O(n+k) | Yes |

### Graph Algorithms
| Problem | Algorithm |
|---------|-----------|
| Connected components | Union-Find / BFS |
| Cycle detection (undirected) | DFS with visited set |
| Cycle detection (directed) | DFS with coloring |
| Topological sort | Kahn's (BFS) / DFS |
| Minimum spanning tree | Kruskal's / Prim's |
| Bipartite check | BFS two-coloring |
| Articulation points | Tarjan's DFS |

---

## Distributed Systems Algorithms

### Consensus
| Algorithm | Use Case | Notes |
|-----------|---------|-------|
| Paxos | Leader election, replication | Complex, hard to implement |
| Raft | Log replication, etcd/Consul | Easier to understand than Paxos |
| Zab | ZooKeeper's atomic broadcast | Variation of Paxos |
| PBFT | Byzantine fault tolerance | High overhead, blockchain origins |

### Hashing
| Algorithm | Use Case |
|-----------|---------|
| Consistent Hashing | Distributed caches, DHTs |
| Rendezvous Hashing | Alternative to consistent hashing |
| Jump Consistent Hash | Minimal overhead, no virtual nodes |
| FNV / MurmurHash | Fast non-cryptographic hashing |
| SHA-256 | Cryptographic integrity |
| MD5 | Checksums (not security) |

### Leader Election
| Algorithm | Notes |
|-----------|-------|
| Bully Algorithm | Highest ID wins, simple |
| Ring Algorithm | Message passing around ring |
| Raft leader election | Timeout-based, production ready |
| ZooKeeper ephemeral nodes | Practical distributed primitive |

---

## Rate Limiting Algorithms

| Algorithm | Accuracy | Memory | Burst Handling |
|-----------|----------|--------|----------------|
| Fixed Window Counter | Medium | Low | Poor (boundary burst) |
| Sliding Window Log | High | High | Good |
| Sliding Window Counter | High | Medium | Good |
| Token Bucket | High | Low | Excellent |
| Leaky Bucket | Medium | Low | Smooths bursts |

**Recommendation**: Token bucket for most use cases; sliding window for strict accuracy.

---

## Caching Eviction Algorithms

| Policy | Best For | Implementation |
|--------|---------|---------------|
| LRU (Least Recently Used) | General-purpose | HashMap + Doubly Linked List |
| LFU (Least Frequently Used) | Frequency-based hotspots | HashMap + Min-Heap |
| FIFO | Simple, fair | Queue |
| Random Replacement | Simple, cache-unfriendly patterns | Random sampling |
| ARC (Adaptive Replacement Cache) | Adaptive workloads | Two LRU queues |
| CLOCK (Second Chance) | OS page replacement | Circular buffer |

---

## Load Balancing Algorithms

| Algorithm | Description | Best For |
|-----------|-------------|---------|
| Round Robin | Cyclic distribution | Equal-weight, stateless |
| Weighted Round Robin | Proportional to weight | Different server capacities |
| Least Connections | Route to least-loaded | Variable-length requests |
| Random | Random selection | Simple, surprisingly effective |
| IP Hash | Hash of client IP | Session affinity |
| Consistent Hash | Hash ring | Cache routing |

---

## See Also
- [Design Numbers CheatSheet](./Design-Numbers-CheatSheet.md)
- [System Components CheatSheet](./System-Components-CheatSheet.md)
