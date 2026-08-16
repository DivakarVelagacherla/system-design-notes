---

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

## Stateless vs Stateful Servers

### What problem does it solve?

When multiple servers handle requests, user state (login, cart, preferences) can get trapped on one server — breaking horizontal scaling and redundancy.

### The concept

- **Stateful server** — holds user state locally in memory or on disk. Requests must return to the same server or the user loses their session.
- **Stateless server** — holds nothing locally. All state lives in an external shared store (e.g. Redis). Any server can handle any request.

### Why stateless is required for horizontal scaling

- With stateful servers, you can't freely add or remove servers — users are stuck to specific ones
- If a stateful server goes down, all users on it lose their sessions
- With stateless servers, any server can be spun up or shut down at any time — users never notice

### Naive solutions and why they fail

| Approach | Problem |
| --- | --- |
| **Sticky sessions** | Load balancer always routes user to same server. Breaks when that server goes down. Creates uneven load. |
| **Servers sharing state directly** | Complex, inefficient, doesn't scale |

### The real solution

Store session data in a **shared external store** — Redis is the standard choice.

- Any server reads/writes session data from Redis
- Servers become identical and interchangeable
- Scale up or down freely without affecting users

### Three-Question Ritual

| Question | Answer |
| --- | --- |
| What problem does it solve? | User state trapped on one server breaks horizontal scaling and redundancy |
| What breaks without it? | Servers go down and users lose sessions; can't freely scale horizontally |
| When NOT to use external session store? | Small scale where sticky sessions is acceptable; truly stateless APIs with no user sessions at all |

---

## DNS (Domain Name System)

### What problem does it solve?

Humans remember names, not IP addresses. DNS translates human-readable domain names (www.google.com) into IP addresses that computers use to route requests.

### The lookup chain

When you type `www.google.com`:

1. **DNS Resolver** (run by your ISP or a service like Cloudflare 1.1.1.1 / Google 8.8.8.8) checks its cache
    - Cache hit → returns IP immediately
    - Cache miss → starts the full lookup
2. Resolver asks **Root DNS servers** → returns address of the `.com` TLD server
3. Resolver asks **`.com` TLD server** → returns address of Google's authoritative server
4. Resolver asks **Google's Authoritative DNS server** → returns the actual IP address
5. Resolver **caches the result** with the TTL specified by Google
6. Resolver returns the IP to your browser
7. Browser connects to that IP

### TTL — Time To Live

Every DNS record has a TTL — how long resolvers should cache it before fetching a fresh copy.

| TTL | Tradeoff |
| --- | --- |
| **Long TTL** (24hrs+) | Faster lookups, less DNS traffic. Stale records persist longer after IP changes. |
| **Short TTL** (5 mins) | Records update quickly after changes. More DNS lookups, more load on servers. |

**Rule of thumb:**

- Normal operations → long TTL (1–24 hours)
- 1–2 days before a planned migration → shorten to 5 minutes so the world picks up the new IP quickly
- After migration is stable → set TTL back to long

### Who sets the TTL?

The **domain owner** sets TTL on their authoritative DNS server — not Cloudflare or the resolver. Resolvers just read and obey whatever TTL the authoritative server specifies.

### Three-Question Ritual

| Question | Answer |
| --- | --- |
| What problem does it solve? | Translates human-readable domain names to IP addresses so humans don't need to remember IPs |
| What breaks without caching/TTL? | Authoritative servers get overwhelmed with every lookup; stale cached records can't be corrected after IP changes |
| When to use short TTL? | 1–2 days before a planned server migration so DNS propagates the new IP quickly |

---