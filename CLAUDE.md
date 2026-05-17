# [TOPIC] Brain — Schema

This is the configuration file for the [TOPIC] second brain. It tells you (the LLM) exactly how this wiki is structured, what the conventions are, and what workflows to follow. Read this at the start of every session.

> **Setup note:** Use the setup prompt from the llm-brain-template README to initialize this file for your topic.

---

## Domain

**Topic**: [TOPIC]

**Purpose of this wiki**: Build and maintain a comprehensive, interlinked knowledge base about [TOPIC]. The wiki is the compiled, synthesized, always-current artifact. Raw sources are the source of truth but are never modified.

---

## Directory Layout

```
[topic]-brain/
├── CLAUDE.md             ← this file (schema)
├── README.md             ← setup and usage guide
├── index.md              ← content catalog (LLM-maintained)
├── log.md                ← append-only activity log (LLM-maintained)
├── changelog.md          ← wiki change history (LLM-maintained)
├── source-actions.md     ← source update tracker (LLM-maintained)
├── clipper-log.md        ← batch-clipper run log (written by Batch Clipper)
├── state/                ← source tracking (LLM-maintained)
│   └── source-state-001.md  ← paginated source registry (500 rows/file)
├── archive/              ← overflow archives (LLM-maintained; auto-created)
├── raw/                  ← immutable source documents (never edit)
│   └── assets/           ← images/attachments from batch-clipper
└── wiki/                 ← LLM-generated knowledge base
    └── (subfolders created organically during ingest, based on topic domain)
```

---

## Wiki Page Format

Every wiki page should follow this structure:

```markdown
---
tags: [one or more descriptive tags for this page, e.g. concept, entity, overview, synthesis, operation, use-case]
sources: [list of raw filenames that informed this page]
updated: YYYY-MM-DD
---

# Page Title

One-sentence summary of what this page covers.

## [Content sections as appropriate]

## Related
- [[link to related page]]
- [[link to related page]]
```

- Use `[[WikiLink]]` syntax for cross-references between wiki pages.
- **Filename must exactly match the H1 title** (e.g., `# Core Concepts` → file named `Core Concepts.md`). This is required for Obsidian WikiLink resolution. Do NOT use kebab-case or other slugified names. Special characters illegal in filenames (`/`) must be replaced (use `and` instead of `/`).
- **Before writing any `[[WikiLink]]`** — verify the target file exists: `find wiki/ -name "EXACT TITLE.md"`. Never write `[[Foo Bar]]` unless `wiki/*/Foo Bar.md` exists on disk. Do not assume a name is correct — check it. This applies when writing links in wiki pages AND in `index.md`.
- Keep page titles short and consistent — they become the link targets.
- `sources` frontmatter lists the raw filenames (without path) that contributed to this page.
- `updated` is the date this page was last meaningfully changed.

### Ingested Sources Table Format

The `### Ingested Sources` table in `index.md` must use this exact format:

```
| Source File | Wiki Pages |
| ----------- | ---------- |
| [Filename.md](<raw/path/to/Filename.md>) | • [[Page One]]<br>• [[Page Two]] |
```

**Column rules:**
- **Source File** — markdown link: display name is the filename only (no path), href is the full `raw/` path in angle brackets — `[Filename.md](<raw/path/to/Filename.md>)`. Never URL-encode spaces or special characters.
- **Wiki Pages** — bullet list using `• [[WikiLink]]<br>• [[WikiLink]]` format; one page per bullet; plain-text note for stub/no-content sources

### Source State File Format

All source tracking lives in paginated files in `state/`. Each file covers 500 rows: oldest sources in `source-state-001.md`, newest in the last numbered file. `clip` owns columns 1-6 (provenance); `ingest` owns columns 7-10 (wiki attribution).

**Schema** (10 columns):
```
| Source File | Source URL | Root | Clipped Date | Source Created | Source Modified | Wiki Pages | Status | Ingested Date | Skipped Reason |
| ----------- | ---------- | ---- | ------------ | -------------- | --------------- | ---------- | ------ | ------------- | -------------- |
| [filename.md](<raw/path/filename.md>) | https://... | raw/subfolder | YYYY-MM-DD |  |  | [[Page One]], [[Page Two]] | ingested | YYYY-MM-DD |  |
```

