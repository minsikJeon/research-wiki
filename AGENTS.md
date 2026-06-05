# AGENTS.md — MSR Research Wiki Schema

This file tells any LLM agent (Claude Code, Codex, OpenCode, etc.) how to operate
this wiki. The wiki is a persistent, compounding knowledge base for a CMU
**Master of Science in Robotics (MSR)** student. Primary fields:
**computer vision** and **robotics** — with everything adjacent that touches
them (foundation models for perception/control, manipulation, SLAM, video
understanding, embodied AI, world models, etc.). See [[llm-wiki-pattern]]
for the high-level idea.

You (the agent) own the `wiki/` layer entirely. The human owns sourcing and
direction. Read this file at the start of every session.

---

## 1. Architecture (three layers)

```
research-wiki/
├── AGENTS.md            ← this schema
├── llm-wiki-pattern.md  ← the pattern idea (immutable reference)
├── index.md             ← content-oriented catalog of the wiki
├── log.md               ← chronological ingest/query/lint log
├── overview.md          ← the evolving thesis / north star
├── raw/                 ← immutable source material (LLM reads, never writes)
│   ├── papers/          ← PDFs + converted markdown
│   ├── articles/        ← clipped web articles
│   ├── repos/           ← code repo notes, READMEs, doc snapshots
│   ├── datasets/        ← dataset cards, schemas
│   ├── notes/           ← personal notes, meeting notes, drafts
│   └── assets/          ← images downloaded from clipped articles
└── wiki/                ← LLM-owned, interlinked markdown pages
    ├── sources/         ← one summary per ingested raw source
    ├── concepts/        ← abstract ideas, phenomena, framings
    ├── methods/         ← algorithms, models, techniques, pipelines
    ├── entities/
    │   ├── people/      ← researchers, practitioners
    │   ├── orgs/        ← labs, universities, companies
    │   ├── tools/       ← libraries, frameworks, software artifacts
    │   ├── venues/      ← conferences, journals, workshops
    │   └── datasets/    ← benchmarks, corpora, mined repos
    ├── questions/       ← open research questions, evolving theses
    └── comparisons/     ← synthesis tables, cross-source analyses
```

