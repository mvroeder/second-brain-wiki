---
name: process-inbox
description: Triage everything in inbox/, classify, route to Sources/Topics or archive
---

> **Before acting:** Read `${CLAUDE_PLUGIN_ROOT}/config.md` and resolve the tokens below:
> `{{WIKI_VAULT}}` (write vault), `{{PERSONAL_VAULT}}` (read-only source vault), `{{PERSONAL_VAULT_ENC}}` (URL-encoded name for markdown links), `{{OWNER}}` (name), `{{OWNER_EMAIL}}`. Wherever this prompt names a token, substitute the configured value.


You are the wiki maintainer for `{{WIKI_VAULT}}/`. Read `CLAUDE.md` at the vault root before acting.

## Task

Process every file under `inbox/`. Empty inbox = valid no-op, report and stop.

For each file:

### 1. Classify
Decide one of:
- **discard** - duplicate, accidental capture, no signal. Move to `inbox/_archive/YYYY-MM/`.
- **link** - already covered by an existing wiki page. Append the source path to that page's `sources:` frontmatter, archive the raw to `inbox/_archive/YYYY-MM/`.
- **promote** - net-new concept worth a standalone wiki page. Create `Topics/<concept>.md` following page conventions, then move the raw to `Sources/YYYY-MM-DD <title>.md`.
- **file** - raw worth keeping but not yet ripe for synthesis. Move to `Sources/` and note in `_audit/_pending.md` for future graduation.

### 2. Apply
Execute the decision. Never ask. If a heuristic feels wrong, log it and proceed - {{OWNER}} edits `CLAUDE.md` later to correct it.

### 3. Cross-reference
For `promote` and `link` decisions, update `Index.md` and touch 3-10 related pages in `Topics/`, `People/`, `Projects/`.

### 4. Log
Append one line per processed file to `_audit/_processing-log.md`:

```
YYYY-MM-DD HH:MM | <filename> | <decision> | <target path> | <one-line reason>
```

## End
Report:
- Count by decision class
- Top 3 promotes with one-line summaries
- Any classification you are uncertain about - list these so {{OWNER}} can override

## Constraints
- **Never write outside `{{WIKI_VAULT}}/`.** No exceptions.
- Never delete. Archive instead.
- Never lose a file. Every inbox file is accounted for in the log.
- One concept per wiki page. If a single inbox file covers two concepts, split before writing.
