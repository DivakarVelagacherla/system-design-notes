**Session:** Phase 1, Week 1, Session 1

**Date:** February 8, 2026

**Duration:** ~1 hour

**Status:** Complete ✅

---

## 🎯 What is Concurrency?

Concurrency means multiple tasks running **simultaneously**. In Java, this is done using **threads**.

**Real-world analogy:**

- Single-threaded = One chef cooking all orders one by one
- Multi-threaded = Multiple chefs cooking different orders at the same time

**Why use threads?**

- Web server handling multiple requests
- Background tasks while UI remains responsive
- Processing large amounts of data in parallel

---

## 💻 Creating Threads

### Method 1: Implement Runnable (Recommended)

```java
class MyTask implements Runnable {
    public void run() {
        System.out.println("Hello from thread!");
    }
}

public class Main {
    public static void main(String[] args) {
        Thread t1 = new Thread(new MyTask());
        t1.start();  // Starts the thread
    }
}
```

**Key points:**

- Implement `Runnable` interface
- Override `run()` method (code that executes in thread)
- Create `Thread` object and pass your task
- Call `start()` to run (NOT `run()` directly!)

---

### Passing Data to Threads

```java
class PrintTask implements Runnable {
    private String message;
    
    public PrintTask(String message) {
        this.message = message;
    }
    
    public void run() {
        System.out.println(message);
    }
}

Thread t1 = new Thread(new PrintTask("Hello from Thread 1"));
Thread t2 = new Thread(new PrintTask("Hello from Thread 2"));
t1.start();
t2.start();
```

---

### Waiting for Threads: join()

```java
Thread t1 = new Thread(new MyTask());
t1.start();
t1.join();  // Wait for t1 to finish before continuing

System.out.println("Thread finished!");
```

**Without `join()`:** Main thread might finish before other threads

**With `join()`:** Main thread waits for other threads to complete

---

## ⚠️ The Problem: Race Conditions

**Race condition** = Multiple threads accessing shared data, causing corruption

### Example: Counter Without Protection

```java
class Counter {
    private int count = 0;
    
    public void increment() {
        count = count + 1;  // NOT thread-safe!
    }
    
    public int getCount() {
        return count;
    }
}
```

**What happens:**

```
count = count + 1 is actually THREE operations:
1. Read current value (0)
2. Add 1 (0 + 1 = 1)
3. Write back (1)

Thread A: Read (0)
Thread B: Read (0)  ← Both read 0!
Thread A: Add 1 = 1
Thread B: Add 1 = 1
Thread A: Write 1
Thread B: Write 1  ← Overwrites A's change!

Result: count = 1 (should be 2!)
```

### Making It Visible

Add a sleep to simulate slow processing and make race condition obvious:

```java
public void increment() {
    int temp = count;
    
    try {
        Thread.sleep(10);  // Simulate processing
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    
    count = temp + 1;
}
```

Now you'll consistently see the race condition!

---

## 🔒 Solution 1: synchronized Keyword

**synchronized** = Lock that ensures only ONE thread can execute the method at a time

```java
class Counter {
    private int count = 0;
    
    public synchronized void increment() {
        int temp = count;
        
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        count = temp + 1;
    }
    
    public int getCount() {
        return count;
    }
}
```

**How it works:**

```
Thread A enters increment() → LOCKS the method
Thread B tries to enter → BLOCKED (waits)
Thread A completes → UNLOCKS
Thread B enters → Can now execute
```

**Result:** No race condition! Count is always correct.

**Key point:** `synchronized` is a keyword, no import needed!

---

## 📦 Solution 2: Thread Pools (ExecutorService)

### The Problem with Manual Threads

```java
// Creating 1000 threads manually - BAD!
for (int i = 0; i < 1000; i++) {
    Thread t = new Thread(new MyTask());
    t.start();
}
```

**Why this is bad:**

- Creating threads is expensive (memory + CPU)
- 1000 threads = system slows down
- Hard to manage and control

---

### Thread Pools = Reuse Threads

**Think:** Restaurant with 10 waiters serving 100 customers

