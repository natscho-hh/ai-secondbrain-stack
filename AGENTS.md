<!-- asbos-template-version: 1.3.0 -->
# AGENTS.md — Vault Rulebook

This file is the single source of truth for how any AI agent works in this vault. `CLAUDE.md` and `GEMINI.md` (and any other agent-specific file) only point here — the rules themselves live in exactly one place.

## Vault structure

The vault follows a PARA-style layout: nine top-level folders, each with one clear job. Every agent and every skill assumes these exact names and this order.

| Folder | Purpose |
|---|---|
| `00 Context` | The context profile — five files (`About me`, `Audience`, `Offer`, `Writing style`, `Branding`) the agent reads before doing content or writing tasks: who the user is, who they serve, what they offer, how they write, how their brand looks. |
| `01 Inbox` | Unprocessed capture: quick thoughts, brain dumps, anything without a home yet. |
| `02 Projects` | Active work with a concrete goal and an end date. New projects start as a single file directly in this folder. |
| `03 Areas` | Ongoing responsibilities with no end date — the standing parts of life and work that keep running. |
| `04 Resources` | Knowledge base: reference material, documentation, anything worth keeping for later lookup. |
| `05 Daily Notes` | Daily log, one file per day, named `YYYY-MM-DD.md`. Gives continuity between sessions. |
| `06 Archive` | Completed projects and inactive areas, moved here only on explicit request. |
| `07 Attachments` | Images, PDFs, and other media referenced from notes. |
| `99 Templates` | Obsidian note templates (daily note, project, inbox capture) — use them when creating new notes so frontmatter stays consistent. |

A handful of files live at the vault root alongside these folders: `todos.md`, the central priority board across all projects (updated in every session that touches project work); `.maintenance-log.md`, the record of when the two upkeep routines last ran; and the agent rule files (`AGENTS.md` and its adapters).

## Vault rules

- Use `[[wikilinks]]` to connect notes to each other.
- **A hub links its own sub-files.** The hub file of a project, area, or resource folder lists every sub-file at least once as a wikilink, with half a sentence saying what it is for. Creating a sub-file means adding it to the hub in the same move. A sub-file missing from its hub is a finding, not a matter of taste. The reverse does not hold: a report or a finished single note is allowed to be a dead end with no outgoing link — never invent links to satisfy a metric. `01 Inbox/`, `05 Daily Notes/`, and `06 Archive/` are exempt.
- **`summary:` describes purpose, never state.** Every active knowledge note carries a `summary:` line in its frontmatter, at most 160 characters, saying what the note is *for* and what it covers. No version numbers, no test counts, no current status, no "waiting on". That one restriction is the whole trick: a sentence about purpose does not go stale when the state changes, which is exactly how priority boards and setup maps rot. Skip it in `01 Inbox/`, `05 Daily Notes/`, `06 Archive/`, `99 Templates/`, and the rule files themselves. It is what the first stage of the search order below reads, so a vault without it falls back to full-text search on every question.
- **A knowledge note that passes roughly 50 KB gets split by subject.** Move whole sections into their own notes and link them from the hub — do not summarize. Summarizing creates a second home for the same information and breaks the one-home rule above. Size alone is not the problem; a note nobody can load in one piece is.
- Keep notes atomic: one idea per note. The one exception is daily notes, which are a running log.
- Every note's YAML frontmatter includes `tags`, `status`, `date`, and `summary`. Allowed `status` values: `active` / `completed` / `paused` / `waiting` (`waiting` = blocked on someone or something external — name the trigger in the note). Daily notes and `00 Context` notes don't need a `status` field. When a project grows into a folder, only its hub/README file carries the project status; sub-notes omit it or inherit it.
- Tags are lowercase kebab-case. Every note carries **two mandatory tags**, then as many topic tags as it needs:
  1. a **type tag** — `project` / `area` / `resource` / `daily` / `inbox` / `archive` / `context`
  2. the **project or area tag** of its folder, but only when the note lives inside a project or area folder. Flat single-reference notes directly in `04 Resources/` don't need one.

  **The order is free.** Obsidian searches tags as a set, not by position, so `#my-project` finds the note wherever that tag sits. Order is never a finding. A rule that demands a fixed position costs enforcement and buys nothing — in the vault this template comes from, that rule held in 78% of notes and its violations had never caused a single problem.

  **Keep a register of project and area tags** in the table below, and read the tag from it instead of deriving it from the folder name. Derivation looks obvious and quietly produces two tags for one thing the first time a folder gets renamed, or the first time a topic appears as both a project and a resource folder.

  | Folder | Tag |
  |---|---|
  | *(add one row per project or area folder as you create it)* | |

  Before inventing a new tag, check whether an existing one fits — no synonyms. Where a word is ambiguous across the vault, pick the more specific tag and record here which spelling is canonical.
- One home per piece of information: the project file holds current state + next step, `todos.md` holds cross-project priorities, the daily note holds the day's log. Never maintain the same open-items list in two places; if they disagree, the project file wins.
- **Keep `todos.md` entries short — five lines at most.** Each entry names the thing, why it matters now, and the next step, then links to the project file. Test numbers, commit hashes, branch names, and state blocks belong in that project file, not on the board. Drop finished entries after two weeks; their substance already lives in the project file and the daily note. An entry that outgrows five lines is not a formatting problem — it is the signal that content is sitting in the wrong place. Left alone, this one file grows into the single largest context cost in every session that touches project work.
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

## Search order

Applies to every question about what the vault already holds. It does not apply to writing daily notes or inbox captures, and it does not apply to the "where was I?" briefing below, which deliberately reads several files.

