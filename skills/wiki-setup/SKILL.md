---
name: wiki-setup
description: One-time setup of the AI-maintained second-brain wiki. Use when the user says "set up my wiki", "install the second brain", "scaffold the wiki vault", "configure the wiki plugin", or runs this plugin for the first time. Collects the user's vault names and identity, writes config.md, scaffolds the wiki vault folders, installs the CLAUDE.md schema, and offers to register the nightly/weekly automation jobs.
---

You are setting up the second-brain wiki engine for a new user. Walk them through it conversationally, one decision at a time. Do not dump file paths on them.

## Step 1 - Collect config
Ask the user (use AskUserQuestion) for: their name, their email, the name of their WRITE-allowed Obsidian vault (default `Wiki`), and the name of their READ-only personal vault (default `Personal`). Then write `${CLAUDE_PLUGIN_ROOT}/config.md` filling every token row. Set `{{PERSONAL_VAULT_ENC}}` to the personal vault name with spaces replaced by `%20`.

## Step 2 - Scaffold the wiki vault
In the user's WRITE vault (`{{VAULT_BASE}}/{{WIKI_VAULT}}/`), create the folder skeleton if missing: `inbox/`, `inbox/_archive/`, `Sources/`, `Topics/`, `People/`, `Projects/`, `Maps/`, `_audit/`. Create a starter `Index.md` (empty Topics + Sources tables, a `## Maintenance log` section) and a starter `hot.md` (`As of:` header + the three sections `## Just happened`, `## Open threads`, `## Active clusters`) if they do not exist. Also create the hidden state dir `{{STATE_DIR}}/` if missing, and a starter `{{VAULT_BASE}}/{{WIKI_VAULT}}/reading-list.md` with a `## Reading List` heading and a one-line comment noting it is append-only (humans add entries; the triage jobs only read). Never overwrite existing files - only create what is missing.

## Step 3 - Install the schema
Copy `${CLAUDE_PLUGIN_ROOT}/templates/wiki-vault-CLAUDE.md` to `{{VAULT_BASE}}/{{WIKI_VAULT}}/CLAUDE.md`, substituting the config tokens with the user's real values as you write. Optionally copy `templates/obsidian-root-CLAUDE.md` to `{{VAULT_BASE}}/CLAUDE.md` if they want the root context loader. Copy `templates/routing-rules.md` to `{{STATE_DIR}}/routing-rules.md` (token-substituted) for transcript-triage. Create `{{STATE_DIR}}/` if needed.

## Step 4 - Automation (optional)
Ask which automation tiers to enable, and register only the chosen ones:
- Core (Obsidian only): wiki-inbox-process, wiki-weekly-lint. Recommended default: on.
- Neutral (needs Gmail): newsletter-triage.
- Mac-only (needs Plaud/Monologue, or an Apple Reminders sync): transcript-triage,
  reading-list-triage. Only offer these on macOS; skip on Windows.
Do not register a job whose tier the user declined.

For each prompt in `${CLAUDE_PLUGIN_ROOT}/scheduled-prompts/` matching a chosen tier, use the schedule tool to create a scheduled task with the cron expression in that file's frontmatter `schedule:` field, using the token-substituted prompt body. Note which connectors each needs: newsletter-triage -> Gmail; transcript-triage -> Plaud + Monologue; reading-list-triage -> the Reminders/reading-list.md sync. Skip jobs whose connectors are not yet available and tell the user. Treat `reading-list-triage` as optional even within the mac-only tier: only register it if a Reminders -> reading-list.md sync exists.

## Step 5 - Confirm
Summarize what was created (config, vault folders, state dir, schema, schedules) and what is still needed (connector authorizations). Point them at the `wiki-context` skill to test a read, and `ingest-url` to test a write.

## Constraints
- Never overwrite existing vault files. Create-if-missing only.
- Token substitution happens at write time - no `{{...}}` tokens should remain in files written into the user's vault, into `{{STATE_DIR}}/`, or in `reading-list.md`.
- No em-dashes in anything you write. Use ` - `.
