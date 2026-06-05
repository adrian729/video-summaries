# Audio DSP & Modern C++ Engineering Roadmap

**Target Goal:** Move from a general CS/Physics background to a high-performance, real-time Audio DSP Engineer capable of building audio-reactive game systems.

---

## 📅 Phase 1: Foundations of Code & Linear Transforms

**Timeline:** Months 1–2 (Aligned with Coursera Weeks 1–4)
**Objective:** Master C++ basic memory models while implementing the Discrete Fourier Transform (DFT) from scratch.

### 1. Coursera Track (Weeks 1–3)

- **Topics:** Digital Audio Basics, Discrete Fourier Transform (DFT), Short-Time Fourier Transform (STFT).
- **Core Theory:** \* Signal discretization ($x[n] = x(n T_s)$).
  - The Nyquist-Shannon sampling theorem constraint ($f_s > 2 \cdot f_{max}$).
  - The DFT viewed as an orthogonal change of basis projecting time-domain vectors onto complex exponential basis functions ($e^{-j\frac{2\pi}{N}kn}$).

### 2. C++ & Systems Track (From Zero)

- **Primary Resources:** \* **The Cherno (YouTube):** "C++ Series" (Focus on: Compilers, Linkers, Pointers, References, Arrays, and Stack vs. Heap).
  - **LearnCpp.com:** Chapters 1–11 (Syntax, data types, control flow, pointers, and dynamic memory allocation).
- **Audio SWE Concepts:** Interleaved vs. De-interleaved memory layout. (Audio DSP heavily favors de-interleaved structures for CPU cache efficiency).

### 3. Practical Integration Project: The Command-Line DFT

- **Setup:** Set up a modern C++20 toolchain using CMake.
- **Task:** Write a command-line application that uses `dr_wav` (or a similar header-only library) to parse a `.wav` file into a heap-allocated array of `float` elements.
- **Implementation:** Implement a naive $O(N^2)$ DFT using `std::complex<float>` and `std::vector`.
- **Verification:** Export the resulting frequency magnitudes to a `.txt` file and verify that the output perfectly matches your Coursera Python assignment matrices.

---

## 📅 Phase 2: Object Lifetimes & The Time Domain

**Timeline:** Months 3–4 (Aligned with Coursera Weeks 5–7 + Supplemental Math)
**Objective:** Master C++ resource management (RAII) and fill the academic gap by building Time-Domain digital filters.

### 1. Coursera Track (Weeks 4–5)

- **Topics:** Sinusoidal Model, Harmonic Model.

### 2. Supplemental DSP Track (Linear Time-Invariant Systems)

- **Primary Resource:** Julius O. Smith’s _Introduction to Digital Filters_ (Stanford W3K Center, available free online).
- **Core Theory:** _ Discrete convolution sum ($y[n] = (x _ h)[n]$).
  - Discretizing continuous differential equations into **Linear Difference Equations**.
  - The Z-Transform, pole-zero mapping on the complex plane, and filter stability analysis.

### 3. C++ Mastery Track (Intermediate)

- **Primary Resources:**
  - **CppCon (YouTube):** "Back to Basics" playlist (Focus on: _Object Lifetimes, RAII, and Move Semantics_).
  - **Book:** _Effective Modern C++_ by Scott Meyers.
- **Language Concepts:** Resource Acquisition Is Initialization (RAII), copy/move constructors, move semantics (`std::move`), and smart pointers (`std::unique_ptr`, `std::shared_ptr`).

### 4. Practical Integration Project: The Biquad Filter Engine

- **Task:** Design an object-oriented, reusable C++ `BiquadFilter` class.
- **Implementation:** Calculate coefficients for low-pass and high-pass filters using the Bilinear Transform. Implement the 2nd-order difference equation:
  $$y[n] = b_0x[n] + b_1x[n-1] + b_2x[n-2] - a_1y[n-1] - a_2y[n-2]$$
- **State Preservation:** Store historical states ($x[n-1], y[n-1]$, etc.) as private member variables within your class, ensuring that filter execution remains continuous across sequential blocks of audio.

---

## 📅 Phase 3: Real-Time Architecture & Machine Listening

**Timeline:** Months 5–6 (Aligned with Coursera Weeks 8–10 + Framework Shift)
**Objective:** Transition to hard real-time streaming constraints and capture live microphone inputs.

### 1. Coursera Track (Weeks 6–8)

