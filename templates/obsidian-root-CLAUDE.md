# CLAUDE.md - Obsidian Root Context

> This file is loaded automatically when a Cowork or Code session mounts the Obsidian folder. It introduces the vault structure and anchors the wiki read pattern.

## Vault structure

```
{{VAULT_BASE}}/
├── {{PERSONAL_VAULT}}/         {{OWNER}}'s personal vault - notes, briefings, daily logs, knowledge articles.
│                         READ-ONLY for you. Never write there unless {{OWNER}} explicitly asks.
│
├── {{PERSONAL_VAULT}} - DEV/   Sandbox vault for experiments. Writing okay when relevant.
│
├── {{WIKI_VAULT}}/             AI-maintained second-brain vault. WRITE-ALLOWED.
│                         The full schema and hard rules live in {{WIKI_VAULT}}/CLAUDE.md - read it BEFORE you write there.
│
└── Last30Days/           Output folder for research from the last30days skill. Read/write depending on the task.
```

## What you do automatically

On every substantial request (research, briefing, decision-making, a person reference, a project reference, a reference to earlier work) you check the wiki **before** you answer:

1. Read `{{VAULT_BASE}}/{{WIKI_VAULT}}/Index.md` or grep it for the request's keywords.
2. On hits, read the relevant pages. The frontmatter field `summary:` tells you quickly whether the page is load-bearing.
3. Use the content in your answer. Cite inline in `[[Topics/...]]` style. Do not paraphrase without attribution.

You **do not announce this step**. No "let me check the wiki". You just look. Mention the wiki only when you actually found something relevant. For trivial questions (quick lookup, debugging, short conversation) skip the step entirely.

For explicit lookups there is the verb `/wiki-context <topic>` (defined in `{{WIKI_VAULT}}/.claude/commands/wiki-context.md`).

## Hard rules

- **Never write to `{{PERSONAL_VAULT}}/`** unless {{OWNER}} explicitly asks. Reading is always allowed - it is the main source for cross-references.
- **Exception, personal vault:** the `transcript-triage` job (a separate productivity automation, not a wiki verb) files confirmed meeting/networking/personal/brainstorm analyses into `{{PERSONAL_VAULT}}/Meetings`, `/Meetings/1-1`, `/Personal`, `/Ideas`. The "never write to the personal vault" rule above governs the wiki agent, not that job.
- **Never delete** in any vault. Move instead of delete.
- **When writing to the wiki** work strictly by `{{WIKI_VAULT}}/CLAUDE.md` - it holds the non-negotiable constraints (page conventions, frontmatter, cross-reference rules).
- **No em-dashes.** ` - ` (hyphen with spaces).
- **Calibrate voice.** Before writing, read 2-3 recent files in `{{PERSONAL_VAULT}}/Knowledge/` to match the style. Sentence-case headers, no marketing language.

## Languages

Match the language of the current session and of the material you are synthesizing.

## Cron jobs

Two scheduled tasks maintain the wiki automatically:

- `wiki-inbox-process` daily - triages `{{WIKI_VAULT}}/inbox/`, classifies into discard/link/promote/file, updates Index.md.
- `wiki-weekly-lint` weekly - health check, writes an audit report to `{{WIKI_VAULT}}/_audit/`.

You normally do not trigger these manually. If {{OWNER}} says "process the inbox now", call the `/process-inbox` verb directly (spec in `{{WIKI_VAULT}}/.claude/commands/process-inbox.md`).