- Don't hire 100 waiters!
- 10 waiters serve customers one by one
- Reuse the same waiters

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Main {
    public static void main(String[] args) {
        // Create pool of 5 threads
        ExecutorService executor = Executors.newFixedThreadPool(5);
        
        // Submit 100 tasks
        for (int i = 0; i < 100; i++) {
            executor.submit(new MyTask());
        }
        
        // Shutdown when done
        executor.shutdown();
    }
}
```

**Benefits:**

- 5 threads process 100 tasks (efficient!)
- Saves memory and CPU
- Better performance
- Easier to manage

---

### Types of Thread Pools

**1. FixedThreadPool** - Fixed number of threads

```java
ExecutorService executor = Executors.newFixedThreadPool(10);
```

**Use when:** You know exactly how many threads needed

**Example:** Web server handling 10 requests at a time

---

**2. CachedThreadPool** - Creates threads as needed

```java
ExecutorService executor = Executors.newCachedThreadPool();
```

**Use when:** Number of tasks varies a lot

**Example:** Background jobs that come randomly

---

**3. SingleThreadExecutor** - Only 1 thread

```java
ExecutorService executor = Executors.newSingleThreadExecutor();
```

**Use when:** Tasks must run one at a time, in order

**Example:** Writing to a log file (can't have multiple writers)

---

## ⚡ Solution 3: Atomic Classes

**Problem with synchronized:** It's relatively slow (uses locks)

**For simple operations**, there's a faster way:

```java
import java.util.concurrent.atomic.AtomicInteger;

class Counter {
    private AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();  // Thread-safe, no synchronized!
    }
    
    public int getCount() {
        return count.get();
    }
}
```

**How it works:**

- Uses CPU-level atomic operations
- Faster than synchronized (no locks)
- Thread-safe by default

---

### Common Atomic Classes

```java
AtomicInteger    // Thread-safe int
AtomicLong       // Thread-safe long  
AtomicBoolean    // Thread-safe boolean
AtomicReference  // Thread-safe object reference
```

**Common operations:**

```java
AtomicInteger count = new AtomicInteger(0);

count.incrementAndGet();  // count++, returns new value
count.getAndIncrement();  // count++, returns old value
count.addAndGet(5);       // count += 5
count.get();              // Read value
count.set(10);            // Set value
```

---

## 🤔 When to Use What?

### Use **AtomicInteger** when:

- Simple operations (increment, decrement, get, set)
- Single variable
- Need best performance

**Example:** Request counter, active users count

---

### Use **synchronized** when:

- Complex operations (multiple steps)
- Multiple variables need protection
- Business logic with conditions

**Example:** Bank account withdrawal (check balance + subtract)

```java
public synchronized void withdraw(int amount) {
    if (balance >= amount) {      // Multiple steps
        balance -= amount;         // Must happen together
    }
}
```

---

### Use **ExecutorService** when:

- Managing many tasks
- Don't want to create threads manually
- Need efficient resource usage

**Example:** Web server, background job processor

---

## 📝 Complete Example

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;

class Counter {
    private AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();
    }
    
    public int getCount() {
        return count.get();
    }
}

class CounterTask implements Runnable {
    private Counter counter;
    
    public CounterTask(Counter counter) {
        this.counter = counter;
    }
    
    public void run() {
        counter.increment();
    }
}

public class Main {
    public static void main(String[] args) throws InterruptedException {
        Counter counter = new Counter();
        
        // Create thread pool
        ExecutorService executor = Executors.newFixedThreadPool(5);
        
        // Submit 100 tasks
        for (int i = 0; i < 100; i++) {
            executor.submit(new CounterTask(counter));
        }
        
        // Shutdown and wait
        executor.shutdown();
        while (!executor.isTerminated()) {
            Thread.sleep(10);
        }
        
        System.out.println("Final count: " + counter.getCount());
        // Output: 100 (always correct!)
    }
}
```

---

## 🎯 Key Takeaways

✅ **Threads** = Multiple workers running simultaneously

✅ **Race conditions** = Data corruption when threads share data without protection

✅ **synchronized** = Lock ensuring only one thread at a time (simple, slower)

✅ **ExecutorService** = Thread pools for efficient task management

✅ **AtomicInteger** = Fast thread-safe operations for simple cases

---

## 🔗 Additional Resources

**YouTube:**

- "Java Multithreading" by Coding with John
- "Java Concurrency" by Defog Tech
- "Java Threads" by Programming with Mosh

**Reading:**

- Oracle Java Concurrency Tutorial (official docs)
- "Java Concurrency in Practice" book

---