---
name: ingest-url
description: Fetch a URL, extract the article, write to Sources/, synthesize into Topics/
argument-hint: <url>
---

> **Before acting:** Read `${CLAUDE_PLUGIN_ROOT}/config.md` and resolve the tokens below:
> `{{WIKI_VAULT}}` (write vault), `{{PERSONAL_VAULT}}` (read-only source vault), `{{PERSONAL_VAULT_ENC}}` (URL-encoded name for markdown links), `{{OWNER}}` (name), `{{OWNER_EMAIL}}`. Wherever this prompt names a token, substitute the configured value.


You are the wiki maintainer for `{{WIKI_VAULT}}/`. Read `CLAUDE.md` at the vault root before acting.

## Task

Ingest the URL: `$ARGUMENTS`

### Step 1 - Fetch
Use WebFetch to retrieve the URL. If the page is JS-rendered (empty body, "enable JavaScript", or boilerplate-only), escalate to the Chrome MCP per the harness rules.

### Step 2 - Source note
Create `Sources/YYYY-MM-DD <Article Title>.md` (today's date, sentence-case title from the article). Match the template {{OWNER}} uses in `../{{PERSONAL_VAULT}}/Knowledge/` - read 2-3 recent files there to calibrate style exactly.

Required frontmatter:

```
---
type: source
summary: One-sentence essence
tags: [tag1, tag2, ...]   # kebab-case lowercase
source: <original URL>
created: YYYY-MM-DD
relevance: 1-10
---
```

Required sections:
- `# {Title}`
- `## TLDR`
- `## Summary` with `### Context` and `### Key Arguments` (numbered)
- `## Implications`
- `## Key Principles` (bullets)
- `## Actionable Items` (bullets)

### Step 3 - Topic synthesis
For each net-new or substantially augmented concept in the article, touch the corresponding `Topics/<concept>.md` page. Create if missing, augment if existing. Follow `CLAUDE.md` page conventions (frontmatter, TL;DR, body ≤600 words, open questions, related).

**When augmenting an existing Topic:** always add the new Source to BOTH places:
1. The `sources:` list in the Topic's frontmatter (append the Source path)
2. The `## Related` section (add a wikilink)

The `sources:` frontmatter is the authoritative record of which articles informed this Topic. A Related link without a corresponding `sources:` entry is incomplete.

### Step 4 - Cross-references
Update 3-10 related pages (Topics, People, Projects). Add the new source under their `## Related` section as a wikilink to the new `Sources/...` page.

### Step 5 - Index
Update `Index.md` with the new pages and a one-liner per page.

### Step 6 - Report
End with a one-paragraph diff: which files were created, which were updated, and one open question worth following up.

## Constraints
- **Never write outside `{{WIKI_VAULT}}/`.** Sources go to `Sources/`, not to {{OWNER}}'s Personal vault.
- Every `[[link]]` must point at a real file in this vault (existence-check before emitting).
- Cross-vault references use `[Title](../{{PERSONAL_VAULT_ENC}}/Knowledge/...)` style markdown links.
- No em-dashes. Use ` - `.
- Match {{OWNER}}'s voice - calibrate by reading existing Knowledge/ notes first.
