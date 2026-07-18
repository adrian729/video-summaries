# Practical LLM Usage — Video Summary & Mini-Lesson

> **Video:** [How I use LLMs](https://www.youtube.com/watch?v=EWvNQjAaOHw) — Andrej Karpathy

---

## 📋 Brief Summary

A hands-on, practical follow-up to Karpathy's more theoretical *"Deep Dive into LLMs"* video: instead of how these models are built, this one is a guided tour of the 2025 LLM app ecosystem — what settings exist, which tool to reach for and when, and how Karpathy personally works with ChatGPT, Claude, Gemini, Grok and friends day to day. It walks through the mental model of tokens/context/training, then layers on the practical stuff: picking a model and a pricing tier, reasoning ("thinking") models, tool use (search, code execution, deep research, file uploads), multimodality (voice, image, video), and quality-of-life features like memory, custom instructions and custom GPTs.

---

## 🎓 Mini-Lesson: The Concepts

### 1. The 2025 LLM ecosystem

- ChatGPT (OpenAI, launched 2022) is the **"Original Gangster" incumbent** — most popular, most feature-rich, simply because it's been around longest.
- But the ecosystem has since filled out: Gemini (Google), Copilot (Microsoft), Claude (Anthropic), Grok (xAI), DeepSeek (China), Llama-based clones, Perplexity, and more.
- Track relative model quality with public leaderboards: **Chatbot Arena** (crowd-sourced ELO scores) and the **Scale AI leaderboard**.
- Practical takeaway: don't assume ChatGPT is automatically the best tool for a given job — the "council" of models is worth having open in parallel tabs.

### 2. Tokens — what the model actually sees

- A conversation isn't text to the model — it's a **one-dimensional sequence of tokens**, chunks of text drawn from a vocabulary of roughly **200,000** entries.
- Example from the video: the sentence *"What it's like to be a large language model"* is 15 tokens; a model's reply was 19 tokens; the whole exchange was 42 tokens.
- Use OpenAI's **Tiktokenizer** tool to see how any string actually gets chopped up — words don't map 1:1 to tokens.

### 3. Pretraining vs. post-training — where knowledge and persona come from

- **Pretraining**: take all of the internet, chop it into tokens, and compress it into a giant neural net — Karpathy's analogy is a **"zip file"** of the internet (for a ~1 trillion-parameter model, roughly a "1TB zip file").
- **Post-training**: swap the dataset for human-built conversations, which is where the assistant persona comes from — like **"attaching a smiley face"** onto the raw zip file of knowledge.
- Because pretraining is expensive and infrequent, every model has a **knowledge cutoff** — a snapshot date after which it knows nothing unless given tools. Model "knowledge" itself is a **"vague recollection of the internet"**: probabilistic, occasionally stale, good for popular/common facts, unreliable for anything niche, recent, or high-stakes.
- Practical rule of thumb Karpathy gives: *"I'm not guaranteed that this is true, but I think probably this is the kind of thing that ChatGPT would know."* Verify with primary sources before anything that matters.

### 4. The context window — the model's expensive working memory

- The context window (the token sequence you and the model are building together) is the **"working memory"** of the model — and every token in it is a **precious, expensive resource**: more tokens = more distraction risk *and* literally more compute to generate the next token.
- Practical hygiene rules: **start a new chat whenever you switch topics**; don't dump irrelevant information in; drop tokens from the window once they've stopped being useful for the next query.

### 5. Picking a model and pricing tier

- Providers segment by capability and price: e.g. ChatGPT's free tier (GPT-4o mini) vs. Plus ($20/mo, GPT-4o) vs. Pro ($100/mo, unlimited + top models); Claude's Pro plan; Gemini's model selector; Grok 2 vs. Grok 3.
- Bigger/smarter models cost more to run — that cost gets passed to you. Karpathy's approach: experiment with the cheap tier first, and pay for the frontier tier only if the intelligence gap actually matters for your use case (especially if you're using it professionally).
- He personally pays for several providers at once and calls this **his "LLM council"** — asking the same question to ChatGPT, Claude, Gemini, etc. and comparing answers (example: asking multiple models to recommend a city to visit, cross-checking which cities show up in more than one answer).

### 6. Thinking (reasoning) models

