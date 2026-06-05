# Harness Engineering — Video Summary & Mini-Lesson

> **Video:** [What is Harness Engineering?](https://www.youtube.com/watch?v=q9Vaoz0hd0U) — BettaTech
> *(Part 1 of the channel's harness series)*

---

## 📋 Brief Summary

The big insight: most of the recent leaps in AI-assisted development came **not from better models but from better environments the models run in** — the *harness*. **Harness Engineering** is the practice of building that environment: the context, tools, memory, and validation wrapped around a model. The video covers the counterintuitive finding that **simpler harnesses work better** (Vercel's d0 case), **context degradation** and external memory, why you must make the AI **prove** its work instead of trusting it, Anthropic's multi-agent research system, the **three pillars** of harness architecture, and a complete walkthrough of a minimal real-world harness (downloadable example repo) built from nothing but markdown files, a JSON task list, and a shell script.

---

## 🎓 Mini-Lesson: The Concepts

### 1. What a harness is

- Metaphor: the model is a **runaway horse** generating thousands of lines of code; the harness is the **reins and saddle** you put on it to control it. (Writing code is getting cheap; *reading* all that generated code is the new bottleneck — controlling the horse is now our job.)
- The harness = everything **around** the model:
  - the **context** you choose to send,
  - the **tools** it may call,
  - a **memory** system (what it's doing, what it did, what it must remember),
  - a **validation** mechanism.
- Big benefit: the harness is model-agnostic — you can **swap the brain** as newer/cheaper models appear.

### 2. Less is more: the Vercel d0 case

- The intuition "give the agent hyper-specialized tools and tons of context" is **wrong** — it's been shown that **the more complex the harness, the worse it performs**.
- **Vercel's d0** (a data/query agent, sibling of v0): they first built lots of specialized tools (SQL helpers, DB connectors) to "guide" the model precisely. Then they asked: *what if we take the training wheels off?*
- They removed ~**80% of the tools** and left simple Unix primitives: `grep`, `cat`, `ls`. Result: **3× faster, 37% fewer tokens (= cheaper)**, and the simple-tool version **won on every request**.

### 3. Context degradation & external memory

- Universal experience: the longer the session, the "dumber" the AI. Degradation appears **well before the window is full** — a GitHub issue cited recommends clearing the context at **~40%**, with degradation starting around **~20%**.
- Therefore: **move information OUT of the context window** into external systems — files or a database. The window should never run full.
- This is why "build me the whole app" / hours-long unsupervised runs produce bad results without a good harness.
- Anthropic's own article on harnesses for long coding sessions recommends e.g. a **JSON file describing the tasks** — structured, outside the context, letting multiple agents check state by reading the file.
- **Subagents** for the same reason: a parent agent with a full context spawns children that get **only the minimal context for their specific task** — less-filled context = better performance.

### 4. Don't trust — verify

- AI is trained to be **plausible**, not correct; AI-generated code can look 100% convincing while quietly failing its goal.
- Rule: the agent must never just *say* something is done — it must **prove it**: automated tests, spinning up a browser (Puppeteer / Chrome DevTools), any check that exploits software's superpower of being **auditable**.
- Emerging pattern: **AI reviewer agents** — adding a verification layer to the harness itself, with instructions on how to validate and review the work.

### 5. Real-world example: Anthropic's multi-agent research system

- From the article *"How we built our multi-agent research system"*: an **orchestrator agent** spawns small **subagents** that do the actual searching and store findings in **memory**; the orchestrator iterates and finally returns the result to the user.
- Pattern: one big agent that *manages*, small agents that *work*.

### 6. The three pillars of harness architecture

1. **The repository IS the system** — stop using AI as a chatbot; the harness lives in the repo's own files and is "vitaminized" by whichever LLM you plug in.
2. **Multi-agent orchestration** — an orchestrator/leader spawns specialized worker agents.
3. **Verification (and self-improvement)** — the harness validates the work; and since the harness is just files in the repo, a misbehaving agent's prompt/rules **can be updated by the harness itself** so next runs improve.

### 7. Walkthrough: a minimal harness (the downloadable example repo)

Nothing magic — start small and understand each piece:

- **`.claude/` agents folder:** three simple markdown agents — *leader*, *implementer*, *reviewer* — each just a name + what to do. Crucial: a shared **protocol** (every agent must read `agents.md` + the architecture/convention docs).
- **`agents.md`:** the **entry point** — first thing loaded into context; the rules every agent follows before starting. Can include a **repo map** (so agents know where to look without reading everything) and norms like *"never mark a task `done` unless the tests ran and pass."*
- **`init.sh`:** a gate script run **before any work**: Python installed? required files present? tests green? → "environment ready, you may work" or "something's broken, **stop**." Its job: decide if the project is in a good enough state for the agent to start.
- **`features.json`:** the task list (the Anthropic-style structured file) — features with descriptions, acceptance criteria, and status (`done` / `pending`). Could be swapped for Linear/Jira via an MCP — the *structure* is what matters.
- **`progress/` folder = the memory system:** the leader explicitly tells subagents to **write their results to files** ("avoid the broken telephone") — current task, per-agent results, a `history.md` changelog. New agents read *progress*, not the whole project. Could be upgraded to SQLite or a remote DB (even shared memory between agents) — same concept.
- **Orchestration rules:** for each pending task the leader decides which agent to spawn (implementer for a task, explorer for research), and always launches the **reviewer** after the implementer. The reviewer approves/rejects against conventions + runs `init.sh`; and being a repo file, it can **modify its own definition** to improve.
- **Hooks:** e.g., on closing Claude Code, run `init.sh` so you never leave the project in a failing state.
- Demo of one prompt ("read agents.md and follow the protocol…") driving the full cycle: init → pick feature → mark in-progress → write progress file → implement tests → verify → mark done → update history → clean current-task file.

### 8. Bonus: good code helps the AI

- A consistent project with clear, **repeatable** structure makes it far easier for the AI to predict how new features should be built. **Good practices don't just help you — they help the AI**: if your project follows them, new generated code likely will too.
- Formula: good practices + programming experience + a good harness = maximum value from AI.

---

## 🧭 The Big Picture (mental map)

```
            MODEL = runaway horse 🐎     HARNESS = reins + saddle
                                │
        ┌───────────┬───────────┼────────────┬─────────────┐
        ▼           ▼           ▼            ▼             ▼
     CONTEXT      TOOLS       MEMORY     VALIDATION   (swap the brain:
    (curated!)  (SIMPLE >   (external    (prove it,    model-agnostic)
                specialized: files/DB,    don't say it:
                Vercel d0:    progress/,  tests, browser,
                grep/cat/ls   task JSON)  reviewer agents)
                = 3× faster,
                −37% tokens)
                                │
       Context degrades EARLY (~20%, clear by ~40%) → externalize state,
       spawn SUBAGENTS with minimal context (no full inheritance)
                                │
              THE THREE PILLARS OF A HARNESS
       1. Repo IS the system (agents.md, init.sh, features.json, progress/)
       2. Multi-agent orchestration (leader → implementer/explorer → reviewer)
       3. Verification & self-improvement (harness = repo files = editable
          by the harness itself)
```

---

## 💡 Key Takeaways

1. **Better environments beat better models** — harness engineering is wrapping context, tools, memory, and validation around a swappable LLM brain.
2. **Simplify ruthlessly:** fewer, simpler tools outperform hyper-specialized ones (Vercel d0: −80% tools → 3× faster, −37% tokens).
3. **Externalize memory** — context degrades from ~20% full; keep state in files/DB and give subagents minimal context.
4. **Never trust "done"** — the agent must prove its work via tests/browser/reviewers; software is auditable, exploit that.
5. **The harness is just repo files** (markdown agents + a gate script + a task JSON + a progress folder) — start minimal, grow as needed, and it can even improve itself.

---

*Related summaries: [ai-development-concepts-video-summary.md](ai-development-concepts-video-summary.md) (the LLM/context/agent fundamentals this builds on) · [claude-code-sdd-harness-video-summary.md](claude-code-sdd-harness-video-summary.md) (part 2: a harness for the SDD workflow) · [uncle-bob-agent-system-video-summary.md](uncle-bob-agent-system-video-summary.md) (another concrete harness: Uncle Bob's flow).*
