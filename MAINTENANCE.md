# Maintenance

This file describes the periodic upkeep for a vault built from AI SecondBrain Stack. `AGENTS.md` points here from its `## Maintenance` section.

## Two routines, and both are mandatory

Upkeep splits into two jobs that are easy to confuse and impossible to substitute for each other.

| | Environment check | Content inventory |
|---|---|---|
| Keeps current | skills, plugins, MCP servers, CLIs, the template itself | what the vault *says* |
| Finds | outdated versions, drifted mirrors, a stale rulebook | orphaned notes, contradicting decisions, rules that no longer earn their context cost |
| Defined in | this file, `## Routine` below | `skills/vault-inventory/SKILL.md` |
| Cadence | first session of a new month | once a month, near month's end |
| Takes | a few minutes | a session |
| Logged as | `last-check` in `.maintenance-log.md` | `last-inventory` in the same file |

**Run both.** A vault whose tooling is perfectly current still rots, and the rot is the expensive kind: a decision gets reversed, the older version stays behind, two notes now disagree, and nobody notices until an agent reads the wrong one and builds on it. No version check catches that, because both files are perfectly well-formed.

The inventory is the slower and less obvious of the two, which is precisely why it has to be scheduled rather than remembered. Put it near the end of the month, when a monthly usage allowance would otherwise expire unused.

**Never let one report stand in for the other.** "Maintenance check done" says nothing about whether the vault still agrees with itself.

## When

Run the environment routine below when either of these is true:

- **First session of a new month.** The agent compares today's date with `last-check` in `template/.maintenance-log.md` (copied into the vault root as `.maintenance-log.md` during setup). If a month or more has passed, offer to run the check.
- **On request.** The user explicitly says "maintenance check" (or equivalent) at any time.

**For the content inventory, the trigger sits at the other end of the month.** From roughly the 26th onward, compare today's date against `last-inventory` in the same file; if this month has none, offer to run `vault-inventory`. Same as above, it stays an offer inside a conversation — the routine is a session's worth of work and nobody should find it already running.

If the vault has a session-start hook, that is the natural place for both nudges. A hook that reads the log file (or the git history) needs no extra state on disk, and unlike an in-session scheduler it survives the session that set it up.

This is a conversational routine the agent runs during a normal session — nothing here requires an external scheduler. If you'd still like a monthly nudge (for example because you don't open the vault every month), you can optionally set up an OS-level reminder that just drops a note into `01 Inbox/` so the agent picks it up next time you talk. These are illustrative examples, not something this template wires up for you:

**Windows Task Scheduler:**

```bash
schtasks /create /tn "SecondBrain Stack Maintenance Reminder" /tr "echo Maintenance check due >> \"C:\path\to\vault\01 Inbox\Maintenance reminder.md\"" /sc monthly /d 1
```

**macOS launchd** (save as `~/Library/LaunchAgents/com.secondbrain-stack.maintenance-reminder.plist`, then `launchctl load` it):

```xml
<key>StartCalendarInterval</key>
<dict><key>Day</key><integer>1</integer></dict>
```

**cron** (crontab line, runs at 09:00 on the 1st of each month):

```bash
0 9 1 * * echo "Maintenance check due" >> "/path/to/vault/01 Inbox/Maintenance reminder.md"
```

None of these run the maintenance routine itself — they only leave a note. The actual check always runs inside a conversation with the agent, per the `## Routine` steps below.

## Routine

1. **Commit the vault first.** Before touching anything, make sure the working tree is clean: `git add -A && git commit -m "chore: pre-maintenance checkpoint"` (skip if there's nothing to commit). This gives every later step a safe rollback point.
2. **Update skills.** For each skill folder that has a git source (see `manifest/skills.md` for provenance), pull or re-fetch the latest version. For any file that changed, re-run the security gate from `SETUP.md` Phase 4 before accepting the update — the same three checks used when a skill is first installed (public repo with real usage, read the files for anything that touches outside the vault, one-sentence summary + explicit OK). Afterwards, refresh both the `.claude/skills/` and `.agents/skills/` mirrors so Claude Code and Codex see the same skill versions as everyone else.

   **Verify the content, not the status message.** Skill managers report state from their own lock file, which records what upstream looked like at install time — not what is on your disk now. After an update, diff the actual `SKILL.md` against upstream before believing "up to date". This bites hardest with mirrors, where one copy gets refreshed and the other silently does not.

   **One canonical source, generated mirrors.** A skill lives once, in `skills/`; `.claude/skills/` and `.agents/skills/` are copies generated from it. Never hand-edit a mirror, and never keep a second independently maintained copy of a skill that a tool already installs globally — pick the location the tool maintains and generate from there. Two hand-maintained copies drift, and the one you are not looking at is always the one an agent loads.
3. **Check Obsidian plugin updates.** The agent can't update Obsidian plugins directly — remind the user to open Obsidian's Community Plugins settings and check for updates there.
4. **Check MCP server package versions.** Compare the installed version of each configured MCP server against its latest published release and flag any that are out of date.
5. **Check agent CLI updates.** Compare the local agent CLI version (for example `claude --version`) against the latest known release and flag if an update is available.
6. **Check Claude Code plugin updates** (skip if the user doesn't use Claude Code). Plugins update separately from the CLI and from skills:

   ```bash
   claude plugin marketplace update
   claude plugin update <name>
   ```

   If that fails with `Plugin not found` even though `claude plugin list` shows the plugin, pass the fully qualified id instead: `claude plugin update <name>@<marketplace>`. Both forms are documented; the short one is not reliable across versions when a name is ambiguous.
7. **Template update.** Read the version comment on line 1 of the vault's local `AGENTS.md` (`<!-- asbos-template-version: … -->` — the `asbos` key is a stable identifier from the project's original name and never changes) and compare it against the current version of `AGENTS.md` in the public `ai-secondbrain-stack` repo. If the public version is newer:
   - show what changed (changelog or diff) between the two versions,
   - offer **selective adoption** — the user picks which changes to bring in,
   - never overwrite the user's personalized content (their structure, rules, and language) as a side effect.
8. **Smoke test.** Run one full session-start routine end to end to confirm everything still works after the updates above.
9. **Close out.** Write today's date into `.maintenance-log.md` as `last-check`, give the user a short report of what was checked and changed, then commit: `git add -A && git commit -m "chore: maintenance check YYYY-MM-DD"`. If `last-inventory` in that same file is more than a month old, say so here — this routine does not cover it.

## Safety

Every step that changes something is preceded by a commit. If any update causes a problem, it can always be undone with `git revert` — nothing here is destructive or irreversible.

**One caution that applies to both routines.** When a step commits inside a repository other than the vault, stage the files you actually changed. A blanket `git add -A` there picks up whatever that repository's `.gitignore` does not cover on the branch you happen to be on, and ignore rules routinely differ between branches. That is how a one-line documentation fix ends up carrying thousands of lines of generated output.
