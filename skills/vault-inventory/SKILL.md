---
name: vault-inventory
description: "Monthly content inventory: orphaned notes, contradicting decisions, stale wikilinks, bloated rule files. Reports and recommends; never repairs on its own."
---

## When to use

Once a month, near the end of the month, and whenever the user asks for a "vault inventory". This is the deep pass over what the vault *says*.

**Three routines, three different jobs.** Do not merge them, and do not let one report claim it did another's work.

| | `MAINTENANCE.md` | `vault-health` | this skill |
|---|---|---|---|
| Checks | the environment | the structure | the content |
| Finds | outdated skills, plugins, MCP servers, CLIs | broken links, missing frontmatter, misplaced files | orphans, contradictions, rules that no longer pay for themselves |
| Runs | monthly, or on request | anytime, takes minutes | monthly, takes a session |
| Costs | a few minutes | a few minutes | real time and tokens |

`vault-health` answers "is anything broken". This skill answers "is anything still true". A vault can pass every structural check and still hold three different answers to the same question.

## Why it exists

The vault is the single source of truth, but decisions get reversed all the time. When a decision is overturned and the older version stays behind, two files now disagree, and nobody notices until someone reads the wrong one. That failure type is invisible to a structural check, because both files are perfectly well-formed.

It is also the failure type that grows. A stale link is one dead end. A stale *decision* gets read, believed, and built on.

## The one rule that beats all the others

**This session repairs nothing.** It collects, checks, judges, and presents. The user decides what gets implemented afterwards.

An agent that tidies up along the way takes that decision away and leaves the user facing accomplished facts. Two exceptions, both uncontroversial: a rollback tag before the run, and the final report itself — without it the session has no result.

**The rollback tag carries the full date,** `rollback/inventory-YYYY-MM-DD`. The month alone is not enough: a second run in the same month would move the tag of the first and take away its safety net. Tags live in their own namespace, so a day-precise name collides with nothing.

If the user does say "go ahead and fix all of it", see **After implementation** at the bottom. That is a second, separate check, and it is not optional either.

## Scope

**Do not maintain the scope list here.** Take it from wherever the vault already documents its repositories — a repo map note in `00 Context/`, or the link notes that point at each external repo. A second list in this skill is exactly the duplicate bookkeeping this session is supposed to find.

So: the vault itself, plus every repository the vault documents. Nothing else.

Explicitly out of scope in every repo checked: `node_modules`, `.git`, `dist`, build output, test artifacts, and vendored or cloned third-party code. A cloned reference repo is not an orphan just because nothing links to it.

**The scope list itself is a finding.** A local repository that is neither documented nor deliberately excluded belongs in the report. So does a documented repository that no longer exists on disk.

## Phases

Splitting the work by model is deliberate. Collecting and measuring is tool work; judging is not.

### Phase 1 — Inventory

One subagent per repository, so the raw inventory never lands in the main context window. Each returns a table, not prose.

Per markdown file: path, date in frontmatter, last commit, incoming wikilinks, outgoing wikilinks, line count.

**A file is a corpse candidate when nothing links to it *and* it has had no commit for over 60 days.** Both conditions, never one. A fresh note has no incoming links yet, and a file can be linked from everywhere and still be a year stale.

### Phase 2 — Wikilinks

Seven checks, in this order:

1. **Dead targets.** A `[[target]]` with no matching file. **The placeholder filter is mandatory,** not a refinement — without it the run reports dozens of false positives from templates and instructions. Exclude targets inside code blocks, generic names like `Note`, `Note Name`, `Target`, `Path`, `...`, and anything with a file extension. A measured run of this filter: 27 raw hits, 0 real. See the exclusion list in `vault-health` for the full set and for the two exclusions that look right and are not.
2. **Orphaned notes.** Files nothing links to. This is the case where linking genuinely changes something, because an agent finds an unlinked note only by accident. **Count the link source, not just the link.** A note referenced only from a daily note or from the archive is not connected — it was mentioned once, in a log. Counting those sources makes the number look healthy while the note stays unreachable in practice; in the vault this template comes from, the two definitions gave 7 orphans versus 14, and the stricter number was the true one.
3. **Sub-files missing from their hub.** For every project, area, and resource *folder*, check that the hub file links each sub-file at least once, as the rulebook requires. This is stricter than the orphan check and catches what it cannot: a note linked from three sibling notes but absent from the hub is reachable by luck, not by structure. The reverse is never a finding — a report or a finished note may have no outgoing links at all.
4. **Ambiguous names.** The same filename in several folders. Obsidian resolves by name; an agent guesses.
5. **Relative paths used as wikilinks.** `[[../../04 Resources/…]]` does not resolve in Obsidian. Always an error.
6. **Agent-memory slugs used as wikilinks.** Memory entries are not vault notes; they belong in plain text.
7. **Renames with links left behind.** Pull renames from `git log --diff-filter=R --name-status` and check whether anything still points at the old name. This is the most productive of the six, because the dead link here is not the error — it is the *trace* of one.

### Phase 3 — Contradictions

The actual point of the exercise. You are looking for one question with two different answers.

Work in that order: first find the places where a topic is described more than once, then hold the versions against each other. Topics that live in vault, plan, spec, and guideline at once are the richest ground.

Typical finds:

- A rule changed and the second occurrence was never updated
- A number stored in two places, stale in one
- A decision reversed, with the old reasoning still standing
- A prohibition and a requirement about the same thing, in the same file

Per find, record both passages verbatim, which version is newer, and whether it is a contradiction at all or merely an older description of the same state. That last distinction saves the user a pointless edit.

