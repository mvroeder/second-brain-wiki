# Scheduled prompts

These are the five automation jobs that keep the wiki fed and healthy. They are NOT
installed automatically - the `wiki-setup` skill offers to register them, or you can
create them yourself with the schedule tool. Each file's frontmatter has a `schedule:`
cron expression and a `name`/`description`.

| Job | Cadence | Needs | What it does |
|---|---|---|---|
| `wiki-inbox-process` | daily 03:30 | Obsidian vault | Triage `inbox/`, route to Sources/Topics, update Index. The single write-instance into the wiki. |
| `transcript-triage` | daily 04:00 | Plaud + Monologue | Pull new voice recordings, classify, propose filing (no auto-write). |
| `reading-list-triage` | daily 04:30 | TASKS.md (Reminders sync) - **optional** | Evaluate new Reading-List URLs -> Wiki / Skip. Skip this job if you have no Reminders -> TASKS.md sync. |
| `newsletter-triage` | daily 05:50 | Gmail | Scan newsletters, recommend ingest / read / skip, drop captures into `inbox/`. |
| `wiki-weekly-lint` | Saturday 11:00 | Obsidian vault | Health check: broken links, orphans, stale pages, contradictions -> `_audit/`. |

All three triage jobs are propose-only: they never file anything without your confirmation.
They dedup via JSONL ledgers in `~/Cowork/productivity/`. The chain is:
newsletter/reading-list/transcript propose -> you confirm -> a capture lands in `inbox/`
-> `wiki-inbox-process` is the only thing that writes durably into the wiki.

Write targets differ per job. `wiki-inbox-process` is the only job that writes into the
**wiki vault**. `transcript-triage` is the exception to "personal vault is read-only": once you
confirm, it files meeting, 1-1, personal and brainstorm analyses into the **personal vault**
(`{{PERSONAL_VAULT}}/Meetings`, `/Meetings/1-1`, `/Personal`, `/Ideas`), per
`templates/routing-rules.md`. `reading-list-triage` is optional and only runs with a
Reminders -> TASKS.md sync.
