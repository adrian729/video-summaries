# Low-Latency C++ — Video Summary & Mini-Lesson

> **Video:** [What is Low Latency C++? (Part 1)](https://www.youtube.com/watch?v=EzmNeAhWqVs) · [Part 2](https://www.youtube.com/watch?v=5uIsadq-nyk) — Timur Doumler, CppNow 2023
> *(One 3-hour double-slot talk, split into two videos. English.)*

---

## 📋 Brief Summary

Timur Doumler (JetBrains developer advocate, ex-decade in music-production software, C++ committee member) spent eight years giving audio-specific low-latency talks before noticing something: the techniques audio programmers invented are the *same* techniques used in gaming, high-frequency trading, and embedded systems. This talk is his attempt to zoom out and give a **cross-industry survey** of what "low latency C++" actually means — not audio-specific, but a genuine common discipline. Part 1 covers **efficient programming** (making code fast in general — measuring, avoiding waste, cache behaviour, hardware quirks). Part 2 covers the harder, less-documented discipline of **programming for deterministic execution time** — code that must never, ever run longer than a hard deadline, which rules out allocation, locks, exceptions, and syscalls, and forces you into a world of lock-free/wait-free data sharing. The throughline: low latency isn't one trick, it's a *worst-case-first* mindset that trades average performance for a guaranteed ceiling.

---

## 🎓 Mini-Lesson: The Concepts

### 1. Latency vs. throughput: not the same axis

Performance splits into two orthogonal dials:

- **Throughput/bandwidth** — how much data you process per unit time.
- **Latency** — how long between asking a question and getting the answer.

Picture your program as a pipe: throughput is the pipe's diameter, latency is how long a single drop takes to get through. Different use cases sit at different points on this spectrum, and the position determines your whole toolkit:

| Where you sit | Example | What you optimize |
|---|---|---|
| Far left (latency-only) | High-frequency trading, real-time audio | Never-exceed-this ceiling, throughput is irrelevant |
| Slightly left | Video games | Prefer dropping *work* (fewer monsters rendered) over dropping *frame rate* |
| Middle (both matter) | Google/Facebook-scale web servers | Balance — thousands of requests, each needs a reasonable response time |
| Far right (throughput-only) | Scientific simulation / HPC | Total job finishes fast; an individual time-step being slow doesn't matter |

**Traffic analogies for the spectrum (Doumler's own):**
- **Optimizing throughput** = add more lanes to the junction.
- **Balancing both** = a roundabout (European-style) — no one waits at a red light, and flow improves for everyone.
- **Optimizing latency** = a bus lane. Doesn't change how often buses come, but *that* bus never sits in traffic.
- **Optimizing latency to the extreme** = remove the junction entirely for one lane (or reroute all city traffic around a marathon) — brutal for total system throughput, but the thing you care about never stops.

**Efficiency sits in the middle** and helps *both* axes — it's not a point on the latency/throughput spectrum but a multiplier on it. Analogy: the telegraph optimizes pure latency (tiny bandwidth, blazing speed); a bigger road optimizes pure throughput; but *inventing the wheel* is efficiency — faster **and** more cargo, at the same time. Definition used in the talk: **efficient code does the same task with less work/energy.**

### 2. The hot path and what "real time" actually means

Every one of these industries has a **hot path** — the one piece of code where speed is non-negotiable — and a **deadline**:

| Industry | What triggers the hot path | Typical deadline |
|---|---|---|
| Video games | Render callback (60 Hz) | ~16 ms |
| Real-time audio | Sound-card callback | 1–10 ms (often 2–3 ms) |
| High-frequency trading | Market data arrives | microseconds (and: be faster than competitors, not a fixed number) |
| Embedded | Interrupt / sensor tick | varies hugely by use case |

One microsecond is roughly how long light takes to travel 300 m (the height of the Eiffel Tower) — a reminder of just how little slack "low latency" leaves.

**Real time = latency + a hard deadline where lateness makes the result *incorrect*, even if numerically right.** Three tiers, by consequence of missing the deadline:

- **Hard real-time** — missing it is a total system failure (can be life-threatening: medical, avionics).
- **Firm real-time** — the late result is simply thrown away/worthless (a dropped audio buffer never reaches the speaker).
- **Soft real-time** — a late result is still useful, just degrades in value over time.

**Jitter** has two meanings worth keeping distinct: irregularity of the *trigger* (is the callback itself evenly spaced?) and irregularity of the *response time* (does the same operation sometimes take 10× longer?).

**The deterministic-vs-average trade-off** — Doumler's audience poll crystallizes the whole talk's philosophy: given a choice between (a) usually 1 µs, occasionally 5-10× that, vs. (b) a guaranteed hard ceiling of 1.8 µs but higher *average* cost — low-latency/real-time work (audio, games, safety-critical) wants **(b)**, always. You are explicitly trading away average-case speed for a worst-case guarantee. (Some HFT use cases are the exception — they don't care about *every* trade completing, just about winning the ones that matter, so they tolerate more variance.)

### 3. Measure first — and don't trust your intuition

The most important discipline is also the most boring: **you don't know your latency until you measure it**, and results are routinely counter-intuitive (e.g. `std::minmax` measured *slower* than calling `min` and `max` separately; fewer instructions ≠ faster). Tools: sampling vs. instrumentation profilers, `perf stat`/cachegrind for cache-miss counts, disassembly inspection, and real benchmarks.

**Micro-benchmarks lie easily** — three concrete traps:
1. Your cache is "warm" in the micro-benchmark in ways the real program's cache never is.
2. Your test data's memory layout (e.g. a freshly-allocated `list`) is unrealistically contiguous compared to a long-lived, fragmented heap.
3. Compiling a micro-benchmark with the same optimization flags as production can let the compiler `constexpr`-away or inline things that would survive in a real, multi-translation-unit program — so you end up measuring something that isn't your actual code path at all.

### 4. Efficient programming — avoiding unnecessary work

Split into "widely-applicable good practice" (this section) vs. "hard real-time only" (Part 2). Recurring theme: **know your language, your library, and your optimizer** — the compiler often *can't* do these transforms for you.

- **Unnecessary object creation:** capturing a computed string by value inside a loop's lambda re-creates it every iteration; moving the computation into an **init-capture** does it once. Classic beginner bug: iterating a container `for (auto x : container)` (by value → copies) instead of `for (const auto& x : container)`. Doumler has personally seen removing one such copy give a **10x** speedup in an audio hot path.
- **Library-specific traps:** `std::map`'s element type is `pair<const Key, Value>`, not `pair<Key, Value>` — writing the wrong type triggers a silent implicit conversion (and copy) on every iteration. `clang-tidy` now warns for this specific case, but plenty of similar traps have no linter coverage.
- **Function-call overhead:** forced inlining can give huge wins the compiler misses on its own (real example: `std::pair`'s constructor failing to inline until force-inlined → **5x** speedup) — but overusing it causes code bloat and hurts the instruction cache, so it's a measure-both-ways decision. Virtual calls cost at least two indirections/cache-miss opportunities (vtable jump); prefer `variant`, compile-time polymorphism (CRTP, or C++23's "deducing this" which makes CRTP much easier).
- **Push work to compile time:** `constexpr` everything reasonable, template metaprogramming, C++20 `constexpr` containers for compile-time lookup tables (heavily used in audio DSP).
- **Fast math approximations:** classic example is Quake III's fast inverse-square-root bit-hack (still famous from the '90s); audio uses similar tricks for `exp`/`log` (mapping log-space ↔ linear-space is constant in DSP code). These get you ~0.1% precision loss for a large speedup — but the old-school bit-fiddling versions are **undefined behavior**. C++20 added safe, defined-behavior equivalents (`std::bit_cast` for reinterpreting bit patterns without UB, "implicit-lifetime types," and C++23's explicit "start the lifetime of an object here" facility) so you can get the same performance *without* UB.

### 5. Riding undefined behavior on purpose: `[[assume]]`

UB is normally framed as purely a safety hazard — but it is also **the mechanism the optimizer uses to generate fast code**: if a compiler can assume something never happens, it can delete the instructions that would have handled it.

C++23's `[[assume(expr)]]` (a paper Doumler co-authored, three years in committee) lets you inject an invariant the compiler can't otherwise deduce (e.g. it's guaranteed elsewhere in the code, or comes from a file format you know the shape of) — **the expression is never evaluated**, the compiler just optimizes as if it's always true.

- Toy example: `assume(x >= 0)` before `x / 32` turns a full division into a single bit-shift.
- Real audio example: `assume`ing a buffer's size is a multiple of 32 and contains no NaN/Inf lets the compiler auto-vectorize a clamp loop it otherwise couldn't touch.
- **The catch:** get it wrong and you get instant, silent UB — "so easy to shoot yourself in the foot." Only use it in genuinely hot, measured code, and only after confirming a real speedup (evidence is mixed across real codebases: one report found assumptions made things *slower*, another found *no* statistical difference vs. plain `assert`, a third found a measurable win — it is extremely case-dependent).
- Related C++20 tool: `std::assume_aligned` (same underlying idea, narrower case that can't be expressed as a boolean expression). Proposed-but-not-yet-standard: a no-alias attribute (like C's `restrict`, which is why Fortran can outrun C++ on some numeric code — no aliasing assumptions needed) and C23's `reproducible`/`unsequenced` function attributes (stateless + effect-less + idempotent + call-order-independent), which let the compiler dedupe repeated calls to the same pure function (e.g. collapse four calls to `cos(x)` into one).

### 6. Cache behaviour usually dominates everything else

A cache miss costs **hundreds of cycles** — bigger than almost every other optimization in this talk combined. Rules of thumb:

- Keep data contiguous and cache-line aligned; this is *also* why concurrent atomics on the same cache line cause "false sharing" even when logically unrelated.
- The obvious algorithmic-complexity intuition can be wrong: a **contiguous `vector`** frequently beats a "better Big-O" node-based container (`map`, `list`) purely because it minimizes cache misses — hence C++23's `flat_map`/`flat_set` (map/set interface, vector-backed storage; worse asymptotic complexity, but faster in practice up to surprisingly large N).
- **You can reshape an algorithm's memory-access pattern to be cache-friendly without changing its Big-O.** Binary search's natural access pattern jumps all over memory (bad heat map); permuting the underlying array's layout can make binary search's *access heat map* contiguous instead — same asymptotic complexity, measurably faster in practice.
- **Instruction cache misses matter too** — virtual calls cost two of them; unrelated code changes can shift code alignment and randomly gain/lose an icache line.
- **Deliberately keeping cold data/code warm:** if a piece of data or a hot-path function is rarely touched but must be *fast* when it is, some codebases periodically "poke" that data on a timer (read it, discard the result, just to stop it aging out of cache) or repeatedly execute the hot-path code with dummy input purely to keep it resident in the instruction cache — an HFT trick to guarantee the order-placement path is always warm when the real order finally needs to fire.

### 7. Hardware surprises: pipelines, branches, throttling, SIMD

- **Pipeline hazards:** branch hazards (mispredicted branches stall the pipeline — sorting a vector before counting positive numbers can be *many times faster* than counting the unsorted version, same instructions, because the branch predictor now has a trivial pattern), data hazards (each instruction needs the previous result — e.g. naive string-to-int parsing), and genuine **hardware bugs**: one Stack Overflow case found that changing a loop counter's type from `unsigned` to `size_t` doubled runtime, purely because a specific CPU's `popcount` instruction has an undocumented false register dependency.
- **Branch-prediction attributes are a trap:** `[[likely]]`/`[[unlikely]]` do **not** hint the branch predictor on modern CPUs (no such instruction exists anymore) — they only affect code *layout*, which can help or hurt instruction-cache behavior depending on context. Sometimes evaluating *both* branches of a condition (no short-circuit) is faster than a branch, because it trades a mispredictable branch for pure throughput work.
- **CPU throttling:** an HFT shop found their overclocked CPU thermal-throttled because their code was unusually hard on the branch predictor, generating heat concentrated in one part of the silicon — the fix was restructuring code to spread thermal load more evenly. A mobile-audio equivalent: phone CPUs downclock during idle gaps, so playing a note right after silence gets crackles from a CPU still spinning up — fixed by keeping the CPU "busy" with `pause`/`wfe` instructions (no real work, but enough activity to block the low-power throttle) between notes.
- **SIMD:** auto-vectorization works but is brittle (needs contiguous arrays, simple for-loops, no aliasing, no data-dependent branches) and sometimes needs a manual loop-index shuffle to expose vectorizable independence to the compiler. When it doesn't kick in, explicit SIMD intrinsics/libraries (or even hand SIMD-within-a-register tricks) are the fallback; cross-platform code without a known target CPU can compile multiple SIMD variants and dispatch at runtime (GCC/Clang "function multiversioning" automates this; MSVC requires doing it by hand).

### 8. Programming for deterministic execution time: the forbidden list

This is the pivot into Part 2's much harder discipline. On the hot path you cannot:

- **Allocate or deallocate memory** (non-deterministic runtime).
- **Block the thread** (mutexes: unknown wait time, plus priority inversion if a low-priority thread holds the lock a high-priority thread needs).
- **Do any I/O**, use exceptions, make syscalls, or context-switch user↔kernel mode.
- **Run an unbounded loop** or call an algorithm without a known, tight upper bound.

Everything in the rest of Part 2 is techniques for working *within* those constraints.

### 9. Living without the heap

- **Non-allocating data types:** `array`, `pair`, `tuple`, `optional`, `variant` are all stack-based and safe. Type-erased things (`std::function`, `std::any`) are *always* a potential heap allocation — avoid on the hot path, or use non-standard bounded/in-place equivalents.
- **Which standard algorithms can allocate?** The standard doesn't forbid allocation in most algorithms; you rely on "quality implementation" not to. But a specific few (identifiable by wording like "if additional memory is available") *are documented to possibly allocate scratch buffers* to hit a better complexity bound — avoid those specifically on the hot path.
- **Custom allocators** (monotonic/arena, pool, frame allocators) pre-allocate everything up front; general-purpose fast allocators like `tcmalloc` are explicitly the *wrong* tool here — they optimize average-case throughput, not worst-case latency, and will eventually fall back to `malloc`.
- **Fixed-capacity containers** (`static_vector` — Doumler's opinion: the name is bad, should be "in-place vector" — stack-allocated backing storage, `push_back` past capacity fails instead of reallocating) are often simpler and cheaper than wiring a custom allocator into a standard container.
- **Lambdas do not heap-allocate** by design (the standard specifies their layout); the trap is passing one into `std::function`, which usually does. **Coroutines are the opposite of safe by default** — the compiler-generated coroutine frame is a heap allocation unless the optimizer can prove it never escapes scope (works "surprisingly often" but is never *guaranteed*). Workarounds exist (allocate the coroutine frame ahead of time elsewhere; override the promise type's `operator new`) but are fragile — realistically, most real-time codebases just avoid coroutines on the hot path entirely.

### 10. Lock-free ≠ wait-free — and why it matters

Precise vocabulary the talk insists on:

- **Atomic** — no torn reads, but says nothing about blocking.
- **Lock-free** — no mutex is involved, but the standard-library guarantee is only that *at least one* thread makes progress. That's not good enough for real-time: your *specific* hot-path thread needs a stronger promise.
- **Wait-free** — *every* thread is guaranteed to make progress in bounded time. This is the actual bar for real-time code, and it rules out spinning on an atomic in a CAS loop just as much as it rules out a mutex.

### 11. Passing a stream of data across threads: the ring buffer

For getting data *into or out of* the hot path (MIDI events into an audio callback, log messages out of it, file bytes streamed in), the near-universal solution across every industry surveyed is a **single-producer single-consumer wait-free ring buffer**: one atomic read-position, one atomic write-position, fixed capacity, no reallocation ever. It's short enough to implement in one slide, and it alone solves the majority of real-time data-passing needs — which is part of why the committee is struggling to standardize "a" concurrent queue: there are at least 16 meaningful variations (SPSC/MPSC, overwrite-on-full vs. error-on-full, default-value vs. error on empty) with genuinely different performance profiles, and no single one serves everyone.

### 12. Sharing a single value with the hot path — five algorithms, real trade-offs

The harder problem: not a stream, but threads that need to **agree on the current value** of something (a volume knob, a filter's coefficients). If the value is small enough to literally fit in `std::atomic<T>`, you're done (but always `static_assert` that it's actually lock-free — larger-than-word-size "atomics" silently degrade into a real mutex under the hood, with real platform landmines: on x86, `std::atomic<double-width>` needs `-mcx16` on Clang, GCC often won't emit the instruction at all for ABI/legacy-CPU reasons, and MSVC won't support it on `std::atomic` for ABI-stability reasons — you often have to use `atomic_ref` or a third-party atomics library instead).

Once the value is *bigger* than a hardware atomic can hold, here are the approaches Doumler has actually seen/used, roughly in order of how much they give up:

| Algorithm | Reader | Writer | Readers | Best when |
|---|---|---|---|---|
| Try-lock on a spinlock (with progressive backoff) | Wait-free *only if* reading is allowed to fail | Spins/blocks | Single | You have a graceful fallback ("use the previous value") |
| Spin-on-write (atomic pointer swap w/ null-pointer sentinel) | Wait-free, but 2 atomic exchanges (expensive) | Blocked by readers, heap-allocates | Single | You need every read to succeed, single reader is fine |
| RCU / deferred reclamation | Wait-free, single atomic load (cheap!) | Never blocked, heap-allocates, must defer-delete old value | Multiple | You want the fastest possible read and can afford writer overhead |
| Double buffering (2 slots + status bits) | Blocks, spins on a busy bit | Wait-free | Single | Writer runs continuously with a lot of data (e.g. visualizer feed) |
| Seqlock (sequence counter + atomic bytewise memcpy) | Lock-free but may retry (spins) | Wait-free | Multiple | Writes are rare and data is trivially-copyable & not huge |

Deeper notes worth remembering:
- **RCU's hard part is "deferred reclamation"** — you can't delete the old value the instant you publish a new pointer, because a reader might still be using it, but you also can't leak it forever. Three known solutions: atomic `shared_ptr` (easy API, but Doumler found — after initially thinking otherwise, in his very first conference talk — that it's genuinely unsuitable: it's only *lock-free*, not *wait-free*, has slow CAS loops, and risks the real-time thread being the one that frees memory), hazard pointers (coming in C++26, powerful but complex, aimed at "hundreds of threads" scenarios more than single-hot-path real-time), and RCU proper (also headed for the standard — a proposal exists, though Doumler disagrees with its name, "snapshot_pointer," and some API choices, particularly around *when and on what thread* reclamation happens, which real-time code needs explicit control over).
- **Seqlocks need very careful memory-ordering** — naive sequentially-consistent loads/stores work but are needlessly expensive; a correct fast version uses relaxed atomics plus manually placed acquire/release fences, and — critically — the actual read/write of the shared payload must use an **atomic, race-free bytewise memcpy** (a proposed but not-yet-standard primitive), not a plain memcpy, or the compiler is legally allowed to treat the whole thing as a data race and optimize it away (this has bitten real seqlock implementations that "worked" only by accident on x86). The tolerance for torn reads is exactly the trick: you're allowed to read garbage mid-write, because you detect it and retry — you just can't let the compiler treat that garbage-read as UB.

### 13. I/O, exceptions, and syscalls on the hot path

- **I/O:** never call `std::cout`/blocking I/O directly on the hot thread. Push through the same wait-free SPSC queue to a non-real-time thread that does the actual output. Cross-process needs shared memory; talking to hardware directly needs DMA to bypass the kernel. A concrete audio pattern: sample libraries (e.g. orchestral sample players) can be tens of gigabytes — too big to preload, too slow to stream cold — so the first ~100 ms of every sound is preloaded into RAM (locked there via OS APIs to prevent swapping) and the rest streams in from disk on a background thread the instant a key is pressed, feeding the audio thread through the same wait-free queue.
- **Exceptions are out** — `throw` requires dynamic allocation and has no static time/space bound. Roughly half the C++ community already disables exceptions in some part of their codebase for exactly this reason. Error codes work but are ugly; `std::optional` loses error detail; **`std::expected` (C++23)** is presented as the best available mechanism — throw-like ergonomics, no allocation.
- **Syscalls / context switches** are non-deterministic by nature. On stock consumer OSes the main lever is **thread priority** (audio famously runs "hard real-time" logic on a *non*-real-time OS and just trusts the scheduler — which works reliably if the system isn't overloaded). If you own the hardware you can go further: disable hyperthreading, pin the hot thread to a core, bypass the kernel for networking (common in HFT), or use Linux's newer `io_uring`, which lets you issue syscalls without blocking the caller via a pair of lock-free queues (one for requests, one for results) — a promising direction Doumler flags as not yet explored deeply.

### 14. Bounded algorithms and the random-number trap

Anything you call on the hot path needs a *known* upper bound on execution time — the closing case study is random numbers. `std::rand()` is not just low-quality (often fine for generating audio dither/noise) but has two real disqualifiers: it's **non-portable** (different sequence per OS, breaking reproducible tests) and the standard explicitly allows implementations to introduce a **lock** inside it. The "better" generators (`mt19937` etc.) are only guaranteed *amortized* constant complexity — meaning they occasionally do an expensive internal state reshuffle, which is exactly the kind of rare-but-unbounded cost real-time code cannot tolerate. The fix is dedicated fast/deterministic PRNGs designed for real-time audio/DSP use (referenced: a talk by Ruoho Ruotsalainen on this specific niche).

---

## 🧭 The Big Picture (mental map)

```
                    PERFORMANCE
                   /           \
            THROUGHPUT       LATENCY
           (more lanes)     (bus lane / no junction)
                   \           /
                  EFFICIENCY (the wheel)
             "same work, less energy"
                        |
        ┌───────────────┴────────────────┐
        │                                │
 EFFICIENT PROGRAMMING          DETERMINISTIC EXECUTION TIME
 (helps everyone, Part 1)       (hard real-time, Part 2)
        │                                │
 - measure, don't guess          FORBIDDEN on hot path:
 - avoid unnecessary copies        allocate · block/mutex
 - inline / avoid virtuals         exceptions · syscalls
 - constexpr, fast approximations  unbounded loops
 - [[assume]] (ride UB on purpose)        │
 - cache: data layout,           ┌────────┴─────────┐
   flat_map, cache-friendly      │                  │
   binary search, icache      NO ALLOC           NO BLOCKING
 - hardware: branch predictor,   │                  │
   pipeline hazards, SIMD    static_vector,    wait-free only:
                              custom allocators  - ring buffer (stream)
                                                 - single value:
                                                   try-lock+spin / spin-on-write /
                                                   RCU / double-buffer / seqlock
                                                        │
                                            underlying puzzle: WHO
                                            frees the old value, and WHEN
                                            (deferred reclamation)
```

---

## 💡 Key Takeaways

1. **Latency and throughput are different axes, and "efficiency" is what improves both at once** — know which one your use case actually needs before picking a technique (the roundabout/bus-lane/marathon analogies are the fastest way to internalize this).
2. **Real-time correctness includes *when*, not just *what*** — a numerically correct answer delivered late is a wrong answer. Design for the **worst case**, not the average, even when that means being slower on average (the audience-poll thought experiment is the whole talk in one slide).
3. **Cache misses dwarf almost every other micro-optimization** — a "worse" Big-O contiguous container regularly beats a "better" Big-O node-based one in practice, which is why `flat_map`/`flat_set` exist and why binary search can be reshaped for a friendlier memory-access pattern without changing its complexity class.
4. **On the hot path: no allocation, no blocking, no exceptions, no syscalls, no unbounded loops** — this single constraint list is the organizing principle of the entire second half, and it's *why* lock-free/wait-free programming exists as a discipline: you need the effect of synchronization without ever calling anything that can block or allocate.
5. **"Lock-free" is not "wait-free," and picking the right value-sharing algorithm is a trade-off table, not a single best answer** — try-lock+spinlock, spin-on-write, RCU, double buffering, and seqlock each trade reader cost, writer cost, allocation, and reader-count support differently; the deep, recurring hard problem underneath most of them is *deferred reclamation* — knowing when it's safe to free a value nobody's reading anymore.

---
*Related summary: [Real-Time Audio Programming](realtime-audio-programming-video-summary.md) — Pete Goodliffe's audio-specific "golden rules" talk covers the same terrain (buffer deadlines, why glitches are so audible, don't allocate/lock/syscall on the audio thread) from inside one industry; this talk is the cross-industry generalization of those same rules, with much deeper coverage of the actual lock-free/wait-free algorithms used to satisfy them.*
