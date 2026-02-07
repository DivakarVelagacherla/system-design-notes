# System Design - FAANG Interview Prep

# System Design - FAANG Interview Prep

Comprehensive system design preparation covering both Low Level Design (LLD) and High Level Design (HLD).

---

## 📊 Progress Overview

**Total Duration:** 24 weeks (Feb 8 - July 19, 2026)

**Weekly Time:** 4 hours (2 hrs Saturday + 2 hrs Sunday)

**Total Hours:** 96 hours

**Current Status:**

- **Phase:** Phase 1 - Fundamentals
- **Week:** 1 of 24
- **Sessions Completed:** 0
- **Projects Built:** 0

---

## 🗺️ Roadmap Structure

### Phase 1: Fundamentals (3 weeks - Feb 8 to Feb 22)

**Goal:** Build foundation in Java, networking, and data storage

**Week 1 (Feb 8-9):**

- Session 1: Java concurrency (threads, executors, locks, atomics)
- Session 2: Networking basics (HTTP/HTTPS, REST vs gRPC, TCP vs UDP)

**Week 2 (Feb 15-16):**

- Session 1: Data storage part 1 (ACID vs BASE, indexing, transactions)
- Session 2: Data storage part 2 (Postgres + Redis hands-on)

**Week 3 (Feb 22-23):**

- Session 1: Tooling (Docker, Postman, JMH microbenchmarks)
- Session 2: Fundamentals review + practice problems

---

### Phase 2: LLD Depth (7 weeks - Mar 1 to Apr 12)

**Goal:** Master design patterns and clean code principles

**Topics:**

- SOLID, DRY, YAGNI, Law of Demeter
- GoF Design Patterns: Factory, Builder, Strategy, Decorator, Observer, State, Adapter, Template Method
- Domain modeling: Entities, Value Objects, Aggregates, Repositories
- Error handling, validation, logging, tracing
- Testing: TDD, mocks vs fakes, contract tests

**Deliverable:** Class + sequence diagrams with clean Java implementation

---

### Phase 3: HLD Foundations (7 weeks - Apr 19 to May 31)

**Goal:** Learn scalability, distributed systems, and architecture

**Topics:**

- Scalability: vertical vs horizontal, replication, sharding, caching, CDN
- Data systems: SQL vs NoSQL, time-series DB, Elasticsearch, Kafka, S3
- Consistency & availability: CAP, PACELC, Raft, quorum
- Performance: throughput/latency, backpressure, circuit breakers, rate limiting
- Observability: metrics, logs, traces, SLO/SLI
- Security: OAuth2/OIDC, secrets, OWASP Top 10, encryption

---

### Phase 4: Systems by Pattern (5 weeks - Jun 7 to Jul 5)

**Goal:** Design and architect real-world systems

**Week 1:** Caching-heavy system (CDN-backed image service)

**Week 2:** Write-heavy system (event-driven orders with Kafka)

**Week 3:** Read-heavy system (news feed with fanout strategies)

**Week 4:** Realtime system (chat with WebSockets + presence)

**Week 5:** Batch/analytics (clickstream ETL pipeline)

For each: Requirements → Capacity estimates → API → Data model → Diagrams → Scaling → Failure modes

---

### Phase 5: Interview Prep (2 weeks - Jul 12 to Jul 19)

**Goal:** Practice timed interview scenarios

- 6-8 timed drills (45-60 min each)
- Alternate LLD and HLD
- Record and self-critique
- Build one-pager library of common components
- Perfect the interview template

---

## 🏗️ Projects to Build

1. **URL Shortener** - Custom domains + analytics
2. **Rate Limiter Service** - Token bucket as library + sidecar
3. **Feature Flag Service** - Rollout rules + audit log
4. **Document Collaboration** - CRDT or Operational Transform

---

## 📚 Weekly Practice Routine

- 1 LLD kata (implement + tests) per week
- 1 HLD design doc per week
- Present 1 design and get feedback
- Review open-source architecture (Kafka, Redis, etc.)

---

**Tools:**

- [draw.io/Excalidraw](http://draw.io/Excalidraw) for diagrams
- Docker for local dependencies
- Postman/cURL for API testing
- JMH for microbenchmarks
- k6/JMeter for load testing
