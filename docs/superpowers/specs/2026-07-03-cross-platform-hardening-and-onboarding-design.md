# Design: cross-platform hardening + non-technical onboarding

Date: 2026-07-03
Repo: `second-brain-wiki` (the generic version)
Status: design, awaiting implementation plan

## Problem

The "generic" version still carries two personal assumptions that break it for anyone
who is not Michael on a Mac:

1. **Local state is hardcoded to `~/Cowork/productivity/`** (12+ references across README,
   SETUP, config.md, wiki-setup, routing-rules, and every scheduled prompt). `~/Cowork/` is
   Michael's private git repo, a personal convention. On a fresh machine (and on Windows) it
   does not exist and has no meaning. The dedup ledgers, `routing-rules.md`, and `TASKS.md`
   all live there.
2. **The vault base is hardcoded to `~/Obsidian/`** in the templates, skills, and the
   vault-root `CLAUDE.md` ("...two parallel vaults at `~/Obsidian/`"). On Windows this path is
   wrong, and Obsidian vaults can live anywhere anyway.

On top of that, a first-time user (never used Obsidian, does not know the Karpathy "LLM Wiki"
pattern, possibly on Windows) has no gentle on-ramp. The README is a dense reference manual,
not an onboarding.

Cowork operates only on folders the user explicitly grants/mounts (`request_directory`). So
the plugin's reachable state is a function of the user's Cowork folder-access configuration.
That makes folder access a first-class part of setup, not an afterthought.

## Goal

Make the generic version genuinely generic and cross-platform (macOS + Windows), give
non-technical users a real onboarding, and make the connector-dependent jobs cleanly
optional. Publish a roadmap so anyone who receives the plugin sees where it is going.

## Non-goals (YAGNI)

- No `second-brain-wiki-anna` fork. We harden the generic repo itself.
- No new connector adapters built in this pass (Google Tasks, MS To Do, Readwise). They are
  roadmap items only. The capture-source abstraction is built; the additional adapters are not.
- No porting of the Apple-only jobs to Windows. They stay Mac-tier and optional. On a Mac they
  keep working unchanged.
- No generalization of the mail seam (Gmail) or voice seam (Plaud/Monologue) in this pass.
  They are named in the roadmap as the same pattern, nothing more.

## Locked decisions

| Decision | Value |
|---|---|
| Scope | Core verbs + platform-neutral automation (`newsletter-triage`/Gmail, `wiki-weekly-lint`). Apple jobs (`transcript-triage`, `reading-list-triage`) stay as an optional Mac tier. |
| Vault model | Two vaults from day 1. Personal may start empty. The one hard rule: the wiki agent only ever writes inside the wiki vault. |
| Client | Cowork Desktop app (macOS + Windows). A Claude Pro/Max account is assumed. |
| State location | Relocate from `~/Cowork/productivity/` into the wiki vault. Operational state (ledgers, routing-rules) in a hidden `{{WIKI_VAULT}}/.wiki-state/`. The reading-list capture file is visible and mobile-editable at `{{WIKI_VAULT}}/reading-list.md`. |
| Vault paths | Config-driven. Add a vault-base/vault-path config value; remove `~/Obsidian/` hardcodes from templates, skills, and the vault-root `CLAUDE.md`. |
| Capture source | Abstracted behind a small contract. Default backend = the vault Markdown file. Apple Reminders becomes one adapter among peers. |
| Connector roadmap | Tier 0 (now): vault file. Tier 1 (roadmap): Google Tasks, Microsoft To Do, Apple Reminders as an equal peer. Tier 2 (later): Readwise Reader. |
| Languages | Spec, `README`, `ROADMAP.md`, user guide: English (repo-consistent, given to a broad audience). Email to Anna: German (personal handover). |

Factual note: "Google Reminders" no longer exists as a separate product; Google merged it
into **Google Tasks** in 2023. The roadmap names Google Tasks.

## Architecture changes

### 1. State relocation

- All local state moves from `~/Cowork/productivity/` to the wiki vault.
  - Ledgers (`ingest-ledger.jsonl`, `newsletter-ledger.jsonl`, `reading-list-ledger.jsonl`)
    and `routing-rules.md` → `{{WIKI_VAULT}}/.wiki-state/` (hidden in Obsidian, no note clutter).
  - The reading-list capture file → `{{WIKI_VAULT}}/reading-list.md` (visible, editable from
    Obsidian Mobile so URLs can be dropped in from a phone).
