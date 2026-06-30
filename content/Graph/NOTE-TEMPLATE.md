# Note Template (every video follows this structure)

```markdown
---
title: "G-<N>. <Video Title>"
tags: [graph, <topic-tags>]
videoLink: "<youtube link>"
difficulty: <Easy|Medium|Hard>   # only for problem videos
---

# G-<N>. <Video Title>

## 🎯 What problem are we solving?
Plain-English statement of the problem/concept, no jargon. 1-3 sentences.

## 💡 Intuition
The "aha" — how a human would think about this before knowing any algorithm name.
Walk through the reasoning step by step, the way the video builds it up.

## 🖼️ Visualizing it
ASCII diagram / step-by-step trace of the algorithm on a small example graph,
so the note is understandable without rewatching the video.

## 🛠️ Algorithm / Approach
Numbered steps — the actual algorithm, language-agnostic.

## 👨‍💻 Code

### Java
\`\`\`java
// full, runnable, commented implementation
\`\`\`

### Python
\`\`\`python
# full, runnable, commented implementation
\`\`\`

## ⏱️ Complexity Analysis
- **Time:** O(...) — explain *why*
- **Space:** O(...) — explain *why*

## ⚠️ Common Pitfalls

> [!warning]
> Mistakes people make (e.g. forgetting to mark visited before pushing vs after popping), as a bulleted list inside this callout.

## 🔗 Related Problems / Next Up
Links to related notes in this repo.
```

---

**Why this structure:** intuition before code (so you actually understand it, not memorize it), a visual trace (so revision doesn't require rewatching), both languages side by side (so it's useful for either track), and pitfalls explicitly called out (these are exactly what trips people up in interviews).

I'll follow this for all 56 notes so the whole repo feels consistent — like one book, not 56 disconnected pages.
