# LLM Brain Template

A template for building a second brain using [Andrej Karpathy's LLM wiki methodology](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — a synthesized, LLM-maintained knowledge base for any topic or domain.

---

## Overview

This template gives you a ready-to-use structure for building a living knowledge base from raw source documents (web pages, GitHub markdown, internal docs, etc.). An LLM reads your sources, synthesizes them into structured wiki pages, maps relationships between concepts, and keeps everything cross-referenced and up to date.

Rather than re-reading raw documents every time you have a question, you query the synthesized wiki — which has already done the synthesis. The raw sources are the source of truth; the wiki is the knowledge layer built on top.

---

## Setup

### 1. Get the template

```bash
# Option A: clone and re-initialize as a new repo
git clone https://github.com/yourpalmark/llm-brain-template my-topic-brain
cd my-topic-brain
rm -rf .git
git init

# Option B: use as a GitHub template repo
# Click "Use this template" on GitHub
```

### 2. Customize for your domain

Open the repo in an AI session and run the following setup prompt (replace `[TOPIC]` in the first line with your topic name — this is the only thing you need to fill in):

```
My topic is: [TOPIC]

I want to set up an LLM brain for this topic. Please:
1. Replace every instance of [TOPIC] in CLAUDE.md and README.project.md with the actual topic name.
2. Delete README.md and rename README.project.md to README.md.
3. When steps 1 and 2 are complete, confirm and then give the user the following next steps exactly as written:

   ---
   **Your brain is ready. Here's what to do next:**

   1. Open this repo as an Obsidian vault (File → Open folder as vault in Obsidian)
   2. Use Batch Clipper to clip source pages directly into `raw/`
   3. Run `ingest [filename]` in this AI session for each source to start building the wiki

   The new `README.md` has full details under **Setup** and **Getting Data In** if you need a reference.
   ---
```

### 3. Gather source documents

Drop your source documents into `raw/` as markdown files. See [Getting Data In](#getting-data-in) below.

### 4. Open as an Obsidian vault

Install [Obsidian](https://obsidian.md) (free) and open the repo folder as a vault. This renders the `[[WikiLink]]` cross-references as a navigable graph.

### 5. Start ingesting

In the AI session run `ingest [filename]` for each source. The LLM builds out the wiki incrementally.

---

## Getting Data In

Get your source documents into `raw/` as markdown files, then run `ingest`.

**[Batch Clipper](https://github.com/yourpalmark/batch-clipper)** — Chrome extension for clipping web pages and Confluence spaces directly into an Obsidian vault as markdown. Think of it as Obsidian's Web Clipper with built-in batch support and asset handling — it clips pages (or entire page trees) at once, downloads images and diagrams alongside the markdown, and saves everything directly into `raw/`. No separate asset download or link-fixing step needed.

> **Large assets**: This repo includes a `.gitattributes` that tracks images and PDFs in `raw/assets/` via [Git LFS](https://git-lfs.com) when available. If your remote supports LFS, run `git lfs install` once after cloning. If not, files commit normally as blobs — no breakage either way.

---

## Key Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | LLM schema — the setup prompt fills in `[TOPIC]` automatically. |
| `README.project.md` | Your project's README — the setup prompt renames this to `README.md` and fills in `[TOPIC]`. |
| `index.md` | Content catalog (LLM-maintained). |
| `source-actions.md` | Tracks gaps between the wiki and source documents (LLM-maintained). |
| `changelog.md` | Append-only wiki change history (LLM-maintained). |
| `log.md` | Append-only activity log (LLM-maintained). |
| `clipper-log.md` | Batch Clipper run log — input for `clip` command. |
| `state/source-state-NNN.md` | Paginated source registry — provenance + wiki attribution (LLM-maintained). |

---

## Repository Structure

```
llm-brain-template/
├── README.md              ← this file (template docs — replaced during setup)
├── README.project.md      ← your project's README (renamed to README.md during setup)
├── CLAUDE.md              ← LLM schema (customized for your domain during setup)
├── index.md               ← content catalog template
├── log.md                 ← activity log template
├── changelog.md           ← wiki changelog template
├── source-actions.md      ← source gap tracker template
├── clipper-log.md         ← batch-clipper run log (written by Batch Clipper)
├── .gitignore             ← ignores .DS_Store, .obsidian/
├── raw/                   ← drop your source documents here
│   └── assets/            ← images downloaded by Batch Clipper
├── state/                 ← source tracking (auto-managed by LLM)
│   └── source-state-001.md  ← paginated source registry (500 rows/file)
├── archive/               ← overflow archives (auto-managed by LLM)
└── wiki/                  ← LLM-generated knowledge base
    └── (subfolders created organically during ingest, based on topic domain)
```

---

## Commands

| Command | What it does |
|---------|-------------|
| `clip` | Register newly clipped files into `state/source-state-NNN.md` |
| `ingest [N=10]` | Batch-ingest next N files from the queue (default: 10) |
| `ingest [filename]` | Ingest a single source file; auto-detects first-time vs. re-ingest |
| `doctor` | Wiki structural health check (no raw source reads): broken links, orphan pages, source-state integrity, SA backlog, README sync |
| `audit` | Source-vs-wiki coverage check (reads raw sources): flags stale ingestions, missing clips, and thin coverage. Resumes via `state/source-state-NNN.md` |
| `todo` | Show all unresolved source action items |
| `resolve SA-NNN` | Mark a source action item as resolved |
| `validate SA-NNN` | Targeted check for a single action item |
| `actions [source filename]` | List all unresolved SA entries for a source, formatted for handoff. Omit filename to see counts per source. |
| `init` | First-time setup verification |
| `help` | List all commands |

---

## How It Works

1. Drop source documents into `raw/`
2. Run `clip` to register sources into `state/source-state-NNN.md`
3. Run `ingest` — the LLM synthesizes wiki pages, adds cross-references, tracks source gaps
4. Query the wiki with natural language or browse it in Obsidian

The LLM reads `CLAUDE.md` at the start of every session — it defines the schema, commands, and all workflows. Raw sources are never modified.
