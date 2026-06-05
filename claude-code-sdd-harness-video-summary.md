# Adapting Claude Code for Spec-Driven Development — Video Summary (delta)

> **Video:** [Esto es lo que Aprendí Adaptando Claude Code para SDD](https://www.youtube.com/watch?v=ElGlTv2A_bM) — BettaTech
> *("This is what I learned adapting Claude Code for SDD" — part 2 of the harness series)*

> ⚠️ **This is a delta summary.** The video explicitly assumes you've seen the Harness Engineering video — all shared concepts (harness components, external memory, context curation, agents-as-markdown, `init.sh`, orchestrator/implementer/reviewer, downloadable repo) are covered in [harness-engineering-video-summary.md](harness-engineering-video-summary.md). Below is only what's **new** here.

---

## 📋 Brief Summary

Better models don't automatically mean better code — **the harness has more impact than the model**. This video builds a concrete harness implementing the **Spec-Driven Development (SDD)** workflow on top of Claude Code, clarifying the difference between Harness Engineering and SDD, and demonstrating a Kiro-inspired spec pipeline (requirements → design → tasks) with **EARS notation** and an explicit **human-in-the-loop approval gate**.

---

## 🎓 What's New vs. the Harness Engineering Video

### 1. Harness Engineering vs. SDD — the clarification

- **Harness Engineering = the discipline** of automating *any* dev workflow with AI (waterfall, iterative, TDD…) — building agents for each stage.
- **SDD = one specific workflow** you can build a harness for. You could equally build a harness for TDD or agile.
- They're orthogonal: *"we create a harness (using harness engineering) to implement the SDD workflow."*

### 2. What SDD actually is

- Software creation with the **specification in first place** as the **source of truth**. Specs can be user stories, client requirements, etc.
- Taken to the extreme: developers **never touch code** — they work on the spec, and the AI acts as a **"compiler/transpiler"** from spec → code (with the caveat of non-determinism vs. a real compiler).
- The loop: **specify → AI implements → AI validates (testing) → repeat.**

### 3. The SDD agent cast (mapped onto the workflow stages)

- **Spec Author** — writes the specification in the negotiated format.
- **Implementer** — implements *only* from the spec the Spec Author produced.
- **Reviewer** — validates code against the spec, tests, and code style.
- **Leader** (orchestrator) — launches each stage **based on the task's current state**.
- Context handoff discipline: the Implementer never sees the Spec Author's chat history — only the **finished, human-approved spec files**. Minimal context per agent.

### 4. The Kiro-inspired 3-file spec structure

For each task, the Spec Author writes three files under `specs/<task>/`:

1. **`requirements.md`** — *what* (user stories or, here, EARS — see below).
2. **`design.md`** — *how*, technically: which files change, which classes/functions to create, which files must NOT be touched.
3. **`tasks.md`** — the concrete task list the Implementer executes (could instead be pushed to Linear/Jira via MCP).

This pipeline progressively **filters and curates context**, so the Implementer ends up with instructions as precise as "go to file X, line 40, add a function called `validation`."

### 5. EARS notation

- A simple requirements notation: *"WHEN the user runs X command [without `--limit`], THE SYSTEM SHALL print at most five notes in descending order."*
- **The killer property: each EARS requirement translates 1:1 into a test** — unlike a user story, which may need an unpredictable amount of code/tests. The format is defined once in a `specs.md` doc that every spec must follow.

### 6. Human-in-the-loop state machine

The presenter is explicitly *not* a fan of removing the human ("you lose context of your own software — dangerous"). The task states enforce review gates:

```
pending ──(Spec Author writes 3 files)──► spec_ready
   ⏸ HUMAN reviews & approves the spec files
spec_ready ──(human flips state)──► in_progress
   Implementer works · human reviews the TESTS as they're written
in_progress ──(Reviewer validates vs spec + runs init.sh)──► done
   memory → history.md changelog
```

- Two explicit human review points: the **design phase** and the **test-implementation phase**.
- Neat detail: each task carries a **boolean `sdd` flag** — short/trivial tasks skip the whole SDD ceremony (it would be "overkill, annoying").

### 7. Philosophy: build YOUR harness

- The internet is full of **opinionated** tools (someone else's workflow). The real skill — *"increasingly sought after"* — is learning to **build your own**: where to store history, when to commit (e.g., "commit after every task"), when to branch ("new branch per spec"), tickets in Linear vs. a JSON file — all of it is just editing your agents' markdown.
- You can even ask Claude to modify the agents/leader for you.
- The CLI doesn't matter (Claude Code, OpenCode…) — only the folder conventions change; **the workflow organization is what matters**.

---

## 💡 Key Takeaways

1. **The harness impacts code quality more than the model version** — stop waiting for better models, build better environments.
2. **SDD = spec as source of truth, AI as the compiler**; Harness Engineering is the discipline that automates it (or any other workflow).
3. **Curate context through the spec pipeline** (requirements → design → tasks): each agent gets the minimum it needs, never the previous agent's chat.
4. **EARS requirements map 1:1 to tests** — a spec format chosen for verifiability.
5. **Keep the human at the gates** (spec approval, test review) and **skip the ceremony for trivial tasks** — and above all, build your own flow instead of adopting someone else's.

---

*Foundation: [harness-engineering-video-summary.md](harness-engineering-video-summary.md) · A different workflow on the same repo: [uncle-bob-agent-system-video-summary.md](uncle-bob-agent-system-video-summary.md) · Fundamentals: [ai-development-concepts-video-summary.md](ai-development-concepts-video-summary.md).*
