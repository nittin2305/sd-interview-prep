# System Components Quick Reference

> One-liner summaries and decision guides for common distributed system components.

---

## Compute

| Component | What It Does | Use When |
|-----------|-------------|---------|
| Web Server (Nginx/Apache) | Serve static files, reverse proxy | Static content, SSL termination |
| App Server | Run application logic | Business logic execution |
| Serverless (Lambda) | Run code without managing servers | Event-driven, low/variable traffic |
| Container (Docker) | Package app + dependencies | Consistent environments |
| Orchestration (Kubernetes) | Manage containerized workloads | Large-scale microservices |
| VM (EC2) | Full machine virtualization | Custom OS, persistent workloads |

---

## Networking

| Component | What It Does | Use When |
|-----------|-------------|---------|
| DNS | Domain → IP resolution | All internet-facing services |
| CDN | Edge-cached content delivery | Static assets, media, global reach |
| Load Balancer (L4) | TCP/UDP routing | High-throughput, low latency |
| Load Balancer (L7) | HTTP-aware routing | URL-based routing, canary |
| API Gateway | Request routing, auth, rate limit | Microservices entry point |
| Service Mesh (Istio) | mTLS, observability, traffic control | Complex microservices |
| VPN / Direct Connect | Secure private connectivity | Hybrid cloud |

---

## Storage

| Component | What It Does | Use When |
|-----------|-------------|---------|
| Block Storage (EBS) | Raw block device | Databases, OS volumes |
| Object Storage (S3) | File storage at massive scale | Media, backups, data lakes |
| File Storage (EFS/NFS) | Shared filesystem | Shared config, legacy apps |
| In-memory Store (Redis) | Ultra-fast key-value | Caching, sessions, pub/sub |
| Document DB (MongoDB) | Flexible JSON documents | Variable schema, nested data |
| Wide-column (Cassandra) | High write, time-series | IoT, activity feeds |
| Relational DB (PostgreSQL) | ACID transactions | Financial, relational data |
| Search (Elasticsearch) | Full-text + aggregations | Logs, search, analytics |
| Graph DB (Neo4j) | Relationships as first-class | Social graphs, fraud detection |
| Time-series DB (InfluxDB) | Time-stamped metrics | Monitoring, IoT |
| Data Warehouse (Redshift) | Analytical queries (OLAP) | Reporting, BI |

---

## Messaging

| Component | What It Does | Use When |
|-----------|-------------|---------|
| Message Queue (SQS) | Decouple producer/consumer | Task queues, async work |
| Message Bus (Kafka) | Durable, ordered event stream | Event sourcing, log aggregation |
| Pub/Sub (SNS) | Fan-out notifications | Multi-subscriber events |
| Real-time (WebSocket) | Bi-directional streaming | Chat, live updates |
| Job Queue (Celery/Sidekiq) | Background jobs | Email sending, report generation |

---

## Caching Layers

| Layer | Technology | Typical TTL |
|-------|-----------|-------------|
| Browser cache | HTTP headers | Minutes to days |
| CDN cache | Cloudfront, Fastly | Hours to days |
| Reverse proxy cache | Nginx, Varnish | Seconds to minutes |
| Application cache | Redis, Memcached | Seconds to hours |
| Database query cache | PostgreSQL, MySQL | Query lifetime |
| CPU cache | L1/L2/L3 | Nanoseconds |

---

## Database Selection Guide

```
Need ACID transactions?
├── Yes → PostgreSQL / MySQL
│   ├── Need horizontal scale? → CockroachDB / Spanner
│   └── Simple needs → SQLite (single node)
└── No → NoSQL options:
    ├── Simple key-value? → Redis / DynamoDB
    ├── Document? → MongoDB / Firestore
    ├── Wide column / high write? → Cassandra / HBase
    ├── Graph? → Neo4j / Neptune
    ├── Full-text search? → Elasticsearch / OpenSearch
    ├── Time-series? → InfluxDB / TimescaleDB
    └── Analytics / OLAP? → Redshift / BigQuery / ClickHouse
```

---

## API Design Comparison

| Style | Protocol | Best For | Not Ideal For |
|-------|----------|---------|---------------|
| REST | HTTP/1.1 | CRUD, public APIs | Real-time, complex queries |
| GraphQL | HTTP | Flexible queries, mobile | Simple APIs, caching |
| gRPC | HTTP/2 | Internal microservices | Browser clients, debugging |
| WebSocket | TCP | Real-time bidirectional | Request-response patterns |
| SSE | HTTP | Server push, one-way | Bidirectional communication |
| Webhook | HTTP | Event callbacks | High frequency events |

---

## Monitoring Stack Reference

| Layer | Tools |
|-------|-------|
| Metrics | Prometheus, Datadog, CloudWatch |
| Logs | ELK Stack (Elasticsearch + Logstash + Kibana), Splunk, Loki |
| Traces | Jaeger, Zipkin, AWS X-Ray, Tempo |
| Alerting | PagerDuty, Opsgenie, Grafana Alerts |
| Dashboards | Grafana, Kibana, Datadog |
| Synthetic monitoring | Pingdom, Datadog Synthetics |
| Error tracking | Sentry, Rollbar |

---

## Security Components

| Component | Purpose |
|-----------|---------|
| WAF (Web Application Firewall) | Block SQLi, XSS, DDoS |
| VPC / Private subnets | Network isolation |
| IAM / RBAC | Identity and access control |
| KMS / HSM | Key management |
| Secrets Manager | Store credentials securely |
| OAuth 2.0 / OIDC | Federated authentication |
| mTLS | Mutual TLS for service-to-service |
| Rate Limiter | Prevent abuse, DDoS |

---

## See Also
- [Design Numbers CheatSheet](./Design-Numbers-CheatSheet.md)
- [AWS Service Mapping](../08-AWS-Mapping/Mapping-Compute-to-AWS.md)
- [Database Tradeoffs](../02-Database-Deep-Dive/Database-Tradeoffs.md)
