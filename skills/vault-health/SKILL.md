---
name: vault-health
description: "Check vault consistency: broken wikilinks, missing frontmatter, misplaced files."
---

## When to use

Use this skill during the monthly maintenance check, whenever the user asks for a vault health check, or any time something feels off about the vault's structure and it's worth confirming nothing is broken.

## Steps

1. Grep the vault for all `[[wikilink]]` targets, and for each one check whether a matching file actually exists. Collect any wikilink whose target has no corresponding note.
2. List every note that is missing a `tags:` field in its YAML frontmatter.
3. List every file sitting directly in the vault root (or any other place outside its expected PARA folder) that should instead live under `00 Context`, `01 Inbox`, `02 Projects`, `03 Areas`, `04 Resources`, `05 Daily Notes`, `06 Archive`, or `07 Attachments`.
4. Report all three findings together as a single table: issue type, file, and what's wrong.
5. Do not change anything yet — present the report and ask the user which issues to fix.
6. Only after the user confirms, apply the agreed fixes (repair or remove broken links, add missing frontmatter, move misplaced files), then commit and push.

## Rules

- Never modify, move, or delete a file based on this check without the user's explicit confirmation.
- Report every finding, even ones that seem minor — let the user decide what matters.
- Never batch unrelated fixes into a single unexplained commit; describe what was changed and why.
