# config.md - fill this once before using the wiki skills

This file resolves the tokens used throughout the skills and scheduled prompts.
Replace the values on the right. The skills read this file before acting.

| Token | Meaning | Your value |
|---|---|---|
| `{{OWNER}}` | Your name (used in agent identity + voice calibration) | _Your name_ |
| `{{OWNER_EMAIL}}` | Your email (used by the Gmail/triage jobs) | _you@example.com_ |
| `{{WIKI_VAULT}}` | Name of your WRITE-allowed Obsidian vault | _Wiki_ |
| `{{VAULT_BASE}}` | The directory that holds your vaults. On Windows set an absolute path, e.g. `C:\Users\<name>\Obsidian`. | `~/Obsidian` |
| `{{STATE_DIR}}` | Where the plugin keeps its operational state (dedup ledgers, routing-rules). Default places it inside the wiki vault so it needs no separate folder grant and is cross-platform. | `{{VAULT_BASE}}/{{WIKI_VAULT}}/.wiki-state` |
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

- `{{STATE_DIR}}/` holds the dedup ledgers (`*-ledger.jsonl`) and `routing-rules.md`.
  It lives inside the wiki vault, so it needs no separate folder grant and is
  cross-platform. `reading-list.md` (the capture file) sits at the wiki vault root,
  visible so you can edit it from Obsidian Mobile.