**Column rules:**
- **Source File** — `[filename.md](<raw/plain path.md>)` — angle-bracket path, never URL-encode
- **Source URL** — original URL the file was clipped from; empty if no URL
- **Root** — the immediate `raw/` subfolder (e.g. `raw/Guides`); use `raw` if file is directly in `raw/`
- **Clipped Date** — YYYY-MM-DD from the file's `clipped` frontmatter field
- **Source Created** — from frontmatter `created` field; blank if not captured at clip time
- **Source Modified** — from frontmatter `modified` field; blank if not captured at clip time
- **Wiki Pages** — comma-separated `[[WikiLink]]` references; empty until ingested
- **Status** — `ingested` | `skipped` | `stale`
- **Ingested Date** — YYYY-MM-DD when file was processed by `ingest`; empty until ingested
- **Skipped Reason** — plain text reason for skipped files; empty for ingested

**Status lifecycle:**
- Absent from source-state → add row via `clip`
- After `ingest`: Status = `ingested` or `skipped`
- Re-clipped after ingested/skipped → Status = `stale`
- After re-ingest of stale row → Status = `ingested` or `skipped`

**Pagination rules:**
- 500 rows per file; oldest in `-001`, newest in last file
- File header: `# Source State — Part NNN` + `Brain-owned. Updated by \`clip\` (columns 1-6) and \`ingest\` (columns 7-10).` + `Rows X-Y`
- When a new file would exceed 500 rows, create the next numbered file
- Filenames: `source-state-001.md`, `source-state-002.md`, etc. (zero-padded to 3 digits)
- Never merge files; never split mid-row

**When `clip` adds or updates rows:**
1. Open the last `source-state-NNN.md` file
2. If file already has a row for this source → update columns 1-6 and set Status = `stale` (if was `ingested` or `skipped`)
3. If not present → append new row (columns 7-10 blank); if current file is at 500 rows, create next file
4. Update the row range in the file header

---

## Key Entities (expand as you learn more)

<!-- Populated during ingest. Add named systems, services, teams, and products as they are encountered. -->

---

## Key Concepts (expand as you learn more)

<!-- Populated during ingest. Add core concepts, mechanics, and terms as they are encountered. -->

---

## Maintenance Requirements (ALWAYS enforce — no exceptions)

These rules apply to **every** wiki change, regardless of what triggered it:

> 🚨 **BEFORE EVERY COMMIT** — run this mental checklist:
> - Did I add a CHG entry to `changelog.md` for every wiki file I touched? (one CHG per file per session of changes)
> - Did I evaluate whether any change needs an SA entry in `source-actions.md`?
> - Did I update `log.md` if this was an ingest, reingest, audit, or doctor operation?
> - Did I update `state/source-state-NNN.md` columns 7-10 for every file I ingested?
> - If I renamed a wiki page or reorganized `index.md` sections, did I update `README.md`?
>
> **Do not commit without completing this checklist.** The user must never have to ask for these — they are part of every change, not an afterthought.

1. **`changelog.md` must be updated** for every wiki page creation or meaningful edit — including diagram changes, prose corrections, and structural edits. Add a CHG entry under today's date header. CHG numbers are sequential. **This includes small changes.** If you touched a wiki file, it gets a CHG entry in the same commit, not a later one.

2. **`source-actions.md` must be updated** whenever the wiki contains information that the raw source does not yet reflect. This includes:
   - Corrections to factual errors in source documents
   - New sections added to the wiki that should be reflected in the source
   - Content synthesized across multiple sources that clarifies or contradicts any single source
   - If the raw source already has the content correctly → no SA entry needed

3. **`log.md` must be updated** after every operation (ingest, reingest, audit, doctor) with a timestamped entry.

