---
name: daily-note
description: Create or update today's daily note as the session log.
---

## When to use

Use this skill at the start of a session to log what's about to happen, during a session to record something worth remembering, or at the end of a session to summarize decisions and open threads — any time the day's activity needs a durable, dated record.

## Steps

1. Compute today's date and the target path: `05 Daily Notes/YYYY-MM-DD.md`.
2. Check whether that file already exists.
3. If it exists, append new content to the existing sections instead of overwriting anything already there.
4. If it does not exist, create it from the `99 Templates/Daily Note.md` template: YAML frontmatter containing `tags: [daily]` and `date: YYYY-MM-DD` (daily notes don't carry a `status` field), followed by three sections: `## What happened`, `## Decisions`, `## Open threads`.
5. Fill in each section from the current session's context: what was done, what was decided (and why), and what's left open or unresolved for next time.
6. Commit the change with a short, descriptive message and push.

## Rules

- Never delete or overwrite existing content in a daily note — always append.
- Keep entries factual and dated; a daily note is a log, not a place for open-ended brainstorming (link out to a project or resource note for that).
- One daily note per calendar day — never create a second file for the same date.
