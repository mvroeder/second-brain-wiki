---
name: graduate
description: Scan {{PERSONAL_VAULT}} Daily note/ and Ideas/ for recurring ideas worth graduating to Topics
---

> **Before acting:** Read `${CLAUDE_PLUGIN_ROOT}/config.md` and resolve the tokens below:
> `{{WIKI_VAULT}}` (write vault), `{{PERSONAL_VAULT}}` (read-only source vault), `{{PERSONAL_VAULT_ENC}}` (URL-encoded name for markdown links), `{{OWNER}}` (name), `{{OWNER_EMAIL}}`. Wherever this prompt names a token, substitute the configured value.


You are the wiki maintainer for `{{WIKI_VAULT}}/`. Read `CLAUDE.md` at the vault root before acting.

## Task

Scan `../{{PERSONAL_VAULT}}/Daily note/` and `../{{PERSONAL_VAULT}}/Ideas/` for the last 30 days. Identify ideas worth graduating to standalone `Topics/` pages here in `{{WIKI_VAULT}}/`.

## Signals to weight

An idea is graduation-worthy if ANY of:
- Mentioned in 3+ daily notes across the window
- Marked with `>>` or `★` ({{OWNER}}'s "this matters" markers)
- Cross-referenced from a `../{{PERSONAL_VAULT}}/Briefings/` note
- Tagged `#graduate` explicitly
- Sits in `../{{PERSONAL_VAULT}}/Ideas/` with body > 200 words and `updated:` within 14 days (active thinking)

## Output

Propose 1-5 ideas for graduation. For each:

```
### Candidate: <Working Title>
- Signal: <which heuristic fired, how strongly>
- Source files: <list of dailies / ideas / briefings, each with relative path>
- Proposed page: Topics/<slug>.md
- Draft TL;DR: <one sentence>
- Related existing wiki pages: [[...]] [[...]]
- Related existing personal-vault content: [Title](../{{PERSONAL_VAULT_ENC}}/...)
```

## Wait for confirmation

DO NOT create the pages yet. Wait for {{OWNER}} to respond with `graduate 1, 3` (or `all`, or `none, here is a different angle`). Then create the confirmed pages in `Topics/` following `CLAUDE.md` page conventions.

When creating the page, link back to the source dailies/ideas in the `sources:` frontmatter and the `## Related` section. **Do not modify the source files in `../{{PERSONAL_VAULT}}/`.**

## Constraint

This is the only verb that asks before acting. `/ingest-url`, `/process-inbox`, `/lint-wiki` all decide and report. `/graduate` proposes and waits.
