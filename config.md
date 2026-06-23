# config.md - fill this once before using the wiki skills

This file resolves the tokens used throughout the skills and scheduled prompts.
Replace the values on the right. The skills read this file before acting.

| Token | Meaning | Your value |
|---|---|---|
| `{{OWNER}}` | Your name (used in agent identity + voice calibration) | _Your name_ |
| `{{OWNER_EMAIL}}` | Your email (used by the Gmail/triage jobs) | _you@example.com_ |
| `{{WIKI_VAULT}}` | Name of your WRITE-allowed Obsidian vault | _Wiki_ |
| `{{PERSONAL_VAULT}}` | Name of your READ-ONLY personal vault | _Personal_ |
| `{{PERSONAL_VAULT_ENC}}` | URL-encoded form of the personal vault name (spaces -> %20), for markdown links | _Personal_ |

## Focus lenses (optional)

Some jobs (`newsletter-triage`, `reading-list-triage`) and `/ingest-scan` rate incoming
material against your personal focus areas. Define yours here (or in the wiki schema) - they
are free-form. If you leave this empty, those jobs simply rate by general relevance.

- Lens 1: _e.g. your primary domain_
- Lens 2: _e.g. a second area you track_
- Lens 3: _e.g. a third_

## Local state (per-user, created on first run)

- `~/Cowork/productivity/` holds the dedup ledgers and `routing-rules.md`:
  - `ingest-ledger.jsonl`        (transcript-triage)
  - `newsletter-ledger.jsonl`    (newsletter-triage)
  - `reading-list-ledger.jsonl`  (reading-list-triage)
  - `routing-rules.md`           (binding routing reference for transcript-triage;
                                  a starter copy ships in templates/routing-rules.md)
  - `TASKS.md`                   (OPTIONAL - your reminders sync file, source for reading-list-triage;
                                  only if you run a Reminders -> TASKS.md sync)
