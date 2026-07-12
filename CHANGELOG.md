# Changelog

All notable changes to this project are documented here. The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.5.0] — 2026-07-11

Onboarding grows up: setup now builds a personalized context profile and shows you the plan before it builds anything — and the rulebook gets an explicit branching policy.

### Added

- **Context profile (five files).** `00 Context/` now ships `About me`, `Audience`, `Offer`, `Writing style`, and `Branding` — the reference set the agent reads before any content or writing task. Setup Phase 2 asks the matching interview questions (one at a time, thin answers welcome) and Phase 3a fills the files with the user's real answers as prose — never invented facts.
- **Structure preview before scaffolding.** Phase 3a now renders the planned vault as a tree with the user's real project/area/resource names and waits for an explicit OK before creating anything.
- **Personalized starter notes.** Instead of only generic examples, setup creates a project file (from the `99 Templates` project template), an area folder + hub note, and a resource folder + hub note for every real item from the interview.
- **`01 Inbox/Brain Dump.md`** — a permanent capture note for quick thoughts that don't warrant their own file; triage empties the list, the note stays.
- **Branching policy** in `AGENTS.md` § Git sync: the vault always works directly on `main` (no session or feature branches — one single truth for hooks, capture channels, and every agent); rollback via commits and tags; on a rejected push `git pull --rebase origin main`, never a branch.

### Changed

- **Migration path (Phase 3b) offers the context profile additively.** Existing vaults are asked which of the five context files they want; existing equivalent notes are registered (or extended only with explicit OK) — nothing is overwritten, and skipping everything is a first-class choice.
- Translation step covers the five context files and Brain Dump.

## [0.4.1] — 2026-07-11

### Fixed

- The Codex SessionStart hook example in `guides/per-agent-tips.md` used `systemMessage`, which Codex only shows to the **user** in the UI. The example now emits `hookSpecificOutput.additionalContext`, the field that actually reaches the **model's context** — the equivalent of plain stdout in Claude Code. Verified against a live Codex setup.

## [0.4.0] — 2026-07-11

Codex parity, field-tested: the source vault wired Codex up as a first-class agent and verified every claim against a live setup — the generalizable pieces now ship in the template.

### Added

- **Native Codex skill discovery via `.agents/skills/`.** Codex scans `.agents/skills/` (vault root and `~/.agents/skills/`), not the vault's `skills/` folder — so setup Phase 4 now mirrors `skills/` into `.agents/skills/`, `AGENTS.md`'s skill reflex documents the mirror, and the maintenance routine refreshes it after skill updates. Claude Code and Codex both discover skills natively now; Gemini CLI and OpenCode still check manually.
- **Codex session-routine hooks** (per-agent tips): Codex's `hooks.json` uses the same JSON shape as Claude Code's hooks, and the Stop git-sync guard works unchanged (exit 2 + stderr blocks). Two Codex-specific rules are documented with a ready-to-copy example: hook stdout must be **JSON** (a `systemMessage` wrapper for the SessionStart pull-and-inbox script), and project-local hooks need a one-time `/hooks` trust approval.
- **Agent compatibility matrix** at the top of `guides/per-agent-tips.md` — rulebook entry, skill discovery, hooks, MCP config, plan mode, and model switch for all four agents in one view, making the portable/vendor-specific split explicit.

### Changed

- **SKILL.md frontmatter must start on line 1** — documented in `AGENTS.md`; a heading above the frontmatter makes Codex silently reject the skill (found the hard way in the source vault).

## [0.3.0] — 2026-07-11

The project has a new name: **AI SecondBrain Stack** (formerly AI SecondBrain OS), and a sharpened agent lineup: Claude Code, Codex CLI, Gemini CLI, and OpenCode.

### Changed

