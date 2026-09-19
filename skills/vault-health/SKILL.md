---
name: vault-health
description: "Check vault consistency: broken wikilinks, missing frontmatter, summaries, project codes, todos board limits, misplaced files."
---

## When to use

Use this skill during the monthly maintenance check, whenever the user asks for a vault health check, or any time something feels off about the vault's structure and it's worth confirming nothing is broken.

**This is the fast structural check, not the content inventory.** It answers "is anything broken" in a few minutes. It does not look for orphaned notes, contradicting decisions, or rule files that outgrew their usefulness — that is `vault-inventory`, which runs monthly and takes a session. A vault can pass every check below and still hold two different answers to the same question. Never report one as if it had covered the other.

## Steps

1. Grep the vault for all `[[wikilink]]` targets, and for each one check whether a matching file actually exists. Collect any wikilink whose target has no corresponding note. **Read the exclusion list below before running this step** — without it, this check reports mostly noise. Two parsing traps make it lie outright: an alias inside a markdown table is written `[[Target\|Alias]]`, so split on `\|` before `|` and strip stray backslashes; and a byte-order mark on line one defeats a `^---` frontmatter test, so strip `﻿` before parsing. Both produce confident false positives.
2. List every note that is missing a `tags:` field in its YAML frontmatter, every note whose `status` value is outside the allowed set (`active` / `completed` / `paused` / `waiting`, or their translated equivalents from `AGENTS.md`), every note that is missing a `status` field outright (except daily notes and `00 Context` notes, which `AGENTS.md` exempts from `status`), and every note that is missing a `date` field. Also flag every note inside a project or area folder that lacks its folder's tag — read that tag from the register in `AGENTS.md`, never derive it from the folder name. **Tag order is never a finding**; the rulebook allows any order deliberately. Note that `tags:` has two legal YAML spellings — inline `tags: [a, b]` and a block list on the following lines. A check that only understands the inline form reports "tags missing" on perfectly correct files.
3. Check `summary:` in every active knowledge note under `00 Context`, `02 Projects`, `03 Areas`, and `04 Resources`: present, at most 160 characters, and describing purpose rather than state. Length is a hard finding; the purpose-versus-state judgement is a warning, because it is a reading, not a measurement. Words that reliably signal a summary has drifted into state: *waiting, done, finished, superseded, currently, today, still, pending, upcoming*, plus any version number or date. Match on **word boundaries** — "finishing a file" contains "finish" but describes the end of a process, not a status.
4. **Project codes, as warnings.** A `code:` that is not two to five capital letters or digits; a `code:` that does not appear in the register in `AGENTS.md`; a hub note without `code:`. A hub means exactly one level: a flat note directly under `02 Projects`, a file named like its own folder, or a `README.md`. An engagement folder inside an area inherits its area's code and is not checked. These stay warnings on purpose — notes written before the vault adopted codes are not wrong, just not yet caught up.
5. **`todos.md`, as warnings.** Finished entries still sitting in a working section instead of "Recently done"; finished entries older than 14 days; entries longer than five lines. **Compute the 14 days against midnight, not against the current time** — otherwise an entry exactly 14 days old counts as overdue in the afternoon and not in the morning, and the report changes depending on when it runs. Why this file gets its own check: the rule that applies while writing — five lines — holds by itself. The two that demand an action *later* — move it when done, clear it after two weeks — did not hold for three weeks in the vault this template comes from, because nothing made the violation visible. Unlike every other finding, over-age entries may be cleared right away; `AGENTS.md` makes that the agent's job.
6. List every file sitting directly in the vault root (or any other place outside its expected PARA folder) that should instead live under `00 Context`, `01 Inbox`, `02 Projects`, `03 Areas`, `04 Resources`, `05 Daily Notes`, `06 Archive`, `07 Attachments`, or `99 Templates` — the rule files, `todos.md`, `Projects.base`, `.gitignore`, and `.maintenance-log.md` legitimately live at the root. Also flag any file sitting directly in the root of `07 Attachments` rather than in an owner folder.
7. Report all findings together as a single table: issue type, file, and what's wrong. Keep hard findings and warnings visually apart.
8. Do not change anything yet — present the report and ask the user which issues to fix.
9. Only after the user confirms, apply the agreed fixes (repair or remove broken links, add missing frontmatter, move misplaced files), then commit and push.

## What is deliberately not a finding

This section is the difference between a check people read and a check people learn to ignore. In the vault this template comes from, a naive broken-link scan returned **27 hits, and not one of them was real.** Every single one was documentation quoting or illustrating a link rather than making one. A check with that hit rate gets skipped after two sessions, and then it protects nothing.

Strip all of the following **before** looking for wikilinks:

- **Fenced code blocks and inline backticks.** A rule file that explains what a broken link looks like will contain one. That is a quotation, not a link. This exclusion alone accounted for all 27 false hits above.
- **Placeholder targets containing angle brackets**, e.g. `[[Reports/<YYYY-MM-DD>-summary]]`. That is a naming pattern, not a destination.
- **The rule files themselves** (`AGENTS.md` and its adapters) for the link check. They describe the rules rather than follow them, so their examples are supposed to look wrong.

Two things that look like exclusions but are not:

- **Do not strip block quotes.** It is tempting, because rule text often gets quoted there. But callouts — `> [!warning] Superseded by [[Other note]]` — are ordinary notes written in quote syntax, and they carry real links. Excluding them means never checking a whole class of banner links, exactly the ones that break when files get renamed. Stripping code and backticks is enough.
- **A dead link inside a daily note is not a finding.** Daily notes are a log of a moment; they are allowed to point at things that no longer exist.

Finally: **the set of files you check and the set of files a link may point to are two different sets.** Excluding a folder from the scan does not mean links into it are broken. Build the resolution index from everything, then check only what you meant to check.

## Rules

- Never modify, move, or delete a file based on this check without the user's explicit confirmation. The single exception is clearing finished `todos.md` entries past 14 days, which `AGENTS.md` assigns to the agent.
- Report every finding, even ones that seem minor — let the user decide what matters.
- Never batch unrelated fixes into a single unexplained commit; describe what was changed and why.
- **If the report is much longer than you expected, suspect the check before you suspect the vault.** Verify a sample of findings by hand first. Acting on a noisy report means changing correct files.
