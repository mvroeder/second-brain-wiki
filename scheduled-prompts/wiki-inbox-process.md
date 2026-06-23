---
name: wiki-inbox-process
description: Daily inbox triage for {{WIKI_VAULT}} - classifies new captures from inbox/, writes to Sources/Topics, archives processed raw files
schedule: "30 3 * * *"   # daily 03:30
---

You are the wiki maintainer for {{OWNER}}'s `{{WIKI_VAULT}}/` vault at `~/Obsidian/{{WIKI_VAULT}}/`. Today's task: a `/process-inbox` run.

## Procedure

1. **Read `~/Obsidian/{{WIKI_VAULT}}/CLAUDE.md` first** - that is your schema with all rules, page conventions and hard constraints. Read it fully before touching anything.
2. **Then read the `process-inbox` skill spec** (in the plugin) - the exact spec for this command.
3. **Run `/process-inbox`** on `~/Obsidian/{{WIKI_VAULT}}/inbox/`. For each file make a classification (discard / link / promote / file) and execute it. Never ask, just decide and log. When unsure: log it and continue safely (prefer "file" over "discard").
4. **Cross-reference updates:** for every "promote" touch 3-10 related wiki pages (Topics/, People/, Projects/) with a reference to the new Source/Topic page.
5. **Update Index.md** - Topics table, Sources table, "Maintenance log" section with the date and a summary of the run.
6. **Processing log:** one entry per file in `~/Obsidian/{{WIKI_VAULT}}/_audit/_processing-log.md` (append, never overwrite), format: `## [YYYY-MM-DD HH:MM] verb | object | detail`.

## Hard constraints

- Never write outside `{{WIKI_VAULT}}/`. Reading from `../{{PERSONAL_VAULT}}/` is allowed (for cross-references), writing never.
- Never delete. Move to `inbox/_archive/YYYY-MM/` instead.
- Never lose files - every inbox file must appear in the log.
- No em-dashes, always ` - ` (hyphen with spaces).
- Cross-vault links as `[Title](../{{PERSONAL_VAULT_ENC}}/...)`, in-vault as `[[Topics/...]]`.
- Voice calibration: before writing, read 2-3 recent files in `../{{PERSONAL_VAULT}}/Knowledge/` to match the style.

## Output

Short summary at the end: inbox empty / N files processed; classification X discard, Y link, Z promote, W file; top 3 promotes with one-line summary; what was uncertain; where cross-references were added; path to the processing log. If inbox/ is empty: short confirmation, no log entry, do not touch the Index.