Rules:
- **Never modify anything in `raw/`.** It is the source of truth.
- **Everything in `wiki/` is yours to create, edit, rename, or delete** (with
  the user's awareness when deleting).
- Keep `AGENTS.md`, `index.md`, `log.md`, `overview.md` in the repo root.

---

## 2. Page types & frontmatter

Every wiki page has YAML frontmatter (Dataview-ready). Always populate
`created` and `updated`; update `updated` on every edit. Use ISO dates
(`YYYY-MM-DD`).

### Common fields (every page)

```yaml
type:        # source | concept | method | entity | question | comparison
title:       # human-readable title
status:      # stub | growing | mature   (sources use ingested|partial|skim)
tags:        # [kebab-case, topical-tags]
sources:     # [[wiki-link]] list of source pages this page draws from
related:     # [[wiki-link]] list of related wiki pages (any type)
created:     # YYYY-MM-DD
updated:     # YYYY-MM-DD
```

### Source pages — `wiki/sources/<slug>.md`

One per ingested raw item. The canonical entry point from the raw layer
into the synthesized layer.

```yaml
type: source
source_type:  # paper | article | repo | notes | dataset
title:
authors:      # [Last, First; Last, First]   (papers/articles only)
year:         # int
venue:        # MSR, ICSE, blog name, etc.   (optional)
url:          # canonical URL or DOI         (optional)
raw_path:     # relative path under raw/, e.g. papers/foo-2024.pdf
status:       # ingested | partial | skim    (source pages use these instead of stub/growing/mature)
tags:
related:
created:
updated:
```

Body sections (in order, omit if not applicable):
1. **TL;DR** — 2-4 sentences.
2. **Why it matters** — relevance to current threads in [[overview]].
3. **Key claims** — bulleted, each with a page/section citation.
4. **Methods** — what they did, briefly.
5. **Results** — headline numbers.
6. **Limitations / open questions** — what to be skeptical of.
7. **Connections** — `[[wiki-links]]` to concepts, methods, entities, prior sources. Note contradictions explicitly.
8. **Citation** — full bibliographic ref for copy-paste.

### Concept pages — `wiki/concepts/<slug>.md`

Abstract ideas synthesized across sources (e.g. `flaky-tests`,
`code-review-as-signal`, `survivorship-bias-in-mined-repos`).

Body: definition → why it matters in MSR → variants/sub-concepts →
evidence across sources (cite `[[sources/...]]`) → contested points → open
questions.

### Method pages — `wiki/methods/<slug>.md`

Concrete algorithms, models, or pipelines (e.g. `bert4code`,
`spr-static-bug-detection`, `commit2vec`).

Body: one-line summary → inputs/outputs → how it works → where it's been
applied (cite sources) → known limitations → related methods.

### Entity pages — `wiki/entities/{people,orgs,tools,venues,datasets}/<slug>.md`

Recurring named things. Use the additional field:

```yaml
entity_type:  # person | org | tool | venue | dataset
```

Body varies by entity_type:
- **person**: affiliation, main contributions, papers in this wiki.
- **org**: type, focus, members in this wiki, sources from them.
- **tool**: what it does, who built it, papers that use it, links.
- **venue**: scope, notable papers in this wiki.
- **dataset**: size, schema, construction method, papers that use it,
  known biases.

### Question pages — `wiki/questions/q-<slug>.md`

Open research questions or evolving theses you're working through. Living
documents — update as new evidence comes in.

Body: the question → why it matters → what we know → what we don't →
relevant sources → tentative position → next things to read.

Filename prefix: `q-`.

### Comparison pages — `wiki/comparisons/cmp-<slug>.md`

Synthesis across sources: tables, ranked lists, methodological
contrasts. Often created as a side effect of answering a query (see §4.2
— file queries back as pages).

Filename prefix: `cmp-`.

---

## 3. File & link conventions

**Slugs.** Kebab-case, lowercase, ASCII only. Examples:
- Paper: `zholus-2025-tapnext`  (first-author-year-shorttitle)
- Article: `<slug-of-title>`
- Concept: `<noun-phrase>`  e.g. `point-tracking`, `masked-token-decoding`
- Method: `<canonical-name>`  e.g. `tapnext`, `cotracker`, `diffusion-policy`
- Person: `<firstname-lastname>` e.g. `carl-doersch`, `pieter-abbeel`
- Org: short canonical name, e.g. `cmu-ri`, `google-deepmind`, `mit-csail`
- Tool: lowercase package name, e.g. `pytorch`, `ros2`, `jax`
- Venue: short acronym, e.g. `cvpr`, `iccv`, `corl`, `rss`, `neurips`
- Dataset: `<name>-dataset` e.g. `tap-vid-dataset`, `kubric-dataset`
- Question: `q-<short-question>`
- Comparison: `cmp-<subject>`

**Wiki-links.** Use bare Obsidian-style `[[slug]]` (no path, no `.md`).
Filenames must be unique across the vault — collisions break links. When
referring to a concept by a different phrase, use `[[slug|display text]]`.

**Backlinks.** Don't write them manually. Obsidian computes them; the
graph view shows them.

**Tags.** Kebab-case, topical. Pick from existing tags before inventing
new ones — check [[index]] for current tag set. Typical CV/robotics tags:
`point-tracking`, `optical-flow`, `manipulation`, `imitation-learning`,
`world-model`, `vlm`, `vla`, `slam`, `nerf`, `gaussian-splatting`,
`diffusion-policy`, `state-space-model`, `transformer`, `benchmark`,
`survey`, `tool-paper`.

**Images.** Stored in `raw/assets/`. Reference with relative paths.
LLMs can't read inline images in one pass — read the text first, then
view referenced images separately when context demands it.

---

## 4. Workflows

### 4.1 Ingest

Triggered when the user drops a new file under `raw/` and says some
variant of "ingest this." Default to **one source at a time** with the
user in the loop. Batch ingest only when explicitly asked.

Steps:
1. **Read the raw file.** For papers, read the full PDF (use the Read
   tool's `pages` parameter for PDFs >10 pages).
2. **Discuss takeaways** with the user before writing. Surface anything
   surprising, contested, or that contradicts existing wiki pages.
3. **Check the existing wiki** before creating new pages: grep
   `wiki/` and read `index.md` to see what concepts/entities/methods
   already exist. Prefer extending existing pages over creating
   parallel ones.
4. **Write the source page** under `wiki/sources/<slug>.md`.
5. **Touch related pages.** A single source typically updates 5-15 wiki
   pages: concept pages it advances, entity pages for authors/tools/datasets
   it introduces, comparison pages it adds a row to, question pages it
   shifts. Create new pages where genuinely new things are introduced.
6. **Flag contradictions explicitly.** If the new source contradicts a
   claim on an existing page, edit the page to note the contradiction
   (don't silently overwrite). Link both sources.
7. **Update [[index]]** with the new source and any new pages created.
8. **Append to [[log]]** using the format in §5.
9. **Update `updated:` frontmatter** on every page touched.
10. **Briefly summarize** to the user what changed (which pages,
    rough why) before yielding.

### 4.2 Query

Triggered by any question against the wiki.

Steps:
1. **Read [[index]] first.** Identify candidate pages.
2. **Read the candidate pages.** Follow `[[links]]` as needed. For large
   wikis, also grep `wiki/` for keywords the index might not surface.
3. **Synthesize an answer with `[[wiki-link]]` citations.** When citing a
   specific claim, cite the source page, not just the concept page.
4. **Offer to file the answer back as a wiki page** when the answer is
   substantive (a comparison, a synthesis, an analysis). File comparisons
   under `wiki/comparisons/` and open threads under `wiki/questions/`.
   Don't let valuable explorations decay in chat.
5. **Append a query entry to [[log]]** if the query led to a new filed
   page or surfaced something worth remembering.

### 4.3 Lint

Triggered when the user asks for a "lint pass," "health check," or
similar. Read the whole wiki and report:

- **Contradictions** — pages making incompatible claims.
- **Stale claims** — pages superseded by newer sources, not yet updated.
- **Orphans** — pages with zero inbound `[[links]]` from other wiki pages.
- **Missing pages** — concepts/entities/methods referenced in source
  pages but lacking their own page. Especially: tools or datasets
  mentioned in 2+ sources.
- **Tag drift** — near-duplicate tags (e.g. `llm-code` vs `llm4code`).
- **Broken `[[links]]`** — wiki-links pointing to nonexistent slugs.
- **Frontmatter gaps** — pages missing required fields, or `updated:`
  older than their last content edit.
- **Open threads worth pursuing** — questions in `wiki/questions/`
  where new sources could move the needle. Suggest specific searches.

Output as a checklist. Don't auto-fix — let the user direct which
issues to address.

---

## 5. Log format

`log.md` is append-only. Each entry starts with a level-2 heading using
this format so `grep "^## \[" log.md | tail -10` shows the last 10:

```
## [YYYY-MM-DD] <op> | <subject>

<one-paragraph note: what was done, which pages touched, anything notable>
```

Where `<op>` is one of: `ingest`, `query`, `lint`, `refactor`, `note`.

Examples:
```
## [2026-05-24] ingest | Zholus 2025 — TAPNext

Added wiki/sources/zholus-2025-tapnext.md. Created wiki/concepts/point-tracking.md
and wiki/methods/tapnext.md. Updated wiki/entities/datasets/tap-vid-dataset.md
(this source is the introducing paper). New entity: wiki/entities/people/carl-doersch.md
(recurring across the TAP series).

## [2026-05-24] lint | weekly

3 contradictions, 7 orphans, 2 tag drifts. See chat for details; user
to triage.
```

---

## 6. Boundaries — what NOT to do

- Don't modify `raw/`.
- Don't auto-ingest without the user confirming the source is in scope.
- Don't silently overwrite contested claims — flag them.
- Don't invent citations. If you can't find a source, say so.
- Don't expand schema or invent new top-level directories without
  proposing the change first and updating this file.
- Don't write summary pages that just restate the raw source — extract
  the *load-bearing* claims and connect them to the rest of the wiki.
- Don't create pages for one-off mentions. Wait until something appears
  in 2+ sources or the user explicitly asks for a page.
