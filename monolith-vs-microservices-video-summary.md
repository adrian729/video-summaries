# Monolith vs. Microservices — Video Summary & Mini-Lesson

> **Videos:** [Pues un Monolito no es tan malo... ¿O Sí?](https://www.youtube.com/watch?v=aVL3AObcb9M) and [¿Amazon deja los MICROSERVICIOS?](https://www.youtube.com/watch?v=RPEM2Kx5-JE) — BettaTech
> *("Actually, a Monolith Isn't That Bad... Or Is It?" and "Is Amazon Leaving MICROSERVICES?")*

---

## 📋 Brief Summary

Two complementary videos. The first — a *book-club* episode on Sam Newman's **Monolith to Microservices** — argues that the goal is **not to marry the perfect architecture but to learn to evolve an architecture without breaking it along the way**, and walks through three of the book's patterns: the **Strangler Fig** for incremental migration, **decomposing the database** (shared DB, views, dedicated data services), and the **"growing pains"** of distributed systems (breaking changes, reporting, local dev experience). The second video is the real-world evidence: the famous **Amazon Prime Video** article claiming a move "from microservices to monolith" with a **90% infrastructure cost cut**. The presenter dismantles the sensationalist headline — Prime Video did *not* abandon microservices; **one specific service** (audio/video quality monitoring) swapped a Step Functions + Lambda + S3 design for a single ECS process, because **the cloud's cost model (pay per state transition, pay per S3 transfer) punished that exact workload**. Both videos land on the same message: *migrate (in either direction) for the right reasons, not for the hype.*

> Recurring theme: *"There is no golden hammer that can both unscrew nuts and do everything else. Every technology has its specific use — it's our responsibility as software engineers to pick the right tool."*

---

## 🎓 Mini-Lesson: The Concepts

### Part I — The framework (Sam Newman's *Monolith to Microservices*)

#### 1. The real goal: evolvable architecture, not "the perfect one"

- Every software team eventually hits the same debate: *stay on the monolith or migrate everything to microservices?*
- The book's core idea: don't marry an architecture — **learn how to evolve one without breaking it on the way**, following patterns that make sure "you don't destroy anything along the path."

#### 2. The Strangler Fig pattern (incremental migration)

- Named by **Martin Fowler** after a real fig: it drops seeds in a tree's canopy, sends roots down to the ground, grows around the host tree until it envelops it completely — and years later you realize **the original tree inside has died**.
- In software: the host tree = the part of the system you want to eliminate (e.g., the monolith); the fig = the **new system growing around it**. Only when everything is migrated, **tested, and proven working** do you switch the traffic — at that point you've *strangled* your monolith.
- The value: migration happens **incrementally**, without constantly touching software that's running in production.
- **The presenter's own story:** at an early job they had a simple Node.js API monolith on a VPS, professionalized the product by moving to AWS — first to a **dedicated instance**, then gradually moved functionality to **serverless Lambda functions** — using the Strangler Fig for years *without knowing the pattern had a name*. Finding it named in the book confirmed they'd taken the right path.

#### 3. Decomposing the database (the hard part)

Migrating code is "the easy part" — the real problem appears when **the data still belongs to the old service**, still monolithic. You split code into separate repos, separate deploys, even separate languages, and feel like "I'm doing microservices, I have a super-scalable architecture"… then realize **every microservice attacks the same database with the same permissions and access** — a risk. The question is no longer *where is my code physically* but **who owns each piece of data and each user flow**.

- Book quote (chapter 4, *Decomposing the Database*): *"If the behavior that changes state (the code) is now distributed around the system, ensuring that the state (the data) is kept consistent across the different machines is a dangerous problem."*
- **Myth busted:** "every microservice must have its own database" — **not necessary**; it depends on the pattern you choose (no need to spin up "a PostgreSQL here, a PostgreSQL there, a PostgreSQL everywhere"):

| Pattern | Idea | Why it helps |
|---|---|---|
| **Database views** | A *virtual table* exposing transformed/limited data per service over one global DB | Each microservice sees **only what you want it to see**; change the monolith's internal tables → just update the view, consumers never know. Book example: a **pricing system for an e-commerce / online store** |
| **Dedicated DB service** | Microservices don't hit the database directly — they go through an intermediate service | Single owner of data access |
| **Data synchronization patterns** | Keep data in sync during/after the split | Plus "a ton of extra patterns" usable from day one |

- **The most important part is semantic, not technical:** finding the real *break points* — reasoning about the system so a separation **makes sense**. Migrate only for adequate reasons: parts that need **independent scaling** or a real **operational improvement**. Otherwise you convert a *local* problem you could reason about inside a monolith into a **distributed** problem with **network failures and availability issues that — believe me — will appear**.

#### 4. Growing pains (chapter 5: the cost of going distributed)

> *"As you adopt a microservices architecture you'll experience lots of problems along the way — more services, more pain."*

- **Breaking changes:** in a monolith your editor tells you *"hey, you changed this function and it's used in these seven places"* — you'll hardly get that in a distributed architecture. Fix: add **typing/structure to the contracts** between microservices in a central repository of expected definitions — break a contract anywhere and an error fires immediately. When a contract genuinely *must* change, the book recommends **staged rollouts** giving teams enough information and time to update their consumers. Microservices force you to care about **communication with the rest of the dev team**, not just the technical implementation.
- **Reporting:** generating data analysis across distributed systems gets complicated (the book offers techniques for reporting and logs in distributed architectures).
- **Local developer experience** (a problem rarely talked about): it's much easier to spin up a monolith on your laptop than to spin up **30 Lambdas** just to develop and test.
- A point the presenter appreciates: Sam Newman **never sells microservices as a magic solution** — he explicitly dislikes his own earlier book (*Building Microservices*) being used that way. Adopt them *if they make technical sense for your organization*, **accepting a set of problems you wouldn't have with a monolith**.

### Part II — The worked case study: Amazon Prime Video (video 2)

#### 5. What actually happened (vs. the headline)

- Prime Video published an article: one service changed implementation "from distributed microservices to a monolith application," cutting infrastructure cost by **~90%**. Twitter erupted: *"I told you so — microservices were a money grab, a fad, complexity for nothing."*
- **Spoiler from the video: Prime Video did not stop using microservices; Prime Video is not a monolith.** The change affected **one specific service**: the **audio/video monitoring service**, which analyzes the quality of videos on the platform (missing frames, audio cuts, file defects). Prime Video is made of many pieces — frontend-serving APIs, the recommendation system, video processing — and this is just one of them. What really changed: **one service moved from serverless to ECS**.

#### 6. The original design — and why it was built that way

- Built on **AWS Step Functions**: a service for building **distributed state machines** — an event occurs, a function watches from above ("step 1 finished → launch step 2; step 1 returned type X → move to another step"), forming a graph of states that orchestrates **Lambda** (serverless) functions quickly and visually.
- The alternative — Lambdas emitting events that call other Lambdas — creates "a very confusing map in your head": hard to know if an execution went A→B→C or skipped B. So Step Functions was the **easiest, fastest thing to implement** — counterintuitively, *easier than building it as a monolith* ("it's very, very easy to make Lambdas that in theory scale forever").
- Context the video stresses: the team **didn't expect the service to be used so much** — it was designed for a specific load, and grew gradually as more things needed monitoring. As it grew, the design turned out **not as cheap or affordable as they expected**. The presenter explicitly does *not* criticize the team: it worked fine as the initial proof of concept.

#### 7. Where the money went: the cloud cost model

| Cost source | The mechanism | Why it exploded |
|---|---|---|
| **Step Functions** | Charges **per state transition** — **$0.025 per 1,000 transitions** | Fine at the initial load (say, a million transitions/day) — but if load keeps growing and you don't control how many *extra* transitions that growth generates, "a bill you didn't expect" arrives |
| **S3 transfers** | S3 is "a huge distributed disk" that **charges per transfer** (putting in and pulling out files) | The **media converter** turned video into frames, pushed frames into S3 *constantly*, only for Lambdas to pull them right back out — pure transfer cost for a high-volume, big-file workload |

- **First cloud lesson** (AWS, Google Cloud, whatever): **know the cost policy** — per use? per call? per hour of being on? If something will be *hammered* with calls and you pay per call, a simple always-on virtual machine billed per hour may be far cheaper.

#### 8. The new design: one process on ECS

- Everything moved into **one ECS task** (Amazon **Elastic Container Service**): Docker container(s) deployed on a virtual machine — "a normal, ordinary server" — with **min/max auto-scaling rules** (CPU/RAM thresholds spawn another instance automatically). "Similar to Kubernetes but much simpler" — and the presenter notes that if they'd called it "a Kubernetes pod," nobody would have screamed about the death of microservices.
- New flow: the **media converter** generates frames, stores them in the **local process/filesystem**, and the **defect detector reads them straight from memory**. Only the **final analysis results** go to S3 — **one S3 upload per video file** instead of an upload *and* download per frame.
- Result: Lambdas (mostly) deleted → no state transitions → no Step Functions cost. They **destroyed the part that generated the most cost and the least value**.
- **Crucially, serverless wasn't abandoned:** a **Lambda still receives requests and distributes the tasks to the replicas** — when one machine is busy processing a video, another spins up and processing continues in parallel. The service still scales up and down.
- Why the new shape fits: this workload is **long processes over very large files** (analyzing entire movies) — "why would you want to analyze that in a distributed way? Analyze it all on one machine and that's it." Serverless isn't useless; **it just wasn't the best solution for this use case**.

#### 9. The analysis checklist (before choosing serverless vs. a process)

Questions the video says you must answer for *your* case:

- How many processes will I have? Do I need concurrency?
- Am I processing **large files**? How **long** will processing take? How much **CPU** will it consume?
- What is the **cost model** of each tool — per state transition? per CPU? per request? per hour?

Only after that analysis can you decide serverless vs. a Docker image in a process vs. something else. You can't decree up front that "my whole system will be serverless" or "my whole system will be ECS" or "everything uses database type X." And **don't be swayed by sensationalist headlines** ("Amazon abandons microservices" — not really true): read the article, form your own opinion, and above all **learn the why** — Step Functions didn't fail because it's bad, but because it was **too expensive for their use case** (too many transitions); ECS works because it's a **closed process that can keep data in memory**.

---

## 🧭 The Big Picture (mental map)

```
            "Monolith or microservices?"  →  WRONG question.
     Right question: "How do I EVOLVE my architecture safely,
                      and does THIS workload fit THIS tool?"

  EVOLVING (Sam Newman patterns)              CHOOSING (Prime Video lesson)
  ───────────────────────────────             ─────────────────────────────
  1. STRANGLER FIG                            Analyze the workload:
     new system grows AROUND the old           • concurrency needed?
     → migrate incrementally                   • large files? long processes?
     → switch traffic only when                • how many calls / transitions?
       tested & proven                         • cost model: per use vs per hour?
            │                                          │
            ▼                                          ▼
  2. DECOMPOSE THE DATABASE                   Quick to build, visual orchestration,
     (the hard part — data ownership)          controlled transition count
     • DB views (virtual tables)               → serverless / Step Functions
     • dedicated data service                 Heavy, long, big-file, hammered w/ calls
     • data sync patterns                      → one process (ECS/VM) + autoscale
     (myth: "1 service = 1 DB" — no!)         …and MIX them: Prime Video kept a
            │                                  Lambda to distribute tasks
            ▼
  3. EXPECT GROWING PAINS
     more services = more pain:
     breaking changes (→ typed contracts),
     harder reporting/logs,
     painful local dev (30 lambdas vs 1 monolith)

  Migrate ONLY for real reasons (independent scaling, operational gain) —
  otherwise: local problem you could reason about  →  distributed problem
             + network failures + availability issues (they WILL appear)

                      NO GOLDEN HAMMER. EVER.
```

---

## 💡 Key Takeaways

1. **Don't marry an architecture — learn to evolve it.** The Strangler Fig lets you migrate incrementally and only switch traffic when the new system is tested and proven.
2. **Code is the easy part; data is the hard part.** Decompose the database deliberately (views, dedicated data services, sync patterns) — and no, each microservice does *not* need its own database.
3. **Microservices trade a local problem for a distributed one:** breaking changes without editor help, harder reporting, painful local dev, network failures. Accept those costs only for real reasons (independent scaling, operational gains).
4. **Prime Video didn't abandon microservices** — one monitoring service swapped Step Functions + per-frame S3 traffic for a single ECS process (cost down ~90%), *and kept a Lambda to distribute work*. Each tool in its place.
5. **Know the cloud's cost model before you build** ($0.025 per 1,000 state transitions, S3 per-transfer fees): pay-per-use punishes hammered, big-file, long-running workloads — and never form your architecture opinions from sensationalist headlines.

---

*Related summaries: [Backend Architecture Patterns](backend-architecture-patterns-video-summary.md), [System Design Thinking](system-design-thinking-video-summary.md)*
