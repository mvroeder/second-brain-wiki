# second-brain-wiki

An AI-maintained Obsidian second brain, packaged as a Cowork plugin. It implements the
Karpathy "LLM Wiki" pattern: two vaults (a read-only personal vault, a write-allowed wiki
vault), a set of verbs that ingest/triage/synthesize, and nightly automation that keeps
the wiki fed and linted.

## What's inside

**Skills (verbs)**
- `wiki-setup` - one-time setup: writes config, scaffolds the vault, installs the schema, registers jobs
- `ingest-url` - fetch a URL -> Source note -> synthesize into Topics, cross-reference, update Index
- `ingest-scan` - turn a `last30days` multi-source scan into a Source note + topic synthesis
- `process-inbox` - triage `inbox/`, classify (discard/link/promote/file), route, log
- `lint-wiki` - 9-check health audit (broken links, orphans, stale, contradictions, ...) -> `_audit/`
- `graduate` - scan personal Daily notes + Ideas for recurring ideas worth promoting to Topics (asks first)
- `synthesize` - surface emergent cross-topic clusters and merge candidates (report only)
- `wiki-context` - read-only lookup: relevant Topics/People/Projects/Sources for a query

**Scheduled prompts** (`scheduled-prompts/`, see its README)
- wiki-inbox-process, wiki-weekly-lint, newsletter-triage, transcript-triage, reading-list-triage (optional - reads `reading-list.md`; an Apple Reminders sync is one optional way to fill it)

**Templates** (`templates/`)
- `wiki-vault-CLAUDE.md` - the full schema that lives at your wiki vault root (the rulebook the verbs read)
- `obsidian-root-CLAUDE.md` - optional root context loader for `{{VAULT_BASE}}/`
- `routing-rules.md` - binding routing reference for transcript-triage

## First run
Run the **wiki-setup** skill ("set up my wiki"). It collects your vault names + identity,
writes `config.md`, scaffolds the folders, installs the schema, and offers to register the
five automation jobs. See `SETUP.md` for the manual setup path and connector requirements, and `USER_MANUAL.md` for the full guide (concepts, verbs, jobs, and end-to-end workflows).

## Configuration
Every skill resolves tokens (`{{OWNER}}`, `{{WIKI_VAULT}}`, `{{PERSONAL_VAULT}}`, ...) from
`config.md` at runtime. Edit that one file to retarget the whole plugin.

## Vaults - who writes where
Two vaults, two write scopes:
- **Wiki vault** (`{{WIKI_VAULT}}/`) - the wiki agent and every verb write here. This is the durable knowledge layer.
- **Personal vault** (`{{PERSONAL_VAULT}}/`) - read-only for the wiki verbs; they only read it for cross-references and `/graduate`. The one automation that writes INTO it is `transcript-triage`: once you confirm, meeting, networking, personal and brainstorm recordings are filed as analyses into `{{PERSONAL_VAULT}}/Meetings/`, `/Meetings/1-1/`, `/Personal/` and `/Ideas/`. So "personal vault is read-only" holds for the wiki agent, not for the transcript job.

## Connectors required
- Obsidian vault access (all skills) - via Cowork folder access
- Gmail (`newsletter-triage`)
- Plaud + Monologue (`transcript-triage`)
- `reading-list.md` (`reading-list-triage`) - **optional**; the job reads a plain file at the wiki vault root. An Apple Reminders sync is one optional way to fill it. Without the file, skip registering reading-list-triage.
- WebFetch / Chrome (URL ingest)

---

An AI-maintained Obsidian second brain, packaged as a Cowork/Claude Code plugin. It implements
the Karpathy "LLM Wiki" pattern: the AI, not you, maintains a structured Markdown wiki that
compounds over time. You feed it raw material (URLs, newsletters, voice notes, scan results);
it extracts, classifies, cross-links, and keeps the whole thing healthy.

This manual covers the concepts, one-time setup, every verb and automation job, and concrete
end-to-end workflows - including how to pull single URLs out of newsletters via Apple Reminders.

---

## 1. The mental model

### Two vaults, one rule

You keep two Obsidian vaults side by side under `{{VAULT_BASE}}/`:

- **Personal vault** (your name for it, e.g. `Personal`) - your own notes, briefings, daily
  logs, knowledge articles. The wiki agent treats this as **read-only**. It reads it for
  cross-references and for the `/graduate` verb, but never writes there. (One exception, see
  the transcript job below.)
- **Wiki vault** (e.g. `Wiki`) - the AI-maintained second brain. Everything the agent writes
  goes here. You generally do not edit it by hand; the verbs do.

The single hard rule: **the wiki agent only ever writes inside the wiki vault.** If it wants to
change something in your personal vault, it suggests it to you instead.

### Inside the wiki vault

