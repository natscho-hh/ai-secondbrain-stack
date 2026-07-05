<!-- asbos-template-version: 0.2.0 -->
# AGENTS.md — Vault Rulebook

This file is the single source of truth for how any AI agent works in this vault. `CLAUDE.md` and `GEMINI.md` (and any other agent-specific file) only point here — the rules themselves live in exactly one place.

## Vault structure

The vault follows a PARA-style layout: nine top-level folders, each with one clear job. Every agent and every skill assumes these exact names and this order.

| Folder | Purpose |
|---|---|
| `00 Context` | Personal profile and reference material the agent should read before doing content or writing tasks — who the user is, how they work, how they write. |
| `01 Inbox` | Unprocessed capture: quick thoughts, brain dumps, anything without a home yet. |
| `02 Projects` | Active work with a concrete goal and an end date. New projects start as a single file directly in this folder. |
| `03 Areas` | Ongoing responsibilities with no end date — the standing parts of life and work that keep running. |
| `04 Resources` | Knowledge base: reference material, documentation, anything worth keeping for later lookup. |
| `05 Daily Notes` | Daily log, one file per day, named `YYYY-MM-DD.md`. Gives continuity between sessions. |
| `06 Archive` | Completed projects and inactive areas, moved here only on explicit request. |
| `07 Attachments` | Images, PDFs, and other media referenced from notes. |
| `99 Templates` | Obsidian note templates (daily note, project, inbox capture) — use them when creating new notes so frontmatter stays consistent. |

Two files live at the vault root alongside these folders: `todos.md`, the central priority board across all projects (updated in every session that touches project work), and the agent rule files (`AGENTS.md` and its adapters).

## Vault rules

- Use `[[wikilinks]]` to connect notes to each other.
- Keep notes atomic: one idea per note. The one exception is daily notes, which are a running log.
- Every note's YAML frontmatter includes `tags`, `status`, and `date`. Allowed `status` values: `active` / `completed` / `paused` / `waiting` (`waiting` = blocked on someone or something external — name the trigger in the note). Daily notes and `00 Context` notes don't need a `status` field. When a project grows into a folder, only its hub/README file carries the project status; sub-notes omit it or inherit it.
- Tags are lowercase kebab-case. The first tag names the note type (`project` / `area` / `resource` / `daily` / `inbox` / `archive` / `context`), followed by topic tags. Before inventing a new tag, check whether an existing one fits — no synonyms.
- One home per piece of information: the project file holds current state + next step, `todos.md` holds cross-project priorities, the daily note holds the day's log. Never maintain the same open-items list in two places; if they disagree, the project file wins.
- Every external repo or file the vault refers to gets its own link note (local path, remote URL, branch, backlink to the project) — so knowledge and code stay connected. Never delete these link notes.
- During inbox triage, add any missing frontmatter (`tags`, `status`, `date`) to captured notes before filing them.
- When creating a new note, start from the matching template in `99 Templates/` so frontmatter stays consistent.
- File names use normal spelling and spaces — no forced kebab-case or underscores.
- A new note with no clear place goes into `01 Inbox/`.
- New projects start as a single `.md` file directly under `02 Projects/`; split into a folder only once a project genuinely needs multiple files.
- Areas are always folders. Resources use a folder per topic; flat single-reference notes directly in `04 Resources/` are fine.
- Move completed work into `06 Archive/` only when the user explicitly asks for it — never automatically.
- Always ask before deleting or overwriting a note.
- When the user says "remember this," file the information in the topically correct place — a writing-style note goes to the relevant style guide, project knowledge goes into the project file, general reference goes into Resources, and vault-wide rules go here, in `AGENTS.md`.

## Session routines

**Session start:**
1. Pull the latest changes: `git pull --no-edit origin main`.
2. Show any new commits since the last session.
3. Check `01 Inbox/` for new notes and offer to triage them.

**On request** ("where was I?" or "what's active right now?"): read the last 2–3 daily notes plus the currently active projects, then give the user a short briefing.

**Session end** (mandatory for any session that did real work — the order matters, project files first):
1. Update the state block of every project file the session touched — not just the daily note.
2. Sync `todos.md`: tick off what got done, add what came up, set the date stamp at the top.
3. Create or update today's daily note. Distill lessons learned into the matching resource note; the daily note only links there.
4. Offer to clean up the inbox; call out pending notes older than 14 days and ask whether to do, schedule, or drop them.

**Automating the routines:** prose routines rely on the agent remembering them — automation beats memory. Claude Code can enforce both routines with hooks (a SessionStart hook that pulls and reports the inbox, and a Stop hook that refuses to end the session while uncommitted changes exist); see the Claude Code section of `guides/per-agent-tips.md` for the pattern. Other agents run these routines conversationally.

## Git sync (mandatory)

After EVERY change to the vault, commit and push automatically:

```bash
git add -A && git commit -m "short description" && git push
```

Do not batch multiple unrelated changes into a single silent commit at the end of a session — sync as you go, so the vault stays recoverable at every step.

## Skill reflex (mandatory)

Before ANY task — not just complex ones — check whether a skill in this vault's `skills/` folder already covers it. Each skill is its own folder with a `SKILL.md` entry point. If a skill matches the task, read its `SKILL.md` and follow it step by step instead of improvising a fresh approach.

Claude Code loads skills from this folder natively. Every other agent (Codex, Gemini CLI, Cursor, Copilot) must do this check manually: list `skills/`, scan for a matching `SKILL.md`, and read it before starting work.

## Model strategy

Plan and brainstorm with the strongest model available and in your agent's plan mode; implement with a cheaper model. When a plan is approved, the agent proposes the switch itself and names the exact command (Claude Code: `/model` + plan mode; Codex: `/model`; Gemini CLI: `-m` flag). Details: `guides/model-strategy.md`.

## Maintenance

On the first session of a new month, compare today's date with the date recorded in `.maintenance-log.md`. If a month or more has passed, offer to run the maintenance check described in `MAINTENANCE.md` — it keeps the rulebook, skills, and adapters current as agents and tools evolve.

## Agent-specific notes

Every agent reads its rules from the same place, `AGENTS.md`, but gets there through a different file:

| Agent | Entry file | Notes |
|---|---|---|
| Codex CLI | `AGENTS.md` | Reads this file natively — no adapter needed. |
| Cursor | `AGENTS.md` | Reads this file natively — no adapter needed. |
| GitHub Copilot | `AGENTS.md` | Reads this file natively — no adapter needed. |
| Claude Code | `CLAUDE.md` | Thin adapter that points to `AGENTS.md`. |
| Gemini CLI | `GEMINI.md` | Thin adapter that points to `AGENTS.md`. |

MCP server configuration is agent-specific and lives separately in `manifest/mcp.md`, not in this file.
