# Cross-Platform Hardening + Non-Technical Onboarding Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make the generic `second-brain-wiki` plugin genuinely cross-platform (macOS + Windows) and onboardable by a non-technical, first-time Obsidian user, without a personal fork.

**Architecture:** Move all local state out of the personal `~/Cowork/productivity/` path into the wiki vault (`.wiki-state/` hidden, `reading-list.md` visible). Make the vault base a config token instead of a hardcoded `~/Obsidian/`. Tier the connector-dependent jobs (core / neutral / mac-only) and make the reading-list capture source pluggable with a vault-file default. Add a public roadmap and a self-contained English user guide; produce a German handover email as a separate personal artifact.

**Tech Stack:** Markdown skill/prompt/template files, `config.md` token substitution, Cowork/Claude Code plugin conventions. No application code, no test framework.

## Verification model (read this first)

This is a Markdown + config project, not a code project. There is no pytest. Each task's
"verify" step is a **grep gate** and/or a **consistency check** with an exact command and an
exact expected result. Treat a failing grep gate exactly like a failing test: do not commit
until it passes.

Repo root for all paths below: the current working directory
(`second-brain-wiki` worktree). Use `rg` (ripgrep) if available, else `grep -rn`.

## Global Constraints

Every task implicitly includes these. Values are copied verbatim from the spec.

