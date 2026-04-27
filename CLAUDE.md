# [TOPIC] Brain — Schema

This is the configuration file for the [TOPIC] second brain. It tells you (the LLM) exactly how this wiki is structured, what the conventions are, and what workflows to follow. Read this at the start of every session.

> **Setup note:** Replace every `[TOPIC]` placeholder in this file with your actual topic (e.g., "WCNP", "Kafka Platform", "Design System"). Replace `[SOURCE TYPE]` with your primary source type (e.g., "Confluence", "GitHub", "internal docs").

---

## Domain

**Topic**: [TOPIC] — brief description of what this brain covers.

**Purpose of this wiki**: Build and maintain a comprehensive, interlinked knowledge base about [TOPIC] — its architecture, concepts, use cases, and operational patterns. The wiki is the compiled, synthesized, always-current artifact. Raw sources are the source of truth but are never modified.

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

## Wiki Page Format

Every wiki page should follow this structure:

```markdown
---
tags: [concept|entity|integration|use-case|operation|roadmap|synthesis]
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

---

## Source Ownership

**Sources editable:** Yes

> Set to `Yes` if your team owns the source documents and can apply changes to them.
> Set to `No` if sources are read-only (third-party docs, another team's pages, external references).
>
> - `Yes` → the LLM creates SA entries as action items to update the source
> - `No` → the LLM skips SA entries; gaps are noted inline on wiki pages with `> ⚠️ Note: source does not reflect this...` instead

---

## Key Entities (seed list — fill in for your domain)

> Replace these with the key named systems, services, teams, and products in your topic area.

- **[Entity 1]** — brief description
- **[Entity 2]** — brief description
- **[Entity 3]** — brief description

---

## Key Concepts (seed list — fill in for your domain)

> Replace these with the core concepts, mechanics, and terms in your topic area.

- **[Concept 1]** — brief description
- **[Concept 2]** — brief description
- **[Concept 3]** — brief description

---

## Maintenance Requirements (ALWAYS enforce — no exceptions)

These rules apply to **every** wiki change, regardless of what triggered it:

> 🚨 **BEFORE EVERY COMMIT** — run this mental checklist:
> - Did I add a CHG entry to `changelog.md` for every wiki file I touched? (one CHG per file per session of changes)
> - Did I evaluate whether any change needs an SA entry in `source-actions.md`?
> - If the changed page is in an external-facing section of `index.md`, did I create an SA entry and mark it `**Editable:** Yes`?
> - Did I update `log.md` if this was an ingest, reingest, audit, or doctor operation?
> - If I renamed a wiki page or reorganized `index.md` sections, did I update `README.md`?
>
> **Do not commit without completing this checklist.** The user must never have to ask for these — they are part of every change, not an afterthought.

1. **`changelog.md` must be updated** for every wiki page creation or meaningful edit — including diagram changes, prose corrections, and structural edits. Add a CHG entry under today's date header. CHG numbers are sequential. **This includes small changes.** If you touched a wiki file, it gets a CHG entry in the same commit, not a later one.

2. **`source-actions.md` must be updated** whenever the wiki contains information that the raw source does not yet reflect. This includes:
   - Corrections to factual errors in source documents
   - New sections added to the wiki that should be reflected in the source
   - Content synthesized across multiple sources that clarifies or contradicts any single source
   - If the raw source already has the content correctly → no SA entry needed
   - Mark `**Editable:** Yes` if you own the source and can apply the change; `No` if the source is read-only

3. **`log.md` must be updated** after every operation (ingest, reingest, audit, doctor) with a timestamped entry.

4. **`README.md` must be checked** whenever a wiki page is renamed or `index.md` sections are reorganized. `README.md` may contain hardcoded page references and section names. Update it in the same commit — never let it drift.

These files are the audit trail of the wiki. Skipping any of them means the wiki has undocumented changes that cannot be traced or reproduced.

---

## Commands

These keyword triggers are recognized when a message starts with them.

### `ingest [filename]`
Ingest a raw source file into the wiki. Auto-detects whether this is a first-time ingest or a re-ingest:
- `[filename]` is the name of a file in `raw/`
- **First-time**: if the filename does not appear in the `index.md` source list → run the **Ingest a Source** workflow
- **Re-ingest**: if the filename already appears → run the **Re-ingest an Updated Source** workflow. Check `source-actions.md` for SA entries linked to this source and resolve any whose Source Quote is no longer present in the updated file.
- Example: `ingest my-topic-overview`

### `doctor`
Run the wiki health check. Follow the **Lint the Wiki** workflow below.

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
Batch re-review of raw sources against the wiki using parallel agents. Reads `audit-state.md` to resume from where the last run stopped. When all sources are audited, prompts the user to optionally restart a full pass.
- Example: `audit`

### `init`
First-time setup. Run once after cloning — safe to run again (idempotent). Follow the **Initialize Repo** workflow.
- Example: `init`

### `help`
List all available commands with a one-line description of each.

---

## Workflows

### Ingest a Source

When the user says "ingest [filename]":

1. **Read** the source file from `raw/`.
2. **Discuss** key takeaways with the user — what's new, what's important, what's surprising.
3. **Write a summary page** in the appropriate `wiki/` subfolder. Name it with a clear display name (e.g., `wiki/concepts/Core Architecture.md`).
4. **Update existing wiki pages** — any entities, concepts, or integrations touched by this source should be updated or created.
5. **Populate `source-actions.md`** — for each wiki correction or addition, check whether the raw source already contains the corrected content. If not, append an SA entry:
   - **Source Quote** — exact text from the raw file that needs changing, or `[missing — content does not exist in source]` for additions
   - **Desired Change** — what the source document should say
   - **Source File** — the raw filename
   - **Ingested** — today's date
   - Only create an entry if there is a genuine gap.
6. **Update `index.md`** — add the new summary page and any new/updated pages to the catalog.
7. **Append to `changelog.md`** — add CHG entries under today's date header.
8. **Append to `log.md`** — one entry: `## [YYYY-MM-DD] ingest | <Source Title>` + 2–3 line summary.

