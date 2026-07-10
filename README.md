# 🎥 Video Summaries & Lessons

A personal library of YouTube videos (mostly the Spanish channel *BettaTech*) distilled into
English mini-lessons. Each summary follows the same shape: **📋 Brief Summary → 🎓 Mini-Lesson →
🧭 Mental Map → 💡 Key Takeaways**. Open the file to read the lesson, or click ▶ to watch the source video.

---

## 🗺️ Suggested learning paths

Pick a thread and follow the files in order:

1. **CS foundations** — `programming-paradigms-video-summary.md` → `data-structures-video-summary.md` → `p-vs-np-video-summary.md` → `design-patterns-video-summary.md` → `testing-and-correctness-video-summary.md`
5. **Functional design** — `programming-paradigms-video-summary.md` → `design-patterns-video-summary.md` → `functional-design-principles-video-summary.md` → `functional-design-patterns-video-summary.md` *(why/direction before how)*
2. **Backend & systems** — `backend-development-concepts-video-summary.md` → `databases-concepts-video-summary.md` → `backend-architecture-patterns-video-summary.md` → `system-design-thinking-video-summary.md` → `scaling-evolution-video-summary.md` → `monolith-vs-microservices-video-summary.md` → `real-world-systems-video-summary.md` → `atlassian-edge-platform-video-summary.md`
3. **AI-assisted development** — `ai-development-concepts-video-summary.md` → `harness-engineering-video-summary.md` → `claude-code-sdd-harness-video-summary.md` → `uncle-bob-agent-system-video-summary.md`
4. **Shipping & operations** — `devops-concepts-video-summary.md` → `continuous-deployment-video-summary.md` → `real-world-systems-video-summary.md` (how systems fail)

---

## 🧮 Computer Science Fundamentals

