# Verification findings for onboarding guide

Date: 2026-07-03
Purpose: resolve the four open items flagged for the second-brain-wiki onboarding guide (Task 9 input). Each finding below carries a CONFIRMED or UNCERTAIN flag and source URLs. Where a claim could not be verified from primary sources, the honest caveat and fallback recommendation for the guide are stated explicitly.

---

## 1. Cowork on Windows: folder access and path resolution

**Finding:**

- Folder access in Cowork is granted through a Settings/folder-picker UI ("Add Folder" / "Grant Access"), not by typing an arbitrary path into a text field. This is consistent across the marketing/help material found.
- On Windows, folder selection is restricted to the user's home directory (`%USERPROFILE%`, e.g. `C:\Users\<name>\...`). Selecting a folder outside the home directory (a secondary drive such as `D:`, `L:`, or a network drive) produces an explicit error dialog: "The selected folder is outside your home directory. Only folders within your home directory can be used."
- This restriction is Cowork-specific: the separate "Code" tab in the same desktop app allows folder selection on any drive, so the limitation is not a general Windows sandboxing constraint of the app, but scoped to Cowork's folder-instruction feature.
- A symlink/junction workaround (`mklink /J "%USERPROFILE%\AI-Hub" "L:\AI-Hub"`) does not work: Cowork resolves junctions/symlinks to their real target path and still rejects it if the real path is outside the home directory.
- Two GitHub issues (one bug report, one feature request asking for a config-based `allowedDirectories` override similar to the filesystem MCP server) document this as a known, currently-unresolved limitation as of the time of writing. The feature-request issue (#27697) was closed as a duplicate of #24964, indicating Anthropic is aware of the ask but no shipped fix was found in public docs.

**On `~` (tilde) expansion specifically:** no primary-source documentation was found that states one way or the other whether Cowork's folder-picker or any path-entry field expands `~`. Windows does not have a native `~` shell convention the way macOS/Linux do (PowerShell aliases `~` to `$HOME` only inside PowerShell itself, not as an OS-level filesystem convention), and nothing in the fetched docs or issues shows a text-entry field for paths at all — selection appears to be picker-only (native OS folder browse dialog), which sidesteps the question of shell-style tilde expansion entirely.

**Flag: CONFIRMED** (home-directory restriction on Windows, folder-picker UI, no typed-path field, symlink workaround fails) / **UNCERTAIN** (whether `~` has any meaning in Cowork's Windows folder picker — likely moot since selection is picker-based, not typed, but not explicitly documented).

**Honest caveat / fallback for the guide:** Recommend Windows users keep their wiki folder under `%USERPROFILE%` (e.g. `C:\Users\<name>\second-brain-wiki`) rather than a secondary drive, and grant access via the native folder-browse dialog rather than typing a path. Do not tell users to type `~/second-brain-wiki` on Windows — instruct them to browse to the folder instead. Flag in the guide that this is a known limitation that may change; point users to the linked GitHub issues if they hit the "outside your home directory" error.

**Sources:**
- https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork
- https://support.claude.com/en/articles/13364135-use-claude-cowork-safely
- https://github.com/anthropics/claude-code/issues/29583
- https://github.com/anthropics/claude-code/issues/27697

---

## 2. `.plugin` packaging + install (macOS and Windows)

**Finding — install steps (CONFIRMED, primary source):**

From Anthropic's own help center and docs, installing a plugin in Cowork works the same way on macOS and Windows:

1. In Cowork, open the **Cowork** tab, then open **Customize** in the left sidebar.
2. Open the **Plugins** tab / page.
3. Click **Browse plugins** to see plugins from the official marketplace (or other added sources), then click **Install** on the desired plugin. If the plugin needs a connector, you'll be prompted to sign in.
4. To install a plugin you built yourself or received from someone else, use the **upload** option on the Plugins page instead of Browse, and select the plugin package file.
5. After installing, open the plugin to see its skills, connectors, agents, and hooks, and enable/disable individual components.

No OS-specific branching was found anywhere in the primary docs — the same UI flow is documented for macOS and Windows.

**Finding — packaging format (UNCERTAIN / conflicting sources):**

This is the item that could not be cleanly verified. Two different claims exist in the wild:

- **Third-party (non-Anthropic) sources** — a Medium walkthrough and several SEO-style "guide" sites — describe building a plugin directory and zipping it with a `.plugin` extension (literally `zip -r ../weekly-report.plugin .`), then uploading that `.plugin` file via Cowork's "Upload plugin" button.
- **Anthropic's own primary documentation** (code.claude.com/docs, the official Claude Code plugin docs, and the "Manage plugins for your organization" help article) **never uses a `.plugin` extension**. It consistently refers to plugin archives as **`.zip`** files:
  - `claude --plugin-dir ./my-plugin.zip` (test a zipped plugin locally)
  - `--plugin-url https://example.com/my-plugin.zip` (load a hosted zip archive)
  - Org admin upload: "the file must be a valid `.zip` under 50 MB" (max 100 plugins per manual marketplace)
  - Cowork-specific upload docs mention a size ceiling of "Plugin package size (uncompressed): 200 MB" and a 5,000-file limit, but do not state the container file's extension.
  - Anthropic's documented, supported path for sharing a self-built plugin is a marketplace (Git repo with `.claude-plugin/marketplace.json`), a hosted `marketplace.json` URL, or a plain compressed file — not a named `.plugin` format.

Since a `.plugin` extension is functionally just a renamed `.zip` (both are read as zip archives by the tooling described), the practical behavior — "zip the plugin directory, upload it through the Plugins page" — is likely accurate regardless of which extension is used. But no Anthropic-authored source that was found actually mandates or even mentions `.plugin` as the required or expected extension. It appears to be a convention some third-party writers adopted, not a documented requirement.

**Flag: CONFIRMED** (install flow via Customize → Plugins → Browse/Install or Upload, identical on macOS/Windows; `.zip` is the extension Anthropic's own docs use for archived plugins; 50 MB org-upload limit, 200 MB/5,000-file Cowork package limit) / **UNCERTAIN** (whether Cowork's local "Upload plugin" button specifically requires/expects a `.plugin`-named file, or accepts any `.zip`; no primary source confirms the `.plugin` extension claim).

**Honest caveat / fallback for the guide:** Do not assert that the file must be named `.plugin`. Recommend instead: "package your plugin directory as a `.zip` archive (some community guides rename this to `.plugin` — functionally identical, both are zip archives) and use the Upload option on the Plugins page in Customize." Tell users that if the upload dialog rejects a `.zip`, try renaming the extension to `.plugin`, since that convention appears in independent walkthroughs even though it's not in Anthropic's own docs. This is the honest middle ground given the conflicting evidence.

**Sources:**
- https://support.claude.com/en/articles/13837440-use-plugins-in-claude
- https://support.claude.com/en/articles/13837433-manage-plugins-for-your-organization
- https://claude.com/docs/cowork/guide/plugins
- https://code.claude.com/docs/en/plugins
- https://code.claude.com/docs/en/discover-plugins
- https://claude.com/resources/tutorials/how-to-build-a-plugin-from-scratch-in-cowork
- (third-party, cited only to show where the `.plugin` claim originates, not as authority) https://medium.com/@Micheal-Lanham/developing-claude-cowork-plugins-is-easier-than-you-think-28d197e50677

---

## 3. Web Clipper target: append to `reading-list.md` vs. create a note in `inbox/`

**Finding (CONFIRMED, primary source — obsidian.md/help/web-clipper/templates):**

The official Obsidian Web Clipper (obsidian.md/clipper) supports three template **Behavior** modes, configured per-template under "edit template," directly below the Title field:

1. **Create a new note** — one new note per clip.
2. **Add to an existing note, at the top or bottom** — appends (or prepends) the clipped content into a single designated note.
3. **Add to daily note, at the top or bottom** — appends/prepends into the active daily note.

So yes: the Web Clipper *can* append to a specific existing file rather than always creating a new note. Behavior #2 is exactly the "append to `reading-list.md`" use case. The target note for "add to existing note" is configured in the template (not automatically inferred per-clip in a way this research could fully pin down — one forum thread implies the destination can be set as a fixed note reference, and template variables like `{{title}}` are otherwise used for note naming when creating *new* notes, not for the append target). A forum bug report (Obsidian forum, "Web Clipper won't append to an existing note") shows this feature has had rough edges in practice (users reporting inconsistent appends, extra line breaks, missed metadata), so it is not perfectly reliable in the wild even though it's officially supported since Obsidian 1.7.2+.

**Recommendation for the guide:**

Given the append feature is officially supported but has documented reliability complaints (extra line breaks, inconsistent triggering), and given the wiki already has an `inbox/` folder designed to receive one-note-per-item for later triage — **pointing the Web Clipper at "Create a new note" targeting the wiki's `inbox/` folder is the more robust recommendation** than relying on "Add to existing note" against a single `reading-list.md`. This avoids the append-reliability issues entirely and fits the wiki's existing inbox-then-triage workflow (raw capture → `second-brain-wiki:process-inbox` skill does the sorting), rather than asking the Clipper to safely co-edit a shared file that also may be edited by hand.

If the user still prefers a single running `reading-list.md`, the "Add to existing note, at the bottom" behavior is the documented mechanism to configure — but the guide should flag the known append-glitches from the Obsidian forum so users aren't surprised.

**Flag: CONFIRMED** (three behavior modes exist; "add to existing note" is real and officially supported since 1.7.2) / **UNCERTAIN** (exact mechanics of how the target note is resolved when using a fixed filename across different source pages, and how reliable append currently is — forum reports of bugs are recent but not comprehensively reproduced here).

**Sources:**
- https://obsidian.md/clipper
- https://obsidian.md/help/web-clipper/templates
- https://obsidian.md/help/web-clipper/variables
- https://forum.obsidian.md/t/the-obsidian-web-clipper-how-do-add-a-webpage-highlight-to-an-existing-note-instead-of-creating-a-new-one/90562
- https://forum.obsidian.md/t/obsidian-web-clipper-wont-append-to-an-existing-note/110132
- https://forum.obsidian.md/t/web-clipper-add-to-existing-note-behavior-includes-extra-line-break/99232

---

## 4. Roadmap API sanity: Google Tasks, Microsoft Graph (To Do), Readwise Reader

**Google Tasks API** — CONFIRMED live and usable: REST API at `https://tasks.googleapis.com/tasks/v1/`, standard OAuth2 (per-user consent, scoped access), free with standard Google Cloud quotas, official client libraries for Java/Python/PHP/Node. No deprecation notice found.
Source: https://developers.google.com/workspace/tasks/overview

**Microsoft Graph To Do API** — CONFIRMED live and usable: part of Microsoft Graph (`todoTaskList` / `todoTask` resources), supports delegated and application permissions, documented at v1.0 (stable, not beta-only), used for creating/syncing tasks across To Do, Outlook, and Teams.
Source: https://learn.microsoft.com/en-us/graph/todo-concept-overview

**Readwise Reader API** — CONFIRMED live and usable: official public API at `https://readwise.io/reader_api`, save endpoint `POST https://readwise.io/api/v3/save/`, token auth via `Authorization: Token XXX` (token from readwise.io/access_token), rate limit 50 requests/minute/token. Documentation notes the API "currently only supports saving new documents and fetching documents" with more endpoints planned — usable today but intentionally limited in scope.
Source: https://readwise.io/reader_api

**Flag: all three CONFIRMED.** All three APIs exist, are documented by their respective owners, use standard OAuth/token auth, and are usable today. The roadmap does not promise anything impossible on the API-existence front; the Readwise API's narrower scope (save + fetch only, more endpoints "in the near future") is worth noting in the roadmap text so expectations stay calibrated.

---

## Summary table

| # | Item | Flag | Key caveat for the guide |
|---|------|------|---------------------------|
| 1 | Cowork Windows folder access | CONFIRMED (restriction) / UNCERTAIN (`~` expansion) | Home-dir-only on Windows; use folder picker, not typed `~` paths |
| 2 | `.plugin` packaging | CONFIRMED (install flow, `.zip` per Anthropic docs) / UNCERTAIN (`.plugin` extension itself) | Say "zip archive (some guides name it `.plugin`)", don't assert `.plugin` is required |
| 3 | Web Clipper target | CONFIRMED (append mode exists) / UNCERTAIN (reliability, exact targeting mechanics) | Recommend `inbox/` + new-note mode over appending to `reading-list.md` |
| 4 | Roadmap APIs | CONFIRMED (all three) | Readwise API scope is currently save+fetch only |
