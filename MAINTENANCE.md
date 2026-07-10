# Maintenance

This file describes the periodic upkeep routine for a vault built from AI SecondBrain Stack. It keeps the rulebook, skills, and adapters current as agents and tools evolve. `AGENTS.md` points here from its `## Maintenance` section.

## When

Run the routine below when either of these is true:

- **First session of a new month.** The agent compares today's date with `last-check` in `template/.maintenance-log.md` (copied into the vault root as `.maintenance-log.md` during setup). If a month or more has passed, offer to run the check.
- **On request.** The user explicitly says "maintenance check" (or equivalent) at any time.

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
2. **Update skills.** For each skill folder that has a git source (see `manifest/skills.md` for provenance), pull or re-fetch the latest version. For any file that changed, re-run the security gate from `SETUP.md` Phase 4 before accepting the update — the same three checks used when a skill is first installed (public repo with real usage, read the files for anything that touches outside the vault, one-sentence summary + explicit OK).
3. **Check Obsidian plugin updates.** The agent can't update Obsidian plugins directly — remind the user to open Obsidian's Community Plugins settings and check for updates there.
4. **Check MCP server package versions.** Compare the installed version of each configured MCP server against its latest published release and flag any that are out of date.
5. **Check agent CLI updates.** Compare the local agent CLI version (for example `claude --version`) against the latest known release and flag if an update is available.
6. **Template update.** Read the version comment on line 1 of the vault's local `AGENTS.md` (`<!-- asbos-template-version: … -->` — the `asbos` key is a stable identifier from the project's original name and never changes) and compare it against the current version of `AGENTS.md` in the public `ai-secondbrain-stack` repo. If the public version is newer:
   - show what changed (changelog or diff) between the two versions,
   - offer **selective adoption** — the user picks which changes to bring in,
   - never overwrite the user's personalized content (their structure, rules, and language) as a side effect.
7. **Smoke test.** Run one full session-start routine end to end to confirm everything still works after the updates above.
8. **Close out.** Write today's date into `.maintenance-log.md`, give the user a short report of what was checked and changed, then commit: `git add -A && git commit -m "chore: maintenance check YYYY-MM-DD"`.

## Safety

Every step that changes something is preceded by a commit. If any update causes a problem, it can always be undone with `git revert` — nothing here is destructive or irreversible.
