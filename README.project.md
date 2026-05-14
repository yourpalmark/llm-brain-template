# [TOPIC] Brain

A second brain for [TOPIC] — a synthesized, LLM-maintained knowledge base.

---

## Overview

This repo is a living knowledge base — not a static wiki. Raw source documents live in `raw/`. An LLM reads those sources, synthesizes them into structured wiki pages, maps relationships between concepts, and keeps everything cross-referenced and up to date.

Rather than re-reading raw documents every time you have a question, you query the wiki — which has already done the synthesis. The raw sources are the source of truth; the wiki is the knowledge layer built on top.

This pattern is based on [Andrej Karpathy's LLM wiki methodology](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

---

## Setup

### 1. Install Obsidian

Download and install [Obsidian](https://obsidian.md) (free). This repo is an Obsidian vault — Obsidian renders the `[[WikiLink]]` cross-references as a navigable graph and is the primary way to browse the wiki.

### 2. Clone the repo and open as a vault

```bash
git clone <repo-url>
```

Then open the repo as an Obsidian vault:
- Launch Obsidian
- Click **Open folder as vault**
- Select the repo folder

The `wiki/` directory is the main browsing surface. Start with `index.md` for the full content catalog.

### 3. Launch an AI session

```bash
cd [topic]-brain
claude         # Claude Code
```

### 4. Next steps

**If you are cloning an already-populated brain** — run `init` inside the AI session to complete first-checkout setup (e.g. initializing submodules):

```
init
```

**If you are building this brain from scratch** — skip `init` and start ingesting. Drop source documents into `raw/` and run `ingest` for each one:

```
ingest [filename]
```

The LLM builds out the wiki incrementally — creating pages, adding cross-references, and tracking source gaps. See [How It Was Built](#how-it-was-built) for tools to capture source documents as markdown files.

> **Getting source documents into `raw/`:** Use [Batch Clipper](https://github.com/yourpalmark/batch-clipper) — a Chrome extension that clips web pages and Confluence spaces directly into your vault as markdown. It handles batching and asset downloads in one step. No separate asset download or link-fixing needed.

---

## Using the Brain

There are two main ways to use this repo:

### 1. Query with an AI agent

Open the repo in an AI session (see Setup step 3) and ask questions in plain English. The AI reads the synthesized wiki pages — not the raw source documents — and answers with citations.

```
# Example queries
"How does [key concept] work?"
"What are the steps to [common task]?"
"What's the relationship between [entity A] and [entity B]?"
"Summarize the [topic area] use cases."
```

The wiki is richer than the raw sources — it adds cross-references, surfaced contradictions, and synthesis across documents the originals never connected.

### 2. Browse in Obsidian

Open the repo as an Obsidian vault (see Setup step 2) and navigate visually. This wiki is designed for **link-following**, not flat file browsing.

**Recommended entry points:**

| Goal | Start here |
|------|-----------|
| I'm new — give me the overview | `index.md` → `[[Overview]]` |
| I want to understand key concepts | `index.md` → `[[Key Concepts]]` |
| I want to explore what exists | `index.md` — full catalog organized by section |
| I want to see how concepts connect | Obsidian Graph View (Cmd+G) |

**Follow the links.** Every wiki page has a `## Related` section with `[[WikiLink]]` cross-references to connected pages.

**Use the synthesis pages.** `wiki/synthesis/` contains the highest-value content: cross-cutting analyses synthesized from multiple sources.

**`index.md` is for discovery, not reading.** It's a catalog of all wiki pages organized by section — use it to find what exists, then open the page and follow links.

**Obsidian Graph View** (Cmd+G) renders the full `[[WikiLink]]` network visually — useful for exploring the topic space when you don't know what to search for.

---

## Updating the Brain

The brain is maintained through Claude Code. Open the repo in your editor, launch an AI session, and use the commands below. The LLM reads `CLAUDE.md` at the start of each session to understand the schema, conventions, and workflows.

### Commands

| Command | What it does |
|---------|-------------|
| `help` | List all available commands |
| `init` | First-time setup verification. Idempotent: safe to run again. |
| `ingest [filename]` | Ingest a raw source file into the wiki. Auto-detects first-time vs. re-ingest. |
| `doctor` | Run a full wiki health check: broken links, orphan pages, stale source actions, README sync. |
| `todo` | Show all unresolved source action items (`source-actions.md`) |
| `resolve SA-NNN` | Mark a source action item as resolved. Example: `resolve SA-009` |
| `validate SA-NNN` | Targeted check for a single action item — reads the source file and verifies whether the change is still needed |
| `audit` | Batch re-review of all raw sources against their wiki pages, resuming from where the last run stopped |

### Adding a new source

1. Clip the source page with [Batch Clipper](#how-it-was-built) — it saves the markdown and assets directly into `raw/`
2. Drop the `.md` file into `raw/` (already there if you used Batch Clipper)
3. Run `ingest [filename]` in your AI session
4. The LLM creates or updates wiki pages, adds cross-references, flags any source gaps, and commits

### Updating an existing source

1. Get the updated version of the source document
2. Replace the file in `raw/` (same filename)
3. Run `ingest [filename]` — the LLM detects it as a re-ingest, resolves any now-stale action items, and merges new content

---

## Key Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Schema — the LLM reads this every session. Defines structure, conventions, and all workflows. |
| `index.md` | Full catalog of all wiki pages, organized by category. Start here for navigation. |
| `source-actions.md` | Tracks gaps between the wiki and the source documents — what ideally should be updated upstream. |
| `changelog.md` | Append-only history of every wiki change. |
| `log.md` | Append-only log of every operation (ingest, audit, doctor, etc.). |
| `audit-state.md` | Tracks audit progress so `audit` can resume across sessions. |

---

## Repository Structure

```
[topic]-brain/
├── CLAUDE.md              ← schema: tells the LLM how this wiki works
├── README.md              ← this file
├── index.md               ← full content catalog (LLM-maintained)
├── log.md                 ← append-only activity log (LLM-maintained)
├── changelog.md           ← wiki change history (LLM-maintained)
├── source-actions.md      ← source gap tracker (LLM-maintained)
├── audit-state.md         ← audit progress checkpoint (LLM-maintained)
├── archive/               ← overflow archives (LLM-maintained; auto-created)
├── raw/                   ← immutable source documents (never edit)
│   └── assets/            ← images downloaded by Batch Clipper
└── wiki/                  ← LLM-generated knowledge base
    └── (subfolders created organically during ingest, based on topic domain)
```

---

## How It Was Built

### The idea

This brain is based on [Andrej Karpathy's LLM wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). The core insight: instead of feeding raw documents to an LLM every time you ask a question (RAG), have the LLM incrementally build a synthesized wiki from those documents once — and then query the wiki. The wiki is richer than the raw sources because it adds cross-references, surfaced contradictions, and synthesis across multiple documents.

### Tools used

**[Obsidian](https://obsidian.md)** — free note-taking application that renders this repo as a browsable, graph-linked wiki. Open the repo root as an Obsidian vault.

**[Batch Clipper](https://github.com/yourpalmark/batch-clipper)** — Chrome extension for clipping web pages and Confluence spaces directly into an Obsidian vault as markdown. Think of it as Obsidian's Web Clipper with built-in batch support and asset handling — it clips pages (or entire page trees) at once, downloads images and diagrams alongside the markdown, and saves everything directly into `raw/`. No separate asset download or link-fixing step needed.

### Build process

1. Used Batch Clipper to clip source pages into `raw/` (markdown + assets in one step)
2. Opened the repo in Claude Code
3. Ran `ingest` on each source
4. The LLM built out `wiki/`, `index.md`, `log.md`, `changelog.md`, and `source-actions.md` incrementally
5. Ran `audit` and `doctor` passes to fill gaps and fix issues
