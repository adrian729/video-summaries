# Backend Development Concepts — Video Summary & Mini-Lesson

> **Video:** [Todo lo que necesitas saber del Desarrollo Backend en 29 minutos](https://www.youtube.com/watch?v=l3HJsXA-Fa4)
> *("Everything you need to know about Backend Development in 29 minutes")*

---

## 📋 Brief Summary

The presenter (a backend developer with 10+ years of experience) builds a **mental map of the core backend concepts** that are usually taken for granted and rarely explained well: APIs, load balancers, reverse proxies, authentication vs. authorization, database indexes, transactions & ACID, the CAP theorem, observability, and — the one that "misapplied can cause disasters like charging a user multiple times" — **idempotency**.

The thesis: backend development is *not* just "building endpoints and returning JSON." Understanding these abstract concepts and their real-world implications is what separates someone who writes endpoints from a real backend engineer.

---

## 🎓 Mini-Lesson: The Concepts

### 1. What an API *really* is

- An **API (Application Programming Interface)** is best understood as a **recipe / map / tutorial** describing *how to interact* with a system. It's a contract — not necessarily an HTTP server (a library's functions are an API too).
- In backend work, most APIs are **HTTP endpoints** (`GET /users`, `GET /orders`, `POST` to create entities), but alternatives exist: **WebSockets** (bidirectional — the server can push messages to the client), **GraphQL**, **tRPC** (which abstracts the protocol away from the programmer).
- **You have total freedom in API design — and that's dangerous.** You *could* build everything with just `GET`, but browsers and intermediaries assume HTTP semantics:
  - **Caching gotcha:** `GET` requests are often cached by default. If a `GET` has side effects (e.g., creates an entity), the first call works, then subsequent calls never reach the server — the browser serves the cached response.
- Good API design also means a good **developer experience**: pagination (page-based or cursor-based), consistent filtering parameters, predictable ordering.

> **Key takeaway:** API design is a skill. Knowing the conventions (HTTP verbs, cacheability) prevents subtle, painful bugs.

### 2. Load Balancer vs. Reverse Proxy

Both distribute traffic, but for **different reasons**. The video uses a restaurant analogy:

| | Load Balancer | Reverse Proxy |
|---|---|---|
| **Analogy** | A burger restaurant with 3 identical kitchens — the host at the door sends each customer to the **least busy** kitchen | A restaurant with burger kitchens *and* pizza kitchens — the host routes you by **what you ordered** |
| **Routes by** | Server load / availability | Feature / path / microservice |
| **Example** | Round-robin (1→2→3→1…) or smarter algorithms based on CPU/pending load | Rule: `/users` → users microservice, `/orders` → orders microservice |

- In practice they're often **combined in the same tool** (e.g., **nginx** can do both).
- Real architectures nest them: a reverse proxy routing to microservice clusters, each cluster with its own internal load balancer.

### 3. Authentication vs. Authorization

Two doormen at the restaurant:

