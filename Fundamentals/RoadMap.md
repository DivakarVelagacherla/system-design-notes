**Duration:** 3 weeks (Feb 8 - Feb 22, 2026)

**Goal:** Build foundation in Java concurrency, networking, and data storage

---

## 📊 Progress

**Week:** 1 of 3

**Sessions Completed:** 1/6

**Status:** In Progress

---

## 📅 Week 1 (Feb 8-9): Concurrency + Networking

### ✅ Session 1: Java Concurrency (Feb 8)

**Topics Covered:**

- Threads and thread creation
- Race conditions
- synchronized keyword
- ExecutorService and thread pools
- AtomicInteger

**Key Learnings:**

- Race conditions happen when threads share data without protection
- synchronized = lock that ensures only one thread at a time
- Executors = efficient thread management (reuse threads)
- Atomics = fast thread-safe operations for simple cases

**Code Examples:**

- Counter with race condition
- Counter with synchronized
- Counter with AtomicInteger
- Using ExecutorService

---

### ⏳ Session 2: Networking Basics (Feb 9)

**Topics to Cover:**

- HTTP/HTTPS fundamentals
- REST vs gRPC
- TCP vs UDP
- Request/response cycle

**Status:** Not Started

---

## 📅 Week 2 (Feb 15-16): Data Storage

### ⏳ Session 1: ACID, Transactions, Indexing

**Topics:**

- ACID vs BASE
- Transaction isolation levels
- Database indexing
- When to use transactions

**Status:** Not Started

---

### ⏳ Session 2: Postgres + Redis Hands-On

**Topics:**

- Setting up Postgres locally
- Basic SQL operations
- Redis basics (key-value store)
- Caching patterns

**Status:** Not Started

---

## 📅 Week 3 (Feb 22-23): Tooling + Review

### ⏳ Session 1: Docker, Postman, JMH

**Topics:**

- Docker basics for local dependencies
- API testing with Postman/cURL
- Microbenchmarking with JMH

**Status:** Not Started

---

### ⏳ Session 2: Fundamentals Review

**Topics:**

- Recap all fundamentals
- Practice problems
- Prepare for LLD phase

**Status:** Not Started

---

## 📖 Resources Used

**Java Concurrency:**

- Coding with John - Java Multithreading
- Defog Tech - Java Concurrency
- Oracle Java Concurrency Tutorial

---

## 📝 Notes

### Java Concurrency Summary

**1. Creating Threads:**

```java
class MyTask implements Runnable {
    public void run() {
        // Code runs in separate thread
    }
}

Thread t = new Thread(new MyTask());
t.start();
```

**2. Race Conditions:**

- Problem: Multiple threads access shared data
- Result: Data corruption
- Solution: synchronized or Atomics

**3. synchronized Keyword:**

```java
public synchronized void increment() {
    count++; // Only one thread at a time
}
```

**4. ExecutorService (Thread Pools):**

```java
ExecutorService executor = Executors.newFixedThreadPool(5);
executor.submit(new MyTask());
executor.shutdown();
```

**Types:**

- FixedThreadPool(n) - exactly n threads
- CachedThreadPool - creates as needed
- SingleThreadExecutor - only 1 thread

**5. AtomicInteger:**

```java
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet(); // Thread-safe without synchronized
```

**When to Use What:**

- Simple counter/flag → AtomicInteger
- Complex operations → synchronized
- Many tasks → ExecutorService

---

[Java Concurrency](//Fundamentals/Java%20Concurrency.md)

[Networking Basics](//Fundamentals/Networking%20Basics.md)

[Data Storage Fundamentals](//Fundamentals/Data%20Storage%20Fundamentals.md)