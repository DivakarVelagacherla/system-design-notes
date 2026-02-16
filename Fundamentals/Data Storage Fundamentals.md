**Week:** Phase 1, Week 1

**Sessions:** 3 & 4 (Feb 8, 2026)

**Duration:** ~2 hours

**Status:** Complete ✅

---

## 📚 Overview

Data storage is the foundation of system design. This session covers how databases work internally, transaction guarantees, scaling strategies, and when to use different storage solutions.

**Topics Covered:**

- ACID vs BASE properties
- Transactions and isolation levels
- Database internals (WAL, RAM, disk)
- Indexing strategies
- Read replicas and replication
- PostgreSQL in practice
- Redis caching
- SQL vs NoSQL comparison

---

## 🎯 ACID vs BASE

### What Problem Do Databases Solve?

**Without a database:**

```java
int balanceA = 500;
int balanceB = 200;

balanceA = balanceA - 100;  // What if crash here?
balanceB = balanceB + 100;  // Money disappears!
```

**Problems:**

1. Server crashes mid-operation → Data loss
2. Two operations happen simultaneously → Race conditions
3. Partial updates → Inconsistent state

**Databases solve these with guarantees!**

---

### ACID Properties

**ACID** = 4 guarantees for reliable transactions

**Transaction** = Group of operations that happen together (all or nothing)

---

#### A = Atomicity (All or Nothing)

**Meaning:** Transaction either completes fully or doesn't happen at all. No partial success.

```sql
BEGIN TRANSACTION;
  UPDATE accounts SET balance = balance - 100 WHERE id = 'A';
  UPDATE accounts SET balance = balance + 100 WHERE id = 'B';
COMMIT;
```

**With Atomicity:**

- Both updates succeed → Transfer complete ✓
- First succeeds, second fails → ROLLBACK both ✓
- Server crashes mid-transaction → ROLLBACK on restart ✓

**Without Atomicity:**

- First succeeds, second fails → $100 disappears! ✗

**Analogy:** Atomic = indivisible. Operations are glued together.

---

#### C = Consistency (Rules Never Broken)

**Meaning:** Database rules/constraints always maintained, even during transactions.

**Rules can be:**

- Balance cannot be negative
- Email must be unique
- Foreign keys must be valid

```sql
ALTER TABLE accounts ADD CONSTRAINT check_balance CHECK (balance >= 0);

BEGIN TRANSACTION;
  UPDATE accounts SET balance = balance - 100 WHERE id = 'A';
  -- If this makes balance negative, transaction FAILS
COMMIT;
```

**With Consistency:**

- Rules enforced always ✓
- Database never in invalid state ✓

**Analogy:** Database is a strict referee - enforces all rules, no exceptions.

---

#### I = Isolation (Transactions Don't Interfere)

**Meaning:** Multiple simultaneous transactions don't see each other's partial changes.

**Problem without isolation:**

```
Account A = $500

Transaction 1: Withdraw $100
  Read A = $500
  A = $500 - $100 = $400
  
Transaction 2: Withdraw $50 (starts before T1 finishes!)
  Read A = $500  ← Sees OLD value!
  A = $500 - $50 = $450
  
Result: A = $450 (should be $350!)
Lost $100!
```

**With Isolation:**

- T1 and T2 run in "bubbles"
- T2 waits for T1 to complete
- T2 sees A = $400, makes it $350 ✓

**Isolation Levels** (strictest to loosest):

1. **Serializable** - Transactions appear to run one-by-one
2. **Repeatable Read** - Same read twice = same result
3. **Read Committed** - Only see committed data (most common default)
4. **Read Uncommitted** - Can see uncommitted data (rare)

---

#### D = Durability (Data Survives Crashes)

**Meaning:** Once committed, data is permanently saved, even if system crashes immediately.

**Without Durability:**

```
User: "Transfer $1000"
Database: "Done! ✓"
[Server crashes 1 second later]
[Restart]
Database: "What transfer?" ✗
```

**With Durability:**

- Data written to disk (not just RAM)
- Survives power outages, crashes
- Uses Write-Ahead Log (WAL)

**How it works:**

1. Write to WAL on disk first
2. Confirm to user
3. Update actual data later (background)
4. If crash, replay WAL on restart

**Analogy:** Database keeps a permanent journal that survives disasters.

---

### ACID Summary

| Property | Guarantee | Example |
| --- | --- | --- |
| **A**tomicity | All or nothing | Transfer succeeds completely or not at all |
| **C**onsistency | Rules never broken | Balance never negative |
| **I**solation | No interference | Concurrent transfers don't conflict |
| **D**urability | Survives crashes | Data safe after confirmation |

**Use ACID when:**