- **Authentication** answers: **"Who are you?"** — the first doorman asking your name (the classic *login*). No identity → no entry.
- **Authorization** answers: **"Are you allowed to do this?"** — a second doorman who already knows your name and says: "Martí can't order burgers, but pizza is fine."
- Authorization **depends on** authentication, but they are different layers.
- Practical rules:
  - Login/registration endpoints are **public** (they're where you authenticate).
  - Nearly everything else should require authentication **and** apply authorization (e.g., on a blog platform, other users must not be able to delete *your* posts).

### 4. The Data Layer: Indexes

"This is where many projects start to fail — the world of data generates the most complexity."

- **Don't scan everything to find one thing.** Reading all posts in a DB to find one user's posts wastes resources.
- A **database index** tells the DB *how to read* your data. Index posts by `user_id` → fetching one user's posts is a fast lookup instead of full-table-scan + filter.
- **How to decide what to index:** know your **access patterns** (the most common queries). Frequent "posts by user" query → index by user.
- **Nothing is free in computing:** indexes make **writes slower**. Always weigh the **read vs. write trade-off** before adding one.

### 5. Transactions & ACID

A **transaction** = a set of operations treated as **one unit of work**: it completes entirely or not at all (e.g., create a post *and* set it to public — never one without the other).

**ACID** properties:

- **A — Atomicity:** all-or-nothing. Prevents inconsistent intermediate states (a post without a status).
- **C — Consistency:** data never ends up corrupt or breaking your design — critical in relational DBs full of links between tables.
- **I — Isolation:** concurrent transactions don't trample each other. The video's bank example:
  1. Account has €10,000. Two deposits of €1,000 arrive simultaneously.
  2. Both transactions **read** €10,000, both write back €11,000.
  3. Result: €2,000 was deposited but the balance shows €11,000 — **€1,000 lost**.
  - This is a **dirty read** (reading data not yet committed by another transaction). Related anomalies: **non-repeatable reads** and **phantom reads**. DBs prevent these with locks — which is why big transactions can hurt read performance.
- **D — Durability:** once a transaction commits, it's persisted and survives — queryable in the future.

Relational databases (PostgreSQL, MySQL) are expected to honor ACID.

### 6. The CAP Theorem

For **distributed systems** (which you get once load balancers + replicated services + distributed data combine), three properties exist — **you can only have two**:

- **C — Consistency:** every user sees the up-to-date data.
- **A — Availability:** every client gets a response even if nodes are down.
- **P — Partition tolerance:** the system keeps working despite communication failures between nodes (queues, retries help here).

The video's analogy: *"good, pretty, cheap — pick two."* If someone claims their distributed system has all three, they're lying.

**Real-world example:** change your Instagram profile photo, then check it on a friend's phone — the old photo may still show. Instagram chose **A + P**, sacrificing strict consistency. When designing distributed systems, you must consciously decide which property to sacrifice.

### 7. Observability

- You don't want users phoning you at night to tell you something is broken — you want to **detect (or predict) failures before they happen**: partitions, slow nodes, dropped requests.
- The toolbox: **logs**, **metrics**, and **traces** (following one request from the API call all the way down to the DB query, across every service it touched).
- Named tools: **CloudWatch**, **AWS X-Ray** (tracing), **canonical logs** (structured, low-noise logging).
- Essential skill for running backends **at scale**.

### 8. Idempotency ⚠️ (the "disaster" concept)

- **Definition:** an operation performed once or many times should produce **the same result**.
- **The horror story:** a "Pay" button that doesn't respond → user clicks 10 times → each click sends a payment request → a €10 product charges **€100**. That platform was built without idempotency.
- **The fix — idempotency keys:** the client generates a **unique ID per action** (not per click). Every click sends the same ID; the server detects the duplicate ("I'm already processing this ID") and executes the action **only once**.
- Other symptoms of missing idempotency: hitting "publish" repeatedly and seeing duplicate posts. Duplicates on a server usually mean a non-idempotent action that should have been idempotent.
- **HTTP connection:** `GET` is defined as idempotent — which is *why* browsers cache it (full circle to concept #1). `POST` is **not** idempotent by default — that's why duplicate creations happen unless you add an idempotency key yourself.

---

## 🧭 The Big Picture (mental map)

```
User ──► API (the contract: endpoints, verbs, pagination, caching)
          │
          ▼
     Load Balancer / Reverse Proxy  (distribute by load / by feature)
          │
          ▼
     AuthN ("who are you?") ──► AuthZ ("can you do this?")
          │
          ▼
     Services + Data layer
       • Indexes (match your access patterns; read/write trade-off)
       • Transactions + ACID (atomic, consistent, isolated, durable)
       • CAP theorem (distributed: pick 2 of C/A/P)
          │
          ▼
     Observability (logs, metrics, traces — see problems before users do)

  Cross-cutting: IDEMPOTENCY — same operation N times = same result
  (idempotency keys prevent double charges & duplicates)
```

---

*Note: the video also mentions a free live AWS workshop (EC2, S3, DynamoDB vs RDS) promoted in its description, and a follow-up video on backend architectures.*
