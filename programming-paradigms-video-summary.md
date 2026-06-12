# Programming Paradigms — Video Summary & Mini-Lesson

> **Video:** [Todos los Paradigmas de Programación, explicados en 14 minutos](https://www.youtube.com/watch?v=GehWsIr6oFo) — BettaTech
> *("All Programming Paradigms, explained in 14 minutes")*

---

## 📋 Brief Summary

Instead of teaching paradigms as **"isolated islands"** — the way most people learn functional, object-oriented, imperative and declarative programming — the video tells them as **one connected story**, a journey starting in the 1930s. The foundational crisis of mathematics leads to Alonzo Church (lambda calculus) and Alan Turing (the Turing machine) answering the same question — *what does it mean to compute?* — in radically different ways, creating the first great fork: **functional vs. imperative**. From the functional branch, AI research on natural language births the **logic paradigm** (Prolog); from the imperative branch, the pain of maintaining ever-larger programs births Alan Kay's **object-oriented programming** (Smalltalk). Framed by **Van Roy's taxonomy**, the core message: paradigms don't appear "just because" — each one is a response to the limits of computing in its era, and understanding *the whys* (not just a language) is what defines your maturity as a programmer.

---

## 🎓 Mini-Lesson: The Concepts

### 1. The 1930s: the crisis that started it all

- The 1930s were a turbulent decade for mathematics — the **crisis of foundations** (*crisis de los fundamentos*). Paradoxes and contradictions had appeared in **set theory**, raising questions that couldn't be answered with the mathematics known at the time.
- Math had advanced very fast but **without logical rigor**: no formal system, no axioms, no solid foundations to grow on robustly. And since set theory is used to some degree *everywhere*, the cracks spread to many other branches.
- Result: an era of deep introspection — lots of analysis and studies trying to figure out **how to rewrite mathematics** to fix these problems.

### 2. Church vs. Turing — "What does it mean to *compute*?"

Two key figures attacked the crisis through **logic**, each in a radically different way, both trying to answer the same question: *what does it mean to calculate?*

| | **Alonzo Church** | **Alan Turing** |
|---|---|---|
| Approach | Purely mathematical | Operational / machine-like — closer to what we'd call a *computer* today |
| Model | **Lambda calculus**: computing = **applying functions and substituting symbols** | **Turing machine**: a machine with an **infinite tape** that stores state, moving the active cell along the tape |
| What it gave us | A theoretical, rigorous way to describe what we now call *algorithms* | An operational definition of *computing* |

- The funny part: working with totally different concepts, **both reached the same conclusion** — *there is no general algorithm that can decide whether a mathematical truth is true or false.* In other words, **there exist mathematical problems no computer can solve**.
- This is the **decision problem** applied to the **halting problem** — one of the great questions of computer science and, as the video puts it, basically the origin of the millennium problem **P versus NP**. The discovery opened a world of opportunities for computer science and programming, plus questions still unresolved today.

### 3. The first fork: functional vs. imperative

These two independent lines of research produced **the first bifurcation in programming paradigms** — the two great paradigm families we have today:

- **Church's school + lambda calculus → functional languages.**
- **Turing's school + the Turing machine → imperative programming.**

Crucially, both are **Turing-complete**: they can solve exactly the same problems, just as Church and Turing had each demonstrated separately. The rest of the video dives into how each branch evolved and which new paradigms were born from them.

### 4. The logic paradigm: Prolog (born from the functional branch)

**Setting:** late 1960s, AI research booming. At the **University of Marseille**, **Alain Colmerauer and Philippe Roussel** are researching **natural language processing (NLP)** — facing a question nobody had faced before: *how do we make a computer understand human language?* (Obvious today with ChatGPT, Claude and neural networks; in the 60s–70s it was brand new, and studied with totally different methodologies.)

- **The core problem:** natural language **is not sequential**, so it's hard to represent with an imperative language. The video's example: *"Juan gave a book to María."* Humans instantly infer the relations — Juan owns/has the book, Juan acts on María, María receives the book from Juan. The relations are obvious, **but there's no sequence of steps you could program in, say, C.**
- The questions Colmerauer's team wanted software to answer: *Who gives the book?* (Juan) — *What does Juan give?* (a book) — *Who receives it?* (María). They needed something that could capture relations and **infer the way humans do**.
- **The missing piece:** in Edinburgh, **Robert Kowalski** was researching exactly this — **SL Resolution** (selective linear resolution), a strategy that **splits programming in two**: on one side the **facts / logic / relations**, on the other the **resolution of the problem** (the algorithm, the way of answering), based on **backtracking** — moving backward and forward to find solutions matching what you're looking for.
- In the summer of **1971** Colmerauer invited Kowalski to Marseille; one year later, in **1972, Prolog is born** — a **logic-paradigm** language. In **Van Roy's taxonomy**, Prolog descends directly from the **functional branch**, sitting in the **logic & relational paradigm**.

Kowalski's separation of logic vs. resolution is crystal clear in Prolog code:

```prolog
% Facts — describe the world
padre(juan, maria).      % Juan is María's father
padre(maria, ana).       % María is Ana's parent (video reuses padre/2)

% Rules — relations / constraints
% X is grandfather of Y IF X is father of Z AND Z is father of Y
abuelo(X, Y) :- padre(X, Z), padre(Z, Y).

% Query — ask, like Colmerauer's NLP team wanted to
?- abuelo(juan, Quien).
% → Quien = ana        ("Whose grandfather is Juan?" → "Ana's")
```

- The logic paradigm took off because it could **express the realities of a messy world** far more naturally than the imperative paradigm. With Prolog and other languages like **CLIPS**, it powered **the AI of that era**: AI focused on solving problems over logical facts — **ontologies and logic solvers**, not neural networks.

### 5. Object-oriented programming: Alan Kay & Smalltalk (born from the imperative branch)

Meanwhile, the **Turing school** kept advancing on its own — and this evolution probably aligns more with what you think of as a programming language today. **Change of continent: the United States.**

- **The problem:** imperative programming was used across tons of industries, but programs were getting ever **longer**, with different needs and modules, becoming **harder and harder to maintain**. Just imagine programming a super-complex system directly in **C** — you'll struggle to keep the code ordered, to keep code **you can reason about**.
- **The atypical engineer:** **Alan Kay** — not just degrees in mathematics / computer science but also in **molecular biology**, which helped him step out of the purely mathematical-theoretical mindset and apply more **human, person-centered** principles. An engineer who cared not only that machines understood each other, but that **we as people understood the machines**.
- In the early 70s Kay joins **Xerox PARC** — the perfect ecosystem to research **without external pressure**: no rush to ship a product, nothing to launch to market, no consumer needs to cover. Just walk in and investigate.
- There, **Smalltalk** is born — the first language where Kay captures his vision of OOP and its pillars: an **expressive** language, one **anyone can understand when reading it**, and above all one capable of **modeling real life**. Smalltalk is where concepts like **classes, objects and messages** appear.

**So Kay and Smalltalk are the precursors of the OOP we find today in Java or TypeScript… or are they?**

- For Kay, languages like **C++ or Java drift very far** from what he considered object-oriented programming. He said on several occasions that **C++ is not an OOP language but a *hybrid* language** — its objects aren't really objects in his philosophy; they're just **data structures with functions glued on**.
- From the Kay archives (1997): *"I actually invented the term object-oriented, and I can tell you that C++ was not what I had in mind."*
- Still, it was Kay's work that took us from imperative programming to languages like **Java** and Smalltalk — and, in the modern era, to today's **multi-paradigm languages**.

### 6. Van Roy's taxonomy — why paradigms exist at all

Telling the history alongside **Van Roy's taxonomy** makes some things crystal clear:

- **Paradigms don't appear "just because."** Each appears to solve **specific problems of its historical context** — e.g., advancing AI research (logic) or improving the expressiveness of programs (OOP). Each one is **a response to the limits of the state of computing in its era**.
- Van Roy defines it well: paradigms are born by **combining fundamental concepts from mathematics, logic and computer science**.
- **Understanding paradigms goes beyond learning a language** — it's about understanding the motivations, the problems they solve, the *whys*. For the presenter, *that* is what defines your **maturity as a programmer**: not just mastering a language, but **learning how to think**.

> The video closes pointing to another video with interviews of working professionals who use less common languages (far from the typical JavaScript or Python), for the practical, real-industry side of these paradigms.

---

## 🧭 The Big Picture (mental map)

```
            1930s — CRISIS OF FOUNDATIONS (paradoxes in set theory)
                  "What does it mean to COMPUTE?"
                              │
            ┌─────────────────┴─────────────────┐
            ▼                                   ▼
     ALONZO CHURCH                         ALAN TURING
     Lambda calculus                       Turing machine
     (apply functions,                     (infinite tape, state,
      substitute symbols)                   moving active cell)
            │     both Turing-complete —        │
            │     same problems solvable;       │
            │     but: undecidability exists    │
            │     (decision/halting problem)    │
            ▼                                   ▼
     FUNCTIONAL paradigm                 IMPERATIVE paradigm
            │                                   │
            │ late 60s–70s: NLP research        │ 70s: huge C programs,
            │ (Colmerauer & Roussel, Marseille  │ unmaintainable code
            │  + Kowalski's SL Resolution)      │ (Alan Kay, Xerox PARC)
            ▼                                   ▼
     LOGIC & RELATIONAL paradigm         OBJECT-ORIENTED paradigm
     Prolog (1972), CLIPS                Smalltalk: classes, objects,
     facts + rules │ resolution          messages — model real life
     (backtracking); era's AI:                  │ (Kay: C++/Java = "hybrid";
     ontologies & logic solvers                 │  data structs + glued functions)
            │                                   │
            └────────────┬──────────────────────┘
                         ▼
          MODERN MULTI-PARADIGM LANGUAGES
   (Van Roy's taxonomy: paradigms = combinations of fundamental
    concepts from math, logic & CS — each born to solve the
    limits of computing in its era)
```

---

## 💡 Key Takeaways

1. **Paradigms are not isolated islands** — they're one family tree, forking from a single 1930s question: *what does it mean to compute?*
2. **Church → functional, Turing → imperative.** Two radically different answers, both Turing-complete — and both proving some problems are unsolvable by any computer.
3. **Every paradigm was born to fix a concrete pain:** logic programming (Prolog) because natural language isn't sequential; OOP (Smalltalk) because giant imperative codebases became impossible to reason about.
4. **OOP ≠ what Alan Kay meant.** He invented the term, and "C++ was not what I had in mind" — for him C++ is a hybrid whose objects are data structures with functions glued on.
5. **Maturity as a programmer = understanding the whys**, not mastering a syntax — learning how to *think*, not just how to write a language.

---

*Related summaries: [ai-development-concepts-video-summary.md](ai-development-concepts-video-summary.md) (today's neural-network AI vs. the logic-solver AI of the Prolog era) · [backend-development-concepts-video-summary.md](backend-development-concepts-video-summary.md) (the concepts-over-tools mindset applied to backend).*