4. **`README.md` must be checked** whenever a wiki page is renamed or `index.md` sections are reorganized. `README.md` may contain hardcoded page references and section names. Update it in the same commit — never let it drift.

These files are the audit trail of the wiki. Skipping any of them means the wiki has undocumented changes that cannot be traced or reproduced.

---

## Commands

These keyword triggers are recognized when a message starts with them.

### `clip`
Register newly clipped source files into `state/source-state-NNN.md`. Run after batch-clipper adds files to `raw/`.
- Reads `clipper-log.md` to find the latest batch of clipped files.
- For each file in the batch: reads frontmatter (`source`, `created`, `modified`, `clipped`), derives `Root` from file path.
- **New file** (not in any source-state row) → append row with columns 1-6 filled, columns 7-10 blank.
- **Existing file** with Status = `ingested` or `skipped` → update columns 1-6, set Status = `stale`.
- **Existing file** with Status = `stale` → update columns 1-6 only (already queued).
- Commit: `clip: register N sources`
- Example: `clip`

### `ingest [N=10]` / `ingest [filename]`
Process source files from the ingest queue and write wiki pages. Two forms only:
- `ingest [N=10]` — process the next N files from the queue (default 10 if N omitted). Queue = rows in `state/source-state-NNN.md` with blank Status OR Status = `stale`.
- `ingest [filename]` — process a specific file by name. If the file has no source-state row, treat as new. If Status = `stale`, run the **Re-ingest an Updated Source** workflow.
- For each file processed: run the **Ingest a Source** workflow, update the source-state row (columns 7-10), commit.
- Example: `ingest` (next 10) | `ingest 5` | `ingest my-topic-overview`

### `doctor`
Wiki structural health check. **Does not read raw sources.** Reports: broken WikiLinks, orphan pages, unresolved SA backlog, source-state integrity issues (duplicate rows, invalid Status values, missing columns), README drift. Follow the **Lint the Wiki** workflow below.

### `todo`
Show all `🔲 Unresolved` entries from `source-actions.md`. For each, show the SA number, source document, source quote, and desired change.

### `resolve SA-NNN`
Toggle the status of an SA entry in `source-actions.md`. **Update BOTH places:**
1. The `### 🔲 SA-NNN` heading → change `🔲` to `✅`
2. The `**Status:**` line → set to `✅ Resolved [YYYY-MM-DD]`
- Example: `resolve SA-009`

### `validate SA-NNN`
Targeted check for a single SA entry. Reads the **Source File** and checks whether the **Source Quote** still exists verbatim. If the quote is gone or the desired change is already present → mark `✅ Resolved [YYYY-MM-DD]` in both the heading and Status line.
- Example: `validate SA-014`

### `audit`
Source-vs-wiki coverage check. **Reads raw sources.** Flags: files whose frontmatter `modified` date is newer than their `Ingested Date` (stale), files present in `clipper-log.md` but absent from `state/source-state-NNN.md` (suggest `clip`), and ingested files where wiki coverage appears thin relative to source content (suggest re-ingest). Uses `state/source-state-NNN.md` to detect new vs. existing files and resume across sessions.
- Example: `audit`

### `init`
First-time setup. Run once after cloning — safe to run again (idempotent). Follow the **Initialize Repo** workflow.
- Example: `init`

### `actions [source filename]`
Generate a clean action list for a specific source document — all unresolved SA entries for that source, formatted for handoff to the doc owner.

- `[source filename]` is the raw filename (partial match is fine)
- Output format: one section per SA entry showing SA number, type, source quote, and desired change — suitable for pasting into a Jira ticket, Slack message, or Confluence comment
- If no filename is provided: group all `🔲 Unresolved` entries by source document and show a summary count per source
- Example: `actions My Source Document`
- Example: `actions` (shows all sources with open items and counts)

### `help`
List all available commands with a one-line description of each.

---

## Workflows

### Ingest a Source

When the user says "ingest [filename]":

1. **Read** the source file from `raw/`.

