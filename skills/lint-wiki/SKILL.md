---
name: lint-wiki
description: Health-check the wiki - broken links, orphans, stale pages, contradictions
---

> **Before acting:** Read `${CLAUDE_PLUGIN_ROOT}/config.md` and resolve the tokens below:
> `{{WIKI_VAULT}}` (write vault), `{{PERSONAL_VAULT}}` (read-only source vault), `{{PERSONAL_VAULT_ENC}}` (URL-encoded name for markdown links), `{{OWNER}}` (name), `{{OWNER_EMAIL}}`. Wherever this prompt names a token, substitute the configured value.


You are the wiki maintainer for `{{WIKI_VAULT}}/`. Read `CLAUDE.md` at the vault root before acting.

## Task

Audit the wiki. **Report only - do not auto-fix** unless {{OWNER}} responds with "fix the X items."

Output a single file at `_audit/audit-YYYY-MM-DD.md` and mention it at the end of your response.

## Checks

### 1. Broken in-vault wikilinks
Every `[[link]]` should point at a real file in `{{WIKI_VAULT}}/`. Report:
- Source page
- Broken link text
- Suggested fix (closest existing page by title or slug)

### 2. Broken cross-vault links
Every `[Title](../{{PERSONAL_VAULT_ENC}}/...)` markdown link should resolve to a real file in `../{{PERSONAL_VAULT}}/`. List broken ones - they may indicate files {{OWNER}} moved in his personal vault.

### 3. Missing frontmatter
Pages without `summary:`, `tags:`, or `type:`. List them.

### 4. Oversized pages
Body sections (excluding frontmatter) over 600 words. Candidates for splitting into atomic concepts.

### 5. Orphans
Pages in `Topics/`, `People/`, `Projects/` with zero inbound links.

### 6. Stale projects
`type: project` pages with `updated:` older than 90 days.

### 7. Open questions aged > 30 days
Scan `## Open questions` sections across the vault.

### 8. Contradictions
Pages making opposing claims about the same concept. Flag pairs, do not resolve.

### 9. Tag singletons
Tags that appear on only one page.

## Output format

```markdown
# Wiki Audit — YYYY-MM-DD

## Summary
- Total wiki pages: N
- Broken in-vault links: N
- Broken cross-vault links: N
- Missing frontmatter: N
- Oversized: N
- Orphans: N
- Stale projects: N
- Aged open questions: N
- Possible contradictions: N
- Singleton tags: N

## Details
[one section per category, only if non-empty]
```

End your response with a one-paragraph headline and a pointer to the audit file.
