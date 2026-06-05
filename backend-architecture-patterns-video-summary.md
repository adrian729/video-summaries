# Backend Architecture Patterns — Video Summary & Mini-Lesson

> **Video:** [Cada Patrón de Arquitectura Explicado en 18 Minutos](https://www.youtube.com/watch?v=q3YQy1lJutw) — BettaTech
> *("Every Architecture Pattern Explained in 18 Minutes")*

---

## 📋 Brief Summary

The presenter walks through the major **backend architecture patterns** — monolith, layered architecture, clean/hexagonal, modular monolith, microservices, CQRS, and serverless — with the central message that **no architecture is good or bad by itself**. Teams fail when they pick architectures by hype ("microservices because they're modern", "serverless because it's trendy", "Clean Architecture because I saw it in a talk") instead of by criteria. The video ends with a **decision algorithm**: a series of questions (team size, coupling, deployment frequency, budget, scalability needs) to choose the right architecture for your next project.

> Recurring theme: *"There is no golden hammer. What separates a senior from a junior is the ability to analyze, weigh, and decide."*

---

## 🎓 Mini-Lesson: The Patterns

### 1. The Monolith (the most hated, unfairly)

- **Definition:** it's not about having all code together or about the tech stack — a monolith is defined by having a **single deployment**: all the code ships at once.
- A monolith is **not bad by definition** — it's bad when implemented badly. The bad reputation comes from monoliths degenerating into a **"Big Ball of Mud"**: highly coupled code, hard to update, with responsibilities all mixed together.
- **The problem isn't the monolith — it's the terrible code.**

### 2. Layered Architecture (your first separation of concerns)

The first structure you should apply *inside* a monolith (or inside each microservice). The presenter's preferred 3-layer split:

| Layer | Responsibility | Examples |
|---|---|---|
| **Transport** | Communication with the outside world; API definition | HTTP, Express, tRPC, CLI |
| **Domain** | Business logic and processing — **doesn't know** if the request came via HTTP, tRPC, or a CLI | Functions, calculations |
| **Data / Repositories** | Everything that calls databases or external APIs | DB queries, third-party calls |

Simple (only three layers) but with very clear responsibilities — it has served him in most backend projects.

### 3. Clean Architecture & Hexagonal Architecture

- One step beyond layers: they help manage **dependencies** when projects grow complex — multiple databases, multiple external APIs, multiple transport layers (public API + CLI + desktop app).
- **The criticism:** hexagonal architecture works best when **business requirements are defined up front** — which clashes with agile methodologies and fast prototyping. In the wrong context, these "purer" architectures slow you down and add boilerplate.
- **Rule:** don't aim for the *purest* architecture — aim for the one **your team can actually maintain**.

### 4. The Modular Monolith (the sweet spot)

- Sounds contradictory but isn't: a modular monolith is simply **a monolith done well**. Everything still deploys at once, but responsibilities and modules are well separated — you can work on payments without breaking purchases.
- Most companies are **comfortable here for years** — you may never need to leave this stage.
- How to modularize: layers, clean/hexagonal, or just any split your team finds simple. Tools like **DDD** can help (though the presenter finds it a bit extreme). A good analysis of features, users, and dependencies is the foundation.
- This stage is the **prerequisite maturity level** before even considering microservices.

### 5. Microservices (the hype machine)

- Historically used as a **marketing word** ("our tech is scalable!") and even to attract investment.
- Data point (O'Reilly article, 2020): many companies adopted microservices since ~2015, but a large share only had "a bit of success" — i.e., incomplete migrations or discovering they didn't solve their problems. Failures were mostly **not technical** — they came from team/context problems.
- **The key question before migrating:** do your dev teams have a clear ownership of their domains/responsibilities? If not, microservices will hurt you.
- Authority backup:
  - **Martin Fowler** (author of *Refactoring*): never *start* with microservices — start with a monolith, make it modular, split only when it's genuinely a problem ("Monolith First").
  - **Sam Newman** (*Building Microservices*): decide based on **team size** — in a small team, microservices are high risk and the benefits (deploy speed, isolation) don't compensate the technical cost.
- The internet joke referenced: needing to traverse a chain of microservices ("Galactus") just to add a birthdate field to a user profile.

### 6. CQRS (Command Query Responsibility Segregation)

- **What it is:** separating **commands** (writes/actions sent to the server) from **queries/responses** — enabling you to **queue requests** and listen for responses **asynchronously**. Often used with WebSockets: each command gets a unique ID; responses are matched back by that ID.
- **When it shines:** very high ingest volume — e.g., recording user events (clicks, scrolls) on a web analytics platform.
- **The cost:** development complexity (every command needs unique identification, async tracking). **Don't** use it where a simple synchronous HTTP request would do.

### 7. Serverless (deployment, not architecture)

- Key clarification: **serverless ≠ microservices**. Serverless is just "a fancy word for delegating server management to a cloud." You can run a **monolith on Lambdas** (the presenter does, for his academy) or microservices on dedicated servers.
- **Caveats:**
  - **Cold starts:** a Lambda that hasn't run recently must boot first, so the first call is slower (people even pinged Lambdas hourly to keep them "warm"). AWS has improved this over time.
  - **Price:** serverless is generally **more expensive** than a dedicated server.
- **When it makes sense (two cases):**
  1. You need very high availability, no matter what.
  2. **Spiky traffic** — e.g., everyone uses your product at 9 a.m., then traffic drops. If traffic is *constant*, a dedicated server is likely cheaper.
- **The Amazon Prime Video story:** a team famously moved from serverless microservices to a dedicated server (EC2) and people cried "microservices are a scam!" The real reason: **cost** — their specific problem fit Lambdas badly, not because Lambdas are bad.

---

## 🧭 The Decision Algorithm (how to choose)

The video's step-by-step questions for picking an architecture:

```
1. Are there strong, frequent transactions BETWEEN different domains?
   (lots of cross-system communication = high coupling)
   ├─ YES → stay with the MONOLITH
   └─ NO (well modularized) → continue ↓

2. Are you in a stable maintenance/testing phase (requirements not changing)?
   ├─ YES → good moment for CLEAN / HEXAGONAL architecture
   │         (these need stability; they add boilerplate)
   └─ NO → continue ↓

3. Is the team big enough?           ← the critical question
   ├─ NO  → stay with the MONOLITH
   └─ YES → MODULARIZE the monolith ↓

4. Does one specific module receive far more traffic than the rest?
   └─ YES → consider extracting THAT module into its own MICROSERVICE
```

**Factors to weigh throughout:** team size, deployment frequency, budget, real scalability needs.

---

## 💡 Key Takeaways

1. **No architecture is good or bad — only well- or badly-matched to a problem.**
2. **Monolith first** (Fowler, Newman): monolith → modular monolith → extract microservices only when a module genuinely needs it.
3. **Team size and domain ownership** matter more than technology when adopting microservices.
4. **Serverless is a deployment choice**, not an architecture — good for spiky traffic, expensive for constant load.
5. Choose the architecture **your team can maintain**, not the purest one.

---

*Note: the presenter mentions a downloadable version of the decision algorithm in the video description, and a related video analyzing the Prime Video serverless→EC2 migration.*
