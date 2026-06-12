# Databases — Video Summary & Mini-Lesson

> **Video:** [Todo lo que necesitas saber de Bases de Datos en 25 minutos](https://www.youtube.com/watch?v=xz1fJ7M2g-o) — BettaTech
> *("Everything you need to know about Databases in 25 minutes")*

---

## 📋 Brief Summary

Starting from the "app that outgrows its files" scenario, the video explains **what databases really are, why different typologies exist, and how to choose the right one for each problem**. It covers relational vs. NoSQL families (document, key-value, graph), scaling (vertical vs. horizontal, sharding), the CAP theorem and the **ACID vs. BASE** trade-off, and the day-to-day design techniques: **indexes** and **normalization** (the three normal forms, worked through a full example). The core message: *no database is the best at everything* — design and choosing well matter more than mastering any specific query language.

---

## 🎓 Mini-Lesson: The Concepts

### 1. What a database is

- A database is just **a program** designed to **store and query information at scale**: you save data, and you ask for data matching certain criteria.
- Different applications need different things — strong relationships, millisecond query speed, or schema flexibility — so **different typologies exist**. Picking the wrong type is a critical decision that complicates development "from now until the future."

### 2. Relational databases

- Created by **Edgar Codd** (1970s, first paper describing the relational model — it spawned the multi-billion-dollar SQL industry).
- Information is organized in **tables**: columns = attributes of an entity (e.g., user's name, email), rows = the entities themselves.
- Their power is **modeling relationships**: a `users` table linked to an `addresses` table. They're called *relational* because they model relations — **not because they use SQL**.

### 3. NoSQL — "Not *Only* SQL"

Important nuance: NoSQL means *not only SQL*, and using SQL doesn't make a database relational. The NoSQL family includes:

| Type | Idea | Examples | Shines when |
|---|---|---|---|
| **Document** | Data stored as JSON-like documents; no closed schema — same collection can hold users with/without certain fields | MongoDB | Startups / evolving business models: "insert and forget", no schema migrations |
| **Key-value** | Like a hash map: know the key → get the value extremely fast | Redis, DynamoDB | Speed-critical workloads — **but only if you model the keys well**; bad key design = very limited query capability |
| **Graph** | Native graph algorithms (path finding, tree traversal) over your data | Neo4j | Social networks, friendship graphs, "shortest path between two users" |

### 4. There is no "best" database — mix them

- "SQL for fixed structure, NoSQL for flexibility" is an **oversimplification**. One product = many different problems → it's fine to use **several databases in one product**.
- Real examples:
  - **Netflix:** Cassandra (NoSQL) for user activity & recommendations + MySQL where rigid relations are needed.
  - **Uber:** Redis (key-value) for caching & real-time data + PostgreSQL for relational models.

### 5. Scaling: vertical vs. horizontal

What happens when 1,000 rows become millions?

- **Vertical scaling:** give the machine more CPU/RAM. Often necessary for relational DBs, whose horizontal scaling is limited.
- **Horizontal scaling:** instead of growing one machine, **spawn an army of servers** with the data spread across them. Easier in less-structured DBs (document, key-value) thanks to **sharding**.
- **Sharding** = rules deciding which server stores which data. Simple example: users from Spain on one server, LatAm on another, US on a third. The goal: a **well-balanced spread** — no server overloaded while another sits idle.

### 6. CAP theorem → ACID vs. BASE

- CAP recap ("good, pretty, cheap — pick two"): **C**onsistency (every read gets the latest data), **A**vailability (the DB always answers), **P**artition tolerance (data can live split across servers — the sharding story). You get **two of three**.
- **Relational DBs choose Consistency** (relations must always hold — no users without their address), following **ACID** principles (covered in the backend video).
- **Distribution-oriented NoSQL DBs choose Availability + Partition tolerance**, following **BASE** principles instead:
  - **BA — Basically Available:** the DB always answers, even if the data is stale (the old Instagram photo your friend changed 10 minutes ago).
  - **S — Soft state:** data can change/propagate over time without you explicitly asking.
  - **E — Eventual consistency:** you *will* see the correct data — just not immediately. (vs. ACID's strong consistency.)

### 7. Indexes (don't let your DB become a turtle)

- Without an index, finding user `#1` among 1 million rows means **scanning everything** — queries get slower as data grows.
- **Book analogy:** you don't read page by page to find a chapter — you check the index and jump straight there.
- Internally implemented as **trees (e.g., B-trees)** stored *alongside* the table: index users by ID → the DB walks the ordered tree straight to the data.
- **The cost:** writes get slower (you write the row *and* update the index). Rule of thumb: indexes for **read-heavy** tables; be cautious on **write-heavy** ones.

### 8. Normalization — the three normal forms (Codd again)

Goal: organize data so **consistency is easy to maintain** (no duplicated info that can drift apart). Worked example — a denormalized orders table: `order_id | customer | email | products(list) | total`. Problems: a *list* in one column, and Alice's name/email repeated in every order (change her email → update every row → inconsistency risk).

The forms are applied **in order**, each building on the previous:

1. **First Normal Form (1NF) — eliminate non-atomic data:** no lists in columns. One row per product, sharing the `order_id` → the key becomes the *composite* (order, product).
2. **Second Normal Form (2NF) — eliminate partial dependencies:** no column may depend on only *part* of the composite key (customer depends on the order, not the product). Fix: split into `orders`, `products`, and a linking `order_lines` table.
3. **Third Normal Form (3NF) — eliminate transitive dependencies:** customer email depends on the order only *through* the customer. Fix: extract a `customers` table; orders just reference the customer ID.

Result: a fully normalized database — no repeated information, much easier consistency.

**…and when to do the opposite:** outside the relational world, **denormalization** can be the right call — e.g., in **DynamoDB** it's sometimes preferable to keep duplicated data to make queries much faster.

> Book recommendation from the video: a *Fundamentals of Database…* textbook used in many university courses — likely *Fundamentals of Database Systems* (Elmasri & Navathe), though the auto-captions render the title as "Fundamentals of Database Design."

### 9. The mindset shift

- The video deliberately skips SQL syntax and JOINs: **design matters more than the technology.** There are endless SQL courses; content on *choosing* the right DB and designing normal forms is rare.
- High-level concepts stay constant regardless of the tool underneath — query syntax is in each DB's documentation.
- **Design your data around the queries you'll actually run.** The simpler and more use-case-fitted your model, the easier it scales. *"Simplicity is a competitive advantage."*

---

## 🧭 The Big Picture (mental map)

```
                       What does your problem need?
                                  │
     ┌──────────────┬─────────────┼─────────────┬──────────────┐
     ▼              ▼             ▼             ▼              ▼
  Relations      Flexibility   Raw speed     Graph queries   Mixed product?
  (SQL/relational) (Document)  (Key-value)   (Graph)         → use SEVERAL
  PostgreSQL,    MongoDB       Redis,        Neo4j           (Netflix: Cassandra
  MySQL                        DynamoDB                        + MySQL)
     │                │            │
   ACID            BASE (BA / Soft state / Eventual consistency)
   (Consistency)   (Availability + Partition tolerance)     ← CAP: pick 2 of 3

  Scaling:  vertical (bigger machine — relational)
            horizontal (more machines + SHARDING — easier in NoSQL)

  Day-to-day design:
   • INDEXES  → B-trees; fast reads, slower writes (read-heavy tables)
   • NORMALIZATION (relational): 1NF atomic → 2NF no partial deps → 3NF no transitive deps
   • DENORMALIZATION (NoSQL/Dynamo): duplicate on purpose for fast queries
   • Always design around your ACCESS PATTERNS — simplicity wins
```

---

## 💡 Key Takeaways

1. **No database is best at everything** — choose per problem, and mixing DBs in one product is normal (Netflix, Uber do it).
2. **NoSQL ≠ "no SQL"** — it's *not only* SQL; relational means *modeling relations*, not *using SQL*.
3. **CAP forces a choice:** relational → consistency (ACID); distributed NoSQL → availability + partitioning (BASE, eventual consistency).
4. **Indexes are a read/write trade-off** — a B-tree on the side that speeds reads and slows writes.
5. **Normalize relational data (1NF → 2NF → 3NF)** for consistency; **denormalize in NoSQL** for speed — and always design around your real queries.

---

*Related summaries: [backend-development-concepts-video-summary.md](backend-development-concepts-video-summary.md) (ACID, CAP, indexes in context) · [system-design-thinking-video-summary.md](system-design-thinking-video-summary.md) (access patterns, read replicas).*