### Re-ingest an Updated Source

1. The user replaces the raw file in `raw/` with the updated version (same filename).
2. Check `source-actions.md` for SA entries linked to this source. For each `🔲 Unresolved` entry, compare the Source Quote against the updated file. If gone or already applied → mark `✅ Resolved [YYYY-MM-DD]`.
3. Read the current wiki page to understand LLM-added content (warnings, cross-links, synthesis) — preserve it.
4. Read the updated raw file and add any net-new content to the wiki.
5. Update `index.md` if new wiki pages are created.
6. Append to `log.md` with entry type `reingest`.

### Audit Raw Sources

When the user says "audit":

**Purpose**: Systematically re-read every raw source and compare it against its wiki pages to find content gaps.

1. **Read `audit-state.md`**. Check the Pending section:
   - **Pending sources exist** → resume from that list.
   - **Pending is empty** → all sources audited. Ask: "All N sources are audited. Run a full audit again?" If yes → reset Pending and proceed. If no → stop.

2. **Reconcile `raw/` against `audit-state.md`**:
   - Run `ls raw/` and compare against audit-state entries (excluding `assets/`)
   - **New files** on disk but not in audit-state → add to Pending automatically; ingest them before auditing
   - **Stale filename pointers** → fix silently
   - **Missing files** → mark as `(file removed — skip)`

3. **Read `index.md`** to get ingested sources and their wiki pages.

4. **For new files from step 2**: run the full Ingest workflow before proceeding.

5. **Divide Pending sources** into batches of ~10.

6. **Launch one Agent per batch** (single parallel call). Each agent:
   - Reads each raw file and its corresponding wiki pages
   - Finds gaps (content in raw absent or understated in wiki)
   - Updates wiki pages directly
   - Preserves all LLM-added content (warnings, cross-links, synthesis)
   - Returns a report: source name, gaps found, wiki pages updated, SA entries needed

7. **Collect agent reports**. For each completed source:
   - Append any SA entries to `source-actions.md`
   - Move source from Pending → Audited in `audit-state.md` with date and gap count