2. **Read associated assets** — after reading the source file, scan it for assets and process each:

   - **Images** (`![[assets/<Title>/<file>.png]]` or similar wikilinks): read each image visually using the Read tool. Interpret what it shows — architecture diagrams, flow charts, data models, screenshots, etc. — and incorporate that understanding directly into wiki page content as substantive prose, not as a footnote. An architecture diagram should inform the design section of a wiki page just as much as the text does.

   - **Linked documents** (URLs or wikilinks pointing to `.pdf`, `.xls`, `.xlsx`, `.docx`, `.csv` files in the assets folder): attempt to read them using the Read tool. If readable, treat as supplementary source material for the same ingest and incorporate their content into related wiki pages.

   - **Other linked files** (`.py`, `.sql`, `.json`, code files, etc.): note their presence in the wiki page with a brief description (e.g., "Attached: `analysis.py` - power estimation simulation") but do not attempt to read them unless they are clearly central to understanding the content.

   Assets live in a single `raw/assets/` folder that mirrors the page hierarchy (e.g. `raw/assets/Parent/Child/<Page Title>/image.png`) — check for that path after reading the source file.

   > 🚨 **ASSET GATE — mandatory before step 3**: Run `find "raw/assets/<Page Title>" -type f 2>/dev/null` (substituting the actual page title). If the directory exists and contains files, read every file before proceeding. Do not skip this even if the source markdown contains no `![[` wikilinks — assets may exist without being referenced. This step is not optional and must not be skipped during bulk batch processing.

3. **Discuss** key takeaways with the user — what's new, what's important, what's surprising. Include insights from images and documents read in step 2.

