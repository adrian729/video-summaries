# Scaling Evolution: From Client-Server to Distributed Systems — Video Summary & Mini-Lesson

> **Video:** [Todo lo que necesitas saber sobre diseño de sistemas en 24 minutos](https://www.youtube.com/watch?v=2nEiIG-xca4) — BettaTech
> *("Everything you need to know about system design in 24 minutes")*

---

## 📋 Brief Summary

The video walks through the **progressive evolution of a system's infrastructure** as its needs grow — starting from the simplest possible architecture (client → server → database) and adding one piece at a time: load balancers, stateless servers, caches, read replicas/sharding, CDNs, async processing with queues, and finally observability. The thread tying it all together is a single rule: **every piece you add must solve a real problem.** Don't reach for Kafka, SQS, or read replicas because they're fashionable — add them when growth forces the need. Before any of it, you must answer the prior question: *what system am I actually designing* — is it read-heavy (Netflix-style) or write-heavy (data ingestion)?

> Core idea: *"Each piece we add to make a system more complex has to play a role, has to solve a problem. Understand the **why** as you reach for each concept — don't build a super-complex infrastructure just because."*

---

## 🎓 Mini-Lesson: The Concepts

### 0. The prerequisite question: what are you designing?

Before any architecture diagram, ask **what problem you're solving**. It's not the same to design:
- A **read-heavy** product (e.g. Netflix — tons of reads), vs.
- A **write-heavy** system (e.g. a data-ingestion pipeline — tons of writes).

The access pattern dictates which tools fit. This step is almost always skipped when people explain system design — don't skip it.

### 1. The starting point: 3-tier client-server-database

For **99.999%** of products, the simplest architecture is the right place to start. Most products don't have millions of users on day one.

| Tier | What it is | Examples |
|------|-----------|----------|
| **Client** | Runs locally on the user's machine; what the user interacts with | Browser/frontend, mobile app, CLI |
| **Server** | Runs in the cloud (or a rented server); collects and executes client requests | Holds **domain logic**, auth, DB communication, data transformation, security layers |
| **Database** | Where the data is persisted | Usually a cloud DB server |

Key points:
- The server is where your **domain logic** and **security layers** live (e.g. not leaking data to users who shouldn't see it).
- This architecture **doesn't have to be in the cloud** — desktop apps follow the same pattern with all three tiers on one machine (a presentation layer talking to a local process acting as server, storing in SQLite/local MySQL). Anyone who used **PHP + XAMPP** knows this.
- Real example: the author's *Comit Academy* runs 3-tier — web client, **AWS Lambda** servers, **PostgreSQL** database.

### 2. Communication between tiers

The tiers must talk to each other. How they communicate depends on the architecture:
- **HTTP** — most common for cloud/internet communication.
- **WebSockets** — for bidirectional/real-time.
- **FTP** — for uploading files to a server.
- **OS sockets** — when the three tiers are separate processes on the *same* machine.

### 3. Hitting the limit & scaling

The 3-tier architecture has a ceiling — though **not a low one** (companies handle millions of requests this way). The problem: when all domain logic lives in one server, **resource-hungry operations cannibalize resources** from lighter ones. Servers get busy doing one thing and can't do others → **degradation**: slow responses, errors. *This* is when system design starts to matter.

**Two orthogonal axes of scaling** (you can do both at once):

| Scaling | What you do |
|---------|-------------|
| **Vertical** | Make the machine bigger — more CPU, more RAM |
| **Horizontal** | Add more machines to handle more traffic |

### 4. Load balancers (the price of horizontal scaling)

When you add machines, you need something to **distribute traffic** among them — a **load balancer**: a single entry point that decides which replica/server gets each request.

| Algorithm | How it works |
|-----------|--------------|
| **Round Robin** | Rotate requests across servers (1→S1, 2→S2, 3→S3, 4→S1…). Doesn't consider request cost or duration. |
| **Least active connections** | Send the next request to whichever server is currently *freest*. Smarter load distribution. |
| *(others)* | By CPU usage, custom preference/logic, etc. |

### 5. Stateless servers (the golden rule of horizontal scaling)

Horizontal scaling breaks if servers hold **state**. Example: server S1 saves an uploaded file to its local disk; a later read request gets routed to S2, which has no idea the file exists.

> **The rule to remember:** servers must be **stateless**. Delegate state to something outside the server so any server can serve any request.

- Store state in a **database**, or files in a service like **S3**.
- **Sticky sessions** (pin a client IP to one server) exist as a workaround — but it's far simpler to design servers stateless from the start.

### 6. Scaling the database (the hardest bottleneck)

Now servers scale dynamically — but the **database** is the remaining, hardest-to-manage bottleneck. Strategies:

**Caching (the easy first win):**
- A layer that **saves calls to the database**. Example: a user's name rarely changes, so don't hit the DB for it every time — cache it. As requests grow, you cache more, and DB/CPU pressure grows far slower than it otherwise would.
- **Cache invalidation** — how you clear stale cache. Strategies:
  - **Write-through invalidation** — invalidate caches whenever you write to the DB (e.g. user changes name → all caches of that name are dropped and re-fetched).
  - **Time-based (TTL)** — drop any cached datum older than e.g. 24h and refresh.
  - Choosing the right strategy matters to keep data correct.

**Other DB scaling methods** (covered in a separate video):
- **Read replicas** — replicate the DB for read scaling.
- **Sharding** — distribute data across multiple servers efficiently.

> With caching + replicas, you can already serve **90–99%** of enterprise application needs.

### 7. CDNs (geographic latency)

Even with all the above, users far from your servers get a worse experience. If your whole infrastructure is in **Spain**, a user in **Latin America** gets data slowly due to physical distance.

A **CDN (Content Delivery Network)** is a global network of servers — conceptually like a geographic cache. Put a server near the Latin-American user holding the same data, and serve from there instead of round-tripping to Spain.
- This is how **Netflix** distributes movies — content infrastructure in each country, not always hitting Ireland/US servers.
- Providers: **CloudFront** (AWS), **Cloudflare**.

### 8. Sync vs. async, and queues

All these intermediate layers (CDN, LB, replicas, caches) **decouple** user requests from data processing. That introduces **asynchrony**: users don't always see the latest DB value (caches), and requests aren't always processed the instant they arrive.

| Process | Behavior | User experience |
|---------|----------|-----------------|
| **Synchronous** | Make a request, **block** and wait for the response | Click button → spinner → result appears |
| **Asynchronous** | Hand off the work, return immediately | Click → "You'll get the report by email in a few minutes" → email arrives later |

**Message queues** make async work:
- A queue is essentially a store of incoming requests (e.g. "generate a PDF report").
- **Synchronous + all servers busy** → user gets an error ("servers too loaded, can't handle your request").
- **With a queue** → save the request, tell the user "we're processing it", and as **worker servers** free up, they consume from the queue and execute.
- Trade-off: it complicates the infrastructure — you must decide how to return results (email, save to DB + notification → dashboard, etc.).
- Tool: **AWS SQS**.

### 9. Observability (don't drive blind)

More pieces = more things that can fail **independently**. In distributed/internet systems, **assume failures will happen** — connectivity always fails eventually. A distributed system without observability is *driving a car blindfolded*.

**What to measure:**
- **Availability** — how ready a server is to accept new requests (vs. always at its limit, always queuing).
- **Latency** — how many ms/s to respond to requests and to communicate server-to-server.

Measuring this (e.g. with **OpenTelemetry**) lets you fix failures faster and react with resilience techniques:
- **Retries** — retry a flaky server before erroring to the user.
- **Circuit breakers** — stop failures from cascading across the whole system.

**The three pillars of observability:**

| Pillar | What it is | Used for |
|--------|-----------|----------|
| **Logs** | Instrumented messages in your app | *Investigating* afterward — must be **queryable**. Don't log "reached this function"; log user ID, server, what's being queried. Use **structured logs** / canonical logging. |
| **Metrics** | Numbers/signals you want to measure (e.g. latency over 3 days) | At-a-glance *"is something wrong right now?"* — spotting spikes/drops |
| **Traces** | A single request followed across all servers/queues via a **unique ID** propagated through every hop | Seeing one complete workflow as a single story across a distributed system |

### 10. The meta-lesson

> Every piece added must **solve a problem**. Understand *why* you reach for SQS, Kafka, or a read replica — add complexity as needs demand it, not preemptively.

**Recommended books:**
- *Building Microservices* — Sam Newman
- *Designing Data-Intensive Applications*

---

## 🧭 The Big Picture (mental map)

```
                    EVOLUTION AS NEEDS GROW
                    (add a piece only to solve a real problem)

  STAGE 1: Simple 3-tier  ──────────────────────────────────────────
     [Client] ──HTTP──> [Server] ──> [Database]
     (browser/app/CLI)   (domain logic,        (persisted state)
                          auth, security)

       │  problem: one server cannibalizes its own resources → degradation
       ▼
  STAGE 2: Horizontal scaling  ─────────────────────────────────────
                       ┌──> [Server S1] ┐
     [Client] ──> [Load Balancer] ──> [Server S2] ┼──> [Database]
                       └──> [Server S3] ┘
                       (round-robin /        ⚠ servers must be STATELESS
                        least-conn)             → state in DB / S3

       │  problem: the database is now the bottleneck
       ▼
  STAGE 3: Scale the data layer  ───────────────────────────────────
     [Servers] ──> [CACHE] ──> [Database] ──> [Read replicas / Sharding]
                   (TTL or write-through invalidation)

       │  problem: far-away users are slow (geographic latency)
       ▼
  STAGE 4: Go global  ──────────────────────────────────────────────
     [User LATAM] ──> [CDN node nearby]   (Netflix model;
     [User ES]    ──> [CDN node nearby]    CloudFront / Cloudflare)

       │  problem: decoupling → asynchrony; heavy/blocking work
       ▼
  STAGE 5: Async processing  ────────────────────────────────────────
     [Server] ──> [QUEUE (SQS)] ──> [Worker servers] ──> result
                                     (consume as they free up)     │
                                                                   ▼
                                            (email / DB + dashboard notify)

  ─────────────────── CROSS-CUTTING THE WHOLE TIME ──────────────────
  OBSERVABILITY (OpenTelemetry):  measure AVAILABILITY + LATENCY
        Logs (queryable, structured)  ·  Metrics (at-a-glance)  ·  Traces (unique ID across hops)
        resilience: retries · circuit breakers
```

---

## 💡 Key Takeaways

1. **Start simple.** A 3-tier client-server-database architecture serves 99%+ of products — and isn't even cloud-specific (desktop apps do it too). Don't over-engineer day one.
2. **Servers must be stateless.** It's *the* rule that makes horizontal scaling clean — delegate state to a database or S3, not local disk.
3. **The database is the real bottleneck.** Caches (with a deliberate invalidation strategy), read replicas, and sharding are how you push it further.
4. **Decoupling buys scale but introduces asynchrony.** Know exactly when a process is synchronous vs. async, and use queues + workers to stay stable under load.
5. **Observability is non-negotiable in distributed systems.** Logs (queryable), metrics (at-a-glance), and traces (unique ID across hops) — you can't fix what you can't see.
6. **The golden rule:** every added piece must solve an actual problem. Understand the *why* before adopting Kafka, SQS, or replicas.

---
*Related summaries: [System Design Thinking](system-design-thinking-video-summary.md) — the access-patterns lens (read-heavy vs write-heavy) that this video's "what are you designing?" step opens with · [Databases Concepts](databases-concepts-video-summary.md) — the replication/sharding deep-dive referenced here · [Backend Architecture Patterns](backend-architecture-patterns-video-summary.md) · [Real-World Systems](real-world-systems-video-summary.md) — CDNs & infrastructure at Netflix/Twitter scale · [DevOps Concepts](devops-concepts-video-summary.md) — observability in practice.*