8. **Append to `log.md`**: `## [YYYY-MM-DD] audit | Sources N–M reviewed (X gaps found, Y remaining)`

9. **Commit** after each batch.

> ⚠️ Large raw files (300KB+) should be read in chunks — do not attempt to read the entire file in one call if it exceeds ~100KB.

### Initialize Repo

When the user says "init":

1. Check if any one-time setup is needed for this project (e.g., git submodule initialization if the project uses one). If none → report "Already initialized. Nothing to do."
2. Verify the `wiki/` subdirectories exist. Create any that are missing.
3. Confirm `raw/` exists and is ready to receive source files.
4. Remind the user to open this repo as an Obsidian vault for wiki browsing, and that they can type `help` to see all available commands.

### Answer a Query

When the user asks a question:

1. **Read `index.md`** to find relevant pages.
2. **Read relevant wiki pages** (not raw sources — the wiki is the compiled knowledge).
3. **Synthesize an answer** with citations to wiki pages.
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
   - Data gaps that could be filled with existing raw sources not yet ingested

4. Check all `🔲 Unresolved` entries in `source-actions.md` against their local raw source files — validate whether the Source Quote still exists verbatim. Mark `✅ Resolved [YYYY-MM-DD]` for any entries whose quote is gone or whose desired change is already present.

5. Report all findings as a list with suggested actions.

6. Suggest new questions or sources to investigate.

---

## Index & Log Conventions

**index.md** — organized by category. Each entry: `- [[Page Title]] — one-line summary`. Updated on every ingest.

**log.md** — append-only. Each entry header: `## [YYYY-MM-DD] operation | Description`. Operation types: `ingest`, `reingest`, `query`, `audit`, `doctor`. New entries go at the **top** (reverse chronological).

**changelog.md** — wiki change history. CHG entries grouped under date headers (`## YYYY-MM-DD`), reverse chronological. CHG numbers are sequential.

**source-actions.md** — source update tracker. SA entries (SA-NNN) each contain a Source Quote anchored to the raw file at ingest time, and a Desired Change describing what the source should ideally say. Entries marked `Editable: Yes` are actionable if you own the source; `Editable: No` entries document known gaps for awareness.

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

**Source:** `Raw Source Filename`
**Change Type:** Brief label (e.g. Correction, New Page, Synthesis)

**What Changed:**
- Bullet list of wiki-local changes only
```

### SA Entry Format (source-actions.md)

```
### SA-NNN — `Source Document Name` — Short Title

**Type:** `content-correction` | `content-addition` | `terminology` | `structural`
**Source File:** `raw/filename.md` | N/A
**Ingested:** YYYY-MM-DD

**Source Quote:**
> Exact text from the source that needs changing.
> Or: `[missing — content does not exist in source]` for additions.
> Or: N/A for structural entries.

**Desired Change:**
What the source document should ideally say. Be specific.

**Status:** [✅ Resolved YYYY-MM-DD | 🔲 Unresolved]
```

> SA entries are only created when **Sources editable: Yes** in the Source Ownership section. When sources are read-only, gaps are noted inline on wiki pages instead.

---

## Git Commit Conventions

> 🚨 **Commit immediately after each operation.** Every ingest, reingest, wiki edit, SA addition, or SA resolution is its own commit. The changelog, log, and source-actions updates must be in the same commit as the wiki changes they document.

### Commit message format
```
<type>: <short description>

<optional body — bullet list of what changed>
```

**Types:** `ingest:` | `reingest:` | `wiki:` | `schema:` | `sa:` | `chore:`

---

## Style Notes

- Write wiki pages for someone who knows the broader domain but may be new to this specific topic.
- Be concrete: include config snippets, API names, field names, state names when known.
- Flag uncertainty: if a source is ambiguous or outdated, note it with `> ⚠️ Note: ...`.
- When multiple sources contradict, note the contradiction explicitly on the relevant page rather than silently picking one.
- **Do not use em dashes (`—`).** Use a regular hyphen with spaces (` - `) instead, or restructure the sentence.