- **Renamed to AI SecondBrain Stack.** Repo moved to `natscho-hh/ai-secondbrain-stack` (GitHub redirects the old repo URL; git remotes keep working). The landing page moved to <https://natscho-hh.github.io/ai-secondbrain-stack/> — the old GitHub Pages URL is **not** redirected, update your bookmarks. The Bootstrap Prompt now clones the new URL. The `asbos-template-version` marker key is intentionally **unchanged** — it's a stable identifier, and renaming it would break the maintenance routine's version check in every existing vault.
- **Supported agents are now Claude Code, Codex CLI, Gemini CLI, and OpenCode.** Cursor and GitHub Copilot are no longer documented targets; their sections and config snippets were removed from the README, landing page, rulebook, per-agent tips, model strategy, MCP catalog, and skills README. (Vaults keep working with any agent that reads `AGENTS.md` — they're just no longer maintained/tested here.)
- **New social card** (`assets/social-card.png`), wired into the landing page via Open Graph tags.

### Added

- **OpenCode support**: it reads `AGENTS.md` natively (project root and `~/.config/opencode/AGENTS.md`), so no adapter file is needed. New landing-page walkthrough, per-agent tips section (install, Tab plan-mode toggle, `/models`, `opencode.json`), model-strategy row, and MCP config snippets (top-level `mcp` key, `command` as array).

### Fixed

- `SETUP.md` Phase 3b told migrating agents to keep a hardcoded `0.1.0` version marker — it now references the current marker from the repo root's `AGENTS.md`.
- `guides/per-agent-tips.md` claimed no other agent has a hook system — Codex meanwhile ships hooks (`hooks.json`, same JSON shape as Claude Code, with a JSON-on-stdout convention and a one-time `/hooks` trust step for project-local hooks); the guide now says so.

## [0.2.0] — 2026-07-05

Field-tested rule upgrades. The source vault this template was derived from went through a full multi-agent audit; every fix that generalizes now flows back into the template. Existing vaults pick these up via the selective-adoption step in `MAINTENANCE.md` — nothing is forced.

### Added

- **`99 Templates/` + Obsidian templates config.** Three note templates (daily note, project, inbox capture) wired into the Templates core plugin via `.obsidian/templates.json`. The audit found that hand-written frontmatter is the root cause of tag and status drift — templates heal it at the source. (v0.1.0 shipped the Templates plugin enabled but no templates folder.)
- **`todos.md` — a root-level priority board** across all projects, plus the "one home per piece of information" rule: project file = current state + next step, `todos.md` = cross-project priorities, daily note = day log. State drift across multiple places was the audit's biggest systemic finding.
- **Tag taxonomy rule**: lowercase kebab-case, first tag names the note type, no synonym tags.
- **`waiting` status** for work blocked on someone/something external (name the trigger in the note), plus clear exemptions: daily notes and `00 Context` notes carry no status; in folder-projects only the hub file does.
- **External link-note rule**: every external repo or file the vault references gets its own link note (path, URL, branch, backlink) so knowledge and code stay connected.
- **Session-routine hooks pattern** (Claude Code): a SessionStart hook that pulls and reports the inbox, and a Stop hook that refuses to end the session with uncommitted changes — automation beats memory. Documented with ready-to-copy scripts in `guides/per-agent-tips.md`.

### Changed

- **Session end is now a mandatory 4-step routine** (was: a loose "offer to…"): update touched project files first, sync `todos.md`, write the daily note (lessons distilled into resource notes), then offer inbox cleanup flagging pending notes older than 14 days.
- **Resources rule relaxed**: folders per topic, but flat single-reference notes directly in `04 Resources/` are fine. The old "always folders" rule was the one rule the source vault consistently broke in practice — the practice won.
- **Inbox triage** now retrofits missing frontmatter on captured notes; **vault-health** additionally validates `status` values against the allowed set.
- `SETUP.md` scaffold and translation steps cover the new folder, templates, and `todos.md`.

## [0.1.0] — 2026-07-04

First public release. AI SecondBrain OS turns an Obsidian vault into an AI-agnostic second brain that any coding agent can drive from a single rulebook.

### Added

- **`AGENTS.md` — one rulebook, every agent.** A single source of truth for vault structure, session routines, git sync, the skill reflex, and model strategy. `CLAUDE.md` and `GEMINI.md` are thin adapters that point to it; Codex, Cursor, and Copilot read `AGENTS.md` natively — so you can switch agents mid-project without losing the rules.
- **`SETUP.md` — the agent is the installer.** Paste one bootstrap prompt into your AI coding agent and it runs a 7-phase setup: environment check (with a dedicated-folder safety warning), dependency check, an open-ended interview, scaffold, skill install, optional MCP config, and a verify + onboarding pass.
- **Migration path for existing vaults.** Already have an Obsidian vault? Setup detects it and migrates non-destructively — commit-first checkpoint, a mapping table you approve row by row, and your notes, wikilinks, and skills left intact.
- **`template/` — a PARA vault scaffold.** Eight folders (`00 Context` … `07 Attachments`) with example starter notes, plus a starter `.obsidian/` config: sensible editor defaults, per-folder graph colors, and a theme selection so the vault looks considered from first launch.
- **Core skills** (`skills/`) in an agent-neutral `SKILL.md` format: `inbox-triage`, `daily-note`, `vault-health`.
- **Curated catalogs** (`manifest/`): a skill catalog with discovery guidance and a security gate, and an MCP catalog with per-agent config snippets for all five agents.
- **User guides** (`guides/`): first steps, token-friendly working, security basics, model strategy (plan with a strong model, build with a cheaper one), and per-agent tips.
- **Maintenance built in** (`MAINTENANCE.md`): a monthly, agent-run routine that keeps skills, plugins, MCP servers, and the template itself up to date — set up once during install, never a chore afterward.
- **Landing page** at [natscho-hh.github.io/ai-secondbrain-stack](https://natscho-hh.github.io/ai-secondbrain-stack/) (URL updated after the v0.3.0 rename): pick your agent, copy the bootstrap prompt, go.
- **Non-English vaults**: choose your language and setup translates the folder names, the rulebook, the Obsidian settings, and the example notes consistently — skills and guides stay English as reference material.

### Notes

- Verified end-to-end with **Claude Code** (full setup, onboarding, and a byte-exact non-destructive migration test). **Codex** correctly reads and reasons over every document; on a locked-down machine its default sandbox blocks first-run writes until you approve file/network access. **Gemini CLI** is currently blocked at Google's own account tier for individual users — unrelated to this project.

[0.5.0]: https://github.com/natscho-hh/ai-secondbrain-stack/releases/tag/v0.5.0
[0.4.1]: https://github.com/natscho-hh/ai-secondbrain-stack/releases/tag/v0.4.1
[0.4.0]: https://github.com/natscho-hh/ai-secondbrain-stack/releases/tag/v0.4.0
[0.3.0]: https://github.com/natscho-hh/ai-secondbrain-stack/releases/tag/v0.3.0
[0.2.0]: https://github.com/natscho-hh/ai-secondbrain-stack/releases/tag/v0.2.0
[0.1.0]: https://github.com/natscho-hh/ai-secondbrain-stack/releases/tag/v0.1.0
