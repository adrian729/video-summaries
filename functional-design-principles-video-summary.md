# Functional Design Principles — Video Summary & Mini-Lesson

> **Video:** [Design Principles in Functional Programming](https://www.youtube.com/watch?v=SxCr-31kiDY) — Aleksandr Granin
> *(57-min companion talk to the same author's "Design Patterns in Functional Programming")*

---

## 📋 Brief Summary

The OOP world has its design *principles* — SOLID, GRASP, low coupling, high cohesion. Are they applicable to functional programming, and does FP add any of its own? Granin's answer: **most mainstream principles aren't really about OOP at all** — SOLID is so universal it applies to electronics or civil engineering, let alone functional code — and FP adds a few principles that flow from its mathematical nature. He first untangles the vocabulary (a *principle* is a compass that guides design, distinct from patterns, approaches, and methodologies), then walks SOLID one letter at a time showing the FP equivalent, then covers the universal principles (Divide & Conquer, Low Coupling, High Cohesion, KISS, DRY-as-"Dumb-but-Uniform", Least Power, Least Knowledge, Pragmatism) and the genuinely FP-flavored ones (**Make Invalid States Irrepresentable**, **Mathematical Nature**). A recurring warning: FP culture loves over-smart, category-theory-soaked code — resist it, reuse mainstream terms, and stay pragmatic if you want FP to be industry-accepted.

---

## 🎓 Mini-Lesson: The Concepts

### 1. What a "design principle" actually is

- A principle is **guidance — a compass** showing a direction to go to get better results. Its job: make the **code, design, and architecture better**.
- Don't confuse the layers of knowledge:

```
  METHODOLOGY        (OO Design · Functional Declarative Design)
      └─ embeds → GENERAL PRINCIPLES   (SOLID, KISS, Divide & Conquer…)
                      └─ guide → LOWER-LEVEL SOLUTIONS (subsystems, modules, structure)
                                    └─ then → IMPLEMENTATION DETAILS (isolate these!)
```

- Principles sit **above the actual coding**. Implementation details follow last and should be kept **isolated** so they can't ruin the design.

### 2. Most mainstream principles are universal — not OOP-specific

- **SOLID** is an acronym of five principles that, on a closer look, **aren't about classes or Java-style interfaces at all**. They're so general they apply outside programming entirely (electronics, civil construction).
- **Modularity** is universal. **Inversion of Control (IoC)** is *naturally embedded* in FP: passing a function into another function is exactly IoC — you hand over a callback and the inner code calls you back indirectly.
- So FP doesn't need to *replace* these principles; it just expresses them with its own tools (functional interfaces, eDSLs, free monads, type-level programming).

### 3. SOLID, translated to functional programming

| Letter | Principle | FP expression (with the talk's example) |
|---|---|---|
| **S** | Single Responsibility | Don't cram unrelated things into one God-like ADT. A `Sandwich` DSL with `start/addIngredient/finish` shouldn't also have `notify` — extract `notify` + `schedule` into a separate `CookingMachine` ADT, creating two clean subdomain layers. |
| **O** | Open/Closed | Open for extension, closed for modification: keep a **stable core**, plug new functionality in independently. Shown via a type-level extensible system — `sandwich` and `coffee` recipes don't know about each other and plug into a cooking program that doesn't know about them (also doable with service handles / free monads at the value level). |
| **L** | Liskov Substitution | We substitute constantly (pass a comparator into `sort`). If that comparator throws an exception `sort` never expected, the **contract is broken**. A GUI interpreter for a terminal DSL that throws where the console interpreter didn't = LSP violation. Always honor the *contract behind the interface*. |
| **I** | Interface Segregation | Keep functional interfaces small and focused; don't merge unrelated ones. Decades of mainstream experience say ignoring this makes systems "hardly maintainable." |
| **D** | Dependency Inversion | Logic depends on **interfaces**, not implementation details; real implementations (interpreters, instances) arrive at runtime. Code that hits a DB / Google directly **isn't testable** — every test hits Google. Detach it behind functional interfaces. |

> ⚠️ **FP's own SRP blind spot:** *effect systems*. People pile `Concurrent`, `Secret`, `Config`, `Logger`, `Env` and domain-specific effects into one giant effect stack, mixing abstraction levels — a very common, very bad habit that makes code unorganized.

### 4. The universal principles

- **Divide & Conquer** — the *umbrella* principle that rules them all: split things small enough for your brain to comprehend and isolated enough not to interfere unexpectedly. First divide into **layers** (don't mix them without real need — performance or an external framework can force it); then put **interfaces on the boundaries** so subsystems interact indirectly.
- **Low Coupling** — letting many clients reference modules directly is a mess; a change in one module breaks everyone downstream. Put **one interface on the boundary** as the single access point, and changes stay isolated.
- **High Cohesion** — one unit, one concern, no distractions. A `TuringMachine` module holds only Turing-machine ADTs — no logger, no database. Applies at every level: modules, interfaces, functions, type classes, ADTs.
- **DRY → "Dumb but Uniform"** — prefer **one tool powerful enough** for most problems over inventing a new solution per problem; knowledge becomes transferable across the project. FP's failing here: over-smart code drowning in functors, applicatives, isomorphisms, comonads — *"a world map where every country speaks a different language."* Avoid it to save time, money, and mind-power for the actual business logic.
- **KISS** — keep it simpler than it could be; **reuse mainstream terms** (say "subsystem" instead of "effect") so FP can become industry-accepted; don't import math notions without real need.
- **No Perfectionism / Pragmatism** — libraries are gorgeous and mathematically correct, but everyday programs sometimes need dirty hacks and cut corners. Everything should be **justified by the final goal**, not by elegance.
- **Least Power** — don't touch what you don't need. In a Minesweeper game with a 3-layer monad stack (IO / immutable-stateful board / immutable-stateful score), `notifyFailure` only prints — but it's typed over the *whole stack*, so it *can* mutate board and score. Give it the **least power** (`IO` only): safer, and reusable from anywhere.
- **Least Knowledge (Law of Demeter)** — don't reach "over heads" deep into a hierarchy to poke the innermost field; that couples you to the whole chain. Route access through a **single point** like `getCell`, which hides the board's internal structure so changing it doesn't break callers.

### 5. The genuinely functional principles

- **Make Invalid States Irrepresentable** — build domain models with **algebraic data types** so invalid states *can't be constructed*. FP sits close to the mathematical world ("invalid mathematics isn't mathematics"), and generics / templates / type-level programming all push this further. (OOP languages are adopting ADTs to chase the same goal.)
- **Mathematical Nature** — hunt for **mathematical properties** in your domain; if you find them you get powerful generic tooling *for free*. A list is a functor, an applicative, and a monad (the *enumeration* monad) whether you noticed or not — discover that and you unlock every generic function written for those abstractions. Not mandatory, but available when you want it.

### 6. Paradigm-*defining* properties (mentioned, not really "design")

Granin separates these out — they define a paradigm rather than guide design:

- **Polymorphism** — dynamic in OOP, static in Haskell/Scala (and duck-typed/dynamic in Clojure). FP has it, but it doesn't *define* FP.
- **Encapsulation** — OOP hides data+methods in a class; FP encapsulates via **lambdas / higher-order functions** (a function is the *simplest possible* functional interface). Done naturally, not as a design act.
- **Inheritance** — mostly an OOP tool; FP can mimic it but rarely needs to.
- **Tell, Don't Ask** — tell the object to change its data; don't pull internals out to change them yourself. An OOP idea with limited FP use.
- **GRASP** — *skipped*: an inconsistent mix of patterns and principles, never as successful as SOLID.
- **Immutability / Purity / Declarativity** — natural to FP; of these, **declarativity** (interpretable declarative DSLs you run later) is a genuine, powerful design technique for beating complexity.

---

## 🧭 The Big Picture (mental map)

```
            DESIGN PRINCIPLE = a compass (guides design; ≠ pattern/methodology)
                                       │
        ┌──────────────────────────────┼──────────────────────────────┐
        ▼                              ▼                               ▼
  UNIVERSAL (apply anywhere)     SOLID (not really OOP)         FUNCTIONAL-FLAVORED
  ─────────────────────────     ──────────────────────         ───────────────────
  Divide & Conquer  ◄── rules   S  one responsibility           Make Invalid States
  Low Coupling       them all   O  stable core, plug-in ext.      Irrepresentable (ADTs)
  High Cohesion                 L  honor the contract           Mathematical Nature
  KISS / Dumb-but-Uniform       I  small focused interfaces       (list = functor/monad,
  Least Power / Least Knowledge D  depend on interfaces,           free tooling)
  Pragmatism (no perfectionism)    inject implementations
        │
        └─ all served by FP tools: functional interfaces · eDSLs · free monads · type level

  PARADIGM-DEFINING (not design): polymorphism · encapsulation(λ) · inheritance · tell-don't-ask
```

---

## 💡 Key Takeaways

1. **SOLID is universal, not OOP.** Read SOLID, low coupling, and high cohesion as engineering principles and they translate cleanly to functional code — FP just expresses them with interfaces, eDSLs, and free monads instead of classes.
2. **IoC is built into FP for free.** Every time you pass a function into a function, you're inverting control — the principle is in the skeleton of the paradigm.
3. **Two principles are distinctly functional:** *make invalid states irrepresentable* (lean on ADTs and the type system) and *mathematical nature* (find the functor/monad hiding in your domain and inherit its tooling).
4. **Least Power + Least Knowledge keep code reusable and decoupled:** give each function only the capability it needs, and never reach across the object graph — funnel access through one point.
5. **Resist FP's over-smartness.** Category-theory-heavy, term-inventing code is the enemy of maintainability and industry adoption. Stay *Dumb-but-Uniform*, reuse mainstream terms, and be pragmatic about cutting corners.

---

*Related summaries: [Functional Design Patterns](functional-design-patterns-video-summary.md) — the companion talk on FP **patterns** (the "how", to this talk's "why/which-direction") · [Design Patterns (OOP / Gang of Four)](design-patterns-video-summary.md) · [Programming Paradigms](programming-paradigms-video-summary.md).*