- Rationale: the wiki vault is already a granted folder (it is the write target), so this
  removes an entire folder grant, is cross-platform for free (Obsidian resolves the path), and
  deletes the `~/Cowork/` personal dependency.
- Every one of the 12+ references is updated. `config.md` documents the new location. A config
  value may point the state dir elsewhere for power users, defaulting to `.wiki-state/`.

### 2. Config-driven vault paths

- Introduce a config value for the vault base (or full per-vault paths) so `~/Obsidian/` is
  never assumed. Windows users set their real path.
- Update `templates/wiki-vault-CLAUDE.md` (the `~/Obsidian/` line and the "runs Cowork + Code"
  assumption), `templates/obsidian-root-CLAUDE.md`, and every skill/scheduled-prompt that
  currently writes `~/Obsidian/{{WIKI_VAULT}}/...` to resolve the base from config.

### 3. Job optionality tiers

Three tiers, made first-class in the README and in `wiki-setup`:

- **Core** (Obsidian only): `wiki-inbox-process`, `wiki-weekly-lint`.
- **Neutral-optional** (one cross-platform connector): `newsletter-triage` (Gmail).
- **Mac-optional** (Apple ecosystem): `transcript-triage` (Plaud/Monologue),
  `reading-list-triage` (when backed by Apple Reminders).

`wiki-setup` asks which tiers to enable and registers only those. Missing connectors already
degrade gracefully; the tiers make that explicit instead of implicit.

### 4. Capture-source abstraction

- Define a minimal contract for a "capture source": it yields items (URL and/or title) and can
  mark an item as processed (so the ledger dedups).
- Ship one backend now: the **vault Markdown file** (`reading-list.md`). `reading-list-triage`
  is reworded to read "a capture source" with the vault file as default, instead of "Apple
  Reminders TASKS.md".
- `reading-list.md` is **human-append-only and read-only to the agent**: the job reads it and
  records processed state in the ledger, it never mutates the file. This keeps the file
  single-writer and sync-conflict-safe regardless of the sync backend (important given iCloud's
  silent-overwrite behavior, see verification item 2).
- Apple Reminders becomes a documented adapter (unchanged behavior on a Mac, via the existing
  Reminders → file sync). Google Tasks / MS To Do / Readwise are roadmap adapters against the
  same contract.

## Documentation deliverables

### A. `ROADMAP.md` (repo root, English, public)

Visible to everyone who receives the plugin. Contents:
- Vision: single-user Mac tool → cross-platform, connector-pluggable second brain.
- Now / shipped.
- Next: capture-source adapters (Google Tasks, Microsoft To Do, Apple Reminders as a peer);
  explicit line that the Apple-Reminders dependency is being generalized.
- Later: Readwise Reader and other read-later services.
- The two further source seams (mail, voice) named as the same pluggable pattern.

### B. User guide (English, non-technical)

A new standalone document, separate from the dense README (which stays the technical
reference). **Self-contained principle:** the reader must never need to open Obsidian's own
documentation. The guide explains exactly the Obsidian pieces this use-case needs (install,
open a vault, edit one file, sync) and nothing more about Obsidian in general, with no pointers
to external Obsidian docs. From zero:
1. What this is: the Karpathy "LLM Wiki" pattern in plain words.
2. The two vaults and the one rule.
3. Prerequisites: Claude account, Cowork.
4. Install step by step, **with both a macOS and a Windows path**: Cowork, Obsidian + two
   vaults, the plugin, `"set up my wiki"`, granting Cowork folder access, connecting Gmail,
   registering the enabled tiers.
