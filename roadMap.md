**Start Date:** February 8, 2026

**End Date:** July 19, 2026

**Weekly Time:** 4 hours (2 hrs Saturday + 2 hrs Sunday)

**Total Hours:** 96 hours

---

## 📊 Progress Overview

**Current Phase:** Phase 1 - Fundamentals

**Current Week:** 2 of 24

**Sessions Completed:** 3/48

**Projects Built:** 0/4

---

## 🗓️ Phase Breakdown

### ✅ Phase 1: Fundamentals (3 weeks - Feb 8 to Feb 22)

**Status:** In Progress

**Week 1 (Feb 8-9):**

- ✅ Session 1: Java concurrency (threads, executors, locks, atomics)
- ✅ Session 2: Networking basics (HTTP/HTTPS, REST vs gRPC, TCP vs UDP)

**Week 2 (Feb 15-16):**

- ✅ Session 1: Data storage part 1 (ACID vs BASE, indexing, transactions)
- ⏳ Session 2: Data storage part 2 (Postgres + Redis hands-on)

**Week 3 (Feb 22-23):**

- ⏳ Session 1: Tooling (Docker, Postman, JMH microbenchmarks)
- ⏳ Session 2: Fundamentals review + practice problems

---

### 📅 Phase 2: LLD Depth (7 weeks - Mar 1 to Apr 12)

**Status:** Not Started

**Topic:** <page url="https://www.notion.so/300ae59f84cd8179b97bdd4b7824e6ae">Low Level Design (LLD)</page>

**Week 1-2:** SOLID principles (SRP, OCP, LSP, ISP, DIP) + DRY, YAGNI

**Week 3-5:** Design Patterns - Factory, Builder, Strategy, Decorator, Observer, State, Adapter, Template Method

**Week 6:** Domain modeling - Entities, Value Objects, Aggregates, Repositories

**Week 7:** Testing & Code Quality - TDD, mocks, contract tests

**Deliverables:**

- 4 LLD projects with class diagrams
- Clean Java implementations with tests
- Design documents with trade-offs

---

### 📅 Phase 3: HLD Foundations (7 weeks - Apr 19 to May 31)

**Status:** Not Started

**Topic:** <page url="https://www.notion.so/300ae59f84cd8148a39cf3c01acc0bad">High Level Design (HLD)</page>

**Week 1:** Scalability (vertical/horizontal, replication, sharding, caching, CDN)

**Week 2:** Data systems (SQL, NoSQL, Kafka, Elasticsearch, S3)

**Week 3:** Consistency & availability (CAP, PACELC, Raft, quorum)

**Week 4:** Performance & resilience (backpressure, circuit breakers, rate limiting)

**Week 5:** Observability (metrics, logs, traces, SLO/SLI)

**Week 6:** Security (OAuth2, OWASP Top 10, encryption)

**Week 7:** Review & integration

---

### 📅 Phase 4: Systems by Pattern (5 weeks - Jun 7 to Jul 5)

**Status:** Not Started

**Week 1:** Caching-heavy - CDN-backed image service

**Week 2:** Write-heavy - Event-driven orders with Kafka + outbox

**Week 3:** Read-heavy - News feed with fanout strategies

**Week 4:** Realtime - Chat with WebSockets + presence

**Week 5:** Batch/analytics - Clickstream ETL pipeline

For each system:

- Requirements → Capacity estimates → API design
- Data model → Component diagram → Scaling plan → Failure modes

---

### 📅 Phase 5: Interview Prep (2 weeks - Jul 12 to Jul 19)

**Status:** Not Started

**Activities:**

- 6-8 timed mock interviews (45-60 min each)
- Alternate LLD and HLD problems
- Record and self-critique sessions
- Build one-pager library of common components
- Perfect the interview template

---

## 🏗️ Projects to Build

1. **URL Shortener** (LLD + HLD)
    - Custom domains
    - Analytics tracking
    - High availability
2. **Rate Limiter Service**
    - Token bucket algorithm
    - Library + sidecar deployment
    - Distributed coordination
3. **Feature Flag Service**
    - Rollout rules (percentage, user groups)
    - Audit log
    - Real-time updates
4. **Document Collaboration**
    - CRDT or Operational Transform
    - Conflict resolution
    - Real-time sync

---

## 📚 Weekly Practice Routine

- ✅ 1 LLD kata (implement + tests)
- ✅ 1 HLD design doc
- ✅ Present 1 design for feedback
- ✅ Review open-source architecture (Kafka, Redis, etc.)

---

## 📖 Resources

**Books:**

- "Designing Data-Intensive Applications" (Martin Kleppmann)
- "System Design Interview" Vol 1-2 (Alex Xu)
- "Java Concurrency in Practice" (Brian Goetz)
- "Clean Code" (Robert Martin)
- "Design Patterns" (Gang of Four)

**Online Courses:**

- Grokking the Low Level Design Interview
- Grokking the System Design Interview
- MIT 6.824 - Distributed Systems (free)

**Tools:**

- **Diagrams:** [draw.io](http://draw.io), Excalidraw, Lucidchart
- **Testing:** JMH (microbenchmarks), k6, JMeter
- **Local dev:** Docker, Postman/cURL
- **Monitoring:** Prometheus, Grafana

---

## 🎯 Success Criteria

By end of 24 weeks:

- ✅ Understand all major design patterns
- ✅ Can design scalable systems with trade-off analysis
- ✅ Built 4 real projects end-to-end
- ✅ Completed 8+ timed mock interviews
- ✅ Have one-pager reference for common components
- ✅ Confident in both LLD and HLD interviews

---

*This roadmap updates as you progress through sessions.*