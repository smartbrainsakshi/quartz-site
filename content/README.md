# DSA Notes

A personal knowledge base for Data Structures & Algorithms, organized as a multi-topic Quartz hub — built with Claude, one video/lecture at a time.

This folder is the **content** of the site — markdown files with frontmatter, organized into top-level **topic** folders (Graph, Sliding Window & Two Pointers, and more added over time), mirroring the structure used by [aayushdw/ML-Notes](https://aayushdw.github.io/ML-Notes/).

## Folder structure

```
graph-notes/
├── index.md                              ← top-level hub — lists every topic
├── README.md
│
├── Graph/                                 ← topic: Graph Algorithms
│   ├── 00-Graph-Index.md                  ← topic home page
│   ├── NOTE-TEMPLATE.md                   ← note format used across this topic
│   ├── 01-Foundations/
│   │   ├── 00-Foundations-Index.md        ← section landing page
│   │   └── G1-Introduction-to-Graphs.md   ← individual video notes
│   ├── 02-BFS-DFS-Problems/
│   ├── 03-Topological-Sort/
│   ├── 04-Shortest-Path-Algorithms/
│   ├── 05-MST-and-Disjoint-Set/
│   ├── 06-Other-Algorithms/
│   └── assets/                            ← images/diagrams for Graph notes
│
├── Sliding-Window-and-Two-Pointers/        ← topic: Sliding Window & Two Pointers
│   ├── index.md         ← topic home page
│   └── L1-...md ... L12-...md             ← individual lecture notes
│
└── Tree/, Dynamic-Programming/, etc.       ← future topics slot in the same way
```

**The pattern for every topic:** a top-level folder, named after the topic, containing one `00-<Topic>-Index.md` home page plus either section subfolders (like Graph) or flat lecture files (like Sliding Window) depending on how big the topic is. The root `index.md` links out to every topic's index page.

## Adding a new topic later
1. Create `<Topic-Name>/` at the repo root.
2. Add `<Topic-Name>/00-<Topic>-Index.md` as its home page (list of lectures/sections + progress tracker).
3. Add a link to it from the root `index.md`'s Topics list.
4. Build out notes the same way as Graph/Sliding Window — one lecture at a time, same template.

## Option A — You already use Quartz elsewhere
Just copy the contents of this folder into your existing Quartz repo's `content/` directory, then commit & push. Quartz will pick up the frontmatter and build the site/graph view automatically.

## Option B — Starting fresh
If you don't have a Quartz repo yet:

```bash
# 1. Scaffold a new Quartz site
npx quartz create

# 2. When prompted, choose "Empty Quartz" (so it doesn't ship demo content)

# 3. Replace the generated content/ folder with this one
rm -rf content
cp -r /path/to/graph-notes content

# 4. Initialize git and connect to your GitHub repo
git init
git remote add origin https://github.com/<your-username>/DSA-Notes.git

# 5. Build & preview locally
npx quartz build --serve

# 6. Deploy to GitHub Pages (Quartz ships a helper for this)
npx quartz sync   # or set up the GitHub Actions workflow Quartz provides
```

Full Quartz docs: https://quartz.jzhao.xyz/

## How we're filling this in

1. You paste the transcript of the next video (we go in playlist order, per topic).
2. Claude writes a complete note (`.md` file) for it — intuition, visuals, Java + Python code, dry runs, complexity, pitfalls.
3. The relevant section index, the topic home page, and the root hub get updated.
4. You get the new/updated files to drop into your repo.

See `Graph/NOTE-TEMPLATE.md` for the exact structure every note follows.
