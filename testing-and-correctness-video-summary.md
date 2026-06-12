# Testing & Correctness — Video Summary & Mini-Lesson

> **Videos:** [¿Estamos haciendo MAL los tests de software?](https://www.youtube.com/watch?v=PQYeWODU8Lo) and [Si usas TYPESCRIPT, DEBERÍAS tener MENOS TESTS](https://www.youtube.com/watch?v=K--Lmy8qUCQ) — BettaTech
> *("Are we doing software testing WRONG?" and "If you use TypeScript, you SHOULD have FEWER tests")*

---

## 📋 Brief Summary

Both videos attack the same underlying question — **how do you gain confidence that your code is correct, and at what cost?** The testing video argues that most teams test the wrong things: they verify the *happy path* when the real goal of testing is **reducing uncertainty** — a safety net that catches you when future changes break things. His battle-tested recipe: prioritize **mock-free integration tests** over piles of unit tests, test **error cases first**, and don't worship coverage. The TypeScript video argues that in statically typed languages you can go further: encode business rules *into the type system* (**type-driven design**) so invalid states **don't even compile** — eliminating whole batteries of tests and error-handling code. Together: types make a whole class of bugs *impossible* at compile time, before the program ever runs; tests remain the (more expensive, runtime) tool for everything types can't see — actual behavior.

---

## 🎓 Mini-Lesson: The Concepts

### 1. What testing is actually for (video 1)

- His definition: testing is **"the philosophy of trying to detect errors before the users do."**
- The common workflow — write all the code, then write tests to *check it works* — misses the point. The aspiration should be using tests **not to prove it works, but to make sure it doesn't break** later.
- Goal: **reduce uncertainty**. If you change things in the future, you'll *notice* when you break something. *"It's a safety net — your airbag to keep you from crashing."*

### 2. The happy-path trap

- The **happy path** is the flow where everything happens exactly as intended. E-commerce example: user selects a product, adds it to the cart, pays.
- Most companies and developers he has worked with implement happy-path tests *first* — making sure "the part that works… works."
- But that flow has many failure points that **must** be tested:
  - You click "add" and a *different product* lands in the cart.
  - The cart shows a *different count* than what you added.
  - Checkout tries to charge a *different price* than the UI showed.
- His recommendation: **test everything that can break — error cases first, happy path last.** Spend real effort thinking about which tests, if they failed, would cause the biggest problems — and start there. Once the error cases are covered, the happy-path test is trivial.

### 3. The map of test types

Manual testing (open the browser, poke at what you built) is where everyone starts — and it's **very unscalable**: as the software grows, so does the pile of things to re-check (re-testing last year's login after every small change is a bad use of developer time). Hence **automated testing** — your manual checks programmed as scripts, each type attacking one abstraction layer:

| Type | Tests… | Example / Tools | Cost & speed |
|---|---|---|---|
| **Unit** | One unit of work — a single function, very close to the code | `sum(1, 2) === 3`; Vitest (Node/JS/TS), pytest (Python) | Cheapest, fastest |
| **Integration** | How parts integrate — a function that hits a web page, a database, or the disk | API-level tests; Testcontainers | More expensive, slower |
| **End-to-end** | The whole system as a user — spin up the page, a browser, click the buttons | Selenium, Cypress | Most expensive (you're simulating a person) |

When you have the unit layer *and* the integration layer tested, you end up with your whole software tested.

### 4. Mocks — and why he avoids them

- A **mock** is a stand-in for an external system (e.g., a fake database, typically implemented via polymorphism) so you can test a unit without spinning up the real thing. Mocks are widespread in most software companies.
- **His stated personal position: he doesn't like mocks much.** The core problem: *when you maintain a simulation of something, every time you change that something you also have to update the simulation.* Mocks can give you **false security**.
- His preference: integration tests where Docker-based tools (he recommends **Testcontainers**) spin up the *real* service during the test run — e.g., a real PostgreSQL, empty, that the test itself configures and fills. Tests become much more realistic because they hit a real database.

### 5. The testing pyramid — and his controversial amendment

- The classic **testing pyramid**: unit tests are cheap and fast → they should be the bulk (the base); integration tests (needing mocks or real databases) are pricier → fewer; **E2E** tests are slowest → the tiny tip.
- His amendment, after years across different software companies — and he flags it as **a bit controversial** versus people who follow the pyramid to the letter: **the most useful tests are integration tests without mocks.**
- Why: unit tests are **tightly coupled** to your implementation. Refactor your business logic (drop the `sum` function, add another) and you must rewrite the tests too — tests that perhaps weren't adding anything. He wants tests that are **stable**: "the just-and-necessary tests, testing the things that must not break."
- Integration tests act at a **higher abstraction layer**: in API/backend work he writes tests that call the **endpoint directly, like a client of the API** (create a user, then verify via an HTTP call that the user exists). He can then refactor internals freely — change controllers, events, the whole internal architecture — without touching the test. But if he breaks something inside, **the test raises the alarm**.

### 6. The coverage trap

- **Coverage** = how many lines/functions your tests execute. It looks like a great metric — surely we want 100%?
- His counterexample: `sum(a, b)` implemented incorrectly to **always return 3**. The test `sum(1, 2) === 3` passes, and coverage of that function is **100%** (every line is executed). Green test, full coverage — *clearly a bug*.
- So coverage gives you *some intuition* about how you're testing, but it **must not be a goal**: better a codebase at **70–80% coverage with well-designed tests** than one at 100% that doesn't test what it should.
- This matters even more in the AI era, where generating tons of tests is very cheap — but they don't always test what *should* be tested. Time spent on **test design** is the scarce ingredient.

### 7. Tests as the contract with AI (video 1's closing argument)

- Quoting Robert C. Martin's *Clean Code*: test code is **as important as production code** — "it is not second-class code; it needs thought, design and care," and must stay as clean as production code.
- Today the test suite is what guarantees that AI-generated code **doesn't break everything you already have** — in an AI harness, the verification stage is exactly where tests shine.
- "But the AI writes the tests too!" — Yes, you *can write* them with AI, but **you must think, design and curate them yourself**: *"tests are the contract you are making with the AI"* to ensure it builds what you actually asked for.

### 8. Static vs. dynamic typing (video 2)

- A **type system** defines the properties and operations allowed on a language's entities (a `number` allows `+`/`−`; a `string` allows concatenation). Important nuance: it's **false** that JavaScript or Python are "untyped" — they are **dynamically typed**.

| | Dynamic typing (JS, Python) | Static typing (TypeScript, Rust, Go, C) |
|---|---|---|
| Types checked at… | **Runtime** — errors appear as the program runs | **Compile/transpile time** — a checking step *before* execution |
| Example | Summing a string and a number fails only when that line actually executes | `number + string` is rejected before the program ever runs |
| Trade-off | Lots of flexibility assigning values — but no guarantee a parameter really has the type you expect | An extra build step — but a whole layer of guarantees for free |

- This pre-execution checking step is why many people are **returning to static typing**: it adds a layer of robustness. And it's the foundation for the technique below — which **only works in statically typed languages**.

### 9. Type-driven design: make invalid states uncompilable

- The leap: use types not just to *describe the shape* of objects, but to **build constraints that make it practically impossible to screw up** — pushing compile-time checking to the max by writing your own rules. **Embed business rules in the type system so wrongly-structured code doesn't even compile.**
- His real example (implemented at one of the companies he works at, just a few months before the video): a **WebSocket** client — a connection that stays open so client and server can send messages bidirectionally (unlike a one-shot HTTP request). You must `connect()` before you can `emit()`.
- **Naive OOP version:** one `WebSocket` class with `connect()` and `emit()`; the first line of `emit` checks "am I connected?" and throws if not. It works, but:
  - You only discover you emitted on a disconnected socket **at runtime** — zero feedback while programming.
  - The only way to detect it is **writing a battery of tests** proving it never happens — i.e., *even more code*.
  - Since the function can throw, you must also write **error-handling code for a case that should never exist**.
- **Type-driven version:** stop thinking of the socket as a simple object and model it as a **state machine** — a graph of state nodes connected by events/transitions:
  - `DisconnectedSocket` — its *only* method is `connect()`, which **returns a `ConnectedSocket`**.
  - `ConnectedSocket` — has `emit(...)` and `disconnect()`; its constructor is hidden, so the **only** way to obtain one is by calling `connect()` on a `DisconnectedSocket` (which acts as a slightly odd factory).
  - Now emitting on an unconnected socket is **impossible to even write** — the compiler/transpiler itself throws the errors if you misuse the socket. This **eliminates both the error handling and the battery of unit tests** that guarded against doing the wrong thing. (More states are possible — e.g., error states for "server doesn't exist"; he keeps the example simple.)
- **Costs and alternatives he discusses:**
  - Slight extra **memory** use — each transition returns a new object. Worth it in his view ("the benefits more than compensate"), though it can become a problem as you keep adding states.
  - The classic **State pattern** mitigates that, but he dislikes it *for this specific use case*: it keeps the state as an **internal attribute** of one parent `WebSocket` object, so the parent must expose all methods and check the state inside them — and you're **back to runtime errors**.
  - With only two states you could just make the **constructor do the connecting** (an unconnected socket simply cannot exist) — simple, but limited: only "constructed/not constructed", and you can no longer operate on the unconnected object at all.
- The mindset shift: program with the type system instead of `if/else` — **type definitions as Lego pieces** you assemble so the rules *must* hold. That's what compile-time checking in statically typed languages exists for.
- His conclusion (the title's claim): **statically typed languages need far fewer tests than dynamically typed ones.** *"If you find yourself writing the same tests you'd write in JavaScript, you're probably not using the type system to its full potential."*

### 10. Tests vs. types — tension or teamwork?

At first glance the videos pull in opposite directions: one says "give your test suite love and care," the other says "you should have fewer tests." Read closely, they converge:

| | Video 1 (testing) | Video 2 (types) |
|---|---|---|
| Skeptical of… | Piles of implementation-coupled **unit tests**, mocks, coverage worship | The **battery of unit tests** (and error handling) guarding states that should never exist |
| Puts confidence in… | **Mock-free integration tests** against real behavior (call the API like a client) | The **compiler** — invalid states can't even be written |
| Class of bug addressed | Wrong *behavior*: wrong product in cart, wrong price charged, broken refactor | Wrong *usage/state*: emit on a disconnected socket, `number + string` |
| When the bug is caught | Test run (runtime) | Compile/transpile time — before the program ever runs |

- **Both reject the same thing:** masses of cheap unit tests that verify the obvious. Video 1 replaces them with fewer, higher-level integration tests; video 2 *deletes* a whole category of them by making the guarded scenario unrepresentable.
- **Neither replaces the other.** Video 2 says *fewer* tests, not zero — types can't see behavior. Video 1's own coverage-trap example proves it: `sum(a, b)` that always returns `3` is **perfectly well-typed** (`(number, number) → number`) and still wrong. Only a (well-designed) test catches that. Conversely, no integration test gives you the compile-time, while-you're-typing guarantee that an invalid state is impossible.
- Net position across both videos: **let the type system eliminate the impossible, then spend your (now smaller) test budget on stable, mock-free, behavior-level tests of what can actually break.**

---

## 🧭 The Big Picture (mental map)

```
            "How do I know my code is correct?"  →  layered defenses
            ────────────────────────────────────────────────────────

  CLASS OF BUG                          CHEAPEST TOOL THAT CATCHES IT
  ────────────                          ─────────────────────────────
  number + string,                      TYPE SYSTEM (static typing)
  wrong shapes/params          ───────▶ caught at COMPILE time, free forever

  illegal states / wrong order          TYPE-DRIVEN DESIGN
  (emit before connect)        ───────▶ state machine in types:
                                        Disconnected ──connect()──▶ Connected
                                        invalid code DOESN'T COMPILE
                                        (deletes tests + error handling)
  ─────────────────────── types can't see below this line ───────────────────────
  wrong BEHAVIOR                        INTEGRATION TESTS, NO MOCKS
  (wrong product in cart,      ───────▶ call the endpoint like a client;
   wrong price; sum() that              real DB via Testcontainers;
   always returns 3 — it                survives internal refactors,
   type-checks fine!)                   alarms when behavior breaks

  tiny pure units              ───────▶ UNIT TESTS (fast & cheap, but coupled
                                        to implementation — keep the useful ones)
  whole user journeys          ───────▶ E2E (browser: Selenium/Cypress) — fewest

  Order of work:  test ERROR CASES first ──▶ happy path last
  Metric warning: 100% coverage + green tests ≠ correct (coverage ≠ goal)
  AI era:         tests = the CONTRACT with the AI — you design, AI may type
```

---

## 💡 Key Takeaways

1. **Test so it doesn't break, not to prove it works** — testing is about reducing uncertainty for *future* changes; test the error cases first, the happy path last.
2. **The most useful tests are mock-free integration tests** (his self-declared controversial take): test through the API like a client, with real services via Testcontainers — refactor-proof, no false security from stale mocks.
3. **Coverage is not a goal** — a 100%-covered, green-tested `sum()` can still always return 3. 70–80% with well-designed tests beats 100% of theater.
4. **Make invalid states uncompilable** — with type-driven design (e.g., `DisconnectedSocket.connect() → ConnectedSocket`), the compiler deletes whole batteries of tests and error handling; if your TypeScript tests look like your JavaScript tests, you're underusing the type system.
5. **Types and tests split the work:** types eliminate the *impossible* at compile time; tests verify the *behavior* types can't see — and in the AI era, that curated test suite is your contract with the machine.

---

*Related summaries: [harness-engineering-video-summary.md](harness-engineering-video-summary.md) (tests as the verification stage of an AI harness — referenced directly in video 1) · [claude-code-sdd-harness-video-summary.md](claude-code-sdd-harness-video-summary.md) (testing inside an agent loop) · [backend-development-concepts-video-summary.md](backend-development-concepts-video-summary.md) (APIs, databases — the things these tests integrate with).*
