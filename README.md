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

### 2. Replace the READMEs

```bash
rm README.md
mv README.project.md README.md
```

### 3. Customize for your domain

Open `CLAUDE.md` and fill in:
- `[TOPIC]` — your topic name throughout the file
- **Key Entities** — named systems, services, teams relevant to your domain
- **Key Concepts** — core terms and mechanics

Open `README.md` (formerly `README.project.md`) and replace `[TOPIC]` throughout.

### 4. Gather source documents

Drop your source documents into `raw/` as markdown files. See [Getting Data In](#getting-data-in) below.

### 5. Open as an Obsidian vault

Install [Obsidian](https://obsidian.md) (free) and open the repo folder as a vault. This renders the `[[WikiLink]]` cross-references as a navigable graph.

### 6. Launch an AI session and start ingesting

```bash
claude         # Claude Code
```

Then run `ingest [filename]` for each source. The LLM builds out the wiki incrementally.

---

## Getting Data In

Get your source documents into `raw/` as markdown files, then run `ingest`.

**[Obsidian Web Clipper](https://obsidian.md/clipper)** — Chrome/Firefox extension that saves web pages as markdown files directly into your Obsidian vault.

> **Images**: Web Clipper can sometimes have trouble downloading assets from authenticated pages — it may capture broken asset links instead. If this happens, two additional tools can help:

**[Asset Clipper](https://github.com/yourpalmark/asset-clipper)** — Chrome extension that downloads assets referenced in clipped pages that Web Clipper couldn't fetch due to authentication.

**[Asset Swapper](https://github.com/yourpalmark/asset-swapper)** — Obsidian plugin that replaces broken asset links in clipped markdown files with the locally downloaded assets.

---

## Key Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | LLM schema — fill in `[TOPIC]`, Key Entities, and Key Concepts for your domain. Everything else works as-is. |
| `README.project.md` | Your project's README template — rename to `README.md` after setup and fill in `[TOPIC]`. |
| `index.md` | Content catalog (LLM-maintained). |
| `source-actions.md` | Tracks gaps between the wiki and source documents (LLM-maintained). |
| `changelog.md` | Append-only wiki change history (LLM-maintained). |
| `log.md` | Append-only activity log (LLM-maintained). |
| `audit-state.md` | Audit progress checkpoint (LLM-maintained). |

---

## Repository Structure

```
llm-brain-template/
├── README.md              ← this file (template docs — replace after setup)
├── README.project.md      ← your project's README (rename to README.md after setup)
├── CLAUDE.md              ← LLM schema (customize for your domain)
├── index.md               ← content catalog template
├── log.md                 ← activity log template
├── changelog.md           ← wiki changelog template
├── source-actions.md      ← source gap tracker template
├── audit-state.md         ← audit progress template
├── .gitignore             ← ignores .DS_Store, Obsidian workspace files
├── raw/                   ← drop your source documents here
├── archive/               ← overflow archives (auto-managed by LLM)
└── wiki/                  ← LLM-generated knowledge base
    ├── concepts/
    ├── entities/
    ├── integrations/
    ├── use-cases/
    ├── operations/
    ├── roadmap/
    └── synthesis/
```

---

## Commands

| Command | What it does |
|---------|-------------|
| `ingest [filename]` | Ingest a source file; auto-detects first-time vs. re-ingest |
| `doctor` | Full wiki health check: broken links, README sync, orphan pages, stale source actions |
| `audit` | Batch re-review of all sources against wiki pages, resuming from last run |
| `todo` | Show all unresolved source action items |
| `resolve SA-NNN` | Mark a source action item as resolved |
| `validate SA-NNN` | Targeted check for a single action item |
| `init` | First-time setup verification |
| `help` | List all commands |

---

## How It Works

1. Drop source documents into `raw/`
2. Run `ingest [filename]` in an AI session
3. The LLM synthesizes wiki pages, adds cross-references, tracks source gaps
4. Query the wiki with natural language or browse it in Obsidian

The LLM reads `CLAUDE.md` at the start of every session — it defines the schema, commands, and all workflows. Raw sources are never modified.