5. **Capture from your phone (optional) - sync setup, step by step.** First the use-case: drop
   a URL into `reading-list.md` on the phone, and the next morning it becomes a wiki entry. Then
   a concrete "what goes where" walkthrough of both routes, with the honest trade-off stated up
   front (Obsidian Sync is recommended but costs money; iCloud is free but has caveats):
   - **Recommended: Obsidian Sync** (~$4-5/month). Enable it, sign in on desktop and in the
     mobile app, pick the vaults to sync. Reliable, cross-platform, proper conflict handling.
   - **Free alternative: iCloud** (Apple only). Where the vault goes (the app-created `Obsidian`
     folder in iCloud Drive, not a hand-made folder), turn OFF "Optimize Mac Storage" / set
     "Keep Downloaded", and the caveat not to edit offline on two devices at once.
   - **Install and operate Obsidian Mobile** (she has never used it, so from zero): download
     from the App Store / Google Play, first launch, and connect it to the vault - noting the
     connect step differs by sync route (iCloud: open the vault from the iCloud `Obsidian`
     folder; Obsidian Sync: sign in and pull the remote vault). Then basic operation on a phone:
     find and open `reading-list.md`, add a line with the URL, it saves automatically. Written
     as concrete step-by-step; real device screenshots, if wanted, are captured by Michael on
     an actual phone (they cannot be generated).
   - **Optional capture helpers (kept minimal on purpose):** recommend the **official Obsidian
     Web Clipper** browser extension for one-click desktop capture (first-party, works in
     Chrome/Edge/Firefox and in Safari on iPhone/iPad, so it doubles as Apple mobile capture);
     on Android/iOS the **built-in share sheet** ("Add text to file" -> `reading-list.md`) needs
     no plugin, and an optional iOS Shortcut makes it one tap (Michael can pre-build it for
     Anna). Deliberately recommend AGAINST heavier community plugins (QuickAdd, Dataview,
     Templater): they add exactly the Obsidian learning cost we are avoiding, and the agent
     already does all structuring. Confirm Web Clipper's exact target (append to
     `reading-list.md` vs. a capture note in `inbox/`) and the mobile share-append flow during
     implementation.
   - State clearly that this whole section is optional: without it she just edits
     `reading-list.md` on the desktop, no phone and no sync needed.
6. First real use: a concrete `ingest-url` → `wiki-context` → `process-inbox` walkthrough.
7. What to ignore at the start (domains, focus lenses).
8. Mini troubleshooting.
9. A short roadmap excerpt so a non-technical reader sees where it is heading.

### C. Email to Anna (German, Michael's voice)

The personal handover. Points her to the guide, and restates the relevant roadmap concretely
(what works today, what is coming, the Apple → cross-platform generalization). Written to the
voice constraints in `~/Cowork/memory/context/voice.md` (no em-dashes, ` - ` instead; no
AI-slop; direct).

## Open verification items (resolve during the plan, before writing the guide)

These are not design blockers, but the guide must not state them on faith:

1. **Cowork on Windows:** how paths resolve, how the folder-access / `request_directory` UI
   works, whether `~` expands. The design deliberately avoids relying on `~` expansion, but the
   guide's Windows screenshots/steps must match reality.
2. **Obsidian Mobile + sync (resolved 2026-07-03):** Obsidian Mobile does NOT require Obsidian
   Sync; the app is free. But the phone-capture use-case needs the vault to sync between the
   phone and the desktop where Cowork runs. iCloud is a known footgun for Obsidian: "Optimize
   Mac Storage" can offload/evict vault files (Obsidian expects them local), and offline edits
   can silently overwrite on conflict (documented data-loss reports on the Obsidian forum). So
   the guide recommends **Obsidian Sync** (~$4-5/month, purpose-built, cross-platform, proper
   conflict handling) as the reliable default for a non-technical user. iCloud is documented
   only as a free Apple-only fallback WITH its caveats (disable Optimize Storage / Keep
   Downloaded, do not edit offline on two devices, let the app create the vault folder).
   Syncthing/Git are free-but-technical. Baseline: desktop-only, no sync, no phone; mobile
   capture is the optional upgrade. Design mitigation: `reading-list.md` is human-append-only /
   read-only to the agent (see section 4), so the worst two-writer conflict on that file never
   arises regardless of backend. Sources: obsidian.md/help/sync-notes, forum.obsidian.md iCloud
   data-loss and optimize-storage threads.
3. **`.plugin` packaging:** how Cowork expects a plugin to be packaged/installed, so the
   distribution step in the guide is correct.
4. **Roadmap accuracy:** confirm Google Tasks API, Microsoft Graph (To Do), and Readwise Reader
   API exist and are viable, so the roadmap does not promise something impossible.

## Risks

- State-in-vault means operational files live inside the knowledge vault. Mitigated by the
  hidden `.wiki-state/` folder (Obsidian hides dot-folders). Reviewer should confirm the hidden
  location over a visible `_state/`.
- Touching 12+ files for the path refactor is broad; risk of a missed reference leaving a
  `~/Cowork/` or `~/Obsidian/` hardcode. The plan includes a final grep gate.
- Scope creep toward building connector adapters. Held off by the non-goals: adapters are
  roadmap text, not code, in this pass.

## Rough phasing (for the implementation plan)

1. Path/state refactor (state → `.wiki-state/`, vault base → config), with a grep gate.
2. Job tiering + `wiki-setup` opt-in.
3. Capture-source abstraction + vault-file default; reword `reading-list-triage`.
4. `ROADMAP.md`.
5. Verification pass (the four open items).
6. User guide (German).
7. Email to Anna (German).