### Phase 4 — Measure the rule files

Measured, not estimated. For the rulebook, its adapters, the hooks, and the skill descriptions:

- Lines and approximate token count
- Which rules appear twice, in the same file or across files
- Which rules demonstrably fired in recent sessions, and which are dead text
- Hook runtime, measured rather than assumed

The question is not "is the file big". It is **"does every line earn back what it costs in context"**.

### Phase 5 — Judgment

Hand over the *findings* from phases 2 to 4, not the raw data. Ask for:

- Which contradiction is real, and which is just an older description
- **Why did it happen**, and which rule would have prevented it. This is the important part — without a cause, everything repeats next month
- What is worth the work, and what is not
- Which rule in the rulebook is not worth its context cost

Use the strongest available model here, and a different one than the one that collected. These questions need judgment, not search.

### Phase 6 — Adversarial read

Hand the judged findings to a *different* model or agent and ask it explicitly to **refute** them, checking each claim against the files. Whatever falls here does not go in the report.

**This phase is not optional.** In the first documented run it overturned or softened roughly half of all claims — measurement errors in the checking script itself, and judgments that had outgrown their evidence. A report without it reliably ships a handful of false alarms, and chasing those costs more time than the read.

### Phase 7 — Report

One file, under `04 Resources/` in a dated inventory folder. Structure:

1. **Numbers** in a table — files, links, dead targets, orphans, contradictions, per repository
2. **Findings by severity,** each with location, assessment, and one concrete proposal
3. **Causes** — the answer to "how did this happen"
4. **Recommendations, numbered,** each with effort and benefit, so the user can tick them off
5. **What is explicitly fine,** so the next run does not re-examine the same places

Name the file `YYYY-MM.md`. **If the inventory runs a second time in the same month,** the report gets a letter: `YYYY-MMb.md`, then `YYYY-MMc.md`. The earlier report is never overwritten. A full date in the file name looks like the obvious alternative and is the wrong one: `2026-08-22.md` collides with the daily note of the same day, and because Obsidian resolves wikilinks by name, every link to either file becomes ambiguous.

Commit it. If a session-start hook nudges for this routine, use whatever commit convention that hook greps for, and record the date in `.maintenance-log.md` as `last-inventory`.

## Pitfalls, learned the hard way

Every one of these comes from a real run.

- **The placeholder filter is not a nicety.** Without it, the real finding drowns in false alarms from templates and instructions.
- **Escaped pipes in tables.** In a markdown table an alias is written `[[Target\|Alias]]`. Split on `|` alone and you keep a trailing backslash on the target and report a dead link that was never dead. Handle `\|` before `|`, and strip stray backslashes.
- **A byte-order mark on line one.** A frontmatter test matching `^---` silently fails on a BOM, and the note shows up as untagged. Strip `﻿` before parsing. One such file in a vault is enough to produce a confident wrong number.
- **Link occurrences and link targets are not the same thing.** Eleven dead targets can be seventeen occurrences. Report both, or the finding looks bigger or smaller than it is.
- **Two conditions for a corpse, never one.**
- **Skill mirrors need their own check, and it is not the orphan check.** Generated mirror folders are not orphans. But do ask two questions about them: have the copies drifted apart, and is anything committed there that does not belong in a repository (compiled bytecode, caches, build leftovers)? Watch for mirrors that are *deliberately* adapted per agent — those must not be "synchronized" back.
- **Whoever copies a file inherits its errors.** Before bringing a drifted mirror back in line, validate the source, not just the difference. In one run, three `SKILL.md` files had carried invalid YAML for two weeks — an unquoted colon inside `description:` — and synchronizing the mirrors neatly doubled the defect.
- **Do not turn a correct raw finding into an overstated verdict.** That a rule was never enforced does not prove the rule is wrong. When two readings are defensible, both belong in the report and the decision belongs to the user.
- **A finding that leads to a deletion recommendation needs the full file list, not the summary.** "Just one leftover script" and "24 files, 18 of them templates" lead to different decisions. And where there is no git history, back the content up before deleting and count the backup against the original.
- **Repair nothing.** Not even the obvious ones. Not even "it's only a typo".
- **Never let the number of guarantees drop silently.** If a proposal removes a rule, the report must say what takes its place.
- **A report without causes is half the work.** The list of corpses takes an hour to generate. The reason they appeared is what saves the following month.

## After implementation

If the user says "implement all of it", the session is not over when the last edit lands.

**Phase 6 checked the report. It did not check the implementation.** Hand the complete diff (`git diff <rollback-tag>..HEAD`) to a strong model with one instruction: find damage, do not confirm success. In the first documented run this second pass found two real defects that had not existed when the report was written:

- **A blanket `git add -A` in a repository that was not the vault.** One README line was intended; twelve generated files came along, because the matching `.gitignore` entries lived only on a feature branch. In a foreign repository, stage the file you changed.
- **A rule moved to the wrong rulebook.** What matters is not who a rule is written for, but **which file the agent that needs it actually loads.** A block addressed to "weaker models" still applies to models that read the main rulebook — moving it to an adapter silently dropped the guarantee for them.

Three smaller patterns show up whenever something is shortened: lost pointers to where the detail now lives, a never-finished sub-item swept into the archive along with a finished block, and a new rule that contradicts an existing one because nobody wrote down the exception.

## Rules

- Never change, move, or delete a file during phases 1 through 6.
- Set a rollback tag before the run and name it in the report.
- Report every finding, including the ones that look minor — the user decides what matters.
- Report what you can prove. Mark anything unverified as unverified rather than rounding it up into a claim.
