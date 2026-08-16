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

> *More sections will be added after each completed session.*
>