- A separate class of models (GPT o1/o1-mini/o1-pro, Grok 3 in "think" mode, DeepSeek-R1) are trained with **reinforcement learning** specifically to do better on hard math/code/reasoning problems, per the DeepSeek paper *"Incentivizing Reasoning Capability in LLMs via Reinforcement Learning."*
- These show their work in a "thinking" trace before answering, and cost more latency for that accuracy boost.
- Karpathy's habit: **default to the fast, non-thinking model**; only switch to a thinking model when a response feels wrong or the problem is clearly hard math/code/logic. This is the single most repeated practical tip in the video.

### 7. Tool use — internet search

- Models can emit a special "search" token that triggers the app to run a real web search and paste the retrieved text into the context window — conceptually just more tokens for the model to condition on.
- Not every app/model has this: ChatGPT has a search button, Claude's knowledge cutoff (April 2024 at time of recording) means it needs it, Gemini 2.0 Flash has it while Gemini 2.0 Pro doesn't. Perplexity was called out as the app that first did this "convincingly."
- Rule of thumb: any question about something recent, time-sensitive, or niche (*"Is the market open today?"*, *"When's the new season of White Lotus out?"*) needs search — otherwise the model is answering from its stale "zip file" memory. Always check the citations an app provides rather than trusting the prose blindly.

### 8. Tool use — deep research

