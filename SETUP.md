# Setup

## Fastest path
Install the plugin, then tell Claude: **"set up my wiki"**. The `wiki-setup` skill does
everything below interactively. The manual steps are here for reference / if you prefer.

## 1. Fill config.md
Open `config.md` and replace each token value with yours:
- `{{OWNER}}` / `{{OWNER_EMAIL}}` - your name + email
- `{{WIKI_VAULT}}` - your write-allowed Obsidian vault (e.g. `Wiki`)
- `{{PERSONAL_VAULT}}` - your read-only personal vault (e.g. `Personal`)
- `{{PERSONAL_VAULT_ENC}}` - same name, spaces as `%20` (for markdown links)

Optionally define your focus lenses in `config.md` (used by the triage jobs and `/ingest-scan`).

## 2. Scaffold the wiki vault
In `{{VAULT_BASE}}/<WIKI_VAULT>/` create: `inbox/`, `inbox/_archive/`, `Sources/`, `Topics/`,
`People/`, `Projects/`, `Maps/`, `_audit/`, plus a starter `Index.md` and `hot.md`.

## 3. Install the schema (important)
Copy `templates/wiki-vault-CLAUDE.md` to `{{VAULT_BASE}}/<WIKI_VAULT>/CLAUDE.md`, replacing the
tokens with your real values. This is the rulebook every verb reads at the start of a run -
without it at the vault root the skills have no schema. Optionally copy
`templates/obsidian-root-CLAUDE.md` to `{{VAULT_BASE}}/CLAUDE.md`.

## 4. Local state for the triage jobs
Create `{{STATE_DIR}}/` and copy `templates/routing-rules.md` there (token-substituted).
The JSONL ledgers (`ingest-ledger`, `newsletter-ledger`, `reading-list-ledger`) are created on
first run. `reading-list-triage` is **optional** - it reads a `reading-list.md` reminders-sync file at
`{{VAULT_BASE}}/{{WIKI_VAULT}}/reading-list.md`. If you do not run a Reminders -> reading-list.md sync, skip this job; `reading-list-ledger`
and `reading-list.md` are then not needed.

## 5. Connect the tools
Authorize the connectors you want: Gmail (`newsletter-triage`), Plaud + Monologue
(`transcript-triage`), and (optional) a Reminders -> TASKS.md sync for `reading-list-triage`.
Skills whose connectors are missing simply won't run until connected.

## 6. Register automation (optional)
For each file in `scheduled-prompts/`, create a scheduled task using its `schedule:` cron and
token-substituted body. `wiki-setup` can do this for you. See `scheduled-prompts/README.md`.

## Notes on what was generalized from the original
- Identity, vault names and cross-vault link paths are tokenized (`config.md`).
- The dedup ledgers and `routing-rules.md` are per-user local state, recreated on first run.
- The personal vault is read-only for the wiki verbs, but `transcript-triage` writes
  meeting/networking/personal/brainstorm analyses into `{{PERSONAL_VAULT}}/Meetings`,
  `/Meetings/1-1`, `/Personal` and `/Ideas`. That is the one automation that files into the
  personal vault - see `scheduled-prompts/README.md` and `templates/routing-rules.md`.
- `reading-list-triage` depends on a Reminders -> TASKS.md sync. If you have none, leave the
  job unregistered; everything else works without it.
