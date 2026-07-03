# Getting started with second-brain-wiki

This guide is for someone who has never used Obsidian, has never heard of an "LLM wiki,"
and just wants a working setup. It assumes nothing except that you have a computer
(macOS or Windows) and a Claude account. Follow it top to bottom.

If a step ever produces a `.plugin` or `.zip` file, a folder path, or a `{{TOKEN}}`
placeholder that this guide does not explain, that is a bug in the guide, not something
you are expected to already know. Everything you need is below.

---

## 1. What this is

Imagine a notebook that organizes itself. You throw things into it: articles you read,
links you save, ideas you jot down. Overnight, an AI reads through what you dropped in,
figures out what it actually is, and files it properly: a new page for a new idea, a link
added to an existing page, or a note that it is not worth keeping. Over months this
notebook becomes a structured knowledge base that actually reflects what you have learned
and thought about, without you ever doing the filing yourself.

That is the whole idea, sometimes called the "LLM wiki" pattern. The AI maintains a
Markdown wiki (Markdown is just plain text files with light formatting) that grows over
time. You feed it raw material. It reads, structures, cross-links, and keeps it tidy. You
almost never edit the wiki by hand; you just keep dropping things in and occasionally ask
it questions.

This plugin runs on Cowork (Claude's environment for taking actions on your files and
apps) and uses an app called Obsidian to store the notebook as files on your own
computer. You do not need to know anything about Obsidian beyond what this guide covers.

### The two vaults

Obsidian calls a folder of notes a "vault." This setup uses two of them, sitting side by
side on your computer:

- **The wiki vault** - this is the notebook the AI maintains. The agent writes here. This
  is where your structured knowledge lives: articles you have read, concepts you have
  learned, projects and people you track.
- **The personal vault** - this is yours. Daily notes, journals, whatever you already
  keep. The AI only ever *reads* this vault, to understand context and occasionally
  suggest something you might want to promote into the wiki. It does not write here.

**The one rule that governs everything:** the agent only ever writes inside the wiki
vault. It never edits your personal notes. If it thinks something in your personal vault
is worth turning into a wiki page, it asks you first.

Both vaults live under one folder on your computer, which this guide (and the plugin's
configuration) calls `{{VAULT_BASE}}`. On macOS that is commonly a folder like
`~/Obsidian`. On Windows it should be a folder under your user profile, for example
`C:\Users\<yourname>\Obsidian` - more on why in the next section.

---

## 2. Prerequisites and installation

You need:

- A **Claude Pro or Max account** (Cowork is part of these plans).
- **Cowork**, Claude's desktop environment that can read and write files and run
  automated jobs on a schedule.

Everything else, the vaults, the plugin, the wiki itself, gets set up in the steps below.

### Step-by-step: macOS

1. **Install Cowork.** Download it from claude.ai and sign in with your Claude Pro/Max
   account.
2. **Install Obsidian.** Download the macOS app and install it like any other
   application. You do not need to create an account to use it locally.
3. **Create your two vaults.** Open Obsidian. On the welcome screen choose "Create new
   vault" twice: once for your wiki vault (for example, name it `Wiki`) and once for your
   personal vault (for example, name it `Personal`). Save both inside the same parent
   folder, for example `~/Obsidian/`, so you end up with `~/Obsidian/Wiki` and
   `~/Obsidian/Personal`. That parent folder is your `{{VAULT_BASE}}`.
4. **Install the second-brain-wiki plugin.** In Cowork, go to the **Cowork** tab, then
   **Customize** in the left sidebar, then **Plugins**. Click **Browse plugins** if you
   want to install it from the marketplace, or use the **Upload** option if you have been
   given the plugin as a file directly (a zip archive; some guides name this file
   `.plugin` instead of `.zip`, but the two are functionally the same thing). Follow the
   prompts to finish installing.
5. **Grant Cowork access to your vault folder.** Cowork needs permission to read and
   write files in `{{VAULT_BASE}}`. When a skill first needs that folder, or from
   Cowork's folder-access settings, use the folder picker to browse to and select your
   `{{VAULT_BASE}}` folder (for example `~/Obsidian`). Granting access to the parent
   folder covers both vaults inside it.
6. **Run setup.** In Cowork, tell Claude: **"set up my wiki."** This runs the
   `wiki-setup` skill, which asks for your name, your vault names, and your email, then
   writes the configuration, creates the wiki's internal folders, and installs the
   schema file the AI reads before every action. Answer its questions as they come.
7. **Connect Gmail (optional).** If you want the wiki to scan your newsletters
   automatically, `wiki-setup` will offer to connect Gmail. You can skip this and add it
   later; nothing else depends on it.
8. **Choose your automation tiers.** `wiki-setup` will offer to register scheduled jobs
   in three tiers (explained in section 4 below). Pick what you want; you can change
   this later.

