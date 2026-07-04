# Obsidian settings — starter configuration

These files are a starter Obsidian configuration that ships with the template so a new vault opens with sensible defaults instead of a blank slate. Obsidian treats the `.obsidian/` folder as its config directory, not as vault content, so these files never show up as notes.

## What's here

- **`app.json`** — editor defaults: new notes go into `01 Inbox/`, attachments into `07 Attachments/`, links update automatically on move.
- **`appearance.json`** — selects the **Cyber Glow** community theme (see below to get the full look).
- **`core-plugins.json`** — a sensible set of Obsidian's built-in plugins enabled (graph, backlinks, tags, daily notes, templates, outline, bookmarks, …). Obsidian Sync is left off by default.
- **`graph.json`** — the graph view is pre-colored per top-level folder (`00 Context` … `07 Attachments`), so the graph reads at a glance. These colors work with zero plugins installed.
- **`canvas.json`** — snap-to-grid canvas defaults.

## Getting the full visual look

The graph colors and editor defaults above work out of the box. The full **appearance** relies on two things you install once from Obsidian's own community browser (Settings → Appearance / Community plugins):

- **Theme: "Cyber Glow"** — referenced in `appearance.json`. Until you install it, Obsidian falls back to its default theme with no error; install it to get the intended look.
- **Plugins (optional, for fine-tuning):** **Style Settings** (lets a theme expose color/spacing controls) and any others you prefer. These are not bundled — third-party plugin code and themes are intentionally left out of this template (licensing + size); install the ones you want yourself.

## What's intentionally NOT shipped

- **`workspace.json`** — your open panes and window layout; per-session state, not a setting.
- **`plugins/`** and **`themes/`** — third-party code and CSS. Install what you want from Obsidian directly so you always get the current version.

If you're **migrating an existing vault**, keep your own `.obsidian/` — setup will not overwrite it.