- **Topics:** Sound Transformations, Sound Description, Feature Extraction.
- **Core Theory:** Autocorrelation functions for fundamental frequency ($f_0$) tracking, and Spectral Flux calculation for onset/transient tracking.

### 2. C++ & Systems Track (The Real-Time Boundary)

- **Primary Resources:** \* Technical presentations by Timur Doumler and Fabian Renn-Giles (CppCon / Audio Developer Conference) on real-time thread safety.
- **The Hard Real-Time Constraints:** The OS audio callback thread runs at a critical priority and must execute deterministically ($O(1)$ or bounded $O(N)$). Inside this thread, you **CANNOT**:
  - Allocate memory on the heap (`new`, `malloc`, resizing containers).
  - Lock a standard mutex (`std::mutex`).
  - Invoke File, Disk, or Network I/O (`std::cout`, logging).

### 3. Audio Frameworks

- **Tool:** Download the **JUCE Framework** (JUCE 8/9).
- **Concepts:** Explore the cross-platform hardware abstraction layer and locate the real-time processing loop (`audioDeviceIOCallback`).

### 4. Practical Integration Project: Real-Time Voice/Instrument Tracker

- **Task:** Build a standalone desktop GUI application using JUCE that acts as a real-time listening engine.
- **Implementation:** Integrate an optimized open-source FFT library (e.g., `pffft` or `KissFFT`). Capture live microphone data via JUCE's real-time buffer loop.
- **Algorithms:** Implement a Root Mean Square (RMS) envelope follower for volume tracking and an Autocorrelation/YIN algorithm for pitch tracking.
- **Concurrency:** Pass the extracted analysis data from the real-time audio thread to the main GUI thread safely using atomic types (`std::atomic<float>`).

---

## 📅 Phase 4: Lock-Free Concurrency & Engine Integration

**Timeline:** Months 7–8 (Post-Coursera Specialization)
**Objective:** Master lock-free communication structures and deploy code into professional game engines.

### 1. Advanced C++ Concurrency

- **Concepts:** Atomic operations, explicit hardware memory fences (`std::memory_order_acquire` / `std::memory_order_release`), hardware cache lines, and avoiding false sharing (`alignas`).
- **Data Structures:** Implement a template-based, thread-safe, **Single-Producer Single-Consumer (SPSC) lock-free ring buffer** to move audio blocks or variable data safely across thread boundaries.

### 2. Game Engine Systems Integration

- **Option A: Wwise (Middleware Approach)**
  - Download the Audiokinetic Wwise SDK.
  - Write a custom C++ **Source Plug-in** that hooks into Wwise's buffer processing. Run your pitch/onset detectors on the microphone matrix and feed the results directly into Wwise Game Parameters (RTPCs).
- **Option B: Unreal Engine 5 (Native Approach)**
  - Study the UE5 C++ Audio API.
  - Construct a custom **MetaSounds Node** by inheriting from `TNodeOperator`. Embed your real-time C++ analyzer directly into the node graph so your game blueprints can query instrument notes or handclaps natively.

---

## 📅 Phase 5: Hardware Optimization (The "Hero" Stage)

**Timeline:** Continuous Mastery
**Objective:** Minimize CPU cycles to make your real-time DSP systems commercially viable.

- **SIMD Vectorization:** Leverage CPU vector hardware. Write audio processing loops using hardware intrinsics (SSE/AVX for Intel/AMD, NEON for ARM) or high-level SIMD wrapper libraries to process 4 to 8 floating-point samples simultaneously in a single clock cycle.
- **Assembly Verification:** Use **Compiler Explorer (godbolt.org)** along with tutorials from _C++ Weekly with Jason Turner_ to verify that your critical DSP loops compile into optimized, branchless assembly code.

---

## 🛠 Summary Checklist of Skills

| Domain           | Junior Audio SWE                           | Professional Audio DSP Hero                                      |
| :--------------- | :----------------------------------------- | :--------------------------------------------------------------- |
| **C++ Standard** | Basic C++11, heavy use of standard locks   | Modern C++20/23, lock-free / wait-free atomics                   |
| **DSP Domain**   | Offline file analysis (Python)             | Real-time block processing, vectorization (SIMD)                 |
| **Architecture** | Modifying sound directly in the GUI thread | Decoupled SPSC queue pipelines between audio and game            |
| **Game Audio**   | Triggering static `.wav` files via code    | High-performance analysis nodes driving procedural runtime audio |
