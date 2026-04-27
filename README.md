# [TOPIC] Brain

A second brain for [TOPIC] — a synthesized, LLM-maintained knowledge base. Based on [Andrej Karpathy's LLM wiki methodology](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

> **This is a template.** Replace `[TOPIC]` throughout this file with your actual topic before using.

---

## What This Is

This repo is a living knowledge base — not a static wiki. Raw source documents live in `raw/`. An LLM (Claude, via Claude Code or Wibey CLI) reads those sources, synthesizes them into structured wiki pages, maps relationships between concepts, and keeps everything cross-referenced and up to date.

The idea is that the wiki is the compiled, authoritative layer. Rather than re-reading raw documents every time you have a question, you query the wiki — which has already done the synthesis. The raw sources are the source of truth; the wiki is the knowledge layer built on top.

**You do not need to own or control your source documents** to use this brain. If your sources are read-only (external docs, third-party references, documentation you don't maintain), the wiki still synthesizes them into a queryable knowledge base. The `source-actions.md` file tracks gaps between the wiki and the sources — marked `Editable: Yes` if you can apply the change, `Editable: No` if not.

---

## Repository Structure

```
[topic]-brain/
├── CLAUDE.md             ← schema: tells the LLM how this wiki works
├── README.md             ← this file
├── index.md              ← full content catalog (LLM-maintained)
├── log.md                ← append-only activity log (LLM-maintained)
├── changelog.md          ← wiki change history (LLM-maintained)
├── source-actions.md     ← source update tracker (LLM-maintained)
├── audit-state.md        ← audit progress checkpoint (LLM-maintained)
├── archive/              ← overflow archives (LLM-maintained; auto-created)
├── raw/                  ← immutable source documents (never edit)
└── wiki/                 ← LLM-generated knowledge base
    ├── concepts/         ← core concepts, mechanics, data models
    ├── entities/         ← named systems, services, teams, products
    ├── integrations/     ← integration-specific pages
    ├── use-cases/        ← concrete use cases and scenario walkthroughs
    ├── operations/       ← guides, runbooks, how-tos
    ├── roadmap/          ← plans, milestones, workstreams
    └── synthesis/        ← cross-cutting analysis and comparisons
```

---

## Setup

### 1. Copy this template

```bash
# Clone or copy this template into a new folder named for your topic
cp -r llm-brain-template/ my-topic-brain/
cd my-topic-brain/
git init
```

Then find and replace `[TOPIC]` in `README.md` and `CLAUDE.md` with your actual topic name.

### 2. Install Obsidian

Download and install [Obsidian](https://obsidian.md) (free). This repo is an Obsidian vault — Obsidian renders the `[[WikiLink]]` cross-references as a navigable graph and is the primary way to browse the wiki.

Open the repo as an Obsidian vault:
- Launch Obsidian
- Click **Open folder as vault**
- Select your `[topic]-brain/` folder

### 3. Gather your source documents

Put your source documents in `raw/` as markdown files. See [Getting Data In](#getting-data-in) below for how to convert different source types.

### 4. Launch an AI session

```bash
cd [topic]-brain/

# Launch your AI assistant (pick one):
wibey          # Wibey CLI (Walmart)
claude         # Claude Code
code-puppy -i  # Code Puppy
```

### 5. Customize CLAUDE.md

Once in the AI session, open `CLAUDE.md` and fill in:
- The domain description (what this brain covers)
- Key Entities (named systems, services, teams relevant to your topic)
- Key Concepts (core terms and mechanics)

The LLM reads `CLAUDE.md` at the start of every session — it's the schema that defines how the brain works.

### 6. Start ingesting

```
ingest [filename]
```

Run `ingest` on each source file in `raw/`. The LLM reads the source, synthesizes wiki pages, and builds out the knowledge base incrementally. Start with your most foundational documents first.

---

## Getting Data In

The brain works with any text-based source. The goal is to get your source documents into `raw/` as markdown files, then run `ingest` on each one.

### Confluence pages

Use [Obsidian Web Clipper](https://obsidian.md/clipper) (Chrome/Firefox extension). Configure it to save clipped pages directly to your `raw/` folder.

> **Images and diagrams**: Web Clipper cannot download assets from authenticated pages — it captures broken links instead. To fix this, use [Asset Clipper](https://github.com/yourpalmark/asset-clipper) (downloads the assets) followed by [Asset Swapper](https://github.com/yourpalmark/asset-swapper) (rewrites the links). Run this after clipping any page with diagrams.

### Web pages and documentation sites

Use [Obsidian Web Clipper](https://obsidian.md/clipper) to clip any web page to markdown. Works for public documentation, blog posts, design docs, etc.

### GitHub markdown files

Copy or download `.md` files directly into `raw/`. For entire repos, clone into a temp directory and copy the relevant files.

### PDFs

Convert to markdown first using a tool like [markitdown](https://github.com/microsoft/markitdown) or [pandoc](https://pandoc.org):

```bash
# Using pandoc
pandoc input.pdf -o raw/my-document.md

# Using markitdown (better for complex PDFs)
markitdown input.pdf > raw/my-document.md
```

### Google Docs / Notion / other wikis

Export as markdown or plain text, then save to `raw/`. Most tools have a "Export as Markdown" option.

### Any text content

If it can be saved as a `.md` or `.txt` file, it can go in `raw/`. The LLM handles the synthesis — the source just needs to be readable text.

---

## Using the Brain

There are two main ways to use this repo:

### 1. Query with an AI agent

Open the repo in an AI session (Setup step 4) and ask questions in plain English. The AI reads the synthesized wiki pages and answers with citations.

```
# Example queries
"Give me an overview of [TOPIC]."
"How does [concept X] relate to [concept Y]?"
"What are the open questions or known gaps in [area]?"
"Walk me through [scenario] step by step."
"What's changed recently in [topic area]?"
```

This is the core value from [Karpathy's methodology](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f): build the wiki once, then query it. The wiki is richer than the raw sources because it adds cross-references, surfaced contradictions, and synthesis across documents the originals never connected.

### 2. Browse in Obsidian

Open the repo as an Obsidian vault (Setup step 2) and navigate visually. This wiki is designed for **link-following**, not flat file browsing.

**Recommended entry points:**

| Goal | Start here |
|------|-----------|
| I'm new — give me the overview | `index.md` → Start Here section |
| I want to understand the full picture | `wiki/synthesis/` — cross-cutting analysis pages |
| I want to explore what exists | `index.md` — full catalog organized by section |
| I want to see how concepts connect | Obsidian Graph View (Cmd+G) |

**Follow the links.** Every wiki page has a `## Related` section with `[[WikiLink]]` cross-references. The LLM adds links across sources that were never connected in the originals — following them surfaces relationships no single document contains.

**Use the synthesis pages.** `wiki/synthesis/` contains the highest-value content: cross-cutting analyses that span multiple sources. These are what you cannot get from reading individual documents.

**`index.md` is for discovery, not reading.** Use it to find what exists, then open the page and follow links from there.

**Obsidian Graph View** (Cmd+G) renders the full `[[WikiLink]]` network visually — useful for exploring when you don't know what to search for.

---

## Maintaining the Brain

The brain is maintained through an AI session. Open the repo, launch your AI assistant, and use the commands below. The LLM reads `CLAUDE.md` at the start of each session.

### Commands

| Command | What it does |
|---------|-------------|
| `help` | List all available commands |
| `init` | First-time setup — verify structure, remind about Obsidian |
| `ingest [filename]` | Ingest a raw source file into the wiki. Auto-detects first-time vs. re-ingest. |
| `doctor` | Full wiki health check: broken links, README sync, contradictions, orphan pages, stale SA entries |
| `todo` | Show all unresolved source action items (`source-actions.md`) |
| `resolve SA-NNN` | Mark a source action item as resolved |
| `validate SA-NNN` | Targeted check for a single action item |
| `audit` | Batch re-review of all raw sources against wiki pages, resuming from where the last run stopped |

### Adding a new source

1. Save the source document as a markdown file in `raw/`
2. Run `ingest [filename]` in your AI session
3. The LLM creates or updates wiki pages, adds cross-references, flags any source gaps, and commits

### Updating an existing source

1. Replace the file in `raw/` with the updated version (same filename)
2. Run `ingest [filename]` — the LLM detects it as a re-ingest, resolves stale action items, and merges new content

### Source Actions

`source-actions.md` tracks gaps between the wiki and the source documents:
- **`Editable: Yes`** — you own this source; apply the change when convenient
- **`Editable: No`** — source is read-only; the entry documents the gap for awareness

Neither type blocks wiki quality — the wiki is always the authoritative layer. SA entries just track what could improve the underlying sources if possible.

---

## How It Was Built (Example)

> Delete this section and replace with your own setup notes after you've initialized the brain.

This template was created from the [Release Ramp Brain](https://gecgithub01.walmart.com/yourpalmark/release-ramp-brain) project, which pioneered this pattern for Walmart's TEX team. That brain was built by:

1. Exporting Confluence pages to markdown via Obsidian Web Clipper
2. Dropping exports into `raw/`
3. Running `ingest` on each source in the AI session
4. The LLM built out `wiki/`, `index.md`, `log.md`, `changelog.md`, and `source-actions.md` incrementally
5. Running `audit` to close content gaps across all sources

---

## Key Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Schema — the LLM reads this every session. Defines structure, conventions, and all workflows. Customize for your domain. |
| `index.md` | Full catalog of all wiki pages, organized by category. Start here for navigation. |
| `source-actions.md` | Tracks gaps between the wiki and source documents. `Editable: Yes` entries are actionable; `No` entries document known gaps. |
| `changelog.md` | Append-only history of every wiki change. |
| `log.md` | Append-only log of every operation (ingest, audit, doctor, etc.). |
| `audit-state.md` | Tracks audit progress so `audit` can resume across sessions. |