- Money involved (banking, payments)
- Legal/compliance requirements
- Data loss unacceptable
- Need transactions

---

### BASE (Alternative to ACID)

**BASE** = **B**asically **A**vailable, **S**oft state, **E**ventually consistent

Used when **speed and availability** > perfect consistency

---

#### BA = Basically Available

System responds even if some parts are down.

**Example - Facebook:**

- Some servers down?
- Show cached/stale data
- Better than showing nothing

---

#### S = Soft State

Data can be in temporary/uncertain state.

**Example - Twitter likes:**

- You like a tweet
- Shows immediately (optimistic)
- Actually saving in background
- Might take 1-2 seconds

---

#### E = Eventually Consistent

All copies become consistent... eventually (not immediately).

**Example - Instagram followers:**

- You follow someone
- They see it immediately
- Your friend checks, doesn't see it yet
- 1 second later, everyone sees it

---

### ACID vs BASE Comparison

|  | ACID | BASE |
| --- | --- | --- |
| **Consistency** | Immediate | Eventually |
| **Availability** | May block | Always responds |
| **Speed** | Slower | Faster |
| **Use case** | Banking | Social media |
| **Database** | PostgreSQL, MySQL | Cassandra, DynamoDB |

---

### Real-World: Hybrid Approach

Most companies use BOTH:

**Instagram:**