### Step-by-step: Windows

The steps are the same as macOS, with one important difference around folder access.

1. **Install Cowork.** Download it from claude.ai and sign in with your Claude Pro/Max
   account.
2. **Install Obsidian.** Download the Windows app and install it.
3. **Create your two vaults, inside your user folder.** Open Obsidian and choose "Create
   new vault" twice, once for the wiki vault and once for the personal vault. Save both
   under your Windows user profile folder, for example
   `C:\Users\<yourname>\Obsidian\Wiki` and `C:\Users\<yourname>\Obsidian\Personal`. That
   parent folder, `C:\Users\<yourname>\Obsidian`, is your `{{VAULT_BASE}}`.

   **Known limitation, read this before you pick a location:** Cowork on Windows can
   currently only be granted access to folders that live inside your home directory
   (`C:\Users\<yourname>\...`). If you try to grant access to a folder on a different
   drive (a `D:` drive, a network share, and so on) it will be rejected with an error
   saying the folder is outside your home directory. There is no known workaround for
   this. Keep both vaults under your user folder and you will not hit it.
4. **Install the second-brain-wiki plugin.** Same as macOS: **Cowork** tab ->
   **Customize** -> **Plugins**. Use **Browse plugins** for the marketplace, or
   **Upload** for a plugin file you already have (a zip archive, sometimes named
   `.plugin`). This flow is identical on Windows.
5. **Grant Cowork access to your vault folder.** When prompted for folder access, use the
   native folder-browse dialog to click your way to `{{VAULT_BASE}}` (for example
   `C:\Users\<yourname>\Obsidian`) and select it. Do not type a path by hand, and do not
   use a `~` shortcut - Windows does not resolve that as a shortcut the way macOS does,
   and the folder picker expects you to browse to the real location. Browsing to the
   folder and selecting it is the reliable way to do this.
6. **Run setup.** Tell Claude: **"set up my wiki."** The `wiki-setup` skill will ask for
   your details and configure everything.
7. **Connect Gmail (optional).** Same as macOS, skip if you do not want newsletter
   scanning yet.
8. **Choose your automation tiers.** Same as macOS, see section 4.

---

## 3. Capture from your phone (optional)

Everything in this section is optional. If you skip it entirely, you can still use the
wiki fully by working on your desktop: opening `reading-list.md` in Obsidian and adding
links there, or running `ingest-url` directly. This section is only for people who want
to save links while out and about, from a phone, and have them show up in the wiki the
next morning.

### The use case

You are on your phone, reading something, and see a link worth keeping for later. You
open a file called `reading-list.md` (it lives at the root of your wiki vault), add one
line with the URL, and move on. The next morning, the `reading-list-triage` automation
job reads that file, fetches each new link, summarizes it, and proposes what to do with
it. You reply with a quick confirmation, and the article becomes a proper wiki entry:
fetched, summarized, cross-linked, filed. You never had to sit down and process it in the
moment.

To make this work on a phone, your two vaults need to be reachable from your phone too,
which means they need to be synced somewhere both your computer and your phone can see.

### Choosing a sync method

You have two realistic options. Pick one before installing anything on your phone.

**Obsidian Sync (recommended).** This is Obsidian's own paid sync service, roughly
$4-5 a month. It works the same way on macOS, Windows, iOS, and Android, keeps your
vaults synced reliably in the background, and is the option with the fewest surprises.
If you are not sure which to pick, pick this one.

**iCloud (free, Apple-only fallback).** If you are on a Mac and an iPhone and do not want
a subscription, you can sync your vault folder via iCloud Drive instead. This works but
has real rough edges, so if you choose it, do these three things:

- **Turn off "Optimize Mac Storage"** for iCloud Drive (or make sure the vault folder is
  set to "Keep Downloaded") so your notes are not silently offloaded to the cloud and
  replaced with placeholder files on your Mac.
- **Let the Obsidian mobile app create or open the vault folder itself** inside iCloud
  Drive rather than moving an existing folder in there by hand, to avoid sync conflicts
  from the start.
- **Do not edit the same vault offline on two devices at once.** iCloud sync is not
  built to merge conflicting offline edits gracefully. If you know you will be offline,
  edit on only one device until you are back online and synced.

If either of these sounds like more risk than you want, use Obsidian Sync instead.

### Installing Obsidian Mobile

1. **Download the app.** Search "Obsidian" on the App Store (iOS) or Google Play
   (Android) and install it.
2. **Open it for the first time.** You will be asked to open an existing vault or create
   a new one.
