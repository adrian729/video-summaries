---
name: video-lesson
description: Summarize a YouTube video (or playlist) into a lesson-style markdown file — brief summary, mini-lesson of the concepts, ASCII mental map, key takeaways. Use when the user shares YouTube links and wants written summaries/lessons instead of watching, or says "do the video summary thing". Supports analyzing first whether a video is worth summarizing.
---

# Video → Lesson Summary

Turn YouTube videos into lesson-style markdown summaries so the user doesn't have to watch them. Established workflow from the `~/projects/todos` sessions (BettaTech series and similar).

## Workflow

### 1. Get metadata & transcript (yt-dlp is installed via homebrew)

```bash
# Title / channel / duration
yt-dlp --skip-download --print "%(title)s | %(channel)s | %(duration_string)s" "URL"

# List a playlist
yt-dlp --flat-playlist --print "%(playlist_index)s | %(title)s | %(id)s" "PLAYLIST_URL"

# Subtitles (auto-generated; videos are often Spanish — try es first, fall back to en)
cd /tmp && yt-dlp --skip-download --write-auto-sub --sub-lang "es,es-orig,en" --sub-format vtt -o "vid_ID" "URL"

# VTT → clean deduplicated transcript
grep -v '^WEBVTT\|^Kind:\|^Language:\|^$\|-->' vid_ID.es.vtt | sed 's/<[^>]*>//g' | awk '!seen[$0]++' > vid_ID_transcript.txt
```

Caveats: auto-subs contain phonetic typos ("Bcken"=backend, "Cloue"/"Cloud Code"=Claude Code, "arnés"=harness, "Gerkin"=Gherkin, mangled English terms) — interpret by context. Requesting `en` subs sometimes 429s; `es` is reliable.

### 2. Analyze before writing (when in doubt, or if the user asks)

Read the transcript and classify:
- **Concept explainer** (evergreen concepts, analogies, no screen dependence) → full lesson. Best case.
- **Workflow/experience video** ("what I learned…", walks through a published workflow) → summarize the workflow + concepts; note that demos lose fidelity.
- **Live-coding/screen-share** (transcript full of "this file here", "if I run this") → recommend skipping or a brief architecture note + repo pointer; the value is on screen, not in audio.
- **Part of a series** → check for a foundation video the others assume; suggest including it. Overlapping sequels get a **slim delta summary** that links to the foundation instead of repeating it.

Always check existing `*-video-summary.md` files in the working directory first — don't re-summarize (the user has asked to double-check this).

### 3. Write the file

- One file per video, in the working directory: `<topic-kebab>-video-summary.md` (e.g. `devops-concepts-video-summary.md`).
- Write in **English**, even if the video is Spanish. Keep the video's analogies and concrete numbers/examples — they're the memorable part.
- Structure (match existing files in the directory):

```markdown
# <Topic> — Video Summary & Mini-Lesson

> **Video:** [Original Title](URL) — Channel
> *("English translation of title")*

---

## 📋 Brief Summary
One paragraph: thesis + what it covers + the core message.

## 🎓 Mini-Lesson: The Concepts
### N. Concept name
Numbered sections per concept; tables for comparisons; keep analogies,
numbers, tool names, and worked examples from the video.

## 🧭 The Big Picture (mental map)
ASCII diagram tying the concepts together in a code block.

## 💡 Key Takeaways
3–5 numbered, punchy takeaways.

---
*Related summaries: [link](other-file.md) — cross-link overlapping summaries both ways.*
```

- Delta summaries: add a `⚠️ This is a delta summary` blockquote pointing to the foundation file, then cover only the new material.

### 4. Update the README index

Whenever you add, remove, or rename summary files, bring the collection's `README.md` back in sync —
invoke the **`video-readme`** skill. It knows the index format (themed sections + tables + suggested
learning paths) and the rules for slotting a new video into an existing section/path or creating a
new one. Run it after the files are written, then give the user a short recap of the files created
and what changed in the index.
