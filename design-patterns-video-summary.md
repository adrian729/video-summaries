# Design Patterns — Video Summary & Mini-Lesson

> **Videos:** [Ya usas estos 5 Patrones de Diseño (aunque no sepas cómo se llaman)](https://www.youtube.com/watch?v=xtaqMX0OvH0) and [7 Patrones de Diseño que todo Programador Debería Conocer](https://www.youtube.com/watch?v=rqOaZf4xMlI) — BettaTech
> *("You already use these 5 Design Patterns (even if you don't know their names)" and "7 Design Patterns Every Programmer Should Know")*

---

## 📋 Brief Summary

Two companion videos that together form **one catalogue of 12 design patterns** — combined here because they share the same thesis and split the material: the "7 patterns" video lays the foundation (the **Gang of Four**, the **creational / structural / behavioral** families) and covers the classics (Factory Method, Strategy, Observer, Decorator, Adapter, Builder, Singleton), while the "5 patterns" video shows patterns you *already* use without naming them (Chain of Responsibility = Express middleware, Composite = Unity hierarchies, Prototype, State, Flyweight) — Strategy appears in both. The shared message: in a multiparadigm world, patterns aren't recipes to memorize — **understand the *problem* each one solves**, know when a typed function replaces a class diagram, combine patterns freely, and never apply one just for the sake of it (that's over-engineering).

---

## 🎓 Mini-Lesson: The Concepts

### 1. Why patterns exist — and the three families

- Patterns are like **small Lego pieces that solve recurring problems**: through programming history, the problems that appear are practically the same regardless of the project.
- In **1994 the Gang of Four** wrote *Design Patterns*, collecting **23 problems and 23 named solutions** — pieces you can pick up and apply as-is.
- They split into **three groups** by the type of problem they solve:

| Family | Solves |
|---|---|
| **Creational** | *How do we create objects?* |
| **Structural** | Relations between objects — how they fit together and communicate |
| **Behavioral** | How to reuse algorithms/behavior transparently |

- Context matters: SOLID and the GoF patterns were born when **OOP was the dominant enterprise paradigm**. Today most languages are **multiparadigm** (OOP + functional + declarative), which leads some people to call patterns unnecessary — both videos strongly disagree: you use many of them daily without knowing their names.

*(Patterns 2–5: creational · 6–9: structural · 10–13: behavioral)*

### 2. Factory Method — create objects without depending on them *(7-patterns video)*

- Problem: SOLID's **D (dependency inversion)** says depend on abstractions — but *somewhere* the real object must be created. You want to keep the famous **`new`** out of your main code.
- Solution: a **factory**. Example — you want to use *vehicles* freely without knowing if they're **cars, motorcycles or trucks**: define an **abstract factory** with a creation function, then via **inheritance** concrete factories (car factory, motorcycle factory, truck factory), each producing one typology.
- Client code asks the factory through the interface; *which concrete factory you hold* decides whether you get a car or a motorcycle — your code never knows how any of them is created.

### 3. Builder — a contractor for complex objects *(7-patterns video)*

- Problem we've **all** suffered: constructors that grow to **5, 10, 20, 30 parameters**, ending in an unreadable `new` full of `true, true, true, "", true, {}, false, false, []` where you have to inspect each argument to know what it means.
- The **builder** object constructs the target **sequentially**: it exposes functions to set one value, set another, compose with another object… and finally a **`.create()` / `.obtain()` / `.build()`** that returns the finished object.
- **Contractor analogy:** when renovating your house you don't show up with the paint and the brush and assemble every piece yourself — you tell a contractor "I want this floor, these walls, this roof", and the contractor hands you back the house.

### 4. Prototype — clone without knowing the internals *(5-patterns video)*

- A name that will surely ring a bell if you've worked with JavaScript.
- Problem: to duplicate an object you'd instantiate a new one and copy every attribute across — but **what if the original has private attributes you can't read?**
- Solution: the object itself exposes a **`clone()` function**, so you can duplicate it **without knowing how it's implemented internally**. A very simple pattern, and more scalable than attribute-by-attribute copying.

### 5. Singleton — the most famous, and the most criticized *(7-patterns video)*

- Probably the most famous pattern, yet one of the most criticized — the presenter himself criticized it in a video of his own a few years back, and it still doesn't fully convince him.
- Singleton tries to solve **two** problems: (1) only **one instance** of a class in the program, and (2) making that instance **globally accessible**.
- The presenter has **no problem with (1)** — e.g., a single database connection. The problem is **(2), global access**: `getInstance()` can be called *anywhere*, which creates two concrete risks:
  - **Testing:** you can't hand the class a different database without some weird hack to change what the singleton returns.
  - **Dependencies become hard to trace:** dependencies normally arrive via the **constructor** or via **setters** — with a singleton, neither; any code anywhere may suddenly depend on the database.
