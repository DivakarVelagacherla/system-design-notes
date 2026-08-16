## Vertical vs Horizontal Scaling

### What problem does it solve?

A single server can only handle so many requests. When traffic grows, you need a strategy to handle more load.

### The concept

- **Vertical Scaling** — make the one server bigger. More CPU, more RAM, more storage. Simple, but has a hard ceiling and requires downtime to upgrade.
- **Horizontal Scaling** — add more servers. No ceiling, just add more. Used by Amazon, Google, Netflix.

### Drawbacks of Vertical Scaling

1. **Hard ceiling** — there's only so big one machine can get
2. **Downtime** — upgrading requires taking the server offline
3. **No redundancy** — if the one server fails, everything fails (no failover)

### New problems Horizontal Scaling introduces

- **Routing** — something needs to decide which server handles each request (→ Load Balancer, Session 2)
- **State management** — servers don't share memory, so session/user state must live somewhere shared (→ Stateless vs Stateful, Session 3)

### When NOT to use Horizontal Scaling

- When load is small enough that vertical scaling is sufficient (2x–3x growth)
- At early stage — two servers are harder to manage than one; don't add complexity before you need it

### Plain-English explanation

Vertical scaling is making your one kitchen bigger. Horizontal scaling is opening more branches. Branches scale forever but require coordination — one kitchen is simpler until it can't keep up.

### Three-Question Ritual

| Question | Answer |
| --- | --- |
| What problem does it solve? | The hard limits of vertical scaling — no ceiling, no single point of failure |
| What breaks without it? | At scale, one server maxes out, goes down for upgrades, and has no backup |
| When NOT to use it? | When load is manageable with one server, or system is too early-stage to justify the complexity |

---

## Load Balancing

### What problem does it solve?

When you have multiple servers, something needs to decide which server handles each incoming request — and stop sending traffic to servers that are down.

### The concept

A **load balancer** sits in front of your servers and:

- Routes incoming requests to healthy servers
- Performs **health checks** — constantly monitoring which servers are alive
- Stops sending traffic to servers that fail health checks

**Real-world analogy:** A host at a busy restaurant — customers don't pick their own table. The host knows which tables are free, which waiters are available, and directs each customer accordingly.

### Routing Strategies

| Strategy | How it works | Problem |
| --- | --- | --- |
| **Round Robin** | Rotates through servers in order | Ignores actual load — heavy requests can pile up on one server |
| **Least Connections** | Routes to server with fewest active connections | Smarter, handles uneven request weights better |

### Load Balancer Redundancy

The load balancer itself is a single point of failure — so you need multiple load balancers:

- **Active-Passive** — one floating IP, primary handles traffic, backup on standby. Heartbeat signal between them — if primary goes silent, backup takes over the floating IP.
- **Active-Active** — DNS round robin across both load balancers. Both handle traffic simultaneously. If one fails, the other absorbs all traffic.

### Layer 4 vs Layer 7

|  | Layer 4 (NLB) | Layer 7 (ALB) |
| --- | --- | --- |
| **Operates on** | IP addresses + ports | HTTP — URLs, headers, cookies |
| **Routing intelligence** | None — just forwards packets | Content-based routing (e.g. `/api/videos` → powerful servers) |
| **Speed** | Faster, lower overhead | Slightly more overhead |
| **Use when** | Ultra-low latency needed (gaming, trading) | Most modern web applications |

### Three-Question Ritual

| Question | Answer |
| --- | --- |
| What problem does it solve? | Routes traffic across multiple servers, removes dead servers from rotation, eliminates single point of failure |
| What breaks without it? | DNS round robin has no health checks — dead servers keep getting traffic, requests fail |
| When NOT to use Layer 7? | When raw speed matters more than smart routing — use Layer 4 (NLB) instead |

---