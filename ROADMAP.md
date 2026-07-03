# Roadmap

Where this plugin is going. It started as a single-user macOS tool and is becoming a
cross-platform, connector-pluggable second brain.

## Now

- Cross-platform (macOS + Windows) via config-driven vault paths.
- State lives inside the wiki vault; no external `~/Cowork/` dependency.
- Automation in three tiers: core (Obsidian only), neutral (Gmail), mac-only (Apple).
- Reading-list capture is abstracted behind a source contract, default is a plain
  vault file (`reading-list.md`) you can edit from Obsidian Mobile.

## Next - capture-source adapters

The reading-list source is pluggable. The Apple Reminders dependency is being
generalized into one adapter among peers:
- Google Tasks (Google/Android ecosystem).
- Microsoft To Do (Windows ecosystem, via Microsoft Graph).
- Apple Reminders stays as an equal peer, not the privileged default.

## Later

- Readwise Reader and other read-later services as capture adapters.
- The same pluggable-source pattern applied to the other two seams: the mail source
  (today Gmail) and the voice source (today Plaud + Monologue).
