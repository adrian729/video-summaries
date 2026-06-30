# Functional Design Patterns — Video Summary & Mini-Lesson

> **Video:** [Design Patterns in Functional Programming](https://www.youtube.com/watch?v=Usr-yxvb9TY) — Aleksandr Granin
> *(1-hour conference-style talk by the author of *Functional Design and Architecture*)*

---

## 📋 Brief Summary

The famous *Gang of Four* book (1994) catalogued **object-oriented** design patterns and shaped how a generation writes software — yet functional programming, which is *older* than OOP, never got an equivalent catalogue. There's even a famous Scott Wlaschin talk arguing FP doesn't *need* patterns: "OOP patterns on the left, a plain function on the right." Granin's thesis is that this is half-right. Yes, you don't need the *OOP* patterns (a function replaces Adapter; pattern matching replaces Visitor) — but FP has its **own** design patterns that nobody has properly catalogued yet. He proposes two design methodologies (**Functional Declarative Design** and **Pragmatic Type-Level Design**), draws a sharp line between a **functional idiom** (a natural mathematical *property* of a structure) and a **design pattern** (an invented *external mechanism*), and walks through three families: architectural, type-level, and "infrastructure" patterns — all backed by working Haskell/C++ demo code.

---

## 🎓 Mini-Lesson: The Concepts

### 1. Why design patterns exist at all

- Patterns let us **solve typical problems with typical solutions** — it's *cheaper* to reuse a well-thought-out piece than to research everything from scratch.
- Used correctly, they **keep complexity low**. Used blindly — sprinkled in "because patterns are good" — they cause **over-engineering**: too many patterns where something simpler would do.
- The fix: never treat patterns as standalone gimmicks. Tie them to a **methodology** that tells you *why* and *when* to apply each one. OOP design is that methodology for OOP; Granin proposes two more for FP:

| Methodology | For building… |
|---|---|
| **Functional Declarative Design** | Programs using ordinary functions, types, ADTs |
| **Pragmatic Type-Level Design** | Apps with heavy *type-level* programming — where the logic itself lives in types |

### 2. The key distinction: idiom vs. design pattern

This is the conceptual heart of the talk. Functional developers often call everything a "pattern," but Granin separates two very different things:

| | **Functional Idiom** | **Design Pattern** |
|---|---|---|
| Nature | *Internal* — a natural property / mathematical essence of a data structure | *External* — an auxiliary, compound mechanism you build |
| Describes | **What the object is** and its inseparable properties | **How the system should work** |
| Existence | Exists **whether you know it or not** | **Doesn't exist until you invent it** |
| Examples | Functor, Applicative, Foldable, Traversable | ReaderT, Higher-Kinded Data, Type Selector |

- A **functor** is a property: you can `map` over a list, changing its contents but not the number of elements. A list *is* a functor, and also *is* a monad (the "nondeterminism" / enumeration monad) — those properties hold even if you never noticed them.
- A pattern like **ReaderT** has no such intrinsic property — it's a compound mechanism assembled from parts. It only exists because someone designed it.
- The talk is about the **patterns**, not the idioms (idioms are a whole separate topic).

### 3. Architectural patterns — shaping the application skeleton

**Three Layer Cake** — the FP take on layering. Three software layers:

```
  Business logic / Domain layer  → depends only on interfaces + DSLs (pure ideas)
  Service layer ("effects")      → interfaces to the outside world
  Application / Implementation   → runs business logic, wires real implementations
```

- The **domain at the center depends on nothing** — it doesn't know about external services, config, or global state. It speaks only the essence of the domain, so it stays (more or less) **pure**.
- Same idea as **Hexagonal** (ports & adapters) and **Onion** architecture — nested shapes where the inner core depends outward on nothing. Consult Domain-Driven Design, Martin Fowler, Bob Martin for the deeper rationale.

**Hierarchical Free Monads** — Granin's own architecture: the interfaces aren't type classes, they're **free-monadic eDSLs** organized hierarchically. In his **Hydra** framework there's a logger subsystem (with its own eDSL), a state subsystem, a key-value DB subsystem — split into a *connectivity* layer and a *business logic* layer. He even mimicked Haskell's parser-combinator library in C++ with this approach.

**Interfaces — four ways to provide them:**

| Approach | What it is | Trade-off |
|---|---|---|
| **Service Handle** | A control structure of method fields, filled with concrete implementations and passed as an object — basically OOP's *service locator* | Dead simple; but a "God handle" gets ugly when you want many small interfaces (interface segregation) |
| **ReaderT pattern** | Hide the handle inside a monadic **environment** you read from the context (`ReaderT env` over `IO`/`AppL`) | Easier to *use* (no manual passing); harder to *implement* (monad stacks) |
| **Final Tagless / MTL style** | One **type class per service**, injected as **constraints** on your business-logic functions | Idiomatic in Haskell/Scala; but John De Goes wrote "the death of Final Tagless" — Granin half-agrees you maybe shouldn't use it |

- Common pain across all of them: when you have **many** interfaces, you must invent a way to **merge** them — or dump everything into one big handle/environment/constraint list. There's no free lunch; pick simplicity vs. segregation deliberately.

### 4. Type-level patterns — when logic lives in types

From his in-progress book *Pragmatic Type-Level Design* (~80% done).

- **Higher-Kinded Data (HKD)** — make the *types of an ADT's fields* configurable via a type parameter. Pass `Identity` and you get a normal record; pass a different wrapper and the compiler generates a variant. Why? You often need **domain types**, **database types**, and **API types** that look different (a relational DB shape ≠ a domain shape ≠ a wire shape). HKD lets you reuse **one model** for all three instead of writing three near-duplicate types.
- **Type Selector** — alter the shape along **two dimensions**: mark each field with a *type tag*, and mark the intended *usage* with a *type-set mark*. A **type family** acts as a lookup table matching `(usage, tag) → concrete type`. Ask for `Domain` and you get domain types; ask for `SQLite`/`Postgres` and you get those. It's effectively **pattern matching on the type level**. (Haskell type families; other languages use traits / template metaprogramming.)
- **Granular Type Selector** — same field-altering idea, but the types have **both** a value-level *and* a type-level representation, so you can program with them in type-level code. On the type level you can't use `String`/`Int` — you use **symbols** and **Nat** (via `DataKinds`). Lets you lift your own ADTs "one level up" into the type level. (Admittedly an advanced topic — not where you'd *start* teaching type-level programming.)