```
Wiki/
  CLAUDE.md     the schema - the rulebook every verb reads first
  inbox/        fresh captures waiting to be triaged
  Sources/      full-text ingested articles and scan notes
  Topics/       atomic concept pages (the durable synthesis layer)
  People/       one page per person referenced 2+ times
  Projects/     one page per active project
  Maps/         Maps of Content - one navigation hub per domain (optional)
  _audit/       lint and synthesis reports
  Index.md      catalog: what exists, where, when last touched
  hot.md        live "where are we right now" cache, read first each session
```

The flow is always the same: **raw material lands in `inbox/` -> it gets triaged -> durable
knowledge ends up in `Sources/` and `Topics/`, cross-linked, with `Index.md` updated.**

### Two organizing ideas you can ignore at first

- **Domains** - an optional `domain:` frontmatter field that lets one vault hold several clearly
  separated subject areas while keeping them in one graph. If you track a single area, skip it.
- **Focus lenses** - your personal "what matters to me" filters, defined in `config.md`. The
  triage jobs rate incoming material against them. If you leave them empty, jobs rate by general
  relevance.

---

## 2. Installation and setup

### Install
Install the `.plugin` file in Cowork (Settings) or Claude Code. The plugin ships eight skills
(verbs), five scheduled automation prompts, and three templates.

### Fastest setup
Tell Claude: **"set up my wiki"**. The `wiki-setup` skill walks you through everything below
interactively. The manual path is here if you prefer to do it yourself.

### Step 1 - Fill `config.md`
Open `config.md` in the plugin and replace the placeholder values. These tokens are resolved by
every skill at runtime, so this one file retargets the whole plugin:

| Token | What it is |
|---|---|
| `{{OWNER}}` | Your name (agent identity + voice calibration) |
| `{{OWNER_EMAIL}}` | Your email (used by the Gmail/triage jobs) |
| `{{WIKI_VAULT}}` | Name of your write-allowed Obsidian vault |
| `{{PERSONAL_VAULT}}` | Name of your read-only personal vault |
| `{{PERSONAL_VAULT_ENC}}` | Same name with spaces as `%20` (for markdown links) |

Optionally fill the focus lenses block.

### Step 2 - Scaffold the wiki vault
In `{{VAULT_BASE}}/<WIKI_VAULT>/` create: `inbox/`, `inbox/_archive/`, `Sources/`, `Topics/`,
`People/`, `Projects/`, `Maps/`, `_audit/`, plus a starter `Index.md` and `hot.md`.
`wiki-setup` does this for you.

### Step 3 - Install the schema
Copy `templates/wiki-vault-CLAUDE.md` to `{{VAULT_BASE}}/<WIKI_VAULT>/CLAUDE.md`, substituting your
token values. This is the rulebook every verb reads at the start of a run. Without it at the
vault root, the skills have no schema. Optionally copy `templates/obsidian-root-CLAUDE.md` to
`{{VAULT_BASE}}/CLAUDE.md` so any session that mounts the folder gets the read pattern.

### Step 4 - Local state for the triage jobs
Create `{{STATE_DIR}}/` and copy `templates/routing-rules.md` there. The dedup ledgers
(`ingest-ledger.jsonl`, `newsletter-ledger.jsonl`, `reading-list-ledger.jsonl`) are created on
first run.

### Step 5 - Connect the tools
Authorize the connectors for the jobs you want (see the connector table in section 6).

### Step 6 - Register automation (optional)
Ask `wiki-setup` to register the scheduled jobs, or create them yourself from the cron in each
file's frontmatter under `scheduled-prompts/`.

---

## 3. The verbs (skills you call)

Call a verb by name in chat (e.g. "run ingest-url on this link") or, in Claude Code, as a slash
command.

| Verb | What it does |
|---|---|
| `wiki-context <query>` | Read-only lookup. Returns the most relevant Topics/People/Projects/Sources with summaries. Writes nothing. |
| `ingest-url <url>` | Fetch a URL, write a full-text `Sources/` note, synthesize into `Topics/`, cross-link, update `Index.md`. |
| `ingest-scan <topic>` | Turn a `last30days` multi-source scan into a `Sources/` note plus topic synthesis. |
| `process-inbox` | Triage everything in `inbox/`: discard / link / promote / file. The main write path into the wiki. |
| `lint-wiki` | 9-check health audit (broken links, orphans, stale pages, contradictions, ...). Report only. |
| `graduate` | Scan your personal Daily notes + Ideas for recurring ideas worth promoting to Topics. Asks before writing. |
| `synthesize` | Surface emergent cross-topic clusters and merge candidates. Report only. |
| `wiki-setup` | One-time setup: writes config, scaffolds the vault, installs the schema, registers jobs. |

