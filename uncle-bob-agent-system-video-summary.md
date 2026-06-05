# Uncle Bob's AI Agent System — Video Summary & Workflow Breakdown

> **Video:** [Implemento el sistema de Agentes de Uncle Bob, te lo muestro](https://www.youtube.com/watch?v=PoXC6XcVa1M) — BettaTech
> *("I'm Implementing Uncle Bob's Agent System, Let Me Show You")*

> Builds on the same example repo and concepts as the harness series — see [harness-engineering-video-summary.md](harness-engineering-video-summary.md). This summary focuses on **the workflow itself**; the live demo is best watched (code in the repo's `unclebob-harness` branch, linked in the video description).

---

## 📋 Brief Summary

Robert C. Martin (Uncle Bob, author of *Clean Code*) has been publicly sharing his AI development workflows. This video implements one of them as a multi-agent harness on Claude Code. The flow combines familiar stages (spec → formalization → TDD implementation) with one stage **rarely seen in other AI workflows: mutation testing** — automatically injecting bugs to prove the test suite actually catches them. The presenter reproduces the whole flow with five markdown agents and an 8-minute end-to-end demo on a small Python notes app.

---

## 🎓 The Workflow (stage by stage)

### Uncle Bob's pipeline

```
 hand-written SPEC ──AI──► HARD SPEC ──AI──► GHERKIN ──AI──► TDD ──► JUDGE ──► MUTATION TESTING
        (human)        ⏸ human review    ⏸ human review   (red→green→     (architecture     (inject bugs →
                                                            refactor)       review)           surviving tests?)
                                                                  ▲────────── handoff if gaps found ──────────┘
```

1. **Spec (by hand):** Uncle Bob writes the initial specification **mostly manually**.
2. **Hard spec (AI):** an agent *expands* it — many more cases and problem scenarios defined. ⏸ **Human reviews** (this is not a 100% automated flow).
3. **Gherkin (AI):** the hard spec is formalized in **Gherkin** — `Feature` + scenarios, each with an ID (S1, S2…) and **Given / When / Then / And** structure. A standardized, formal requirements format (one of many — cf. EARS in the SDD video). ⏸ **Human reviews** again.
4. **Implementation via TDD (AI):** no code straight away — tests are written **directly from the Gherkin** (each scenario is already formally defined). Classic TDD: red → green → refactor.
5. **Architecture / "Judge" (AI):** Uncle Bob calls it the architecture stage; the presenter frames it as an extreme code review.
6. **Mutation testing (AI):** the star of the show — see below.

### Mutation testing 🧬 (the rarely-seen stage)

- **The intuition you already have:** when all tests are green, you've probably at some point *flipped a condition or deleted a line* just to check that the tests actually fail. Mutation testing automates exactly that.
- **Mechanism:** a script applies small **mutations** to the source code — `>=` → `>`, `==` → `!=`, `and` → `or`, `True` → `False` — then runs the suite.
- **Survivors = a problem:** if tests still pass after a mutation, your suite has a coverage gap — a missing test for that exact case → back to writing tests.
- In the implementation it's a deliberately simple `tools/mutate.py` script driven by a *Mutation Tester* agent. The presenter's verdict: automating this with AI is *"really powerful"* — worth adopting **whatever workflow you use**.

---

## 🤖 The Agent Implementation (five markdown agents)

| Agent | Responsibility |
|---|---|
| **Spec Partner** | Doesn't write the spec from zero — **asks questions, converses, debates** with you to close the hard spec (mirrors Uncle Bob writing specs by hand) |
| **Gherkin Author** | Converts a hard-spec section into an **executable Gherkin contract** in the specs folder; then sets the task state to `spec` |
| **TDD Craftsman** | Implements following the **three laws of TDD**: ① no production code except to pass a failing test, ② no more tests than necessary, ③ no more production code than necessary to pass. Tests one by one (reviewable), logging each red→green cycle to files |
| **Judge** | Sanity checks: every Gherkin scenario has ≥1 test, refactor cycles were followed — reads the TDD log files to verify the actual red/green history |
| **Mutation Tester** | Runs `mutate.py` across files, reports survivors — and does **not** fix anything itself: it does a **handoff** to the TDD Craftsman ("these mutations survived, add the missing tests") |
| *(+ Craftsman Lead)* | The parent/orchestrator defining the order, the gates, and task decomposition |

**Concepts worth stealing** (independent of this exact flow):

- **Handoff:** an agent finishes, writes its result to a file, and passes the task to another agent with a determined context — agent A's artifacts are agent B's input.
- **File-based memory between stages:** the TDD log lets the Judge *verify the process happened* (red → green per scenario, 9 traceable cycles in the demo), not just the end state. Could be markdown or an external memory system (e.g., engram).
- **Context economy:** many small agents each holding only their stage's context, instead of one agent that did everything with a saturated window — "optimizing token usage across a flow this big."

---

## 🧪 The Demo (in brief)

- Feature: a `--since` date-filter command for a small Python notes CLI (same repo as the SDD video).
- One prompt ("implement feature cli-since") drives: init checks → Spec Partner asks **design questions** ("validate real dates or just the pattern?", "day-level or exact timestamp?") → Gherkin with 9 scenarios → human approval gate → TDD cycle by cycle → Judge approves → Mutation Tester finds 4 survivors, all out of the feature's scope → handoff confirms OK → history updated, session closed. **~8 minutes** total on this small project.
- Presenter's honest caveats:
  - This flow suits developers who **review as the AI goes** (he reviews every test) — it's *not* a "leave it running 20 hours" flow.
  - ⚠️ **Token cost:** multi-agent flows reduce per-agent context but multiply total AI work — "they can be a token eater."

---

## 💡 Key Takeaways

1. **Mutation testing is the standout idea:** automatically inject bugs (`>=`→`>`, `==`→`!=`…) — any test that survives reveals a coverage gap. Powerful to automate regardless of your workflow.
2. **Formal spec formats (Gherkin) make implementation mechanical:** each Given/When/Then scenario translates directly into a test.
3. **Handoff + file-based memory:** agents communicate through artifacts, letting later agents *audit the process* (the Judge reads the red→green log), not just the result.
4. **Humans gate the spec stages** — Uncle Bob writes specs by hand and reviews both formalization steps; the value is conversation (Spec Partner asks, doesn't assume).
5. **Mind the token bill** — many specialized agents = lean contexts but lots of total inference.

---

*Related: [harness-engineering-video-summary.md](harness-engineering-video-summary.md) (the foundations: agents-as-markdown, init gate, memory files) · [claude-code-sdd-harness-video-summary.md](claude-code-sdd-harness-video-summary.md) (the SDD flow on the same repo — compare EARS vs Gherkin, Leader vs Craftsman Lead).*
