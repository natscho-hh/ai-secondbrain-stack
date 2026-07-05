---
name: inbox-triage
description: "Process 01 Inbox: discard, park as pending, or file into the PARA structure."
---

## When to use

Use this skill whenever `01 Inbox/` has one or more notes waiting to be processed — at the start of a session, when the user asks to clean up the inbox, or any time a quick capture needs to find a permanent home.

## Steps

1. List every file currently in `01 Inbox/`.
2. For each note, read its content, then ask the user what to do with it. Offer exactly three options:
   - **Discard** — the note is no longer needed.
   - **Pending** — the idea is worth keeping but isn't ready to be filed yet.
   - **File** — the note is ready to move into the PARA structure (`02 Projects`, `03 Areas`, or `04 Resources`).
3. If the user chooses **discard**: confirm the exact file, then delete it.
4. If the user chooses **pending**: rename the file to `Pending - <topic>.md` and restructure it with three sections: `## Source` (where the idea came from), `## Analysis` (what it is and why it matters), `## Next step` (the concrete next action). Leave it in `01 Inbox/`.
5. If the user chooses **file**: move the note into the correct destination folder, and set its YAML frontmatter (`tags`, `status`, `date`) to match the target area's conventions.
6. Whatever the choice (pending or file), retrofit any missing frontmatter (`tags`, `status`, `date`) — quick captures often arrive without any.
7. After processing all notes in the batch, commit the changes with a short, descriptive message and push.

## Rules

- Never delete a note without asking the user first, even if it looks obviously obsolete.
- Never automatically move a processed note into `06 Archive/` — archiving always requires an explicit, separate request from the user.
- Never file a note into a PARA folder without the user's confirmation of the destination.
- Keep each note atomic: if a single inbox note contains multiple unrelated ideas, split it before filing.
