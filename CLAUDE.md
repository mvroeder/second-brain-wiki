# CLAUDE.md - second-brain-wiki (maintainer notes)

This is the **generic, public** version of the second-brain-wiki plugin. The personal fork
with Notion/Dex integration and concrete identity lives in a separate repo,
`second-brain-wiki-mvr` - do not conflate the two.

## What this repo is

A Cowork/Claude Code plugin: an AI-maintained Obsidian "LLM wiki". Skills (verbs) under
`skills/`, scheduled automation prompts under `scheduled-prompts/`, vault templates under
`templates/`, manifest in `.claude-plugin/plugin.json`. There is no build step and no test
suite; correctness is checked with grep gates (see the design trail below).

## Repeatable tasks

- **Regenerate the guide PDF** (deps already present: pandoc + Google Chrome):
  1. `pandoc GETTING-STARTED.md -f gfm -t html5 -s --include-in-header=<css-header>.html --variable title-meta="Getting started with second-brain-wiki" -o guide.html`
  2. `"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new --disable-gpu --no-pdf-header-footer --print-to-pdf=GETTING-STARTED.pdf file://<abs-path>/guide.html`

  The A4 print CSS is a small `<style>` block passed as the header include; recreate it if lost.
- **Package / install the plugin:** zip the repo (Anthropic docs call it a `.zip`; some
  community guides rename it `.plugin`) and install via Cowork -> Customize -> Plugins ->
  Upload, or publish a marketplace and use Browse.

## Conventions

- No em-dashes anywhere; use ` - ` (hyphen with spaces). Sentence-case headings. Repo docs in
  English.
- Config tokens only: `{{OWNER}}`, `{{OWNER_EMAIL}}`, `{{WIKI_VAULT}}`, `{{PERSONAL_VAULT}}`,
  `{{PERSONAL_VAULT_ENC}}`, `{{VAULT_BASE}}`, `{{STATE_DIR}}`. No hardcoded `~/Obsidian` or
  `~/Cowork` in operational files (the only allowed literal is the documented default value in
  `config.md`).
- State lives in the wiki vault: ledgers + routing-rules in `{{STATE_DIR}}` (`.wiki-state/`),
  and `reading-list.md` at the vault root is human-append-only (the agent never writes it).

## Where the context lives

- Design trail: `docs/superpowers/` holds the spec, the implementation plan, and the
  verification notes (Cowork-on-Windows folder access, packaging, Obsidian Mobile sync). Read
  those before changing behavior.
- Roadmap: `ROADMAP.md` (capture-source adapters: Google Tasks, Microsoft To Do, Readwise;
  the same pluggable pattern later for the mail and voice seams).
- Machine-local project memory (not committed here): the Claude Code project-memory folder for
  this repo path holds a dated state note plus a MEMORY.md index. Check it at session start.
