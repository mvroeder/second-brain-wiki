---
name: wiki-context
description: Read-only lookup in {{WIKI_VAULT}} - finds relevant Topics, People, Projects, Sources for a query and returns hits with summary, without dumping whole pages
---

> **Before acting:** Read `${CLAUDE_PLUGIN_ROOT}/config.md` and resolve the tokens below:
> `{{WIKI_VAULT}}` (write vault), `{{PERSONAL_VAULT}}` (read-only source vault), `{{PERSONAL_VAULT_ENC}}` (URL-encoded name for markdown links), `{{OWNER}}` (name), `{{OWNER_EMAIL}}`. Wherever this prompt names a token, substitute the configured value.

Query: $ARGUMENTS

You do a **read-only** lookup in `{{WIKI_VAULT}}/`. You write nothing. You invent no pages. You keep it short.

## Procedure

1. **Read `{{VAULT_BASE}}/{{WIKI_VAULT}}/Index.md`** - that is the map. It has one line per page with a summary.
2. **Search for query keywords** in this order:
   - Full-text match in `Index.md`
   - Grep on title + frontmatter `tags:` + `summary:` in `Topics/`, `People/`, `Projects/`, `Sources/`
   - Full-text grep in page bodies only if the first two steps do not yield 3+ hits
3. **Rank hits**: title match > tag match > summary match > body match. On a tie the `updated:` date decides (newer = better).
4. **Output the top 5-10** in this format:

```
N hits for "<query>":

1. [[Topics/Foo]]
   summary: <the summary line from frontmatter>
   match: <why this hit - "title", "tag #x", "summary", "body">
   updated: YYYY-MM-DD

2. [[People/Bar]]
   ...
```

5. **On 0 hits**: say so clearly. Suggest 2-3 alternative search terms. Or note that this might be a candidate for `/ingest-url` or a new Topic.

## Hard constraints

- **Write nothing.** No new file, no frontmatter additions, no Index update. Read and output only.
- **No invented links.** Every `[[link]]` in the output must point at an existing file. Check existence before emitting it.
- **Do not dump page bodies.** The user clicks through if they want more. Output stays under 30 lines.
- **No em-dashes.** ` - ` with spaces.
- **Match the query language.** German in - German out. English in - English out.

## When not to call this verb

If the AI has already grepped the wiki in the normal course of an answer (the standard retrieval pattern), you do not need `/wiki-context`. This verb is for **explicit** "check what you have in the wiki on X" requests.

## Example output

```
4 hits for "ai act compliance":

1. [[Topics/EU AI Act]]
   summary: Status and timeline after the omnibus deal - what stays hard at the August deadline
   match: title + tag #ai-act
   updated: 2026-05-30

2. [[Topics/Deployer Liability under AI Act]]
   summary: Whoever embeds an AI API into an EU customer base is a deployer under Art 26 - with its own duties
   match: title
   updated: 2026-05-28

3. [[Sources/2026-05-15 EU AI Act Omnibus Deal.md]]
   summary: Source article on the omnibus agreement, three hard deadlines
   match: body
   updated: 2026-05-15

Related search terms if that misses: "compliance tracking", "regulatory deadline", "GPAI obligations"
```