4. **Write a summary page** in the appropriate `wiki/` subfolder. Choose or create a subfolder that fits the content (e.g., `wiki/concepts/`, `wiki/entities/`, `wiki/operations/`, `wiki/synthesis/` — or any name that fits the domain). Use a display-name filename matching the H1 title (e.g., `wiki/concepts/Overview.md` — NOT `wiki/concepts/overview.md`).

   **Content types to recognize** (from Karpathy's methodology):
   - **Entity pages** — named things: people, systems, teams, products, services
   - **Concept pages** — ideas, mechanics, patterns, terminology
   - **Summaries** — digested overview of a single source
   - **Comparisons** — side-by-side analysis of two or more things
   - **Overviews** — high-level orientation for a topic area
   - **Synthesis pages** — cross-cutting analysis connecting multiple sources

   Use these types to guide what pages to create and where to put them. Subfolder names are yours to define — the above are examples, not requirements.

5. **Update existing wiki pages** — any entities, concepts, or integrations touched by this source should be updated or created.

6. **Populate `source-actions.md`** — for each wiki correction or addition, check whether the raw source already contains the corrected content. If not, append an SA entry. Only create an entry if there is a genuine gap.

7. **Update `index.md`** — add the new summary page and any new/updated pages to the catalog.

7b. **Update `state/source-state-NNN.md`** — find the existing row for this file (added by `clip`) and update columns 7-10: set Status = `ingested`, fill Wiki Pages with `[[PageName]]` links for every wiki page this source contributed to, set Ingested Date = today. If no row exists (file was not clipped via batch-clipper), append a new row with all columns filled.

8. **Append to `changelog.md`** — add CHG entries under today's date header. Follow the CHG Entry Format: `**Source File:**` with angle-bracket link, `**Change Type:**`, `**What Changed:**` bullets.

9. **Append to `log.md`** — one entry: `## [YYYY-MM-DD] ingest | <Source Title>` + 2–3 line summary.

A single source may touch 5–15 wiki pages. That's expected and correct.

### Re-ingest an Updated Source

Used when `ingest [filename]` is run on a file whose Status = `stale` in source-state.

1. Check `source-actions.md` for SA entries linked to this source. For each `🔲 Unresolved` entry, compare the Source Quote against the updated file. If gone or already applied → mark `✅ Resolved [YYYY-MM-DD]`.
2. Read the current wiki page to understand LLM-added content (warnings, cross-links, synthesis) — preserve it.
3. Read the updated raw file and add any net-new content to the wiki.
4. Update `index.md` if new wiki pages are created.
5. **Update `state/source-state-NNN.md`** — update the existing row: set Status = `ingested` (or `skipped`), update Ingested Date, update Wiki Pages if new pages were added.
6. Append to `log.md` with entry type `reingest`.

### Clip Sources

When the user says "clip":

**Purpose**: Register newly clipped files from batch-clipper into `state/source-state-NNN.md`. This is always the first step after a batch-clipper run — before any ingestion.

1. **Read `clipper-log.md`** — find the most recent batch entry. Extract the list of filenames clipped in that run.

2. **For each file in the batch**:
   a. Read the file's frontmatter from `raw/` — extract `source` (URL), `created`, `modified`, `clipped`.
   b. Derive `Root` from the file path: the immediate subfolder under `raw/` (e.g. `raw/Guides` for `raw/Guides/My Page.md`; `raw` if directly in `raw/`).
   c. Search all `state/source-state-NNN.md` files for an existing row matching this filename.
   d. **Not found** → append new row: fill columns 1-6, leave columns 7-10 blank.
   e. **Found, Status = `ingested` or `skipped`** → update columns 1-6, set Status = `stale`.
   f. **Found, Status = `stale`** → update columns 1-6 only (already queued for re-ingest).

3. **Update row ranges** in any source-state file headers that changed.

4. **Commit**: `clip: register N sources` (substitute actual count).

5. **Report**: "N new sources registered, M marked stale. Run `ingest` to process the queue."

### Audit Raw Sources

When the user says "audit":

**Purpose**: Systematically re-read every raw source and compare it against its wiki pages to find content gaps. Uses the same advisor-gated ingest workflow as `ingest` for maximum quality — each file gets full read → plan → advisor → write → commit treatment. New files (never ingested) are processed first; existing files are re-ingested sequentially.

**Resume state**: `state/source-state-NNN.md` is the source of truth. New files = not present in any source-state row. Already-ingested files are re-audited only on opt-in (step 3). No separate audit-state file is needed.

1. **Discover new files**:
   - Run `find raw/ -name "*.md" -not -path "raw/assets/*"` to get all raw files on disk.
   - For each, check whether its filename appears in any `state/source-state-NNN.md` row (Status = `ingested` or `skipped`).
   - **New files** (not in any source-state row) → queue for processing.
   - **Already-ingested files** → skip for now (offered in step 3).
   - Report: "N new files found, M already ingested."

2. **Process NEW files first** (if any), sequentially in batches of ~10:

   For each new file:
   a. Read the raw source file.
   a2. **Read associated assets** — scan the source for asset wikilinks and process each (same rules as `ingest` step 2: read images visually, attempt documents, note code files). Assets live in `raw/assets/[...hierarchy]/<Page Title>/` — check for that path after reading the source file.

   > 🚨 **ASSET GATE — mandatory before step b**: Run `find "raw/assets/<Page Title>" -type f 2>/dev/null`. If the directory exists and contains files, read every file before proceeding. Do not skip this during bulk batch processing — each file in the batch must pass this gate individually.

   b. Read all existing wiki pages that may be related (from `index.md` or by topic).
   c. Form a plan: which wiki pages to create or update, what SA entries are needed.
   d. **Call the advisor** (before writing anything) — the reviewer catches wrong target pages, missing cross-references, and blind spots.
   e. Execute the plan: write wiki pages, update `index.md`, append to `changelog.md` and `source-actions.md`.
   f. **SA dedup check**: before adding any SA entry, scan `source-actions.md` for an existing `🔲 Unresolved` entry with the same `**Source File:**` and `**Source Quote:**`. If a match exists → skip the new entry.
   g. **Update `state/source-state-NNN.md`** — append a row for this file (Status = `ingested`, Wiki Pages filled).
   h. Commit: `ingest: <source title>`
   i. Append to `log.md`: `## [YYYY-MM-DD] audit-ingest | <source title> (N gaps found)`

   After all new files are processed, report: "N new files ingested."

   > 🚨 **MANDATORY NEXT STEP**: After reporting, you MUST proceed to step 3. Do not commit and stop. Do not summarize and end the session. Step 3 is required even when all new files had no gaps.

3. **Prompt about EXISTING files**: "M existing sources could be re-audited for gaps. Re-ingest them now? (Y/N)"
   - If **N** → stop.
   - If **Y** → proceed to step 4.

4. **Process EXISTING files** sequentially in batches of ~10:

   For each existing file:
   a. Read the raw source file.
   a2. **Read associated assets** — scan the source for asset wikilinks and process each (same rules as `ingest` step 2: read images visually, attempt documents, note code files). Assets live in `raw/assets/[...hierarchy]/<Page Title>/` — check for that path after reading the source file.

   > 🚨 **ASSET GATE — mandatory before step b**: Run `find "raw/assets/<Page Title>" -type f 2>/dev/null`. If the directory exists and contains files, read every file before proceeding. Do not skip this during bulk batch processing — each file in the batch must pass this gate individually.

   b. Read all existing wiki pages linked to this source (from `sources:` frontmatter or `index.md`).
   c. Form a plan: what content is in raw but absent or understated in the wiki.
   d. **Call the advisor** (before writing anything).
   e. If gaps found: update wiki pages, append to `changelog.md` and `source-actions.md` (with SA dedup check). Update the file's source-state row Wiki Pages column if new pages were added.
   f. If no gaps: note "no gaps" — batch the commit with the next file.
   g. Commit after each file that had changes: `reingest: <source title>` (batch no-gap files: `reingest: audit pass — <source A>, <source B>, ...`)

5. **Report** a summary after each batch: sources reviewed this run, gaps found and fixed (or "no gaps"), SA entries added, sources remaining.

6. **Append to `log.md`** after each batch: `## [YYYY-MM-DD] audit | Sources N–M reviewed (X gaps found, Y remaining)`

> ⚠️ Large raw files (300KB+) should be read in chunks or via grep for headings — do not attempt to read the entire file in one call if it exceeds ~100KB.

### Initialize Repo

When the user says "init":

1. Check if any one-time setup is needed for this project (e.g., git submodule initialization if the project uses one). If none → report "Already initialized. Nothing to do."
2. Verify the `wiki/` directory exists. Create it if missing.
3. Confirm `raw/` exists and is ready to receive source files.
4. Remind the user to open this repo as an Obsidian vault for wiki browsing, and that they can type `help` to see all available commands.

### Answer a Query

When the user asks a question:

1. **Read `index.md`** to find relevant pages.
2. **Read relevant wiki pages** (not raw sources — the wiki is the compiled knowledge).
3. **Synthesize an answer** with citations to wiki pages. Deliver in the format most useful for the question — prose, comparison table, step-by-step list, etc.
4. **Offer to file the answer** as a new `wiki/synthesis/` page if it's meaningful analysis worth keeping.

### Lint the Wiki

When the user says "doctor":

1. **Check for broken WikiLinks** (run first — highest priority):
   - Collect all `[[WikiLink]]` references across wiki pages AND `index.md`: `grep -roh '\[\[[^\]]*\]\]' wiki/ index.md`
   - For each unique link target, check whether a file with that exact name (plus `.md`) exists anywhere in `wiki/`
   - A link is broken if no file matches the exact display name
   - **Auto-fix**: for each broken link, check if a file exists whose H1 title matches the link text. If yes → rename using `git mv`. If no match → flag as a genuinely missing page.
   - Also check the reverse: for each wiki file, verify its filename exactly matches its H1 title. If not → rename using `git mv`.
   - After fixes, report: N broken links fixed, M genuinely missing pages (if any).

2. **Check README.md is in sync**:
   - Read `README.md` and extract all wiki page references and index section names
   - For each referenced page, verify the file exists in `wiki/`
   - Verify each section name exists in `index.md`
   - **Auto-fix**: update any stale page paths or section names in README
   - Report: N README references checked, M stale references fixed (if any).

3. Scan all wiki pages for:
   - Contradictions between pages
   - Stale claims superseded by newer sources
   - Orphan pages (no inbound links)
   - Concepts mentioned but lacking their own page
   - Missing cross-references between related pages
   - Data gaps that could be filled with existing raw sources not yet ingested, or via web search

4. **Check source-state integrity** — scan all `state/source-state-NNN.md` files for:
   - Duplicate rows (same filename in multiple rows)
   - Invalid Status values (anything other than `ingested`, `skipped`, `stale`, or blank)
   - Rows missing required columns
   - Pagination invariants: row count per file ≤ 500, row ranges in headers match actual content
   - Report any issues found.

5. Check all `🔲 Unresolved` entries in `source-actions.md` — validate whether the Source Quote still exists verbatim in the source file. Mark `✅ Resolved [YYYY-MM-DD]` for any entries whose quote is gone or whose desired change is already present.

6. Report all findings as a list with suggested actions.

7. Suggest new questions or sources to investigate.

---

## Index & Log Conventions

**index.md** — organized by category. Each entry: `- [[Page Title]] — one-line summary`. Updated on every ingest.

**log.md** — append-only. Each entry header: `## [YYYY-MM-DD] operation | Description`. Operation types: `ingest`, `reingest`, `query`, `audit`, `doctor`. New entries go at the **top** (reverse chronological). Parseable with: `grep "^## \[" log.md | head -5`

**changelog.md** — wiki change history. CHG entries grouped under date headers (`## YYYY-MM-DD`), reverse chronological. CHG numbers are sequential. Every `###` heading in `changelog.md` must be a `CHG-NNN` entry — no other content should use `###`.

**source-actions.md** — source update tracker. SA entries (SA-NNN) each contain a Source Quote anchored to the raw file at ingest time, and a Desired Change describing what the source should ideally say. SA entries track every gap regardless of whether you own the source — what you do with them is up to you. New entries go at the **top** (reverse chronological).

**state/source-state-NNN.md** — paginated source registry. One row per raw file. `clip` populates provenance columns (1-6); `ingest` populates wiki attribution columns (7-10). See Source State File Format above.

### Automatic Archiving (file length management)

**Trigger**: Before appending to any of the files below, check its line count. If it exceeds the threshold, archive entries first, then append.

| File | Threshold | Archive file | What to move |
|------|-----------|--------------|--------------|
| `changelog.md` | 3 000 lines | `archive/changelog-archive-NNN.md` | Oldest ~1 000 lines (bottom block) |
| `log.md` | 1 000 lines | `archive/log-archive-NNN.md` | Oldest ~400 lines (bottom block) |
| `source-actions.md` | 2 000 lines | `archive/source-actions-archive-NNN.md` | All `✅ Resolved` entries |

**Always move complete entries — never split mid-block.**

**Archive procedure**:
1. Check line count: `wc -l <file>`
2. If over threshold → identify entries to move
3. Next archive sequence number: `ls archive/<prefix>-archive-*.md 2>/dev/null | wc -l` (start at `001` if none)
4. Create archive file with header: `# <Filename> — Archive NNN` + `Entries moved from <file> on <date>. Do not edit.`
5. Append moved entries to archive file
6. Remove those entries from the main file
7. Add note at bottom of main file's header: `> Older entries archived in <archive-filename> and prior.`
8. Commit both files: `chore: archive <file> overflow → <archive-filename>`

### CHG Entry Format

```
### CHG-NNN — `wiki/path/to/page.md` — Short Title

**Source File:** [filename.md](<raw/path/to/filename.md>)
**Change Type:** New Page | Content Correction | New Content Added | Synthesis | Stub | SA Stamps

**What Changed:**
- Bullet list of wiki-local changes only
```

**Rules:**
- `**Source File:**` — display name is the filename only, href is the full `raw/` path in angle brackets. Multiple sources: comma-separated links on the same line.
- For SA Stamps entries (recording files evaluated as not wiki-worthy): omit `**Source File:**`; use `**Change Type:** SA Stamps` and list dispositions in bullets.
- For entries that touch meta-files only (`CLAUDE.md`, `index.md`, etc.): use the file path as the title; `**Source File:**` is not needed.
- Every `###` heading in `changelog.md` must be a `CHG-NNN` entry. No other content (e.g. batch summaries, verification notes) should use `###`.

### SA Entry Format (source-actions.md)

```
### SA-NNN — `Source Document Name` — Short Title

**Type:** `content-correction` | `content-addition` | `terminology` | `structural` | `outdated` | `archive` | `conflict`
**Source File:** [filename.md](<raw/path/to/filename.md>) | N/A

**Source Quote:**
> Exact text from the source that needs changing.
> Or: `[missing — content does not exist in source]` for additions.
> Or: N/A for structural entries.

**Desired Change:**
What the source document should ideally say. Be specific.

**Status:** [✅ Resolved YYYY-MM-DD | 🔲 Unresolved]
```

### SA Type Reference

| Type | When to Use |
|------|-------------|
| `content-correction` | Source has wrong or outdated information; wiki has the correct version |
| `content-addition` | Source is missing content that should be there |
| `terminology` | Source uses wrong or outdated terminology |
| `structural` | Organizational issue — wrong location, wrong doc, incomplete structure |
| `outdated` | Source is stale or superseded by newer sources or wiki synthesis |
| `archive` | Source is no longer relevant and should be removed from `raw/` entirely |
| `conflict` | Two sources directly contradict each other; human resolution needed |

### Source Archive Process

When a raw source should be retired (flagged via `archive` SA entry):

1. Create SA entry of type `archive` documenting why it should be retired.
2. **Do not move or delete `raw/` files yourself** — the user decides when to act on SA entries.
3. When the user resolves the SA entry: move the file from `raw/` to `raw/archive/` (create the directory if needed).
4. Update the relevant `state/source-state-NNN.md` row: set Status = `skipped`, set Skipped Reason = `archived`.
5. Update any wiki pages that cited the archived source — remove the source from their `sources:` frontmatter if other sources cover the same content.
6. Commit: `chore: archive source — <filename>`

---

## Git Commit Conventions

> 🚨 **Commit immediately after each operation.** Every ingest, reingest, wiki edit, SA addition, or SA resolution is its own commit. The changelog, log, and source-actions updates must be in the same commit as the wiki changes they document.

### Commit message format
```
<type>: <short description>

<optional body — bullet list of what changed>
```

**Types:** `clip:` | `ingest:` | `reingest:` | `wiki:` | `schema:` | `sa:` | `chore:`

- `clip:` — source registration after a batch-clipper run (e.g. `clip: register 12 sources`); no visibility suffix

---

## Style Notes

- Write wiki pages for someone who knows the broader domain but may be new to this specific topic.
- Be concrete: include specific names, values, steps, and examples when known.
- Flag uncertainty: if a source is ambiguous or outdated, note it with `> ⚠️ Note: ...`.
- When multiple sources contradict, note the contradiction explicitly on the relevant page rather than silently picking one.
- **Do not use em dashes (`—`).** Use a regular hyphen with spaces (` - `) instead, or restructure the sentence.

---

## AI Review Checkpoints

For LLM assistants that support a reviewer or advisor tool:

### When to call the reviewer

1. **Before writing** — after reading all raw sources for a batch, but before creating or editing any wiki pages. The reviewer catches content placement errors (wrong target page), structural issues, and blind spots that are invisible mid-execution.

2. **After committing** — once the commit is durable, call the reviewer to catch anything missed (numbering gaps, broken WikiLinks not yet verified, pages that should have been updated but weren't).

3. **When stuck** — if a source is ambiguous about where content belongs, or two pages seem like valid targets, ask the reviewer before proceeding.

### What to ask

You do not need to pass parameters — the reviewer sees the full conversation. Just call it. Expect a short enumerated response (~100 words) with specific corrections.

### Exemptions

Single-file reactive fixes (e.g. fixing a broken WikiLink found during `doctor`, resolving one SA entry) do not require a reviewer call.
