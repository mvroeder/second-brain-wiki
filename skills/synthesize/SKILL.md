---
name: synthesize
description: Surface emergent cross-topic patterns and clusters - report only, no edits
argument-hint: [optional domain/tag or time-window]
---

> **Before acting:** Read `${CLAUDE_PLUGIN_ROOT}/config.md` and resolve the tokens below:
> `{{WIKI_VAULT}}` (write vault), `{{PERSONAL_VAULT}}` (read-only source vault), `{{PERSONAL_VAULT_ENC}}` (URL-encoded name for markdown links), `{{OWNER}}` (name), `{{OWNER_EMAIL}}`. Wherever this prompt names a token, substitute the configured value.


You are the wiki maintainer for `{{WIKI_VAULT}}/`. Read `CLAUDE.md` and `hot.md` at the vault root before acting.

## Task

Surface the emergent cross-topic patterns that per-ingest linking misses: clusters of pages converging on an unnamed concept, merge candidates, themes in recent activity. **Report only - do not edit the vault.** {{OWNER}} decides what to act on.

Scope: if `$ARGUMENTS` is given, restrict to that domain/tag or time-window (e.g. `security`, `last-7-days`). Default: the whole vault, with emphasis on activity since the last synthesis report in `_audit/`.

## Inputs (read these, cheaply)

- `Index.md` - the catalog (Topics + Sources tables).
- The `summary:` and `tags:` frontmatter of every `Topics/` page (load-bearing and lean - do NOT read full bodies unless a cluster needs confirmation).
- The `## Related` / `[[wikilink]]` graph across Topics and Sources.
- Recent `Sources/` (since the last synthesis run).
- `_audit/_pending.md` - graduation candidates already noted.
- The most recent `_audit/synthesis-*.md`, if any - do not re-surface clusters you already reported unless they have grown.

## Detection

1. **Tag co-occurrence clusters** - tags that repeatedly appear together across pages but have no umbrella Topic.
2. **Related-link density** - 3 or more Sources/Topics linking to each other or to the same small set, converging on a concept that is not yet its own Topic.
3. **Recent-ingest themes** - what the Sources added since the last run circle around.
4. **Merge candidates** - two Topics whose summary and tags overlap heavily (near-duplicates).
5. **Orphan groups** - orphan pages (zero inbound links) that share a theme and could anchor a new Topic together.

## Output

Write `_audit/synthesis-YYYY-MM-DD.md`. Per cluster: the emergent theme, the member pages, the pattern you detected, exactly ONE suggested action (mapping to an existing verb), and a confidence level. Shape:

```markdown
# Wiki Synthesis - YYYY-MM-DD

## Summary
- Clusters surfaced: N
- Suggested: X new Topics, Y merges, W watch

## Clusters
### 1. <Cluster name>  [confidence: high|medium|low]
- Members: [[...]], [[...]]
- Pattern: <one or two sentences - the signal you detected>
- Suggested action: <create Topic "<title>" | merge [[A]] + [[B]] | leave (watch)>
```

## Constraints

- **Report only.** No new Topics, no merges, no edits. {{OWNER}} triggers the chosen actions in a follow-up.
- Every suggested action must map to a real existing verb (`/graduate` or a manual Topic create, or a merge).
- Cite specific pages by `[[wikilink]]`. No vague "various pages".
- Be selective: report a cluster only if it suggests a concrete action. Do not pad.
- No em-dashes (use " - ").

## Final step

Rewrite `hot.md` per the CLAUDE.md hot.md rule: note the synthesis run and any open cluster decisions in `## Open threads` (e.g. "synthesis YYYY-MM-DD: N clusters to decide, see _audit/synthesis-...").

End your response with a one-paragraph headline (the top 1-2 clusters and their suggested actions) and a pointer to the report file.