### How `process-inbox` classifies
Every inbox file gets exactly one decision:
- **discard** - duplicate or accidental capture. Moved to `inbox/_archive/`.
- **link** - already covered by an existing page. The source path is appended to that page; raw archived.
- **promote** - a net-new concept worth its own page. A `Topics/` page is created; the raw moves to `Sources/`.
- **file** - worth keeping but not yet ripe. Moved to `Sources/`, noted for future graduation.

---

## 4. The automation jobs (scheduled prompts)

These run on a schedule and are **propose-only**: they never file anything without your
confirmation. They dedup via the JSONL ledgers in `{{STATE_DIR}}/`.

Automation jobs come in three tiers. Enable only the tiers you want; wiki-setup asks.

**Core (Obsidian only):**
- `wiki-inbox-process` (daily) - triage inbox/, route to Sources/Topics, update Index.
- `wiki-weekly-lint` (weekly) - health audit into _audit/.

**Neutral-optional (one cross-platform connector):**
- `newsletter-triage` (daily, needs Gmail) - scan newsletters, drop confirmed captures into inbox/.

**Mac-optional (Apple ecosystem):**
- `transcript-triage` (daily, needs Plaud + Monologue) - classify voice recordings, propose filing.
- `reading-list-triage` (daily, needs a capture source) - evaluate new reading-list URLs.

The chain for the triage jobs is always: **the job proposes -> you confirm -> a capture lands in
`inbox/` -> `wiki-inbox-process` is the one thing that writes it durably into the wiki.**

One exception to "the wiki is the only write target": `transcript-triage` files confirmed
meeting, networking, personal and brainstorm analyses into your **personal vault**
(`Meetings/`, `Meetings/1-1/`, `Personal/`, `Ideas/`), because those are personal, not wiki,
material.

---

## 5. Workflows

### 5A. Drop a link in `reading-list.md` -> wiki  (the reading-list flow)

This is the path for "I am reading something, I see a few links worth keeping, I want them in
my wiki - but later, not now." You add URLs to a plain file during the day, on desktop or from
your phone via Obsidian Mobile; the wiki picks them up the next morning.

**The setup it assumes.** `reading-list-triage` reads a plain-text file at
`{{VAULT_BASE}}/{{WIKI_VAULT}}/reading-list.md`. This file is yours: you create it, you add
lines to it, and the agent never edits or prunes it - it only reads it. The file must contain a
section like:

```
## Reading List
- https://example.com/some-article  #i-read
- https://blog.example.org/post
```

The job captures two kinds of entries from the **Reading List** section only: any line tagged
`#i-read`, and any line whose text starts with `http://` or `https://`. Everything else is
ignored. Apple Reminders is one optional way to fill this file (via a Reminders -> file sync you
set up yourself - an Apple Shortcut, a small export script, or a reminders sync tool; the plugin
does not ship one) but it is an adapter, not a requirement - editing the file directly is the
primary path.

**The daily loop.**

1. **During the day**: reading something (a newsletter, an article, wherever), you spot a link
   worth keeping. Open `reading-list.md` - on your Mac, or on your phone via Obsidian Mobile -
   and add a line under `## Reading List`: paste the URL, or add a line of text and tag it
   `#i-read`. (Optional: if you run an Apple Reminders -> file sync, adding to a Reminders list
   works too, since it writes into the same file.)
2. **Early morning**: `reading-list-triage` reads the file. For each new URL it fetches the page,
   writes a title + TL;DR, and proposes a category: WIKI (fits your focus lenses, a new concept),
   READ (too nuanced to auto-handle), or SKIP. It records each in `reading-list-ledger.jsonl` so
   it never re-rates the same URL, and never writes back to `reading-list.md` itself.
3. **You get a chat summary** like: "3 new URLs. 1. <title> - WIKI; 2. <title> - SKIP; 3.
   <title> - READ. Reply with numbers and destinations, e.g. 'ingest 3 wiki, 5 skip'."
4. **You confirm**: reply "ingest 1 wiki". The agent then runs `/ingest-url` on that link: it
   writes a full `Sources/` note, synthesizes the concepts into `Topics/`, cross-links related
   pages, and updates `Index.md`. The ledger entry flips to its final verdict.
5. Done. The article is now durable, sourced, and linked in your wiki - and you never lost the
   link or had to process it in the moment.

If you do not want this job, skip it entirely and use 5B instead.

### 5B. Capture a single URL right now

You have a link in front of you and want it in the wiki immediately. Just say:
"ingest-url https://...". The `ingest-url` verb does the full pipeline (Source note ->
Topic synthesis -> cross-links -> Index) in one shot. No Reminders, no waiting.

### 5C. Newsletter triage from Gmail (the automated alternative to 5A)

If you would rather the wiki scan your inbox than have you stash links by hand:
`newsletter-triage` runs each morning, finds newsletters from the last 24h, and proposes
INGEST / READ / SKIP per newsletter. On "ingest 1, 3" it drops a capture into `inbox/`, and the
next `wiki-inbox-process` run turns it into Sources/Topics. Use 5A when you want to pick
individual links; use 5C when you want to triage whole newsletters.