- **Deep Research** (ChatGPT Pro $200/mo tier; also in Perplexity and Grok as "Deep Search") combines search + reasoning over 10+ minutes to produce a long, cited report — Karpathy calls it **"a custom research paper on any topic you'd like."**
- Demoed on things like analyzing a supplement ingredient (C-AKG, from Brian Johnson's longevity mix) and comparing Brave vs. Arc browsers for privacy.
- Important caveat: treat the output as a **first draft, not ground truth** — one demo report had real inaccuracies (missing xAI, wrongly including Hugging Face in a list of LLM labs). Go check the underlying sources yourself.

### 9. Tool use — feeding it documents (and reading books together)

- Uploading a PDF or pasting in text puts the document directly into the context window, so the model can reference it the way a person would reference a specific source rather than guessing from vague pretraining memory.
- Demoed with an Arc Institute genomics paper (summarized by Claude 3.7) and with Adam Smith's *The Wealth of Nations* (1776) — pasting in a chapter and asking for a summary before reading, then asking questions as you go.
- Karpathy's suggestion: **don't read books alone anymore** — this dramatically helps with old texts or unfamiliar fields (biology, economics, Shakespeare), even though the workflow today is still clunky manual copy-paste.

### 10. Tool use — the code interpreter (and why models hallucinate math)

- Simple arithmetic a model does "in its head" (probabilistically) — which is why it can get large multiplications wrong. Give it a real Python interpreter tool and it writes and runs actual code instead of guessing.
- Example: `30 * 9` — fine either way. `12345678901234567890 * 9876543210987654321` — ChatGPT (has Python) gets it right; Grok 3 (no Python at the time) confidently hallucinated a wrong answer.
- Not every model/app has this tool, so check before you trust a "confidently wrong" numeric answer — Claude was noted to use JavaScript instead, Gemini 2.0 Pro did math in its head and hallucinated.

### 11. Building things with an LLM: Advanced Data Analysis, Artifacts, diagrams, Cursor

- **Advanced Data Analysis** (ChatGPT) writes code to plot/extrapolate data — but Karpathy calls it **"a very very junior data analyst"**: in one demo it silently mis-set a variable (assumed a $0.1M valuation instead of $1.7B), skewing an extrapolation by 10x. Always ask it to *"print this variable directly by itself"* to sanity-check intermediate values, and actually read the generated code.
- **Artifacts** (Claude) generates small, self-contained interactive apps (e.g., a flashcard app built from a Wikipedia page) as React components running client-side, no backend.
- Claude can also draw **Mermaid diagrams** to visualize concepts — used to map out a book chapter's argument as a tree, which Karpathy says helps retention a lot for visual thinkers.
- **Cursor** is a dedicated coding app (not a browser chat) that gives an LLM (Claude 3.7 Sonnet under the hood) access to your local files. Karpathy coined **"vibe coding"** for the `Composer`/`Ctrl+I` mode where you hand off control and just describe what you want (e.g., "set up a new React repository," "add a confetti effect when a player wins") rather than editing code by hand.

### 12. Multimodality — voice, image, video

- **Voice** comes in two flavors: "fake audio" (speech-to-text → text tokens → text-to-speech, e.g. mobile ChatGPT's mic icon, or the desktop tool **SuperWhisper** bound to a hotkey) vs. "true audio" (native audio tokens, e.g. ChatGPT's **Advanced Voice Mode** or Grok's voice). Karpathy: roughly half his queries are voice because it's faster than typing — "don't type stuff out, use voice, it works quite well."
- **Image input**: he does it in two steps whenever possible — first ask the model to transcribe an image (e.g. a nutrition label or blood test screenshot) into text so you can verify it read the numbers correctly, *then* ask your real question against the verified transcription.
- Advanced Voice on mobile can also take live camera input (roughly one frame/second, not true video) to identify objects in the real world — useful for casual/non-power users, less so for daily power use.
- **Image generation** (DALL-E 3 in ChatGPT, plus competitors like Ideogram/Midjourney) for things like YouTube thumbnails; **video generation** tools were name-checked as evolving fast but not something Karpathy uses much personally.
- **NotebookLM** (Google) turns uploaded documents into a custom "deep dive" podcast — demoed turning a genomics paper into a ~30-minute two-host podcast for passive listening on a walk or drive, with an interactive mode to ask questions mid-episode.

### 13. Personalization — memory, custom instructions, custom GPTs

- **Memory** (ChatGPT-specific at the time): explicitly say *"please remember this"* and the model stores a durable preference (e.g., "the late 1990s/early 2000s was peak Hollywood") that gets silently prepended to all future conversations. Manage it under settings.
- **Custom instructions** set a standing tone/behavior globally — Karpathy's own example: *"don't be like an HR business partner, just talk to me normally,"* plus a rule to use a specific level of formality when replying in Korean.
- **Custom GPTs** are reusable mini-tools built with a system prompt and a few worked examples (**few-shot prompting**). His demoed set: a Korean vocabulary extractor, a detailed Korean translator that breaks a sentence down piece by piece, and an OCR-plus-translate GPT for subtitle screenshots — built because, in his view, "ChatGPT is just so much better than Google Translate" once specialized this way.

### 14. Everyday-life use-case grab bag

A running theme through the video is that once you internalize the mental model above, an LLM becomes a reach-for-first tool for small everyday tasks: interpreting a blood test screenshot (paste it in piecemeal, treat the output as a first draft to bring to a doctor), analyzing whether toothpaste/supplement ingredients are "essential" vs. filler, explaining a meme's joke, or working through a hard math expression copied out of a paper. The common thread: treat the answer as a fast first draft, not final authority, and verify anything that matters.

---

## 🧭 The Big Picture (mental map)

```
PRETRAINING (internet → "zip file" of knowledge, has a cutoff date)
        │
POST-TRAINING (human conversations → assistant persona/"smiley face")
        │
        ▼
   THE MODEL  ──lives inside──►  CONTEXT WINDOW (expensive working memory)
        │                              new chat per topic; keep it lean
        │
        ├─ no tools → answers from vague pretraining memory (verify!)
        │
        ├─ THINKING MODE (RL-trained) ──► for hard math/code/reasoning
        │
        └─ TOOLS attached:
              ├─ internet search        → fresh/current facts
              ├─ deep research           → long cited first-draft reports
              ├─ file upload             → reason over YOUR documents
              ├─ code interpreter        → real math instead of guessing
              ├─ artifacts/diagrams/IDE  → build things (Claude, Cursor)
              └─ voice/image/video I/O   → multimodal front doors

  + quality-of-life layer: memory, custom instructions, custom GPTs
  + ecosystem layer: many providers/tiers — pick per task, keep a "council"
```

---

## 💡 Key Takeaways

1. **A model is a compressed, dated snapshot of the internet plus a bolted-on persona** — treat its unaided answers as a probabilistic first draft, not ground truth.
2. **The context window is expensive working memory** — start new chats per topic and don't pollute it with irrelevant tokens.
3. **Default to the fast non-thinking model; switch to a "thinking"/reasoning model only for hard math, code, or logic** — this was the single most repeated tip in the video.
4. **Tools turn a text predictor into something trustworthy for specifics**: search for fresh facts, code interpreters for real math, file upload for reasoning over your own documents — and not every app/model has every tool, so know what you're using.
5. **Always scrutinize generated code and cited sources** — Karpathy's own demos show these tools silently getting numbers wrong; the fix is always a cheap verification step (print the variable, check the citation, re-transcribe the image) before trusting the output.

---

*Related summaries: [ai-development-concepts-video-summary.md](ai-development-concepts-video-summary.md) (shares the tokens/context-window fundamentals, from a developer-tooling angle).*