3. **Connect to your vault.** This step depends on which sync method you chose:
   - **With Obsidian Sync:** sign in with the same Obsidian account you set up sync with
     on your desktop, and choose your wiki vault from the list of synced vaults offered.
   - **With iCloud:** choose "Open folder as vault," then browse to the vault folder
     inside your iCloud Drive (on iOS this appears under "iCloud Drive" in the file
     picker; on Android you will need an iCloud-for-Windows-style bridge or you should
     use Obsidian Sync instead, since iCloud Drive is not natively available on Android).
4. **Open `reading-list.md`.** Once the vault is open on your phone, use the file browser
   inside the app (usually a folder icon in a corner) to find `reading-list.md` at the
   root of your wiki vault.
5. **Add a line.** Under the `## Reading List` heading, add a new line with the URL you
   want to save, for example:
   ```
   - https://example.com/some-article
   ```
   Save the note (Obsidian saves automatically as you type). That is the entire capture
   action from your phone.

### Optional capture helpers

Two small additions make capture faster, but neither is required:

- **The official Obsidian Web Clipper**, a browser extension for saving web pages
  straight into your vault. If you install it, configure its template to **create a new
  note** and target the wiki vault's `inbox/` folder, rather than appending to
  `reading-list.md`. Creating a fresh note per clip is the more reliable setting and
  fits how the wiki already processes new captures (anything landing in `inbox/` gets
  triaged by the `process-inbox` verb). Appending to a single running file is also
  offered by the Web Clipper, but it has known reliability quirks (occasional extra line
  breaks, inconsistent triggering), so this guide does not recommend it.
- **The phone's built-in share sheet**, or a simple Shortcut on iOS, to send a link
  from another app straight into Obsidian without copy-pasting. This is a convenience
  layer on top of the same "add a line to `reading-list.md`" action above, not something
  new.

Avoid installing heavier community plugins for capture beyond these. They add
maintenance overhead this setup does not need.

---

## 4. Your first real session

Once setup is done, here is a concrete first run to get a feel for the whole loop.

1. **Pick a link you want in your wiki.** Anything: an article, a blog post, a
   documentation page.
2. **Tell Claude:** `ingest-url <the link>`. Watch it fetch the page, write a full note
   about it into the wiki vault's `Sources/` folder, and synthesize the key ideas into
   one or more pages in `Topics/`, cross-linked to anything related that already exists.
3. **Ask a question about it.** Tell Claude: `wiki-context <a word or phrase from the
   article>`. This is a read-only lookup: it returns the most relevant Topics, People,
   Projects, and Sources pages already in your wiki, with short summaries. Nothing gets
   written; this is just how you check what the wiki already knows.
4. **Drop something rougher into the inbox.** Save a half-formed note or a second link
   into the wiki vault's `inbox/` folder by hand (or via any capture method from section
   3), then tell Claude: `process-inbox`. It looks at everything waiting in `inbox/` and,
   for each item, decides: discard it (duplicate), link it (already covered elsewhere,
   just adds a reference), promote it (a genuinely new idea gets its own page), or file
   it (worth keeping, not ripe yet). This is the main way raw material becomes durable
   knowledge.

### What to ignore at first

Two features exist but are not needed on day one:

- **Domains** - an optional tag that lets one wiki hold several separate subject areas.
  Skip this unless you are already tracking clearly distinct topics and want to keep
  them apart.
- **Focus lenses** - your personal "what matters to me" filters, set in `config.md`.
  They only affect how automated jobs rate incoming material. If you leave them blank,
  the wiki simply rates by general relevance, which is a fine default to start with.

### Troubleshooting

- **A scheduled job did nothing.** Check two things: whether its connector (Gmail,
  Obsidian folder access, and so on) is still authorized in Cowork, and whether there
  was actually anything new to process. The dedup ledgers deliberately skip anything
  already seen, so "nothing happened" often just means "nothing new."
- **You see raw placeholders like `{{WIKI_VAULT}}` inside your vault files instead of
  your actual vault name.** This means the setup schema was copied without its values
  being filled in. Tell Claude "set up my wiki" again to re-run `wiki-setup`, which
  rewrites the schema file with your real values.
- **An ingest produced nothing.** The agent only writes into the wiki vault, and only
  for genuine new material; a thin or duplicate item is correctly skipped rather than
  turned into a low-value page. This is expected behavior, not a bug.

---

## 5. Where this is headed

A short excerpt of the current roadmap, so you know what to expect as the plugin
evolves:

- The reading-list capture method described in section 3 is being generalized. Apple
  Reminders support today is just one option; Google Tasks and Microsoft To Do are
  planned as equal alternatives, so the phone-capture flow will work regardless of
  which ecosystem you are in.
- Further out, read-later services such as Readwise Reader are planned as additional
  capture sources, alongside the same pattern being extended to how the wiki pulls in
  email and voice-note material.

None of this changes anything about the setup in this guide; it only adds more ways to
get material into the same `inbox/` -> `process-inbox` -> wiki pipeline you just used.