- User payments → ACID (PostgreSQL) 💰
- Photos, likes → BASE (Cassandra) 📸
- User auth → ACID (can't lose passwords) 🔐

**Amazon:**

- Cart, checkout, payment → ACID 💳
- Product recommendations → BASE ⭐

---

## 🔒 Transactions and Isolation Levels

### The 4 Isolation Levels

From strictest to loosest:

1. Serializable
2. Repeatable Read
3. Read Committed (default)
4. Read Uncommitted

---

### Problem 1: Dirty Read

**Reading uncommitted data from another transaction.**

```
Time    Transaction 1              Transaction 2
----    -----------------          -----------------
1       BEGIN
2       UPDATE balance = 900
3                                  BEGIN
4                                  SELECT balance
                                   → Reads 900 (dirty!)
5       ROLLBACK (error!)
6                                  Uses 900 in calculation
                                   → WRONG!
```

**Problem:** T2 read data that was never committed!

**Solution:** Read Committed or higher

**Banking example:**

```
T1: Transfer $100 (not committed)
T2: Check if enough for $200 withdrawal
    Reads uncommitted balance
    Approves withdrawal
T1: ROLLBACK
Result: Overdraft! ✗
```

---

### Problem 2: Non-Repeatable Read

**Same query, different results within one transaction.**

```
Time    Transaction 1              Transaction 2
----    -----------------          -----------------
1       BEGIN
2       SELECT balance
        → Returns 1000
3                                  BEGIN
4                                  UPDATE balance = 500
5                                  COMMIT
6       SELECT balance
        → Returns 500 (changed!)
```

**Problem:** Same query gave different results!

**Read Committed:** Allows this (you see latest data) ✓

**Repeatable Read:** Prevents this (same snapshot) ✓

**When it matters: Generating reports**

```java
BEGIN;
  total = SELECT SUM(balance);  // $10,000
  count = SELECT COUNT(*);      // 100 accounts
  average = total / count;      // $100
  
  // Meanwhile someone inserts new account
  
  count = SELECT COUNT(*);      // 101 accounts (changed!)
  // Calculations now inconsistent!
COMMIT;
```

**Solution:** Use Repeatable Read for reports

---

### Repeatable Read: How It Works

**Creates a snapshot at transaction start:**

```
Time 0: BEGIN TRANSACTION (Repeatable Read)
        [Snapshot created: 1000 accounts, $1M total]

Time 1: SELECT SUM(balance)
        → Returns $1M (from snapshot)

Time 2: [Someone deposits $10K]
        [New accounts created]
        → YOUR TRANSACTION DOESN'T SEE THIS!

Time 3: SELECT COUNT(*)
        → Returns 1000 (still from snapshot)

Time 4: Calculate average = $1M / 1000 = $1000
        → CONSISTENT! ✓

COMMIT
```

**Key insight:** Your transaction sees database as it was at start time.

**Think of it like:** Taking a photograph (frozen) vs watching live video (changing)

**Implementation:** Multi-Version Concurrency Control (MVCC)

- Database keeps multiple versions of each row
- Your transaction always sees the version from your start time
- Other transactions see latest version
- No locks needed! ✓

---

### Problem 3: Phantom Read

**New rows appear in query results (even with Repeatable Read).**

```
Time    Transaction 1              Transaction 2
----    -----------------          -----------------
1       BEGIN (Repeatable Read)
2       SELECT * WHERE status = 'pending'
        → Returns 10 rows
3                                  BEGIN
4                                  INSERT status = 'pending'
5                                  COMMIT
6       SELECT * WHERE status = 'pending'
        → Returns 11 rows (phantom!)
```

**Problem:** New rows "appeared" like phantoms 👻

**Repeatable Read:** Protects existing rows, not new inserts ✗

**Serializable:** Prevents phantom reads ✓

**When it matters: Inventory management**

```java
BEGIN;
  available = SELECT COUNT(*) WHERE status = 'available';
  // Returns 5 items
  
  // Can I fulfill order for 5 items? Yes!
  
  // Meanwhile: Someone else reserves 2 items (INSERT)
  
  // Try to reserve 5 items
  // Now only 3 actually available!
  // OVERSOLD by 2! ✗
COMMIT;
```

**Solution:** Use Serializable for critical inventory

---

### Isolation Levels Summary

| Level | Dirty Read | Non-Repeatable | Phantom |
| --- | --- | --- | --- |
| Read Uncommitted | ❌ Possible | ❌ Possible | ❌ Possible |
| Read Committed | ✅ Prevented | ❌ Possible | ❌ Possible |
| Repeatable Read | ✅ Prevented | ✅ Prevented | ❌ Possible |
| Serializable | ✅ Prevented | ✅ Prevented | ✅ Prevented |

---

### When to Use Which

**Read Committed (Default - 95% of cases):**

- Banking transactions
- E-commerce orders
- Most applications

**Repeatable Read:**

- Financial reports
- Analytics queries
- Batch processing

**Serializable (Rare):**

- Critical financial operations
- Inventory management (prevent overselling)
- ⚠️ SLOW! Locks everything

**Read Uncommitted (Almost never):**

- Only for approximate analytics
- Example: "Roughly how many users online?"

---

## 💾 Database Internals

### Write-Ahead Log (WAL)

**The Problem:**

```
User: "Transfer $100"
Database updates RAM (fast!)
Database: "Done! ✓"
[Power outage before writing to disk]
Data lost! ✗
```

**Solution: WAL**

---

### How WAL Works

```
1. User: "Transfer $100"

2. Database writes to WAL (on disk):
   "Transaction 123: UPDATE accounts SET balance=400 WHERE id='A'"
   "Transaction 123: UPDATE accounts SET balance=300 WHERE id='B'"
   "Transaction 123: COMMIT"
   
3. Database updates data in RAM

4. Database tells user: "Done! ✓"

5. Later (background): Write RAM → disk
```

**Key:** WAL is sequential write (fast!) vs data files are random writes (slow)

---

### Recovery After Crash

**Crash after WAL write, before data write:**

```
[Crash]
[Restart]
Database reads WAL: "Oh, Transaction 123 was committed"
Database replays: Updates A and B from WAL
Result: Data recovered! ✓
```

**Crash before WAL write:**

```
[Crash]
[Restart]
Database reads WAL: "No record of Transaction 123"
Result: Transaction never happened (user retries)
```

---

### What is a "Crash"?

**Crash = Server stops unexpectedly**

Causes:

1. **Power failure** - Electricity lost, RAM → gone
2. **Hardware failure** - RAM/CPU/disk dies
3. **Software bug** - Process crashes, kernel panic
4. **Out of memory** - OS kills process
5. **Kill signal** - `kill -9` terminates process

**What happens:**

- RAM (volatile) → ALL DATA LOST instantly
- Disk (persistent) → DATA SURVIVES

**Recovery:**

1. Server restarts
2. Database reads WAL from disk
3. Replays committed transactions
4. Database ready!

---

### Database Memory Architecture

```
┌─────────────────────────────────────┐
│           Computer RAM              │
├─────────────────────────────────────┤
│  PostgreSQL (6 GB)                  │
│  ├── Shared buffers (4 GB)         │  ← Cache frequently used data
│  ├── WAL buffers (16 MB)           │  ← Staging for WAL writes
│  └── Work memory (2 GB)            │  ← For queries, sorting
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│           Disk                      │
│  ├── Data files (800 GB)           │  ← Your actual tables
│  ├── WAL files (10 GB)             │  ← Transaction logs
│  └── Indexes (190 GB)              │  ← Speed up queries
└─────────────────────────────────────┘
```

**Speed comparison:**

- RAM: 100 nanoseconds
- SSD: 100 microseconds (1,000x slower)
- HDD: 10 milliseconds (100,000x slower)

**Why databases use RAM:**

- Cache frequently accessed data
- Only hit disk when necessary
- Write to disk in background

---

### How a Query Works

```sql
SELECT * FROM users WHERE id = 123;
```

**Behind the scenes:**

```
1. Check RAM cache (shared buffers)
   ├── Found? Return immediately! (fast!)
   └── Not found? Go to step 2

2. Read from disk
   ├── Load page containing row 123 into RAM
   └── Keep in RAM for next time

3. Return result

4. (Background) If data modified:
   ├── Write to WAL (disk, sequential)
   ├── Mark RAM page as "dirty"
   └── (Later) Flush dirty pages to data files
```

---

### What You Control vs What Database Does

**Database handles automatically:**

- ✅ WAL management
- ✅ RAM caching (shared buffers)
- ✅ Background writes to disk
- ✅ Crash recovery
- ✅ MVCC (multi-version concurrency)

**You control:**

- 🎛️ Transactions (BEGIN/COMMIT/ROLLBACK)
- 🎛️ Creating indexes
- 🎛️ Configuration tuning (optional)
- 🎛️ Isolation level (optional)

**Think of it like a car:**

- Database = Engine (automatic)
- You = Driver (steering, when to stop)

---

## 📇 Indexing

### What is an Index?

**Without index:**

```sql
SELECT * FROM users WHERE email = 'john@example.com';

Scans entire table:
Row 1: alice@... (no)
Row 2: bob@... (no)
...
Row 1,000,000: john@... (found!)

Time: 145ms 🐌
```

**With index:**

```sql
CREATE INDEX idx_email ON users(email);

SELECT * FROM users WHERE email = 'john@example.com';

Uses index (like book index):
"john@... is at Row 1,000,000"
→ Goes directly there

Time: 0.2ms ⚡ (600x faster!)
```

**Analogy:** Finding word in dictionary vs reading entire dictionary.

---

### Why NOT Index Everything?

**Problem 1: Space**

```
Table: 1 GB
Index on email: +200 MB
Index on name: +150 MB
Index on age: +100 MB

Total: 1.63 GB (63% overhead!)
```

**Problem 2: Slower Writes**

```sql
INSERT INTO users VALUES (...);

Without indexes: 1ms
With 4 indexes: 5ms (must update all indexes!)
```

**Problem 3: Maintenance**

- Must keep indexes sorted
- Rebalance tree structures
- Uses RAM to cache indexes

**Trade-off:**

- ✅ Faster reads
- ❌ Slower writes
- ❌ More disk space
- ❌ More RAM needed

---

### When to Create an Index

**Create index when:**

- ✅ Column in WHERE clause (frequently)
- ✅ Column in JOIN conditions
- ✅ Column in ORDER BY
- ✅ Read-heavy workload (90% reads)

**Skip index when:**

- ❌ Column rarely queried
- ❌ Write-heavy workload
- ❌ Few unique values (boolean)
- ❌ Small table (< 1000 rows)

**Example: Banking**

```sql
-- Good
CREATE INDEX idx_transaction_date ON transactions(transaction_date);
-- Users check "last month's transactions" often

-- Bad
CREATE INDEX idx_is_active ON users(is_active);
-- Only 2 values (true/false), not selective
```

---

### Types of Indexes

**1. Single Column**

```sql
CREATE INDEX idx_email ON users(email);
-- Fast for: WHERE email = '...'
```

**2. Composite (Multiple Columns)**

```sql
CREATE INDEX idx_city_age ON users(city, age);
-- Fast for: WHERE city = 'NYC' AND age = 25
-- Also fast: WHERE city = 'NYC' (uses first column)
-- NOT fast: WHERE age = 25 (doesn't use first column)
```

**Order matters!** `(city, age)` ≠ `(age, city)`

**3. Unique Index**

```sql
CREATE UNIQUE INDEX idx_unique_email ON users(email);
-- Enforces uniqueness + speeds up lookups
```

---

### Composite Index Best Practices

**Query:**

```sql
SELECT * FROM users 
WHERE city = 'NYC' AND age > 25 
ORDER BY created_at DESC
LIMIT 10;
```

**Best index:**

```sql
CREATE INDEX idx_city_age_created ON users(city, age, created_at);
```

**Rule:** `(equality, range, order_by)`

**Why?**

- Filters by city and age (fast!)
- Data already sorted by created_at (no sort needed!)
- Just grab first 10 rows

---

### How Indexes Work (B-Tree)

```
Think of it like a tree:

                    [M]
                   /   \
            [D - L]     [N - Z]
           /   |   \       /    \
      [A-C] [E-G] [H-L] [N-P] [Q-Z]
        |     |     |     |      |
      Data  Data  Data  Data   Data
```

**Finding "John":**

1. Start at root: J < M, go left
2. J > L, go middle-right
3. Found! Jump to data

**Time:** O(log n) - Very fast!

---

## 🔄 Read Replicas and Scaling

### The Idea

**Problem:**

- Primary DB handles ALL reads and writes
- Writes slow down with many indexes
- Can't scale reads and writes independently

**Solution: Read Replicas**

```
┌─────────────────┐
│  Primary DB     │ ← All WRITES
│  (Few indexes)  │    Fast writes!
└────────┬────────┘
         │ Replication
         ├──────────┬──────────┐
         ▼          ▼          ▼
   ┌─────────┐ ┌─────────┐ ┌─────────┐
   │ Replica │ │ Replica │ │ Replica │ ← All READS
   │ (Lots   │ │ (Lots   │ │ (Lots   │    Many indexes!
   │  of     │ │  of     │ │  of     │
   │ indexes)│ │ indexes)│ │ indexes)│
   └─────────┘ └─────────┘ └─────────┘
```

**This is how YouTube, Netflix, Instagram scale!**

---

### Replication Flow

```
1. User signs up
   → Write to PRIMARY
   → Primary: INSERT succeeds

2. Primary sends changes to replicas
   → "New user: john@example.com"
   → Replicas update (async, ~100ms delay)

3. User searches
   → Read from REPLICA
   → Fast! (lots of indexes)
```

---

### Replication Lag Problem

**The issue:**

```
Time 0: User updates profile
        → Write to PRIMARY

Time 50ms: User refreshes page
        → Read from REPLICA
        → Replica hasn't synced yet!
        → Shows old data ✗
```

---

### Solutions to Replication Lag

**Solution 1: Read from Primary After Write**

```java
// User updates profile
writeDB.execute("UPDATE users SET name = 'John' WHERE id = 123");

// Read from PRIMARY (not replica)
User user = writeDB.query("SELECT * FROM users WHERE id = 123");
```

**Solution 2: Read Your Own Writes**

```java
String userId = getCurrentUserId();

if (queryingOwnData(userId)) {
    readFromPrimary();  // Your own data
} else {
    readFromReplica();  // Others' data (can be stale)
}
```

**Solution 3: Session Stickiness**

```java
// After write, remember timestamp
session.set("last_write", System.currentTimeMillis());

// On read:
if (withinOneSecondOfWrite()) {
    readFromPrimary();
} else {
    readFromReplica();
}
```

---

### Real-World Examples

**Instagram:**

- You post photo → Write to primary
- You view YOUR photos → Read from primary
- Friends view → Read from replica (okay if 100ms delay)

**Amazon:**

- Add to cart → Write to primary
- View YOUR cart → Read from primary
- Browse products → Read from replicas

---

### Implementation: Two Connection Pools

```java
// Setup
DataSource primaryDB = createConnection("primary-db.amazonaws.com");
DataSource replicaDB = createConnection("replica-db.amazonaws.com");

// Lambda function
public void processOrder(Order order) {
    // Read from replica
    int stock = replicaDB.query(
        "SELECT stock FROM inventory WHERE id = ?", 
        productId
    );
    
    if (stock > 0) {
        // Write to primary
        primaryDB.execute(
            "UPDATE inventory SET stock = stock - 1 WHERE id = ?", 
            productId
        );
    }
}
```

**You manually choose which connection!**

---

## 🗄️ PostgreSQL in Practice

### Transaction Example

```sql
-- Bank transfer
BEGIN;

UPDATE accounts 
SET balance = balance - 100 
WHERE account_id = 'A';

UPDATE accounts 
SET balance = balance + 100 
WHERE account_id = 'B';

COMMIT;  -- Writes to WAL, returns success
```

**Behind the scenes:**

1. BEGIN - Creates transaction
2. Updates written to WAL (durability)
3. Data updated in RAM (speed)
4. COMMIT - Makes permanent
5. Background flush RAM → disk

---

### Setting Isolation Level

```sql
-- Default: Read Committed
BEGIN;
SELECT * FROM users WHERE id = 1;
COMMIT;

-- For reports: Repeatable Read
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT SUM(balance) FROM accounts;
-- ... time passes ...
SELECT COUNT(*) FROM accounts;  -- Same snapshot!
COMMIT;

-- Critical: Serializable
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT * FROM inventory WHERE product_id = 123;
INSERT INTO orders (...);
COMMIT;
```

---

### Checking Index Performance

```sql
-- Before index
EXPLAIN ANALYZE
SELECT * FROM users WHERE email = 'john@example.com';
-- Result: Seq Scan, 145ms

-- Create index
CREATE INDEX idx_email ON users(email);

-- After index
EXPLAIN ANALYZE
SELECT * FROM users WHERE email = 'john@example.com';
-- Result: Index Scan, 0.2ms (600x faster!)
```

---

### View Index Usage

```sql
SELECT 
    tablename,
    indexname,
    idx_scan  -- How many times used
FROM pg_stat_user_indexes
WHERE tablename = 'users';

-- If idx_scan = 0 → Index never used, DELETE IT!
```

---

### Configuration (Optional Tuning)

```sql
-- See settings
SHOW shared_buffers;  -- RAM for caching
SHOW max_connections;
SHOW work_mem;
```

**Common production settings:**

```
shared_buffers = 4GB      (25% of RAM)
work_mem = 64MB           (per query operation)
max_connections = 100
```

---

## 🚀 Redis Basics

**Redis = Remote Dictionary Server**

In-memory key-value store (think HashMap in the cloud)

---

### Why Redis?

**Problem:**

```
User requests profile
→ Query Postgres: 50ms
→ User refreshes
→ Query Postgres again: 50ms
→ Wasted time!
```

**Solution:**

```
First request:
→ Check Redis: Not found
→ Query Postgres: 50ms
→ Store in Redis

Second request:
→ Check Redis: Found! (0.5ms)
→ 100x faster!
```

---

### Redis Data Structures

**1. String**

```
SET user:123:name "John Doe"
GET user:123:name
→ "John Doe"

SETEX session:abc 3600 "user_data"  -- Expires in 1 hour
```

**2. Hash (mini-document)**

```
HSET user:123 name "John" age 30 city "NYC"
HGET user:123 name
→ "John"

HGETALL user:123
→ {name: "John", age: 30, city: "NYC"}
```

**3. List**

```
LPUSH recent_orders:user123 "order_456"
LRANGE recent_orders:user123 0 9
→ Last 10 orders
```

**4. Set (unique items)**

```
SADD online_users "user_123"
SMEMBERS online_users
→ ["user_123", "user_456"]
```

**5. Sorted Set (with scores)**

```
ZADD leaderboard 100 "player1"
ZADD leaderboard 250 "player2"
ZRANGE leaderboard 0 -1 WITHSCORES
→ player1: 100, player2: 250
```

---

### Caching Pattern (Cache-Aside)

```java
public User getUser(int userId) {
    // 1. Try cache
    String cached = redis.get("user:" + userId);
    if (cached != null) {
        return JSON.parse(cached);  // Cache hit!
    }
    
    // 2. Cache miss, query DB
    User user = postgres.query(
        "SELECT * FROM users WHERE id = ?", 
        userId
    );
    
    // 3. Store in cache (1 hour TTL)
    redis.setex(
        "user:" + userId, 
        3600, 
        JSON.stringify(user)
    );
    
    return user;
}
```

---

### When Data Changes

```java
public void updateUser(int userId, String newName) {
    // 1. Update database
    postgres.execute(
        "UPDATE users SET name = ? WHERE id = ?", 
        newName, userId
    );
    
    // 2. Invalidate cache
    redis.del("user:" + userId);
    // Next read will get fresh data from DB
}
```

---

### Common Use Cases

**1. Session Storage**

```
SET session:abc123 "user_id=456" EX 3600
```

**2. Rate Limiting**

```
INCR api_calls:user123
EXPIRE api_calls:user123 60
-- Track API calls per minute
```

**3. Leaderboards**

```
ZADD game_scores 1000 "player1"
ZREVRANGE game_scores 0 9  -- Top 10
```

**4. Pub/Sub**

```
PUBLISH chat:room123 "Hello!"
SUBSCRIBE chat:room123
```

---

### Redis vs PostgreSQL

|  | PostgreSQL | Redis |
| --- | --- | --- |
| **Storage** | Disk | RAM |
| **Speed** | 50ms | 0.5ms |
| **Data size** | Terabytes | Limited by RAM |
| **Durability** | ACID | Can lose data |
| **Queries** | SQL (complex) | Key-value (simple) |
| **Use case** | Primary DB | Cache, sessions |

---

## 🗃️ SQL vs NoSQL

### SQL Databases (PostgreSQL, MySQL)

**Structure:** Tables with fixed schema

```
users table:
| id | name  | email         | age |
|----|-------|---------------|-----|
| 1  | John  | john@...      | 30  |
| 2  | Jane  | jane@...      | 25  |
```

**Characteristics:**

- ✅ ACID transactions
- ✅ Complex queries (JOINs)
- ✅ Data consistency
- ✅ Mature ecosystem
- ❌ Harder to scale horizontally
- ❌ Schema changes painful

**Use when:**

- Financial data
- Relationships matter
- Need transactions
- Structured data

---

### NoSQL: Document Store (MongoDB, DynamoDB)

**Structure:** JSON-like documents

```json
{
  "_id": "123",
  "name": "John",
  "email": "john@...",
  "addresses": [
    {"type": "home", "city": "NYC"},
    {"type": "work", "city": "SF"}
  ]
}
```

**Characteristics:**

- ✅ Flexible schema
- ✅ Scales horizontally
- ✅ Fast simple queries
- ❌ No JOINs
- ❌ Eventual consistency

**Use when:**

- Rapid development
- Massive scale
- Document-like data
- Schema changes often

---

### NoSQL: Key-Value (Redis, DynamoDB)

```
key: "user:123"
value: {...}
```

**Use when:**

- Caching
- Session storage
- Simple lookups

---

### NoSQL: Column Store (Cassandra)

```
user:123 → name:John, email:john@..., login_2024_01_01:success, ...
```

**Use when:**

- Time-series data
- Massive writes
- Analytics

---

### NoSQL: Graph (Neo4j)

```
(User:John) -[FOLLOWS]-> (User:Jane)
(User:Jane) -[LIKES]-> (Post:123)
```

**Use when:**

- Social networks
- Recommendations
- Complex relationships

---

### Decision Matrix

**Choose SQL when:**

- ✅ ACID transactions needed
- ✅ Complex queries with JOINs
- ✅ Data has relationships
- ✅ Financial/critical data
- Example: Banking, E-commerce

**Choose NoSQL when:**

- ✅ Massive scale
- ✅ Flexible schema
- ✅ Simple queries by ID
- ✅ Eventual consistency OK
- Example: Social media, IoT

**Choose Redis when:**

- ✅ Need <1ms speed
- ✅ Caching
- ✅ Temporary data
- ✅ Real-time features
- Example: Sessions, leaderboards

---

### Real Architecture (Instagram)

```
PostgreSQL:
- User accounts
- Payments
- Anything needing ACID

Cassandra:
- Photos metadata
- Likes/comments
- Activity feeds
- Massive scale, eventual consistency OK

Redis:
- User sessions
- Recent photos cache
- Real-time notifications
- Feed cache
```

**All three together!**

---

## 🎯 Key Takeaways

**ACID vs BASE:**

- ACID for banking (consistency)
- BASE for social media (speed/availability)
- Most companies use both

**Transactions:**

- Read Committed (default)
- Repeatable Read (reports)
- Serializable (critical inventory)

**Database Internals:**

- WAL ensures durability
- RAM caching for speed
- Databases are just programs using RAM/disk

**Indexing:**

- Fast reads, slow writes
- Don't index everything
- Composite indexes: (equality, range, order_by)

**Scaling:**

- Read replicas for scaling reads
- Primary for writes, replicas for reads
- Handle replication lag carefully

**Redis:**

- In-memory cache (100x faster)
- Use for sessions, rate limiting, leaderboards
- Complement to primary database

**SQL vs NoSQL:**

- SQL for transactions and relationships
- NoSQL for scale and flexibility
- Redis for speed and caching
- Use all three in real architectures

---

## 📚 What's Next

**Week 2 (Feb 15-16):**

- More data storage patterns
- Tooling and observability
- Monitoring and debugging

**Week 3 (Feb 22-23):**

- Review fundamentals
- Prepare for LLD phase

---

## ⚡ Redis Caching Performance - Hands-On Experiment

### The Problem

You're building an API that fetches user profiles. Each database query takes time. At 1000 requests/second, your database is struggling. **What happens when you add Redis as a cache layer?**

---

### The Experiment

Built a Java application to measure actual performance difference between:

1. Querying PostgreSQL directly (no cache)
2. Using Redis as a cache layer

**Setup:**

- PostgreSQL with 1000 test users
- Redis running in Docker
- Artificial delay (`pg_sleep(0.1)`) to simulate realistic query time
- Java application using Jedis (Redis client) and JDBC (PostgreSQL)

---

### Results Observed

**Without Cache (10 direct DB calls):**

```
Time: 1276ms
~127ms per query
```

**With Cache - First Call (cache miss):**

```
Time: 136ms
Goes to DB + stores in Redis
```

**With Cache - Next 9 Calls (cache hits):**

```
Time: 5ms total
~0.56ms per call
```

**Speedup: 227x faster when data is cached!**

---

### Why Redis is Faster

**PostgreSQL (slow):**

- Data stored on **disk** (SSD/HDD)
- Must parse SQL, plan query, find row
- Even simple queries have overhead
- Typical query: 50-100ms

**Redis (fast):**

- Data stored in **RAM** (memory)
- Simple key-value lookup (HashMap)
- Hash table lookup is O(1)
- RAM access is ~100,000x faster than disk
- Typical lookup: 0.5-1ms

**The Trade-off:**

- Redis: Fast but **limited by RAM** (expensive, volatile)
- Postgres: Slower but **durable** (data survives crashes)

---

### Cache-Aside Pattern (What We Built)

```java
public User getUserWithCache(Jedis redis, int userId) {
    String cacheKey = "user:" + userId;
    
    // 1. Try cache first
    String cached = redis.get(cacheKey);
    if (cached != null) {
        return gson.fromJson(cached, User.class);  // Cache hit!
    }
    
    // 2. Cache miss - fetch from DB
    User user = getUserFromDB(userId);
    
    // 3. Store in cache with TTL (60 seconds)
    if (user != null) {
        redis.setex(cacheKey, 60, gson.toJson(user));
    }
    
    return user;
}
```

**Flow:**

1. Check Redis first
2. If found → return immediately (fast!)
3. If not found → query DB, then cache it
4. Set expiration (TTL) to avoid stale data

---

### When to Use Redis Caching

**✅ Use Redis for:**

- **Read-heavy data** - Product catalogs, user profiles
- **Expensive queries** - Queries with joins, aggregations
- **Frequently accessed data** - Homepage content, trending posts
- **Temporary data** - Sessions, rate limiting

**❌ Don't use Redis for:**

- **Frequently changing data** - Real-time stock prices (cache becomes stale)
- **Security-sensitive data** - Passwords, API keys (Redis is not encrypted)
- **Data requiring 100% accuracy** - Bank balances (can't risk serving stale data)
- **Large datasets rarely accessed** - Wastes precious RAM

---

### Cache Invalidation Strategies

When you update data in the database, the cached copy becomes **stale**. Three ways to handle this:

**Strategy 1: Write-Through Cache**

```java
public void updateUser(int userId, String newName) {
    // 1. Update database
    postgres.execute("UPDATE users SET name = ? WHERE id = ?", newName, userId);
    
    // 2. Update cache immediately
    User updated = getUserFromDB(userId);
    redis.setex("user:" + userId, 3600, gson.toJson(updated));
}
```

- **Pros:** Cache always fresh
- **Cons:** Extra write operation, what if cache update fails?

**Strategy 2: Cache Invalidation (Delete from cache)**

```java
public void updateUser(int userId, String newName) {
    // 1. Update database
    postgres.execute("UPDATE users SET name = ? WHERE id = ?", newName, userId);
    
    // 2. Delete from cache
    redis.del("user:" + userId);
    // Next read will be cache miss → fetch from DB → cache it
}
```

- **Pros:** Simpler, no stale data
- **Cons:** Next request is slower (cache miss)

**Strategy 3: TTL (Time To Live)**

```java
redis.setex("user:" + userId, 60, userData);  // Expires after 60 seconds
```

- **Pros:** Simple, automatic cleanup, eventual consistency
- **Cons:** Might serve stale data until expiration

**Real-world:** Most companies use **Strategy 2 (Invalidation) + TTL** as backup

---

### Interview Connection

**Interviewer:** "Your API is slow. Users complain about loading times. How do you fix it?"

**Answer Framework:**

1. **Measure first** - "I'd profile to find the bottleneck. If it's database queries..."
2. **Identify pattern** - "Are we doing expensive queries repeatedly? Reading same data often?"
3. **Propose caching** - "Add Redis cache layer between API and database"
4. **Explain trade-offs:**
    - ✅ 100-200x speedup for cached data
    - ❌ Need to handle cache invalidation
    - ❌ Limited by RAM (need to monitor memory usage)
    - ❌ Adds complexity (one more system to maintain)
5. **When NOT to cache** - "For real-time data or security-sensitive info, I'd optimize the database query instead with indexes or read replicas"

**Follow-up:** "What if cached data becomes stale?"

- "I'd use cache invalidation - when data updates, delete the cache key. Next read gets fresh data."
- "Plus add TTL as a safety net - data expires after X seconds anyway."

---

### Key Takeaways

1. **Redis gives 100-200x speedup** for frequently accessed data
2. **Redis stores data in RAM** (fast but limited and volatile)
3. **Cache-Aside pattern:** Check cache → if miss, query DB and cache it
4. **Cache invalidation is critical** - delete or update cache when data changes
5. **Use TTL as safety net** - data auto-expires to prevent serving stale data forever
6. **Not everything should be cached** - consider data update frequency and criticality

---

## ⚡ Redis Caching - Hands-On Performance Testing

### The Problem

You're building an API that fetches user profiles. Each database query takes time. At 1000 requests/second, your database is struggling.

**Question:** What happens when you add Redis as a cache layer?

---

### The Experiment

We built a Java application to measure the actual performance difference between:

1. Querying PostgreSQL directly (no cache)
2. Using Redis as a cache layer

**Setup:**

- PostgreSQL with 1000 test users
- Redis running

---