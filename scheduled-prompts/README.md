# Scheduled prompts

These are the five automation jobs that keep the wiki fed and healthy. They are NOT
installed automatically - the `wiki-setup` skill offers to register them, or you can
create them yourself with the schedule tool. Each file's frontmatter has a `schedule:`
cron expression and a `name`/`description`.

Automation jobs come in three tiers. Enable only the tiers you want; wiki-setup asks.

**Core (Obsidian only):**
- `wiki-inbox-process` (daily 03:30) - triage `inbox/`, route to Sources/Topics, update Index. The single write-instance into the wiki.
- `wiki-weekly-lint` (Saturday 11:00) - health check: broken links, orphans, stale pages, contradictions -> `_audit/`.

**Neutral-optional (one cross-platform connector):**
- `newsletter-triage` (daily 05:50, needs Gmail) - scan newsletters, recommend ingest / read / skip, drop captures into `inbox/`.

**Mac-optional (Apple ecosystem):**
- `transcript-triage` (daily 04:00, needs Plaud + Monologue) - pull new voice recordings, classify, propose filing (no auto-write).
- `reading-list-triage` (daily 04:30, needs a capture source) - evaluate new Reading-List URLs -> Wiki / Skip. Skip this job if you have no Reminders -> TASKS.md sync.

All three triage jobs are propose-only: they never file anything without your confirmation.
They dedup via JSONL ledgers in `{{STATE_DIR}}` (the wiki vault is already the granted folder; the state dir is `{{STATE_DIR}}` inside it). The chain is:
newsletter/reading-list/transcript propose -> you confirm -> a capture lands in `inbox/`
-> `wiki-inbox-process` is the only thing that writes durably into the wiki.

Write targets differ per job. `wiki-inbox-process` is the only job that writes into the
**wiki vault**. `transcript-triage` is the exception to "personal vault is read-only": once you
confirm, it files meeting, 1-1, personal and brainstorm analyses into the **personal vault**
(`{{PERSONAL_VAULT}}/Meetings`, `/Meetings/1-1`, `/Personal`, `/Ideas`), per
`templates/routing-rules.md`. `reading-list-triage` is optional and only runs with a
Reminders -> TASKS.md sync.