### 5. Infrastructure / "uncategorized" patterns

- **Control Structure** — define a subsystem (e.g. a command-line subsystem) and get back a control structure you talk *through*. Works best when subsystems are **actors**: the structure is your interface to send messages, receive data, or shut them down.
- **MVar Request-Response** — reliable communication between two threads built on **MVars**. A requester puts a request in an MVar; the responder is *blocked* waiting on it, processes it, puts the response in a second MVar; the requester blocks until the response arrives. Guarantees no events are missed and both sides stay synchronized.
- **Typed/Untyped + Typed Avatar** — how to store *heterogeneous* user state in a *homogeneous* collection. The typed side (free-monadic eDSLs) knows the generic `a`; the **interpreters** can't, so values are converted to an **untyped `any`** (`unsafeCoerce`/`GHC.Any` in Haskell, `std::any` in C++) and kept in homogeneous storage. The **Typed Avatar** (e.g. a `TVar`) holds *no data itself* — it's a lightweight **key/accessor** (his STM `TVar` stores only a creation timestamp) pointing into the untyped store. Used in his custom Software Transactional Memory library.
- **Functor over an impure async value** — to get `async`/`await`-style syntax with free monads, make an impure async value (an `MVar`) behave as a **functor** without reading it: instead of mutating the contents (impure), you **grow the function** that will transform the contents *when it's eventually run* inside the interpreter. Business logic stays pure; the impure read happens only at interpretation.
- **Bracket (RAII)** — the FP analogue of C++ RAII / `using`. A `bracket` function ties **resource acquisition + release** together, is **exception-safe** (resources are freed even if something throws), and **stacks** cleanly. Rule of thumb: **one resource per bracket** — mixing two means a failure between them could leak the other; stack many single-resource brackets instead.

### 6. Postscript — "but free monads are slow!"

Granin's defense against the Haskell crowd:

| Free monad variant | Stays fast until… |
|---|---|
| **Normal** free monad | ~5,000–10,000 operations per scenario, then degrades |
| **Church-encoded** free monad | ~**1 million** operations — with ~1.1× evaluation overhead |

You almost never have a single business-logic scenario with a million operations, and you can split long scenarios anyway — so the performance objection mostly doesn't bite.

---

## 🧭 The Big Picture (mental map)

```
   "FP doesn't need design patterns" (Wlaschin)  ──► HALF TRUE
        │                                              │
        │  true: drop the OOP patterns                 │  false: FP has its OWN
        │  (function replaces Adapter,                 │  patterns, just uncatalogued
        │   pattern-match replaces Visitor)            │
        ▼                                              ▼
   ┌──────────────────────────────────────────────────────────────┐
   │            FUNCTIONAL DESIGN PATTERNS (Granin)                 │
   └──────────────────────────────────────────────────────────────┘
        │
        ├── IDIOM  ≠  PATTERN
        │     idiom = intrinsic property (Functor, Foldable…) — exists already
        │     pattern = invented mechanism (ReaderT, HKD…) — built from parts
        │
        ├── ARCHITECTURAL ── Three Layer Cake · Hierarchical Free Monads
        │        interfaces: Service Handle → ReaderT → Final Tagless
        │
        ├── TYPE-LEVEL ───── Higher-Kinded Data → Type Selector → Granular Type Selector
        │        (logic lives in types; one model → domain/DB/API)
        │
        └── INFRASTRUCTURE ─ Control Structure · MVar Request-Response
                 Typed/Untyped + Typed Avatar · Functor-over-async · Bracket(RAII)

   Backed by: Hydra framework, Labyrinth/Astro demos, books
   (Functional Design & Architecture · Pragmatic Type-Level Design)
```

---

## 💡 Key Takeaways

1. **FP has design patterns too.** Dropping the *OOP* patterns (functions and pattern matching subsume them) doesn't mean FP is pattern-free — it has its own, just no *Gang of Four* book to name them yet.
2. **Idiom ≠ pattern.** An idiom is an *intrinsic mathematical property* a structure already has (a list *is* a functor); a pattern is an *external mechanism you invent* (ReaderT). Don't conflate them.
3. **Patterns need a methodology.** Applied without a "why/when" framework they breed over-engineering. Granin organizes his around Functional Declarative Design and Pragmatic Type-Level Design.
4. **Interfaces are a spectrum of effort.** Service Handle (simplest) → ReaderT (easier to use, harder to build) → Final Tagless (idiomatic, controversial). The recurring tax is *merging many small interfaces* — choose your trade-off on purpose.
5. **Type-level programming is a real toolbox.** Higher-Kinded Data and Type Selectors let one model serve domain, database, and API shapes — the logic moves into the type system itself.

---

*Related summaries: [Design Patterns (OOP / Gang of Four)](design-patterns-video-summary.md) — the OOP catalogue this talk deliberately contrasts itself against · [Programming Paradigms](programming-paradigms-video-summary.md) — where OOP and functional sit relative to each other.*