- Recommendation: **limit singleton use** to cases where it truly simplifies your life (it *does* simplify code). The alternative — instantiating the connection outside and injecting it hierarchically into every class — can be too much work; a better option is a **dependency-injection system** (Angular's works very well).

### 6. Adapter — a translation layer between old and new *(7-patterns video)*

- Similar to Decorator, but with a **different intention**.
- The classic situation: integrating a **legacy system** with a new one through a **translation layer** — receiving JSON in the legacy format and converting it to the new JSON, or converting **XML to JSON**.
- The adapter lets you **integrate new things and old things without the risk of modifying either** — you just generate an adapter that transforms the necessary information.

### 7. Decorator — add functionality without modifying the function *(7-patterns video)*

- Sounds like a contradiction — how do you add functionality to something without touching its implementation? — but the decorator pulls it off: it's essentially a **wrapper**.
- Example: to add **logging** to a class that runs database queries, wrap it and put the logging *around* its functions; your code calls the decorated class and gets *log → query → log*.
- Heavily used in **API/web-server definitions**: **Flask** in Python, and TypeScript too — the famous **`@` annotations** above functions are decorators with **syntactic sugar**. E.g., adding a **permission check** on an endpoint without modifying the endpoint's function.

### 8. Composite — Russian dolls / trees of entities *(5-patterns video)*

- For data with a **hierarchical structure**; if you've worked with **Unity**, you've applied it unconsciously.
- Videogame example: a big **enemy that spawns minions**, where killing the parent must also kill the minions because they're linked. The composite lets your code **treat the group (enemy + minions) exactly like a single enemy**.
- Conceptually it's **Russian dolls**: open a doll, find another inside, and so on. Implementation: the enemy class holds children enemies **recursively**, down to an enemy with no children — a **leaf** — so you're building a **tree**. This abstracts your game-management code away from the enemy hierarchies.
- The presenter's university project: an **octopus boss with a flaming arm made of fireballs** — each ball killed you on touch and could be destroyed independently, and you had to destroy them all to kill the octopus. The balls weren't separate entities: the octopus was the **root** holding an array of ball-enemies, instantiated as one hierarchy; processing the octopus **delegated** processing to all its balls.
- Not just games: a **filesystem** is the same — folders containing folders.

### 9. Flyweight — a cache of shared attributes *(5-patterns video)*

- The pattern that took the presenter longest to understand: books explain it with a **text processor's characters, fonts and colors**, which never clicked — what did was **particle systems** in game engines (smoke, sparks).
- A spark system renders **hundreds or thousands of elements**, each with attributes: position **X, Y, Z**, rotation, transparency, **time alive** (older = more transparent, for the fade-out effect) and a **texture** (the little spark drawing). Yet you can have **millions of particles** on screen and the program runs like nothing's happening. Why? Flyweight.
- Key split: some attributes are **unique per particle** (position, rotation, transparency — can't be shared), but the **texture** is another story: there may be only one texture for *all* particles, so holding millions of copies in memory would be a waste.
- Flyweight is **nothing more and nothing less than a cache**: **one instance** of the texture in RAM, and every particle stores a **reference** to it instead of a clone. The presenter only truly got it after using it in a **graphics engine he built at university**.

### 10. Strategy — inject behavior (covered in BOTH videos)

- *7-patterns video:* the behavioral pattern **par excellence** — **inject behavior and change it dynamically**. Videogame example: enemies attack differently with a **sword** vs. a **spear**, and can **drop weapons or pick them up from the ground**. Define an abstract `Weapon` class, inherit `Sword` and `Spear` which implement `attack()`; the enemy holds a `weapon` attribute and **delegates attacking to the weapon** — that's Strategy.
- *Both videos make the same modern caveat:* injecting behavior is no different from **injecting functions**. In multiparadigm languages (JavaScript/TypeScript) where **functions are first-class citizens and can be typed**, you can just pass a typed function instead of the classic Gang of Four class diagram — so Strategy is falling somewhat into disuse. That's not calling it a bad pattern: it's *understanding its use case* well enough to recognize when a simpler solution solves the same problem. In a strictly OOP language, the classic version still applies.

### 11. Observer — notify subscribers of changes *(7-patterns video)*

- Heavily used today — if you've worked with **Angular**, observables are everywhere.
- Purpose: **notification** — when something changes in one object, a set of other objects must be told so they can react.
- **YouTube-channel analogy:** notify all **subscribers** every time a video is uploaded. The channel (the notifying class) keeps a **list/array of subscribers**; they all implement a **common interface** for how they're notified; on upload, the channel **iterates the list and sends the notification**.
- Especially common in **frontend**, where notifying external agents of changes is everyday work — e.g., pressing a button should trigger an action elsewhere on the screen.

### 12. State — a class per state instead of an `if` maze *(5-patterns video)*

- Problem: objects whose **usage changes depending on an internal state**. Example — a **social-network post** that can be **draft, private or public**, with different allowed actions per state.
- The common approach is a pile of `if state == draft … else if state == private …` — you end up with functions that constantly re-check the state to pick a logic branch, and **state transitions you can't control**.
- The State pattern: **one class per state**; processing the post **delegates the logic to the state class**. What you're really building is a **state machine** — a graph navigated by actions.
- Crucially, **the state itself has access to the change-state functionality**: the draft state's own logic can fire the transition to *published* — that's an **edge (action)** of the state machine. Result: a **robust, typed state machine** implemented with classes instead of a tangle of if/elses.

### 13. Chain of Responsibility — you've used it if you've used middleware *(5-patterns video)*

- Definition sounds odd at first: **delegate a process along a chain of handlers**. But if you've done backend with **Express or Flask**, you've used it: **middlewares** — functions that receive the request, process it somehow, and call **`.next()`** to pass the request to the next handler. That *is* Chain of Responsibility in your daily work.
- Same structure for other problems, e.g. a **permission system** where DB access requires a series of requirements: program the requirements as a series of **short-circuits**, passing the process through the different checks instead of dumping all checks into one function.

### 14. The meta-lesson: understand, combine, don't over-engineer *(both videos)*

- **Patterns aren't to be memorized, they're to be understood** — knowing them is useless if you can't apply them at the right moment. Knowing the *problem* each one solves is how you recognize when a pattern is genuinely helping vs. when you're just complicating your code.
- **Patterns aren't used in isolation — combine them.** Examples from the 5-patterns video:
  - **State + Flyweight** in a particle system: give each particle a small state (alive/dying) and act on it, e.g. changing its color.
  - **State + Chain of Responsibility**: give each middleware an active/inactive state — inactive ones just let the request pass through the chain.
- **Decision rule** (7-patterns video's bonus tip): given a problem, ask whether the functional paradigm already gives you a simpler solution (e.g., injecting typed functions instead of Strategy). If not — you're in a strictly OOP language, or you need polymorphism — go to the patterns and find your solution there.
- Only use patterns to **solve problems you've actually run into**; applying patterns for their own sake is the famous **over-engineering**.
- These are 12 of the **23** GoF patterns — the rest follow the same logic: problem first, pattern second.

---

## 🧭 The Big Picture (mental map)

```
            GANG OF FOUR (1994): 23 named problems → 23 named solutions
                       12 covered across the two videos
                                     │
        ┌────────────────────────────┼────────────────────────────┐
        ▼                            ▼                            ▼
   CREATIONAL                   STRUCTURAL                   BEHAVIORAL
  "how do we create         "how objects relate          "how to reuse behavior
      objects?"               and communicate"               transparently"
        │                            │                            │
  • Factory Method           • Adapter                    • Strategy (both videos)
    (no `new` in main          (legacy ⇄ new:               (inject Weapon: sword/spear
     code; vehicle              XML → JSON                   …or just pass a typed
     factories)                 translation layer)           function)
  • Builder                  • Decorator                   • Observer
    (contractor: set           (wrapper: logs               (YouTube channel →
     floor/walls/roof           around queries,              iterate subscriber list)
     → .build())                @decorators in Flask/TS)   • State
  • Prototype                • Composite                     (class per state →
    (.clone() without          (Russian dolls / trees:       typed state machine,
     reading private            octopus boss + fireballs,    not an if/else maze)
     attributes)                folders in folders)        • Chain of Responsibility
  • Singleton ⚠              • Flyweight                     (Express/Flask middleware:
    (1 instance OK;            (cache: 1 texture,            process → .next())
     global getInstance         millions of particles
     hurts testing &            hold a reference)
     dependency tracing
     → prefer DI)

  META: understand the PROBLEM, not the diagram · combine patterns
        (State+Flyweight, State+Chain) · multiparadigm may offer a
        simpler route (functions > Strategy) · no over-engineering
```

---

## 💡 Key Takeaways

1. **You already use these patterns** — Express middleware *is* Chain of Responsibility, Unity hierarchies *are* Composite, `@decorators` *are* Decorator, Angular observables *are* Observer. Naming them lets you reason about them.
2. **Learn the problem, not the diagram** — patterns are understood, not memorized; they only pay off when you can spot the right moment to apply (or skip) them.
3. **Multiparadigm changes the answer:** when functions are first-class and typed, Strategy can be replaced by simply passing a function — same problem, simpler solution.
4. **Singleton: single instance fine, global access risky** — `getInstance()` everywhere hurts testing and hides dependencies; prefer dependency injection and limit singleton to where it truly saves you.
5. **Combine patterns and stop there** — State + Flyweight for particles, State + Chain for toggleable middleware; but applying patterns for their own sake is over-engineering.

---

*Related summaries: [Backend Architecture Patterns](backend-architecture-patterns-video-summary.md) · [testing-and-correctness-video-summary.md](testing-and-correctness-video-summary.md) (why Singleton's global access hurts testability) · [Functional Design Patterns](functional-design-patterns-video-summary.md) (the FP counterpoint — why most of these GoF patterns dissolve into functions, and which patterns FP has instead).*
