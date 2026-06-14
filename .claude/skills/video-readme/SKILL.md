---
name: video-readme
description: Build or update the README.md index for a folder of video-summary lessons (the `~/projects/todos` collection or any directory of `*-video-summary.md` files). Use right after video summaries are added, removed, or renamed — e.g. immediately after the video-lesson skill — or whenever asked to update/rebuild the video README or index. Sorts summaries into themed sections (tables) plus suggested learning paths, creating new sections/paths or folding entries into existing ones as the collection grows.
---

# Video README Index

Keep a browsable `README.md` index in sync with a folder of `*-video-summary.md` lessons. This skill
describes **how** to build the index, not a fixed list of sections — so it adapts as new (possibly
unrelated) videos are added: it folds an entry into an existing section/path when it fits, and spins
up a new section/path when it doesn't.

Run this **whenever the set of summary files changes** (after the `video-lesson` skill writes/removes
files), or on demand to rebuild. The README is regenerated from the files, so it's safe to re-run.

## What the index is for

A reader lands on `README.md` and can (a) see every lesson grouped by theme, (b) jump to the source
video, and (c) follow a suggested reading order. The summary files themselves are the source of
truth; the README is a derived catalog.

## Step 1 — Gather data from the files

Work in the collection directory. Separate **video summaries** (files containing a `> **Video(s):**`
blockquote) from **other docs** (e.g. original roadmaps with no video).

```bash
# Titles
for f in *-video-summary.md; do echo "$f"; grep -m1 '^# ' "$f"; done
# Source video line(s) with URLs
for f in *-video-summary.md; do echo "=== $f ==="; grep -m1 '^> \*\*Video' "$f"; done
```

For each file extract: the **H1 title**, the **video URL(s)** from the `> **Video(s):**` line, and a
**one-line "what you'll learn"** condensed from that file's `## 📋 Brief Summary`. Note any
`⚠️ delta summary` flag. (Channel/source isn't shown in the index — only the title, ▶ link, and
filename — so don't add a channel column.)

## Step 2 — Read the existing README (if any) and preserve its shape

If `README.md` exists, keep its current sections, ordering, and learning paths; only **add, update,
or rebalance** per the rules below. Don't reshuffle a stable structure just to re-derive it. If there
is no README, build one fresh from the skeleton.

## Step 3 — The skeleton

```markdown
# 🎥 <Collection title>

<One-line intro: what the collection is + the shared summary shape
(📋 Brief Summary → 🎓 Mini-Lesson → 🧭 Mental Map → 💡 Key Takeaways) +
"open the file to read, click ▶ to watch the source video.">

---

## 🗺️ Suggested learning paths
<ordered filename chains, one per thread — see rules>

---

## <emoji> <Theme name>
| File | What you'll learn |
|---|---|
| **<Human Title>** ([▶](URL))<br>`<filename>.md` | <one scannable line> |

## <emoji> <Next theme>
...

---

## 📌 Also in this repo
| File | What it is |
|---|---|
| **<Title>**<br>`<filename>.md` | <non-video doc — short description> |
```

### Row format (do this consistently)

- **File cell:** bold human title, then the video link(s) in parens, then `<br>` and the **filename in
  backticks**. The filename must appear as visible text — `md → md` links don't navigate in the
  user's local viewer, so the filename is how they find the file. Keep the ▶ links (those work).
- **Video links:** single video → `([▶](URL))`; multiple videos → `([▶ 1](URL1) · [▶ 2](URL2))`.
  Pull the URLs verbatim from each file's `> **Video(s):**` line.
- **What you'll learn:** one line, condensed from the file's Brief Summary — keep the concrete hooks
  (tools, numbers, case studies), drop filler.
- **Delta summary:** append an italic note, e.g. `*(delta — read <foundation> first)*`.

## Step 4 — Decide sections & paths (the part that generalizes)

**Sections (group by theme, not by source/format):**
- Aim for ~3–7 sections, each a theme a reader would browse by (e.g. *CS Fundamentals*, *Backend &
  Architecture*, *DevOps & Infrastructure*, *AI-Assisted Development*, *Software Design & Quality*).
- **Adding a video:** if it clearly fits an existing section, insert a row (order roughly
  foundational → advanced, or logical reading order). If it fits none and no section can reasonably
  be broadened, create a new section with a fitting emoji + name.
- **Balance:** a single-item section is fine when the theme is genuinely distinct; otherwise fold a
  lone item into the closest broader section. **Split** a section that grows large (~8+) and clearly
  contains two sub-themes.
- **Non-video docs** go under *Also in this repo*, never in the themed video tables.

**Learning paths (ordered reading sequences for a thread):**
- Extend an existing path when the new video continues that thread; re-order if a new entry is more
  foundational and should come first.
- Create a **new path** only when **≥3 related files** form a coherent progression. Don't force
  unrelated one-offs into a path — it's fine for some files to be in no path, and for a file to
  appear in more than one path.
- Each path is a theme label followed by an ordered chain of **filenames in backticks**:
  `**Backend & systems** — `a.md` → `b.md` → `c.md``.

## Step 5 — Write and verify

Write `README.md`, then confirm it covers the collection exactly:

```bash
# every video summary is listed (in its table row; it may also appear in learning paths)
for f in *-video-summary.md; do grep -q "\`$f\`" README.md || echo "MISSING: $f"; done
# each summary sits in exactly one table row (the row line carries a youtube URL)
for f in *-video-summary.md; do n=$(grep -F "\`$f\`" README.md | grep -c 'youtube.com'); [ "$n" = 1 ] || echo "ROWS=$n: $f"; done
# nothing references a file that doesn't exist
grep -oE '`[a-z0-9-]+\.md`' README.md | tr -d '`' | sort -u | while read f; do [ -f "$f" ] || echo "STALE: $f"; done
# every ▶ URL matches the source file's > **Video** line
for f in *-video-summary.md; do
  s=$(grep -m1 '^> \*\*Video' "$f" | grep -oE 'v=[A-Za-z0-9_-]+' | sort | tr '\n' ' ')
  r=$(grep -F "\`$f\`" README.md | grep 'youtube.com' | grep -oE 'v=[A-Za-z0-9_-]+' | sort | tr '\n' ' ')
  [ "$s" = "$r" ] || echo "URL-DIFF: $f  src=[$s] readme=[$r]"
done
```

All loops should print nothing. Then give the user a one-line recap of what changed (new sections,
new paths, rows added/moved).
