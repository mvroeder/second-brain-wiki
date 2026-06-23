---
name: wiki-setup
description: One-time setup of the AI-maintained second-brain wiki. Use when the user says "set up my wiki", "install the second brain", "scaffold the wiki vault", "configure the wiki plugin", or runs this plugin for the first time. Collects the user's vault names and identity, writes config.md, scaffolds the wiki vault folders, installs the CLAUDE.md schema, and offers to register the nightly/weekly automation jobs.
---

You are setting up the second-brain wiki engine for a new user. Walk them through it conversationally, one decision at a time. Do not dump file paths on them.

## Step 1 - Collect config
Ask the user (use AskUserQuestion) for: their name, their email, the name of their WRITE-allowed Obsidian vault (default `Wiki`), and the name of their READ-only personal vault (default `Personal`). Then write `${CLAUDE_PLUGIN_ROOT}/config.md` filling every token row. Set `{{PERSONAL_VAULT_ENC}}` to the personal vault name with spaces replaced by `%20`.

## Step 2 - Scaffold the wiki vault
In the user's WRITE vault (`~/Obsidian/<WIKI_VAULT>/`), create the folder skeleton if missing: `inbox/`, `inbox/_archive/`, `Sources/`, `Topics/`, `People/`, `Projects/`, `Maps/`, `_audit/`. Create a starter `Index.md` (empty Topics + Sources tables, a `## Maintenance log` section) and a starter `hot.md` (`As of:` header + the three sections `## Just happened`, `## Open threads`, `## Active clusters`) if they do not exist. Never overwrite existing files - only create what is missing.

## Step 3 - Install the schema
Copy `${CLAUDE_PLUGIN_ROOT}/templates/wiki-vault-CLAUDE.md` to `~/Obsidian/<WIKI_VAULT>/CLAUDE.md`, substituting the config tokens with the user's real values as you write. Optionally copy `templates/obsidian-root-CLAUDE.md` to `~/Obsidian/CLAUDE.md` if they want the root context loader. Copy `templates/routing-rules.md` to `~/Cowork/productivity/routing-rules.md` (token-substituted) for transcript-triage. Create `~/Cowork/productivity/` if needed.

## Step 4 - Automation (optional)
Ask if they want the nightly/weekly jobs registered. For each prompt in `${CLAUDE_PLUGIN_ROOT}/scheduled-prompts/`, use the schedule tool to create a scheduled task with the cron expression in that file's frontmatter `schedule:` field, using the token-substituted prompt body. List the five jobs and their cadence so they can pick. Note which connectors each needs: newsletter-triage -> Gmail; transcript-triage -> Plaud + Monologue; reading-list-triage -> the Reminders/TASKS.md sync. Skip jobs whose connectors are not yet available and tell the user. Treat `reading-list-triage` as optional: only register it if a Reminders -> TASKS.md sync exists.

## Step 5 - Confirm
Summarize what was created (config, vault folders, schema, schedules) and what is still needed (connector authorizations). Point them at the `wiki-context` skill to test a read, and `ingest-url` to test a write.

## Constraints
- Never overwrite existing vault files. Create-if-missing only.
- Token substitution happens at write time - no `{{...}}` tokens should remain in files written into the user's vault or `~/Cowork/productivity/`.
- No em-dashes in anything you write. Use ` - `.