1. Search titles, tags, and `summary:` first — not the full text. Judge the candidates without opening any file.
2. Only when that returns nothing, or nothing that plausibly fits, search the full text.
3. Open exactly **one** file, the best one, and read only the relevant section. At most one counter-candidate when two notes are genuinely in the running.
4. Then answer, and name the file the answer came from.

Follow a reference at most once. **If what you read contradicts itself or plainly does not carry the answer, keep reading — and say so, because that is a finding.** If the ladder produces no hit at all, that is also a finding, not an invitation to read everything after all.

**Having read one file is not evidence.** This rule saves searching; it does not replace checking whether the passage actually answers the question.

Why the ladder starts at metadata rather than at a generated index: in the vault this template comes from, a full-text scan across every note took 53–118 ms, while a complete index of the same vault weighed 48,509 characters and had to be loaded into context on every session. Finding things was never the expensive part. Measure before you build an index — the answer flips once a vault grows past a few hundred notes, but it is worth knowing which side you are on.

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

**Automating the routines:** prose routines rely on the agent remembering them — automation beats memory. Claude Code and Codex can both enforce these routines with hooks (a SessionStart hook that pulls and reports the inbox, and a Stop hook that refuses to end the session while uncommitted changes exist); see their sections in `guides/per-agent-tips.md` for ready-to-copy patterns. Gemini CLI and OpenCode run the routines conversationally.

## Git sync (mandatory)

After EVERY change to the vault, commit and push automatically:

```bash
git add -A && git commit -m "short description" && git push
```

Do not batch multiple unrelated changes into a single silent commit at the end of a session — sync as you go, so the vault stays recoverable at every step.

**Branching policy.** It depends on one thing: whether you are there while the agent runs.

- **Agents you are sitting in front of work directly on `main`.** No session branches, no feature branches. A knowledge base has no "broken intermediate state" that a branch would protect, and every routine in this rulebook (session-start pull, capture channels, any agent reading the vault) relies on one single truth. Rollback is covered by the commit history and by tags — set a rollback tag before risky bulk operations, e.g. `pre-cleanup-2026-07-11`. If a push is rejected (non-fast-forward, usually a parallel session or a web edit): `git pull --rebase origin main`, resolve the conflict (for binary files the newer version wins — check timestamps), push again. Never fall back to a branch.
- **Agents that run without you always work on a branch and never push to `main`.** That means cloud and background runs, scheduled jobs, anything whose output you read afterwards rather than while it happens. Branch name `agent/<topic>`. You review the branch locally and merge from there. The reason is not code safety, it is review: an unattended run writes without anyone reading along, and its context ends when the run ends. The branch is the only place where that reading can still happen.
- **A rollback tag set during an unattended run is not a safety net.** Tags don't push automatically, so a tag created in a throwaway sandbox dies with it. An unattended agent relies on the predecessor commit in its branch and does not claim a net it never hung. If you genuinely want the tag, push it explicitly with `git push origin <tag>`.

Branches otherwise belong in code repos, and there per topic or feature — never per session.

## Skill reflex (mandatory)

Before ANY task — not just complex ones — check whether a skill in this vault's `skills/` folder already covers it. Each skill is its own folder with a `SKILL.md` entry point. If a skill matches the task, read its `SKILL.md` and follow it step by step instead of improvising a fresh approach.

Skills live in `skills/`, the canonical source. Claude Code discovers them from `.claude/skills/` and Codex from `.agents/skills/`, so setup generates both folders as mirrors of `skills/` — refresh both whenever a skill is added or updated. Gemini CLI and OpenCode have no native project-skill directory; they read `skills/` because this rulebook points them there, doing the check manually: list `skills/`, scan for a matching `SKILL.md`, and read it before starting work.

One formatting rule matters for native discovery: a `SKILL.md`'s YAML frontmatter (`name` + `description`) must start on **line 1** — a heading or blank line above it makes Codex reject the whole skill.

## Model strategy

Plan and brainstorm with the strongest model available and in your agent's plan mode; implement with a cheaper model. When a plan is approved, the agent proposes the switch itself and names the exact command (Claude Code: `/model` + plan mode; Codex: `/model`; Gemini CLI: `-m` flag; OpenCode: `Tab` toggles plan/build, `/models` switches model). Details: `guides/model-strategy.md`.

## Maintenance

Upkeep has two mandatory halves, both described in `MAINTENANCE.md`, both tracked in `.maintenance-log.md`. Running one does not cover the other, and neither report may claim it did.

- **Environment check.** On the first session of a new month, compare today's date with `last-check`. If a month or more has passed, offer to run the maintenance routine — it keeps the rulebook, skills, and adapters current as agents and tools evolve.
- **Content inventory.** From roughly the 26th onward, compare today's date with `last-inventory`. If this month has none, offer to run the `vault-inventory` skill — it finds orphaned notes, contradicting decisions, and rules that stopped earning their context cost. The environment check cannot find any of those, because the files it looks at are all perfectly well-formed.

## Agent-specific notes

Every agent reads its rules from the same place, `AGENTS.md`, but gets there through a different file:

| Agent | Entry file | Notes |
|---|---|---|
| Codex CLI | `AGENTS.md` | Reads this file natively — no adapter needed. |
| OpenCode | `AGENTS.md` | Reads this file natively — no adapter needed. |
| Claude Code | `CLAUDE.md` | Thin adapter that points to `AGENTS.md`. |
| Gemini CLI | `GEMINI.md` | Thin adapter that points to `AGENTS.md`. |

MCP server configuration is agent-specific and lives separately in `manifest/mcp.md`, not in this file.
