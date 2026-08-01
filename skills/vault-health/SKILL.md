---
name: vault-health
description: "Check vault consistency: broken wikilinks, missing frontmatter, misplaced files."
---

## When to use

Use this skill during the monthly maintenance check, whenever the user asks for a vault health check, or any time something feels off about the vault's structure and it's worth confirming nothing is broken.

**This is the fast structural check, not the content inventory.** It answers "is anything broken" in a few minutes. It does not look for orphaned notes, contradicting decisions, or rule files that outgrew their usefulness — that is `vault-inventory`, which runs monthly and takes a session. A vault can pass every check below and still hold two different answers to the same question. Never report one as if it had covered the other.

## Steps

1. Grep the vault for all `[[wikilink]]` targets, and for each one check whether a matching file actually exists. Collect any wikilink whose target has no corresponding note. Two parsing traps make this check lie: an alias inside a markdown table is written `[[Target\|Alias]]`, so split on `\|` before `|` and strip stray backslashes; and a byte-order mark on line one defeats a `^---` frontmatter test, so strip `﻿` before parsing. Both produce confident false positives.
2. List every note that is missing a `tags:` field in its YAML frontmatter, every note whose `status` value is outside the allowed set (`active` / `completed` / `paused` / `waiting`, or their translated equivalents from `AGENTS.md`), every note that is missing a `status` field outright (except daily notes and `00 Context` notes, which `AGENTS.md` exempts from `status`), and every note that is missing a `date` field.
3. List every file sitting directly in the vault root (or any other place outside its expected PARA folder) that should instead live under `00 Context`, `01 Inbox`, `02 Projects`, `03 Areas`, `04 Resources`, `05 Daily Notes`, `06 Archive`, `07 Attachments`, or `99 Templates` — the rule files, `todos.md`, and `.maintenance-log.md` legitimately live at the root.
4. Report all three findings together as a single table: issue type, file, and what's wrong.
5. Do not change anything yet — present the report and ask the user which issues to fix.
6. Only after the user confirms, apply the agreed fixes (repair or remove broken links, add missing frontmatter, move misplaced files), then commit and push.

## Rules

- Never modify, move, or delete a file based on this check without the user's explicit confirmation.
- Report every finding, even ones that seem minor — let the user decide what matters.
- Never batch unrelated fixes into a single unexplained commit; describe what was changed and why.
