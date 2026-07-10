# Real-Time Audio Programming — Video Summary & Mini-Lesson

> **Video:** [The Golden Rules of Audio Programming — Pete Goodliffe — ADC16](https://www.youtube.com/watch?v=SJXGSJ6Zoro) — ADC · Audio Developer Conference
> *(English talk. Pete Goodliffe heads software at inMusic — Akai, Alesis, M-Audio — and writes the long-running "Becoming a Better Programmer" column.)*

---

## 📋 Brief Summary

A 50-minute conference talk on the hard constraints of **real-time audio code** — the audio render thread that must hand the sound card a fresh buffer of samples on a relentless clock, or the user hears a click. Goodliffe frames the field as a handful of "golden rules" passed down between audio devs, then spends the talk doing the thing the title promises: stating each rule and asking *how do we break it?* The honest answer is that you mostly **don't break them — you design around them**. The meta-rule, repeated as a joke and as the real thesis, is *"the golden rule is there are no golden rules"*: learn them like a pro (so you understand the machine, latency, jitter, data tearing, and float vs. fixed point) so you can bend them like an artist when your specific platform and problem allow. The whole thing is essentially a catalogue of war wounds from shipping audio products — every rule is a bug he has watched destroy someone's audio.

---

## 🎓 Mini-Lesson: The Concepts

### 0. What "real-time audio" means (the assumptions)

The rules apply to a specific world; if your context is different, they may not bind you:

- **Real-time performance matters:** a DAW, a synth (standalone or VST), a DJ app, an insert effect. The defining trait is that audio must be produced **the instant it's needed**, as fast as possible.
- A media player like iTunes is *not* this — you can buffer up huge amounts of audio ahead of time; the constraints are completely different.
- You're in a **C / C++–like environment**, on **desktop or embedded** hardware (most rules apply to both).

### 1. The landscape: the machine your code lives in

Before any micro-optimization, look at the system from the highest level. Two things dominate: **the CPU/execution environment** and **the buffer size**.

- **CPU clock speed has flatlined.** Moore's Law (transistors per die keep doubling) is *still* roughly holding — but the extra transistors now go into **more cores**, not faster clocks. *"The free lunch is over"* (Herb Sutter). Single-core machines basically don't exist anymore — not on desktop, not in phones/tablets, not even in many microcontrollers (multi-core ARM, xmos).
- **Multicore lets you do more — but only if you do the right things.** It's a happy coincidence: audio code is *inherently* multi-threaded anyway (keep the UI off the audio thread), so multiple cores map naturally onto designs you already want. But it is **not** a license to be sloppy. *"I still threaten to cut off the hands of anyone on my team who writes the word `thread`."* Threads are pernicious; wield them with respect.
- **Fast CPUs are not an excuse to stop optimizing.** You'll want to run *5 instances* of your plugin in one DAW, not one. You'll keep adding features (deeper modulation, more voices). And the day you **port to mobile**, every assumption about that lovely desktop processor and its memory bandwidth falls on the floor.
- **Memory bandwidth and cache topology matter too**, especially on embedded targets — the same code performs very differently across platforms.

### 2. Buffer size, latency, and the "honk button"

Time marches on relentlessly; every period you must produce one audio packet (say 256 samples). This sets up the core tension:

- **Latency:** the user hits the "honk" button expecting an *immediate* honk — but the current buffer is already in flight, so they wait up to a full buffer (256 samples — an age in CPU terms) before they hear it. **Smaller buffers = more responsive.**
- **But smaller buffers are harder on the CPU:** less opportunity to exploit cache effects, loop unrolling, and algorithmic batching, even though you're doing the same total work. So the goal is code that stays performant *with small buffers*.
- **You cannot cheat by processing input more often inside the buffer.** Processing the input 4× within one buffer doesn't remove the latency — the output buffer still has to drain. You get "h-onk" instead of a clean "honk", but you haven't solved it.

**How fast is fast enough?** (his cited research)

| Domain | Threshold perceived as "instantaneous" | Source |
|---|---|---|
| UI response | ~**100 ms** (1 s = tolerable; 10 s = lost their attention) | 1968 study |
| Sharp audio (e.g. drum pad hit) | **1–5 ms** — this is the real-time target | 1949 study |
| Complex sound playback | up to ~40 ms (iTunes-style, not us) | 1949 study |

Audio has **tighter** constraints than graphics, and a single audio glitch is far more noticeable than a dropped frame. *One* click can kill a DJ's live set. That's the bar.

> ⚠️ **The fundamental rule (stated slightly garbled in the talk):** your audio render must **complete within** the buffer period. If the render thread "takes a nap" and overruns even once, audio doesn't reach the sound card in time → dropout → pop/click, and then every following buffer is scrambling to catch up.

---

### Rule 1 — Keep within your CPU budget

This is the rule you genuinely **cannot break**. The render must finish inside its time slice. The good news: the budget is *huge*.

- At 44.1 kHz, a **64-sample** buffer = only **1.45 ms** of audio to generate; 128 samples = 2.9 ms; 256 = ~5.8 ms. All very achievable.
- On a pedestrian 2.4 GHz i5 that's *millions* of instructions per buffer — and you have ~4 cores. *"We are spoiled."* (*"I remember when I was a lad, we had 16 instructions to rub together…"*)
- Multicore means UI work and audio work needn't contend for one core — **as long as the UI never blocks, stalls, or interacts badly with the audio thread.** (The rest of the talk is mostly ways we get this wrong.)
- Always keep the **deployment target** in mind. Whatever you assume about the platform today, *"I bet you in 10 years it'll be wrong. Sucks to be you."*

**How to "break" it (= just design well):** don't do slow work on the audio thread. Push coefficient calculation elsewhere; punt non-real-time track rendering off; never do audio visualization/analysis on the audio thread; **amortize** intensive work across render blocks. Smear the work around the system so the audio thread only ever shuffles numbers.

---

### Rule 2 — If you don't know how long it takes, don't do it (on the audio thread)

The shiniest golden rule, and most of the talk. This is about **jitter**, not just latency.

- **Jitter is the killer.** A 3 ms budget where you average 2 ms is lovely (time left over for "lemoncello on the veranda"). But if *every now and again* a render takes 4 ms, that's **unacceptable** — it must *always* be under budget. Occasional jitter → an occasional click → game over. And it's nearly impossible to find with a profiler.
- **Big-O is the *average* case.** Most algorithms and library functions are optimized for the average. On the audio thread you must reason about the **worst** case.
- **Instrument the audio thread (at least in debug) from day one.** Time each pipeline stage. **Count underruns** — even the ones you didn't hear, because the same underrun could destroy someone else's speakers. Treat any underrun as your most important bug.

The forbidden list — anything that runs in *unbounded* or *unknown* time:

| Don't | Why | How to break it |
|---|---|---|
| **Append/write/remove on a `std::vector`** | Vectors are perfect on the audio thread for *reading* (contiguous, cache-friendly — use them everywhere). But a one-in-a-million **reallocation** triggers a hidden memory allocation → untraceable stall. Reading = fine; mutating = never. | Pre-size it; never grow/shrink it on the audio thread. |
| **Take a lock / mutex** | Not deadlock (that's a separate disaster) — **priority inversion**: if the UI thread holds the lock, your audio thread is wedged until the UI finishes "printing a spinning teapot." "Used responsibly" mutex sections quietly grow over two years until they cause dropouts everywhere. | Lock-free / wait-free data structures + **atomic** data. But verify: some "lock-free" library primitives actually lock (he was bitten by one as a RAS plugin on win32). Trust no one; measure. |
| **Allocate memory** (the classic everyone agrees with and everyone violates) | Allocators (a) take locks and (b) don't run in fixed time — fragmentation/table rebuilds run in unbounded time. And things you call allocate behind your back. | Custom allocator you trust (boost memory pool, dlmalloc); or **pre-allocate everything up front** (worst case); or allocate on the UI thread and pass memory down safely. |
| **Log** (`printf`/`NSLog`) | Disastrously slow — and the temptation when debugging glitches is to add *more* logging, making it worse. | A trusted logger: drop data into a circular buffer, do the `printf` on another thread. |
| **Any OS call** | The OS may take a lock for all sorts of functionality. | Only safe on a real-time OS that *guarantees* syscall timing (most of us aren't on one, except some embedded). |
| **Disk access** | An `fread` on the audio thread is "a disaster waiting to happen" — unknown seek times, reads likely longer than a buffer period, and the filesystem takes locks. *"That's not a streaming engine, that's a disaster."* | Read on a separate thread; hand audio buffers across to the audio thread carefully. |
| **Network activity** | Same family of problems. | Off-thread. |
| **Untrusted third-party / framework calls** | Anything you call could allocate, lock, or call yet more code. Some libs advertised as "thread-safe *and* performant" are performant *because* they're not actually thread-safe — *"I didn't lock, but I fell apart."* | Inspect everything; measure, measure, measure. Only call code that ships a real-time guarantee. |

**A sneaky one — smart pointers / RAII:** great in general, but if you release the **last `shared_ptr`** to an object *on the audio thread*, you deallocate *on the audio thread* without noticing — a hidden latency bug. Use a smarter smart pointer that defers the destruction to another queue. (Same caution for interpreted languages: fine *if* guaranteed within budget, but most carry garbage-collection / ref-counting baggage.)

> The recurring theme: **almost all of these are baked into the design.** You can't retrofit a good threading model or remove all audio-thread allocation once you have six million lines of code. *"If you're four years into a project and haven't done it right, you're doomed."*

---

### Rule 3 — Respect threads

- **Don't presume the audio thread *is* "the audio thread."** He wrote careful lock-free structures assuming the audio thread is always the high-priority, single-producer/single-consumer realtime thread — then a host (Logic) pumped him for audio *on the UI thread* and destroyed the assumption.
- **Data tearing.** Pushing coefficient calculation off the audio thread is good — but if the audio thread reads a coefficient set *mid-update*, it runs a filter in a half-written state → instability, pops, clicks. The fix is **not** a mutex: keep **two coefficient sets**, let the audio thread read via a pointer to the *current* one, update the *other* set in the background, then **switch the pointer atomically**.
- **Know which types are tear-free.** On most major platforms, `int`, `bool`, `float`, and **pointers** are atomic (tear-free) — which is exactly why the pointer-swap trick is safe. `long` and `double` *may* tear depending on platform (a 64-bit value on a 32-bit CPU is written in two halves — a reader can catch it mid-write).
- **Decide where your data lives — early.** The right model: **all the data lives on the UI thread; the audio thread holds a renderable *copy*.** If the audio thread tries to *edit* the master data, you end up mutexing everything. Real-time automation (changes driven *on* the audio thread that must flow back to the UI-side model) is a genuine design problem — solve it at the start, not bolted on later.

---

### Rule 4 — Choose the right data representation

- Most people process audio in **`float`** because that's what the frameworks hand them. Fine — **but if you do audio DSP and don't understand how floats work, stop and learn IEEE 754 first.**
- **Understand fixed-point too** (he is *not* saying rewrite your float path). Fixed-point is ugly and forces you to be explicit about underflow/overflow — which is the point: it makes you reason about where precision and range fail, and it sidesteps float's nastier surprises.
- **IEEE 754 is brilliant but has invisible failure modes:** precision loss at very large magnitudes, and **denormals/subnormals** near zero (which can even tank performance — there are tricks like injecting tiny noise to avoid them). These failures aren't visible in the source; in fixed-point, overflow/underflow is right there in front of you.
- **`double` doesn't "fix" anything** — it just moves the resolution limit elsewhere and can reintroduce **tearing** in a 32-bit world.
- **Don't reach for a C++ fixed-point library** — they tend to renormalize after *every* operation, losing the efficiency of hand-rolled fixed-point (where you do several ops, then renormalize once, later in the chain).

---

## 🧭 The Big Picture (mental map)

```
        REAL-TIME AUDIO = produce a buffer every period, ON THE CLOCK, or → CLICK
        ───────────────────────────────────────────────────────────────────────

   time ──▶ │period│period│period│period│ ...   each = 1.45 ms (64 smp @ 44.1k)
                         ▲
            render must FINISH inside its period.  overrun ONCE → pop, then cascade.

   THE ENEMY IS JITTER, not the average:
        budget 3ms, avg 2ms  ✅ lovely        one render at 4ms  ❌  → a click → game over
        (Big-O is the AVERAGE case; on this thread you live or die by the WORST case)

   ┌─────────────────────────────  THE AUDIO THREAD  ─────────────────────────────┐
   │  ALLOWED:  shuffle numbers. read from pre-sized vectors. atomic loads.        │
   │  BANNED (unbounded/unknown time):                                             │
   │     ✗ malloc / new / vector grow      ✗ locks (→ priority inversion)          │
   │     ✗ logging / printf                ✗ OS calls   ✗ disk   ✗ network         │
   │     ✗ dropping the last shared_ptr    ✗ untrusted 3rd-party calls             │
   └───────────────────────────────────────────────────────────────────────────────┘
        do the slow stuff on ──▶  THE UI THREAD  (owns the master data; allocates;
        and hand results down                     calc coefficients; reads disk)
        via lock-free queues / atomic pointer swap / double-buffered coefficients

   FOUR GOLDEN RULES                          DATA:  master lives on UI thread,
     1. keep within CPU budget                       audio thread gets a COPY
     2. unknown duration → not on this thread  TYPES: int/bool/float/ptr = tear-free
     3. respect threads (tearing, ownership)          long/double = maybe not
     4. right representation (know float AND fixed)

        META-RULE:  "the golden rule is there are no golden rules"
        learn them like a pro  →  so you can break them like an artist
```

---

## 💡 Key Takeaways

1. **Jitter, not average speed, is what kills audio.** A render that *occasionally* overruns produces a click — and one click can ruin a performance. Budget for the **worst case**; remember Big-O describes the *average*. Instrument the audio thread and count even the underruns you didn't hear.
2. **On the audio thread, do nothing whose duration you can't bound.** No allocation, no locks (priority inversion, not deadlock, is the real danger), no logging, no OS/disk/network calls, no growing a `std::vector`, no dropping the last `shared_ptr`. When in doubt, **trust no one and measure.**
3. **Move the slow work to the UI thread and hand results across safely** — lock-free/wait-free queues, atomic data, and the double-buffer-plus-atomic-pointer-swap trick for things like filter coefficients. The master data lives on the UI thread; the audio thread renders a *copy*.
4. **Know which types tear.** `int`, `bool`, `float`, and pointers are atomic on most platforms (so the pointer-swap is safe); `long`/`double` can tear on a 32-bit CPU. And understand **both** float (IEEE 754, with its denormals and precision cliffs) **and** fixed-point — even if you ship in float.
5. **These constraints are architectural — bake them in from day one.** You cannot retrofit a clean threading model or de-allocate the audio thread after six million lines. Learn the rules so well that you know exactly when and how it's safe to bend them.

---

*Related summaries: [data-structures-video-summary.md](data-structures-video-summary.md) — why `std::vector` reallocation and worst-case complexity matter; [testing-and-correctness-video-summary.md](testing-and-correctness-video-summary.md) — instrumentation and measuring as a discipline; [low-latency-cpp-video-summary.md](low-latency-cpp-video-summary.md) — the cross-industry generalization of these same rules, with the actual lock-free/wait-free algorithms (ring buffers, seqlocks, RCU) that satisfy them.*
