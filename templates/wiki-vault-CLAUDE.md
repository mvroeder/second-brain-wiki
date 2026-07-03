# CLAUDE.md — Schema for {{WIKI_VAULT}} vault

> This file is the schema. You read it on every session and follow it. The user does not edit the wiki by hand — you do.
> Reference: Karpathy's "LLM Wiki" pattern (gist 442a6bf555914893e9891c11519de94f) — three layers, three verbs.
> Layout: this is a SEPARATE Obsidian vault from `{{PERSONAL_VAULT}}/`. You read across vaults; you write only here.

## Identity

You are the maintainer of {{OWNER}}'s second brain. The vaults live under
`{{VAULT_BASE}}/` (two parallel vaults: the wiki and the personal vault). Treat the
wiki as production: every write is durable, every link is real, every quote is sourced.

## Vault layout — two vaults, hard separation

```
{{VAULT_BASE}}/
├── {{PERSONAL_VAULT}}/           READ-ONLY for you
│   ├── Knowledge/          long-form article notes (you read)
│   ├── Briefings/          meeting/call notes (you read)
│   ├── Daily note/         daily logs (you read for /graduate)
│   ├── Ideas/              raw captures (you read for /graduate)
│   └── ...                 (other folders: you read if relevant, never write)
│
└── {{WIKI_VAULT}}/               WRITE-ALLOWED for you (this vault)
    ├── CLAUDE.md           this file
    ├── .claude/commands/   slash commands
    ├── inbox/              fresh captures: Web Clipper, raw paste-dumps, automated captures
    ├── Sources/            full-text ingested articles (from /ingest-url)
    ├── Topics/             atomic concept pages
    ├── People/             one file per person referenced 2+ times
    ├── Projects/           one file per active project
    ├── Maps/               Maps of Content (MOCs) - one navigation hub per domain (optional)
    ├── _audit/             /lint-wiki output reports
    ├── Index.md            single source of truth: what exists, where, when last touched
    └── hot.md              lean live-state cache: read FIRST every session, rewritten as the final step of every mutating verb
```

## hot.md - the live-state cache

`hot.md` (vault root) is the lean "where are we right now" cache. It is the FIRST thing you read each session, before acting: it tells you the current state and open threads without scanning Sources. It is separate from `Index.md` (the catalog of what exists) and from "Maintenance log" in Index.md (permanent history). hot.md is the present, Maintenance log is the past.

**Read:** at session start, read `hot.md` first. This instruction lives in CLAUDE.md, which survives context compaction, so re-read hot.md after any compaction.

**Write (keep it current):**
- Every mutating verb (`/ingest-url`, `/process-inbox`, `/lint-wiki`, `/graduate`, `/synthesize`) rewrites hot.md as its FINAL step.
- A free-form session that changed wiki content without running a verb refreshes hot.md before finishing.
- Keep it under 250 words, three sections: `## Just happened` (3-6 bullets of substantive changes), `## Open threads` (RESUME HERE: pending decisions, half-finished threads), `## Active clusters` (live topic clusters). Update the `As of: YYYY-MM-DD` header.

**Session-state handovers: hot.md only.** The live "where are we right now" handover is `hot.md` and nothing else. Never create ad-hoc session-handover files inside the wiki, not in `inbox/`, not in `Sources/`. Two lookalikes are *not* banned: (a) a project-local "read me first" note a session leaves beside a self-contained work package *outside* this vault (e.g. under `../{{PERSONAL_VAULT}}/`); (b) a capture whose title merely contains "Handover" but whose content is knowledge transfer (a vocabulary or domain primer), that is ordinary inbox material, classified on content. If a real session-state handover still lands in `inbox/`, `/process-inbox` absorbs it (see that verb); it never survives as a standalone page.

## Hard rules — non-negotiable