- **No em-dashes** in any produced text. Use ` - ` (hyphen with spaces). en-dash `–` only for number ranges.
- **State location:** ledgers (`ingest-ledger.jsonl`, `newsletter-ledger.jsonl`, `reading-list-ledger.jsonl`) and `routing-rules.md` live in `{{STATE_DIR}}` (default `{{VAULT_BASE}}/{{WIKI_VAULT}}/.wiki-state`, hidden in Obsidian). The capture file lives at `{{VAULT_BASE}}/{{WIKI_VAULT}}/reading-list.md` (visible, mobile-editable).
- **Vault paths are config-driven** via `{{VAULT_BASE}}` (default value `~/Obsidian`). No `~/Obsidian/` or `~/Cowork/` literal remains in any skill, scheduled prompt, or template. The only permitted `~/Obsidian` occurrences are the documented default value in `config.md` and illustrative text in `ROADMAP.md`/guide.
- **`reading-list.md` is human-append-only and read-only to the agent.** The job reads it and records processed state in the ledger; it never mutates the file.
- **Job tiers:** core (`wiki-inbox-process`, `wiki-weekly-lint`) / neutral-optional (`newsletter-triage`, Gmail) / mac-optional (`transcript-triage`, `reading-list-triage`).
- **No new connector adapters are built.** Google Tasks / Microsoft To Do / Readwise are roadmap text only.
- **Languages:** `README`, `SETUP`, `config.md`, `ROADMAP.md`, and the user guide are English. The email to Anna is German (voice per `~/Cowork/memory/context/voice.md`).
- **Two vaults; the wiki agent writes only inside the wiki vault** (transcript-triage's personal-vault writes remain the one sanctioned exception, mac-only tier).

---

## File Structure

**Modified:**
- `config.md` - add `{{VAULT_BASE}}`, `{{STATE_DIR}}` tokens; document new state + capture locations
- `SETUP.md` - config-driven paths, state relocation, tiered connectors
- `README.md` - config-driven paths, state relocation, three job tiers, files table
- `skills/wiki-setup/SKILL.md` - scaffold `.wiki-state/` + `reading-list.md`, write to new locations, tier opt-in
- `skills/wiki-context/SKILL.md` - config-driven wiki path
- `skills/ingest-url/SKILL.md`, `skills/ingest-scan/SKILL.md`, `skills/process-inbox/SKILL.md`, `skills/graduate/SKILL.md`, `skills/lint-wiki/SKILL.md`, `skills/synthesize/SKILL.md` - config-driven paths where present
- `templates/wiki-vault-CLAUDE.md` - config-driven `~/Obsidian/`, de-personalize the "runs Cowork + Code" line
- `templates/obsidian-root-CLAUDE.md` - config-driven root path
- `templates/routing-rules.md` - state path -> `.wiki-state/`
- `scheduled-prompts/wiki-inbox-process.md`, `wiki-weekly-lint.md`, `newsletter-triage.md`, `transcript-triage.md`, `reading-list-triage.md`, `scheduled-prompts/README.md` - config-driven paths, state relocation, tier labels, capture-source rewording

**Created:**
- `ROADMAP.md` (repo root)
- `GETTING-STARTED.md` (repo root) - the self-contained English user guide
- `docs/superpowers/notes/2026-07-03-verification-findings.md` - research output feeding the guide
- Email artifact in the session scratchpad (NOT committed to the public repo)

---

## Task 1: Config schema - new tokens and locations

**Files:**
- Modify: `config.md`

**Interfaces:**
- Produces: two new tokens every later task relies on:
  - `{{VAULT_BASE}}` - directory holding the vaults. Default value `~/Obsidian`. Windows users set an absolute path, e.g. `C:\Users\<name>\Obsidian`.
  - `{{STATE_DIR}}` - operational state dir. Default value `{{VAULT_BASE}}/{{WIKI_VAULT}}/.wiki-state`.
- The capture file path `{{VAULT_BASE}}/{{WIKI_VAULT}}/reading-list.md` is referenced by name (no token).

- [ ] **Step 1: Read the current config.md**

Run: `cat config.md`
Note the existing token block (`{{OWNER}}`, `{{OWNER_EMAIL}}`, `{{WIKI_VAULT}}`, `{{PERSONAL_VAULT}}`, `{{PERSONAL_VAULT_ENC}}`) and the `~/Cowork/productivity/` line (~line 26).

- [ ] **Step 2: Add the two new tokens**

Add to the token table/list, after `{{WIKI_VAULT}}`:

```markdown
- `{{VAULT_BASE}}` - the directory that holds your vaults. Default `~/Obsidian`.
  On Windows set an absolute path, e.g. `C:\Users\<name>\Obsidian`.
- `{{STATE_DIR}}` - where the plugin keeps its operational state (dedup ledgers,
  routing-rules). Default `{{VAULT_BASE}}/{{WIKI_VAULT}}/.wiki-state` (hidden in Obsidian).
```

- [ ] **Step 3: Replace the `~/Cowork/productivity/` description**

Replace the line documenting `~/Cowork/productivity/` (holds the dedup ledgers and `routing-rules.md`) with:

```markdown
- `{{STATE_DIR}}/` holds the dedup ledgers (`*-ledger.jsonl`) and `routing-rules.md`.
  It lives inside the wiki vault, so it needs no separate folder grant and is
  cross-platform. `reading-list.md` (the capture file) sits at the wiki vault root,
  visible so you can edit it from Obsidian Mobile.
```

- [ ] **Step 4: Verify no `~/Cowork` remains in config.md**

Run: `grep -n "Cowork/productivity" config.md`
Expected: no output (exit 1).

- [ ] **Step 5: Verify both new tokens are present**

Run: `grep -c "{{VAULT_BASE}}\|{{STATE_DIR}}" config.md`
Expected: a count >= 3.

- [ ] **Step 6: Commit**

```bash
git add config.md
git commit -m "feat(config): add VAULT_BASE and STATE_DIR tokens, relocate state into vault"
```

---

## Task 2: Config-driven paths in skills and scheduled prompts

Mechanical sweep of the operational files. Two transformations:
1. `~/Obsidian/{{WIKI_VAULT}}` -> `{{VAULT_BASE}}/{{WIKI_VAULT}}` (and any bare `~/Obsidian/` -> `{{VAULT_BASE}}/`).
2. `~/Cowork/productivity` -> `{{STATE_DIR}}` for ledgers/routing-rules; the reading-list capture file `~/Cowork/productivity/TASKS.md` becomes `{{VAULT_BASE}}/{{WIKI_VAULT}}/reading-list.md` (handled fully in Task 6, but update the path token here).

**Files:**
- Modify: `skills/wiki-context/SKILL.md`, `skills/ingest-url/SKILL.md`, `skills/ingest-scan/SKILL.md`, `skills/process-inbox/SKILL.md`, `skills/graduate/SKILL.md`, `skills/lint-wiki/SKILL.md`, `skills/synthesize/SKILL.md`
- Modify: `scheduled-prompts/wiki-inbox-process.md`, `scheduled-prompts/wiki-weekly-lint.md`, `scheduled-prompts/newsletter-triage.md`, `scheduled-prompts/transcript-triage.md`, `scheduled-prompts/reading-list-triage.md`, `scheduled-prompts/README.md`
- (Leave `skills/wiki-setup/SKILL.md` for Task 4; leave templates for Task 3.)

**Interfaces:**
- Consumes: `{{VAULT_BASE}}`, `{{STATE_DIR}}` from Task 1.

- [ ] **Step 1: List every operational file that still hardcodes a path**

Run: `grep -rln "~/Obsidian\|~/Cowork/productivity" skills scheduled-prompts`
Expected: the files listed above (minus wiki-setup, which is Task 4).

- [ ] **Step 2: Rewrite `~/Obsidian/{{WIKI_VAULT}}` occurrences**

In each listed file, replace `~/Obsidian/{{WIKI_VAULT}}` with `{{VAULT_BASE}}/{{WIKI_VAULT}}` and any standalone `~/Obsidian/` with `{{VAULT_BASE}}/`. Edit precisely (these are natural-language instructions to the agent; keep the surrounding wording intact).

- [ ] **Step 3: Rewrite `~/Cowork/productivity` occurrences (ledgers + routing-rules)**

In `newsletter-triage.md`, `transcript-triage.md`, `reading-list-triage.md`, and `scheduled-prompts/README.md`, replace `~/Cowork/productivity` with `{{STATE_DIR}}` for the ledger and routing-rules references. Also replace the mount instruction wording ("mount `~/Cowork/productivity`", "`request_cowork_directory`") with "the wiki vault is already the granted folder; the state dir is `{{STATE_DIR}}` inside it". (The reading-list capture *source* path in `reading-list-triage.md` is finalized in Task 6.)

- [ ] **Step 4: Grep gate - no hardcoded home paths remain in these files**

Run: `grep -rn "~/Obsidian\|~/Cowork" skills scheduled-prompts | grep -v "wiki-setup"`
Expected: no output (exit 1). If any line prints, fix it and re-run.

- [ ] **Step 5: Consistency check - tokens are well-formed**

Run: `grep -rno "{{[A-Z_]*}}" skills scheduled-prompts | sort -u`
Expected: only known tokens appear (`{{OWNER}}`, `{{OWNER_EMAIL}}`, `{{WIKI_VAULT}}`, `{{PERSONAL_VAULT}}`, `{{PERSONAL_VAULT_ENC}}`, `{{VAULT_BASE}}`, `{{STATE_DIR}}`). No typo'd token (e.g. `{{VAULTBASE}}`).

- [ ] **Step 6: Commit**

```bash
git add skills scheduled-prompts
git commit -m "refactor: config-driven vault + state paths in skills and scheduled prompts"
```

---

## Task 3: Config-driven paths and de-personalization in templates and top-level docs

**Files:**
- Modify: `templates/wiki-vault-CLAUDE.md`, `templates/obsidian-root-CLAUDE.md`, `templates/routing-rules.md`
- Modify: `README.md`, `SETUP.md`

**Interfaces:**
- Consumes: `{{VAULT_BASE}}`, `{{STATE_DIR}}` from Task 1.

- [ ] **Step 1: Fix the personalized line in `wiki-vault-CLAUDE.md`**

Line ~9 currently reads: "...runs Claude Cowork (desktop) and Claude Code (CLI) against two parallel vaults at `~/Obsidian/`." Replace with a client-neutral, config-driven version:

```markdown
You are the maintainer of {{OWNER}}'s second brain. The vaults live under
`{{VAULT_BASE}}/` (two parallel vaults: the wiki and the personal vault). Treat the
wiki as production: every write is durable, every link is real, every quote is sourced.
```

- [ ] **Step 2: Sweep remaining `~/Obsidian/` in templates**

In `wiki-vault-CLAUDE.md` (the folder tree at ~line 14, the Claude Code cwd note at ~line 192), `obsidian-root-CLAUDE.md` (~lines 8, 24), replace `~/Obsidian/{{WIKI_VAULT}}` -> `{{VAULT_BASE}}/{{WIKI_VAULT}}` and bare `~/Obsidian/` -> `{{VAULT_BASE}}/`.

- [ ] **Step 3: Point routing-rules template at the new state dir**

In `templates/routing-rules.md` (~line 3, "Copy this to `~/Cowork/productivity/routing-rules.md`"), replace with "Copy this to `{{STATE_DIR}}/routing-rules.md`".

- [ ] **Step 4: Sweep README.md and SETUP.md**

Apply the same two transformations across `README.md` and `SETUP.md`: `~/Obsidian/` -> `{{VAULT_BASE}}/`, and `~/Cowork/productivity/` -> `{{STATE_DIR}}/` (with `TASKS.md` -> `reading-list.md` in the reading-list rows; the files table row for `routing-rules.md` and `*.jsonl` now points at `{{STATE_DIR}}/`). Leave the tier restructuring of these two files to Task 5; here only the paths change.

- [ ] **Step 5: Grep gate across templates + top-level docs**

Run: `grep -rn "~/Cowork" templates README.md SETUP.md`
Expected: no output.
Run: `grep -rn "~/Obsidian" templates README.md SETUP.md config.md`
Expected: at most the single documented default-value line in `config.md`; nothing in templates/README/SETUP.

- [ ] **Step 6: Commit**

```bash
git add templates README.md SETUP.md
git commit -m "refactor: config-driven paths in templates and docs, de-personalize schema"
```

---

## Task 4: wiki-setup - scaffold new locations and tier opt-in

**Files:**
- Modify: `skills/wiki-setup/SKILL.md`

**Interfaces:**
- Consumes: `{{VAULT_BASE}}`, `{{STATE_DIR}}` from Task 1; the tier definitions from Global Constraints.

- [ ] **Step 1: Update scaffolding paths**

In the scaffold step, change the write-vault path to `{{VAULT_BASE}}/{{WIKI_VAULT}}/`. Add creation of `{{STATE_DIR}}/` (the hidden state dir) and a starter `{{VAULT_BASE}}/{{WIKI_VAULT}}/reading-list.md` containing a `## Reading List` heading and a one-line comment explaining it is append-only.

- [ ] **Step 2: Update schema + routing-rules copy targets**

Change "Copy `templates/routing-rules.md` to `~/Cowork/productivity/routing-rules.md`" to `{{STATE_DIR}}/routing-rules.md`, and "Create `~/Cowork/productivity/`" to "Create `{{STATE_DIR}}/`". The vault-root `CLAUDE.md` copy target becomes `{{VAULT_BASE}}/{{WIKI_VAULT}}/CLAUDE.md`.

- [ ] **Step 3: Add the tier opt-in prompt**

Add an explicit step: after scaffolding, ask the user which automation tiers to enable and register only those:

```markdown
Ask which automation tiers to enable, and register only the chosen ones:
- Core (Obsidian only): wiki-inbox-process, wiki-weekly-lint. Recommended default: on.
- Neutral (needs Gmail): newsletter-triage.
- Mac-only (needs Plaud/Monologue, or an Apple Reminders sync): transcript-triage,
  reading-list-triage. Only offer these on macOS; skip on Windows.
Do not register a job whose tier the user declined.
```

- [ ] **Step 4: Update the token-substitution guard**

Ensure the "no `{{...}}` tokens remain" guard lists the new locations: files written into the vault, into `{{STATE_DIR}}/`, and `reading-list.md`.

- [ ] **Step 5: Grep gate**

Run: `grep -n "~/Cowork\|~/Obsidian" skills/wiki-setup/SKILL.md`
Expected: no output.

- [ ] **Step 6: Consistency check - tier names match the scheduled prompts**

Run: `ls scheduled-prompts/*.md`
Confirm the five job filenames referenced in the tier prompt (`wiki-inbox-process`, `wiki-weekly-lint`, `newsletter-triage`, `transcript-triage`, `reading-list-triage`) all exist.

- [ ] **Step 7: Commit**

```bash
git add skills/wiki-setup/SKILL.md
git commit -m "feat(wiki-setup): scaffold vault state dir + reading-list, add tier opt-in"
```

---

## Task 5: Job optionality tiers in the docs

**Files:**
- Modify: `README.md`, `scheduled-prompts/README.md`

**Interfaces:**
- Consumes: the tier definitions from Global Constraints.

- [ ] **Step 1: Restructure the jobs table in README.md**

Replace the flat automation-jobs table with a three-tier presentation. Content:

```markdown
Automation jobs come in three tiers. Enable only the tiers you want; wiki-setup asks.

**Core (Obsidian only):**
- `wiki-inbox-process` (daily) - triage inbox/, route to Sources/Topics, update Index.
- `wiki-weekly-lint` (weekly) - health audit into _audit/.

**Neutral-optional (one cross-platform connector):**
- `newsletter-triage` (daily, needs Gmail) - scan newsletters, drop confirmed captures into inbox/.

**Mac-optional (Apple ecosystem):**
- `transcript-triage` (daily, needs Plaud + Monologue) - classify voice recordings, propose filing.
- `reading-list-triage` (daily, needs a capture source) - evaluate new reading-list URLs.
```

- [ ] **Step 2: Mirror the tiers in `scheduled-prompts/README.md`**

Add the same three-tier grouping so the prompts folder's README matches the top-level README.

- [ ] **Step 3: Consistency check**

Run: `grep -rn "Core\|Neutral\|Mac-optional" README.md scheduled-prompts/README.md`
Expected: the three tier labels present in both files.

- [ ] **Step 4: Commit**

```bash
git add README.md scheduled-prompts/README.md
git commit -m "docs: present automation jobs in core/neutral/mac tiers"
```

---

## Task 6: Capture-source abstraction - vault-file default, append-only

**Files:**
- Modify: `scheduled-prompts/reading-list-triage.md`
- Modify: `README.md` (the reading-list workflow section 5A and quick-reference)

**Interfaces:**
- Consumes: `{{VAULT_BASE}}`, `{{WIKI_VAULT}}`, `{{STATE_DIR}}`.
- Produces: the capture contract wording reused by the guide and ROADMAP.

- [ ] **Step 1: Reword the source in `reading-list-triage.md`**

Replace "scan `~/Cowork/productivity/TASKS.md`" and the Apple-Reminders framing with a source-agnostic default:

```markdown
Every morning you scan the capture source for new URLs and propose where each belongs:
`{{WIKI_VAULT}}` (wiki) or Skip. The default capture source is the file
`{{VAULT_BASE}}/{{WIKI_VAULT}}/reading-list.md`, under its `## Reading List` heading.
Captured: every line whose text starts with `http://` or `https://`, and any line tagged
`#i-read`. Apple Reminders (via a Reminders -> file sync) is an optional adapter that
writes into the same file; it is not required.
```

- [ ] **Step 2: Enforce append-only / read-only-to-agent**

Add an explicit constraint line to the prompt:

```markdown
Treat `reading-list.md` as READ-ONLY: never edit or prune it. Processed state lives only
in `{{STATE_DIR}}/reading-list-ledger.jsonl`. The human owns the file; you only read it.
```

- [ ] **Step 3: Update the ledger path**

Ensure the ledger reference reads `{{STATE_DIR}}/reading-list-ledger.jsonl`.

- [ ] **Step 4: Update README workflow 5A**

Rewrite section 5A so the primary path is "add a URL to `reading-list.md` (on desktop, or from your phone via Obsidian Mobile)"; Apple Reminders is mentioned as one optional adapter, not the assumed mechanism.

- [ ] **Step 5: Grep gate**

Run: `grep -n "TASKS.md\|Cowork" scheduled-prompts/reading-list-triage.md`
Expected: no output.

- [ ] **Step 6: Commit**

```bash
git add scheduled-prompts/reading-list-triage.md README.md
git commit -m "feat: abstract capture source, default to append-only vault reading-list.md"
```

---

## Task 7: ROADMAP.md

**Files:**
- Create: `ROADMAP.md` (repo root)

- [ ] **Step 1: Write the roadmap**

Create `ROADMAP.md` with this content (English, no em-dashes):

```markdown
# Roadmap

Where this plugin is going. It started as a single-user macOS tool and is becoming a
cross-platform, connector-pluggable second brain.

## Now

- Cross-platform (macOS + Windows) via config-driven vault paths.
- State lives inside the wiki vault; no external `~/Cowork/` dependency.
- Automation in three tiers: core (Obsidian only), neutral (Gmail), mac-only (Apple).
- Reading-list capture is abstracted behind a source contract, default is a plain
  vault file (`reading-list.md`) you can edit from Obsidian Mobile.

## Next - capture-source adapters

The reading-list source is pluggable. The Apple Reminders dependency is being
generalized into one adapter among peers:
- Google Tasks (Google/Android ecosystem).
- Microsoft To Do (Windows ecosystem, via Microsoft Graph).
- Apple Reminders stays as an equal peer, not the privileged default.

## Later

- Readwise Reader and other read-later services as capture adapters.
- The same pluggable-source pattern applied to the other two seams: the mail source
  (today Gmail) and the voice source (today Plaud + Monologue).
```

- [ ] **Step 2: Verify no em-dashes**

Run: `grep -n "—" ROADMAP.md`
Expected: no output.

- [ ] **Step 3: Commit**

```bash
git add ROADMAP.md
git commit -m "docs: add public roadmap"
```

---

## Task 8: Verification pass - resolve the open items

Research task. Produces a findings note the guide depends on. Use WebSearch/WebFetch and,
where a claim cannot be verified, record the honest caveat rather than guessing.

**Files:**
- Create: `docs/superpowers/notes/2026-07-03-verification-findings.md`

- [ ] **Step 1: Cowork on Windows**

Determine how the Cowork desktop app on Windows grants folder access and resolves paths
(does the folder-access UI accept a Windows path; does `~` expand). Record what is
confirmed and what remains uncertain.

- [ ] **Step 2: `.plugin` packaging + install**

Determine how a Cowork plugin is packaged into an installable artifact and how a user
installs it from Settings on macOS and Windows. Record the exact steps.

- [ ] **Step 3: Web Clipper target**

Confirm whether the official Obsidian Web Clipper can append to a specific file
(`reading-list.md`) or whether it creates a note (and if so, whether pointing it at the
wiki `inbox/` is the better recommendation). Record the recommended configuration.

- [ ] **Step 4: Roadmap API sanity**

Confirm Google Tasks API, Microsoft Graph (To Do), and Readwise Reader API exist and are
usable, so the roadmap does not promise the impossible. One line each.

- [ ] **Step 5: Write the findings note**

Record each finding with its source URL(s) and a clear confirmed/uncertain flag. This is
the input to Task 9. No commit gate beyond the file existing and citing sources.

- [ ] **Step 6: Commit**

```bash
git add docs/superpowers/notes/2026-07-03-verification-findings.md
git commit -m "docs: verification findings for onboarding guide"
```

---

## Task 9: GETTING-STARTED.md - the self-contained user guide

**Files:**
- Create: `GETTING-STARTED.md` (repo root)

**Interfaces:**
- Consumes: the findings from Task 8; the tier + capture wording from Tasks 5-6; the roadmap from Task 7.

Write in English, no em-dashes. Self-contained principle: explain only the Obsidian pieces
this use-case needs; no pointers to external Obsidian docs. Sections (each must contain real
content, not a heading placeholder):

- [ ] **Step 1: Concept + vaults**

Section "What this is": the Karpathy LLM Wiki pattern in plain words (the AI maintains a
Markdown wiki that grows; you feed raw material, it structures). Section "The two vaults":
wiki (AI writes) vs personal (yours, AI only reads), and the one rule (the agent writes
only in the wiki).

- [ ] **Step 2: Prerequisites + install (macOS and Windows)**

A Claude Pro/Max account and Cowork. Then step by step, with a macOS path and a Windows
path each: install Cowork; install Obsidian and create the two vaults under
`{{VAULT_BASE}}`; install the plugin (from Task 8's packaging findings); run "set up my
wiki"; grant Cowork folder access to the vaults; connect Gmail (optional); register the
chosen tiers. Where Windows behavior is uncertain (Task 8), state the honest caveat.

- [ ] **Step 3: Capture from your phone (optional) - sync setup**

Use-case first (drop a URL into `reading-list.md`, next morning it becomes a wiki entry).
Then the sync walkthrough with the trade-off up front: **Obsidian Sync** recommended
(~$4-5/month, reliable, cross-platform); **iCloud** free Apple-only fallback WITH caveats
(turn off Optimize Mac Storage / Keep Downloaded, let the app create the vault folder, do
not edit offline on two devices). Then install + operate Obsidian Mobile from zero
(App Store / Google Play, first launch, connect to the vault - differs by sync route, open
`reading-list.md`, append a line). Then optional capture helpers, kept minimal: the
official Obsidian Web Clipper (per Task 8's recommended config) and the built-in mobile
share sheet / optional iOS Shortcut. Recommend against heavier community plugins. State the
whole section is optional.

- [ ] **Step 4: First real use + what to ignore + troubleshooting**

A concrete walkthrough: `ingest-url` on a link, then `wiki-context`, then `process-inbox`.
What to ignore at first (domains, focus lenses). A short troubleshooting list (a job did
nothing -> connector or folder access; tokens like `{{WIKI_VAULT}}` visible in files ->
re-run wiki-setup). Close with a short roadmap excerpt (from `ROADMAP.md`).

- [ ] **Step 5: Verify no em-dashes and no Obsidian-doc pointers**

Run: `grep -n "—" GETTING-STARTED.md`
Expected: no output.
Run: `grep -ni "help.obsidian\|obsidian.md/help\|see the obsidian documentation" GETTING-STARTED.md`
Expected: no output.

- [ ] **Step 6: Commit**

```bash
git add GETTING-STARTED.md
git commit -m "docs: add self-contained non-technical getting-started guide"
```

---

## Task 10: Email to Anna (personal artifact, not committed)

**Files:**
- Create: `<scratchpad>/email-to-anna.md` (session scratchpad, NOT the repo)

**Interfaces:**
- Consumes: `GETTING-STARTED.md` (Task 9), `ROADMAP.md` (Task 7).

- [ ] **Step 1: Draft the email**

German, in Michael's voice per `~/Cowork/memory/context/voice.md` (no em-dashes, ` - `
instead; no AI-slop; a short human opener then straight to the point). Content: what this
is in two sentences, that it is a gift she can install herself, a pointer to the guide,
and a concrete restatement of the roadmap (what works today, that the Apple dependency is
being generalized, Google Tasks / Microsoft To Do coming). Keep it short.

- [ ] **Step 2: Present to Michael**

Show the draft in chat for him to send. Do not commit it to the public repo (it is
personal). No git step.

---

## Self-Review

Spec coverage: state relocation (T1-T4), config-driven vault paths (T1-T4), job tiers
(T4-T5), capture-source abstraction + append-only (T6), ROADMAP (T7), verification items
(T8), self-contained English guide incl. sync + Obsidian Mobile + Web Clipper (T9), German
email (T10). All spec sections map to a task.

Placeholder scan: new-file content is given inline (ROADMAP full text; guide section briefs
with concrete required content). The path sweeps use exact transformation rules + grep
gates rather than pasting all 15 files, which is the honest concrete unit for a Markdown
refactor.

Type/name consistency: token names `{{VAULT_BASE}}`, `{{STATE_DIR}}` are defined in T1 and
used identically in T2-T9. State paths (`{{STATE_DIR}}/`, `reading-list.md`) are consistent
across tasks. Tier labels (core / neutral / mac-only) match between T4, T5, and the guide.