| File | What you'll learn |
|---|---|
| **Programming Paradigms** ([▶](https://www.youtube.com/watch?v=GehWsIr6oFo))<br>`programming-paradigms-video-summary.md` | The historical family tree: the 1930s crisis → Church (lambda/functional) vs. Turing (machine/imperative), plus the logic and OOP branches |
| **Data Structures (and Why Analysis Matters)** ([▶ 1](https://www.youtube.com/watch?v=iDHzQO2r3xo) · [▶ 2](https://www.youtube.com/watch?v=J1_UIf17p7I))<br>`data-structures-video-summary.md` | The 8 core structures (arrays, lists, stacks, queues, graphs, trees, heaps, tries); why hash collisions are inevitable; external merge sort |
| **P vs NP** ([▶](https://www.youtube.com/watch?v=tBBfsM_0hrE))<br>`p-vs-np-video-summary.md` | Complexity theory: easy-to-solve (P) vs. easy-to-verify (NP), the Millennium Prize, TSP, and non-constructive proofs |

## 🏗️ Backend & Architecture

| File | What you'll learn |
|---|---|
| **Backend Development Concepts** ([▶](https://www.youtube.com/watch?v=l3HJsXA-Fa4))<br>`backend-development-concepts-video-summary.md` | The vocabulary: APIs, load balancers, reverse proxies, auth, indexes, ACID, CAP, observability, idempotency |
| **Databases** ([▶](https://www.youtube.com/watch?v=xz1fJ7M2g-o))<br>`databases-concepts-video-summary.md` | Relational vs. NoSQL (document, key-value, graph); CAP theorem; ACID vs. BASE; indexes; normalization (3NF) |
| **Backend Architecture Patterns** ([▶](https://www.youtube.com/watch?v=q3YQy1lJutw))<br>`backend-architecture-patterns-video-summary.md` | A decision algorithm for monolith, layered, clean, modular, microservices, CQRS, and serverless architectures |
| **System Design Thinking** ([▶](https://www.youtube.com/watch?v=dbLQ_0Ivg4U))<br>`system-design-thinking-video-summary.md` | Start from access patterns: read-heavy (fan-out: CDNs, caches, replicas) vs. write-heavy (fan-in: queues, workers); resilience |
| **Scaling Evolution: From Client-Server to Distributed Systems** ([▶](https://www.youtube.com/watch?v=2nEiIG-xca4))<br>`scaling-evolution-video-summary.md` | The step-by-step infra evolution: 3-tier → load balancers → stateless servers → caches → CDNs → queues/workers → observability, adding each piece only to solve a real problem |
| **Monolith vs. Microservices** ([▶ 1](https://www.youtube.com/watch?v=aVL3AObcb9M) · [▶ 2](https://www.youtube.com/watch?v=RPEM2Kx5-JE))<br>`monolith-vs-microservices-video-summary.md` | Sam Newman's evolution patterns (Strangler Fig, DB decomposition) and the Amazon Prime Video cost case study |
| **Real-World Systems: How They're Built and How They Fail** ([▶ 1](https://www.youtube.com/watch?v=zkSSoEpL7zU) · [▶ 2](https://www.youtube.com/watch?v=waHk1grLR5U))<br>`real-world-systems-video-summary.md` | Twitter's search infra (Elasticsearch, Kafka, read/write separation) and why the internet goes down (CDN/DNS centralization, cascading failure) |

## 🚀 DevOps, Deployment & Infrastructure

| File | What you'll learn |
|---|---|
| **DevOps** ([▶](https://www.youtube.com/watch?v=UTNbLoZCOgM))<br>`devops-concepts-video-summary.md` | Culture over job title: blameless culture, CI/CD, IaC, shift-left, monitoring vs. observability (logs/metrics/traces), the nines, SLA/SLO/SLI |
| **Continuous Deployment Without Fear** ([▶](https://www.youtube.com/watch?v=WP2yXIEDJ9c))<br>`continuous-deployment-video-summary.md` | Feature flags to separate deploy from release: A/B testing, trunk-based dev, canary rollouts *(delta — pairs with DevOps)* |
| **Building Atlassian's Edge Platform** ([▶](https://www.youtube.com/watch?v=55pTFVoclvE))<br>`atlassian-edge-platform-video-summary.md` | Case study: a ~2,000-proxy Envoy edge platform, self-service provisioning, and centralizing cross-cutting concerns at the edge |

## 🎨 Frontend

| File | What you'll learn |
|---|---|
| **Frontend Development Concepts** ([▶](https://www.youtube.com/watch?v=Rla0IMxIlNc))<br>`frontend-development-concepts-video-summary.md` | Browser loading, DNS, HTML/CSS/JS, the DOM, the event loop (micro/macrotasks), state, the Virtual DOM, and rendering strategies (CSR/SSR/SSG) |
| **How Clay's UI Layout Algorithm Works** ([▶](https://www.youtube.com/watch?v=by9lQvpvMIc))<br>`clay-ui-layout-algorithm-video-summary.md` | Building a flexbox-style layout engine from scratch: UI as a tree, why sizing must precede positioning, multi-pass fit/grow/shrink/wrap, and a standalone reproducible algorithm spec |

## 🤖 AI-Assisted Development

| File | What you'll learn |
|---|---|
| **AI Development Concepts** ([▶](https://www.youtube.com/watch?v=MgtM_Ktuc5A))<br>`ai-development-concepts-video-summary.md` | The vocabulary of AI-assisted dev: tokens, transformers, context windows, why models "get dumber," and agents, skills & MCPs |
| **Harness Engineering** ([▶](https://www.youtube.com/watch?v=q9Vaoz0hd0U))<br>`harness-engineering-video-summary.md` | Building context/tools/memory/validation around LLMs; context degradation; the three pillars of harness architecture |
| **Adapting Claude Code for Spec-Driven Development** ([▶](https://www.youtube.com/watch?v=ElGlTv2A_bM))<br>`claude-code-sdd-harness-video-summary.md` | A concrete SDD harness: spec pipeline (requirements → design → tasks), EARS notation, human-in-the-loop gates *(delta — read Harness Engineering first)* |
| **Uncle Bob's AI Agent System** ([▶](https://www.youtube.com/watch?v=PoXC6XcVa1M))<br>`uncle-bob-agent-system-video-summary.md` | A multi-agent workflow: spec → hard spec → Gherkin → TDD → mutation testing, with five markdown agents and file-based memory |

## 🧩 Software Design & Quality

| File | What you'll learn |
|---|---|
| **Design Patterns** ([▶ 1](https://www.youtube.com/watch?v=xtaqMX0OvH0) · [▶ 2](https://www.youtube.com/watch?v=rqOaZf4xMlI))<br>`design-patterns-video-summary.md` | 12 Gang-of-Four patterns across the creational, structural, and behavioral families |
| **Functional Design Patterns** ([▶](https://www.youtube.com/watch?v=Usr-yxvb9TY))<br>`functional-design-patterns-video-summary.md` | FP has its own patterns: idiom vs. pattern; architectural (Three Layer Cake, free monads, Service Handle/ReaderT/Final Tagless); type-level (HKD, type selectors); infrastructure (bracket/RAII, MVar request-response) |
| **Functional Design Principles** ([▶](https://www.youtube.com/watch?v=SxCr-31kiDY))<br>`functional-design-principles-video-summary.md` | SOLID is universal, not OOP-specific — translated to FP — plus Divide & Conquer, Least Power, Least Knowledge, and the FP-flavored "make invalid states irrepresentable" and "mathematical nature" |
| **Testing & Correctness** ([▶ 1](https://www.youtube.com/watch?v=PQYeWODU8Lo) · [▶ 2](https://www.youtube.com/watch?v=K--Lmy8qUCQ))<br>`testing-and-correctness-video-summary.md` | Test design (error cases first, mock-free integration tests, the coverage trap) and type-driven design (making invalid states uncompilable) |

## ⚡ Low-Latency & Real-Time Programming

| File | What you'll learn |
|---|---|
| **The Golden Rules of Audio Programming** ([▶](https://www.youtube.com/watch?v=SJXGSJ6Zoro))<br>`realtime-audio-programming-video-summary.md` | Pete Goodliffe's four rules for the real-time audio thread: stay in CPU budget; never do work of unknown duration (no malloc/locks/logging/disk/OS calls); respect threads (data tearing, atomic pointer swaps); know float vs. fixed-point — *jitter, not average speed, is the killer* |
| **What is Low Latency C++?** ([▶ 1](https://www.youtube.com/watch?v=EzmNeAhWqVs) · [▶ 2](https://www.youtube.com/watch?v=5uIsadq-nyk))<br>`low-latency-cpp-video-summary.md` | Cross-industry generalization (audio, gaming, HFT, embedded) of the same rules: latency vs. throughput vs. efficiency, deterministic-worst-case-over-average, cache-friendly data/algorithms, `[[assume]]`, and five real lock-free/wait-free algorithms (ring buffer, spin-on-write, RCU, double buffering, seqlock) for sharing data with a hot path |