1. **Never write outside `{{WIKI_VAULT}}/`.** If you find yourself wanting to
   update a file in `../{{PERSONAL_VAULT}}/`, suggest it to {{OWNER}} in your response
   instead. (This rule governs the wiki verbs. The separate `transcript-triage`
   automation is not a wiki verb and does file analyses into
   `../{{PERSONAL_VAULT}}/Meetings|Meetings/1-1|Personal|Ideas` - that write path is
   outside this agent's scope, by design.)
2. **Never delete.** Move to `_audit/_archive/YYYY-MM/` instead.
3. **Always cite.** Every wiki page's frontmatter lists the source files it was derived from. References in body use markdown links `[Title](../{{PERSONAL_VAULT_ENC}}/Knowledge/...)` or `[[Topics/...]]` for in-vault.
4. **One concept per file.** If a wiki page covers two concepts, split before writing.
5. **No hallucinated cross-references.** A `[[link]]` must point at a real file in this vault. Existence-check before emitting.
6. **No em-dashes.** Use ` - ` (single hyphen with spaces). {{OWNER}} writes that way.
7. **Match {{OWNER}}'s voice.** Before writing prose, read 2-3 recent files in `../{{PERSONAL_VAULT}}/Knowledge/` to calibrate. He uses sentence-case headers, no marketing language.

## Page conventions

Every page in `Topics/`, `People/`, `Projects/` must have:

```
---
type: topic | person | project | source
domain: <optional - your domain>
summary: One sentence. The whole point of this page.
tags: [kebab-case, lowercase, max-5]
sources:
  - "../{{PERSONAL_VAULT}}/Knowledge/2026-04-13 Foo.md"
  - "../{{PERSONAL_VAULT}}/Briefings/2026-04-15.md"
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# {Title}

## TL;DR
Two sentences. What this is, why it matters.

## Body
Prose paragraphs. ≤600 words. Split if larger.

## Open questions
- One bullet per unanswered thing. These drive future research.

## Related
- [[Topics/Related-Topic]]
- [[People/Person]]
- External: [Title](../{{PERSONAL_VAULT_ENC}}/Knowledge/...)
```

The `summary` and `tags` are load-bearing - they let other agents (and you in future sessions) decide relevance without reading the full page. Do not skip them. `domain` is optional; if you use domains, set it on every `Topics/` and `Sources/` page - see "Domains" below.

## Domains - one wiki, optionally several domains

This is ONE vault with ONE graph, split not into several wikis but optionally via the frontmatter field `domain`. Rationale: the wiki's value is in the cross-links (compounding); two separate vaults would cut exactly the most valuable layer. Zettelkasten/PKM best practice is clear: one system, navigation via Maps of Content, not separate boxes.

`domain` is optional and freely definable. If you maintain a single subject area, leave the field out. If you run several clearly separated areas, give each `Topics/` and `Sources/` page exactly one `domain` and create one MOC hub in `Maps/` per domain.

Rules (only if you use `domain`):
1. Every `Topics/` and `Sources/` page carries exactly one `domain`.
2. Cross-links between domains are welcome - but ONLY on a real substantive connection, not reflexively. Bridges are the justified exception, not the rule.
3. Each domain has a MOC hub in `Maps/`. Add new Topics to the matching MOC. The Index stays the flat full list, the MOC the curated navigation.

When to REALLY split into a second vault (only then, a documented tripwire):
- a domain gets its own audience or its own write rights/governance
- a domain gets its own maintainer or its own cron job
- a genuine subject break: an area with no connection to the rest
Until then: one graph, domains via `domain` + MOCs.

## The three verbs

### `/ingest-url <url>`

1. Fetch the URL. Use WebFetch. If JS-rendered, escalate to Chrome MCP.
2. Write the full-text article to `Sources/YYYY-MM-DD <slug>.md`. Format: same heavy template {{OWNER}} uses in `../{{PERSONAL_VAULT}}/Knowledge/` (frontmatter with tags + source + date + relevance, then TLDR, Summary with Context + Key Arguments, Implications, Key Principles, Actionable Items).
3. For each net-new or substantially augmented concept, touch `Topics/<concept>.md`. Create if missing, augment if existing. Follow page conventions strictly (incl. `domain` if you use domains). Add new Topics to the matching `Maps/`-MOC.
4. Update 3-10 related pages (Topics, People, Projects) to add a `Related` entry pointing at the new content. If you use domains, respect the Cross-Link rule (cross-domain links only on real bridges).
5. Update `Index.md`.
6. End with a one-paragraph diff: created N files, updated M files, one open question worth following up.

### `/process-inbox`

1. Read every file in `inbox/`. Empty inbox = valid no-op.
2. For each file decide one of:
   - **discard** - duplicate / accidental capture. Move to `inbox/_archive/YYYY-MM/`.
   - **link** - already covered by existing wiki page. Append the source path to that page's `sources:` frontmatter, archive the raw.
   - **promote** - net-new concept. Create `Topics/<concept>.md` following page conventions (incl. `domain`). Move raw to `Sources/`. Register the new Topic in the matching MOC under `Maps/`.
   - **file** - raw worth keeping but not yet ripe. Move to `Sources/` (set `domain` if you use domains) and note in `_audit/_pending.md` for future graduation.
3. Update `Index.md`. If you use domains, set `domain` on every new Source/Topic and add new Topics to the matching `Maps/`-MOC (no reflexive cross-domain links).
4. Append a per-file decision line to `_audit/_processing-log.md`:
   ```
   YYYY-MM-DD HH:MM | <filename> | <decision> | <target path> | <one-line reason>
   ```
5. End with: count by decision class, top 3 promotes with one-line summaries, any classifications you're uncertain about.

**Automated extractor captures:** files carrying an `origin:` field in frontmatter are pre-extracted Source candidates dropped automatically by an upstream tool (alongside Web Clipper and paste-dumps). Treat them like any other inbox file (discard / link / promote / file); they are already full extractions, so `source:` holds the original URL and `relevance:` the score - no need to re-fetch.

**Stray session-state handovers:** an inbox file that is really a session-state handover (a "where were we" note that belonged in hot.md) is not filed as a Source. Absorb it: lift any reusable architecture or decisions into the right Topic/Source, fold still-open threads into `hot.md`, then archive the raw to `inbox/_archive/YYYY-MM/`. Decide by content, not by the word "handover" in the title, a knowledge primer that merely says "Handover" takes the normal discard / link / promote / file path.

### `/lint-wiki`

Health-check the wiki. **Report only**, do not auto-fix unless {{OWNER}} says "fix the X items."

Output a single file at `_audit/audit-YYYY-MM-DD.md`. Checks:

1. Broken `[[wikilinks]]` (in-vault).
2. Broken `../{{PERSONAL_VAULT}}/...` markdown links (verify file exists across vault boundary).
3. Pages without `summary:`, `tags:` or `domain:` frontmatter. If you use domains: `Topics/` pages whose `domain` is missing, and domain Topics not listed in their matching `Maps/`-MOC (MOC-Drift).
4. Pages over 600 words in body - candidates for splitting.
5. Orphans - pages with zero inbound links.
6. Stale projects - `type: project` with `updated:` older than 90 days.
7. Open questions still open after 30 days.
8. Possible contradictions - pages making opposing claims about the same concept (flag pairs).
9. Singleton tags - tags used only once, likely typos.
End your response with a one-paragraph headline and a pointer to the audit file.

## Extra verb: `/graduate`

Per Internet Vin pattern. Scan `../{{PERSONAL_VAULT}}/Daily note/` and `../{{PERSONAL_VAULT}}/Ideas/` for the last 30 days. Identify ideas worth graduating to standalone `Topics/` pages.

Signals (any one triggers):
- Mentioned in 3+ daily notes across the window
- Marked `>>` or `★` ({{OWNER}}'s "this matters" markers)
- Cross-referenced from a Briefings/ note
- Tagged `#graduate` explicitly
- Sits in `Ideas/` with body > 200 words and recent `updated:` (active thinking)

**Output the proposals only.** Wait for {{OWNER}}'s response (e.g., `graduate 1, 3`, `all`, `none`) before creating the pages. This is the only verb that asks before acting.

## Extra verb: /synthesize [domain]

Surface emergent cross-topic patterns that the per-ingest linking misses. REPORT ONLY, like `/lint-wiki` - do not auto-edit the vault.

Reads Index.md, the `summary`/`tags` frontmatter of all Topics, the `## Related` graph, recent Sources, and `_audit/_pending.md`. Detects: tag co-occurrence clusters, Related-link density (pages converging on an unnamed concept), themes among recent ingests, merge candidates (near-duplicate Topics), and orphan groups that together form a theme.

Writes `_audit/synthesis-YYYY-MM-DD.md`: per cluster, the member pages + the emergent theme + ONE suggested action (create a Topic, merge two Topics, or leave) + a confidence level. Every suggested action must map to an existing verb. {{OWNER}} decides - nothing is auto-applied; he triggers the chosen actions in a follow-up.

Optional `$ARGUMENTS` scopes it to a domain/tag or a time-window; default is the whole vault with emphasis on recent activity. As the final step, rewrite `hot.md` (note the run and open cluster decisions in `## Open threads`).

Full spec: `.claude/commands/synthesize.md`

## Cross-runtime notes

- **Scheduled automation:** the plugin ships nightly/weekly jobs in `scheduled-prompts/` (inbox-process, weekly-lint, and optional triage jobs). Register the ones you want via the schedule tool or the `wiki-setup` skill.
- **Claude Code (CLI):** invoke from `{{VAULT_BASE}}/{{WIKI_VAULT}}/` as cwd. The CLI auto-discovers `.claude/commands/*.md` and reads this `CLAUDE.md`.
- **Both runtimes share this schema.** Edit `CLAUDE.md` to change behavior - never duplicate rules into runtime-specific places.

## Working memory hints

When the personal vault is referenced:

- Its long-form notes may use a heavy template (TLDR, Summary with Context + Key Arguments, Implications, Key Principles, Actionable Items). Match that style when writing new entries to `Sources/`.
- Match the tag convention used there (e.g. lowercase + kebab-case).
- Daily-note format is typically `YYYY-MM-DD.md`; same for Briefings/.
- If the owner uses a "this might matter" marker (e.g. `>>`) in daily notes, `/graduate` scans for it.
- Match the source language of the material you're synthesizing.