### 5D. A research scan -> wiki

Run the `last30days` scan on a topic, then "ingest-scan <topic>". The verb turns the scan into a
single `Sources/` note and layers its findings onto existing `Topics/` (it usually augments more
than it creates).

### 5E. Voice notes and meetings

`transcript-triage` pulls new recordings from Plaud and Monologue each night, classifies them
(strategy, customer, meeting, network, personal, conference, brainstorm), and proposes where
each goes. Conference talks become wiki ingest candidates; meetings and personal recordings are
filed as analyses into your personal vault (never verbatim transcripts).

### 5F. Promote recurring ideas

`graduate` scans your personal Daily notes and Ideas folders for ideas that keep coming up
(mentioned repeatedly, marked important, or tagged) and proposes promoting them to standalone
`Topics/`. It always asks before writing.

### 5G. Keep the wiki healthy

- `lint-wiki` (or the weekly job) reports broken links, orphans, stale projects, contradictions,
  and more, into `_audit/`. It fixes nothing unless you say so.
- `synthesize` surfaces emergent clusters across topics and suggests merges or new Topics.
- `wiki-context <query>` is your read lookup when you want to see what the wiki already holds.

---

## 6. Connectors reference

| Connector | Used by | Required? |
|---|---|---|
| Obsidian folder access | all skills | Yes |
| WebFetch / Chrome | `ingest-url`, `reading-list-triage` | Yes for URL ingest |
| Gmail | `newsletter-triage` | Only for that job |
| Plaud + Monologue | `transcript-triage` | Only for that job |
| `reading-list.md` (Apple Reminders sync optional) | `reading-list-triage` | Optional |

Skills whose connectors are missing simply do not run until connected.

---

## 7. Files and folders reference

| Path | What it is |
|---|---|
| `{{VAULT_BASE}}/<WIKI_VAULT>/` | the wiki vault (write target) |
| `{{VAULT_BASE}}/<WIKI_VAULT>/CLAUDE.md` | the schema the verbs read first |
| `{{VAULT_BASE}}/<PERSONAL_VAULT>/` | your personal vault (read-only to the wiki agent) |
| `{{VAULT_BASE}}/{{WIKI_VAULT}}/reading-list.md` | reading-list source (mirror of Apple Reminders) |
| `{{STATE_DIR}}/routing-rules.md` | binding routing reference for transcript-triage |
| `{{STATE_DIR}}/*.jsonl` | dedup ledgers (newsletter, reading-list, ingest) |

---

## 8. Conventions and rules

- The wiki agent **never writes outside the wiki vault** (the transcript job is the one
  sanctioned exception, and it writes only to your personal vault's Meetings/Personal/Ideas).
- **Never delete** - things are moved to an `_archive/`, not removed.
- **Always cite** - every wiki page lists the sources it was derived from.
- **One concept per page** - if a page covers two ideas, it gets split.
- **No invented links** - every `[[link]]` points at a real file.
- **No em-dashes** - the wiki uses ` - ` (a hyphen with spaces).
- **Voice calibration** - before writing prose, the agent reads a few of your recent notes to
  match your style.

---

## 9. Troubleshooting

- **A job did nothing.** Its connector is probably not authorized, or there was nothing new
  (the ledgers prevent re-processing). Check the connector and the relevant ledger.
- **`reading-list-triage` never finds anything.** Confirm `{{VAULT_BASE}}/{{WIKI_VAULT}}/reading-list.md`
  exists and has a `## Reading List` section that your Reminders sync actually writes to.
- **Tokens like `{{WIKI_VAULT}}` show up in your vault files.** The schema was copied without
  substitution. Re-run `wiki-setup` or re-copy `templates/wiki-vault-CLAUDE.md` with your real
  values.
- **The agent wrote nothing on an ingest.** It only writes inside the wiki vault and only on a
  real concept; a thin or duplicate page is correctly skipped or filed, not promoted.

---

## 10. Quick reference

| I want to... | Do this |
|---|---|
| Save a link now | `ingest-url <url>` |
| Save links during the day for later | Add a line to `reading-list.md` (tag `#i-read`), on desktop or via Obsidian Mobile -> `reading-list-triage` handles it next morning |
| Triage my newsletters | `newsletter-triage` (daily) -> confirm -> `wiki-inbox-process` |
| Turn a research scan into wiki pages | `ingest-scan <topic>` |
| File voice notes / meetings | `transcript-triage` (daily) |
| Promote recurring ideas to Topics | `graduate` |
| See what the wiki knows about X | `wiki-context <X>` |
| Check the wiki's health | `lint-wiki` |
| Find hidden cross-topic patterns | `synthesize` |