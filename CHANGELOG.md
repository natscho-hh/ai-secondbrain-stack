# Changelog

All notable changes to this project are documented here. The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
- **Landing page** at [natscho-hh.github.io/ai-secondbrain-os](https://natscho-hh.github.io/ai-secondbrain-os/): pick your agent, copy the bootstrap prompt, go.
- **Non-English vaults**: choose your language and setup translates the folder names, the rulebook, the Obsidian settings, and the example notes consistently — skills and guides stay English as reference material.

### Notes

- Verified end-to-end with **Claude Code** (full setup, onboarding, and a byte-exact non-destructive migration test). **Codex** correctly reads and reasons over every document; on a locked-down machine its default sandbox blocks first-run writes until you approve file/network access. **Gemini CLI** is currently blocked at Google's own account tier for individual users — unrelated to this project.

[0.1.0]: https://github.com/natscho-hh/ai-secondbrain-os/releases/tag/v0.1.0
