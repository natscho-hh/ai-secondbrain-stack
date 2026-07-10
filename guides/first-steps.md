# First Steps

A short walkthrough for the first time you sit down with your new vault and an AI coding agent. If you're a complete beginner to AI agents, read this straight through — each chapter builds on the last.

## 1. What is an AI coding agent?

An AI coding agent is a program that reads your instructions in plain language, then reads and writes files, runs commands, and reports back — the same way a person would, but from a terminal or editor panel instead of a chat window. It can see your vault's folders and files, make edits, and run `git` commands on your behalf, all with your approval. Unlike a plain chatbot, it can actually *do* things in your vault, not just talk about them.

## 2. Your first session

Open your agent (Claude Code, Codex, Gemini CLI, or OpenCode) **inside your vault folder** — the same folder `SETUP.md` created for you. This matters: an agent only sees and touches the folder it's started in, so starting it in the vault is what keeps it scoped to your second brain.

Once it's running, ask it to do the **session-start routine** described in `AGENTS.md` (your agent will do this automatically most of the time, but it's fine to ask). That routine:

1. Pulls the latest changes from git — so you're never working on a stale copy.
2. Shows you any new commits since your last session — a quick "what changed while I was away."
3. Checks `01 Inbox/` for new notes and offers to triage them — so nothing you captured gets forgotten.

You don't need to remember these three steps yourself. They live in `AGENTS.md`, and any agent following this vault's rulebook will run them for you.

## 3. Capture your first thought

Don't overthink where a new idea belongs. Just tell your agent the thought — a task, a question, something you noticed — and ask it to add it to `01 Inbox/`. That's the vault's front door: an unsorted holding area for anything without a home yet.

Then say "triage the inbox" and watch what happens. Your agent will read the note, ask where it should really live (a project? an area? a resource?), and — with your OK — file it there, often adding `[[wikilinks]]` to connect it to related notes. This is the vault doing its actual job: turning a stray thought into something findable later.

## 4. How to talk to an agent

State the **goal**, not the micro-steps. A good agent can figure out the individual file edits and commands on its own — what it needs from you is the outcome you want and any constraints that matter.

- **Good:** *"Set up a new project note for the kitchen renovation — budget, timeline, and a couple of open questions I still need to answer."*
- **Bad:** *"Create a file called kitchen-renovation.md, then add a heading called Budget, then add a heading called Timeline, then add a bullet point under Timeline that says..."*

The bad version turns you into the planner and the agent into a typist — you're doing the thinking it's supposed to help with. The good version gives it a goal and lets it use its judgment, ask if something's unclear, and show you the result.

## 5. When things go wrong

Every change to your vault is a git commit, which means every change is undoable. If an agent makes an edit you don't want, you can always ask it to revert — `git revert` undoes a commit while keeping the history of what happened, so you can see what was tried and why.

If you're not sure *why* an agent did something, just ask it: "why did you make that change?" or "walk me through your reasoning." A good agent can explain its own decisions in plain language — and if the explanation doesn't hold up, that's your cue to revert and try again with clearer instructions.

## 6. An optional pattern: draft with one AI, process with another

You don't have to capture everything through your coding agent. Once you're comfortable, a second, complementary pattern works well: use a **chat-based AI for drafting**, and your **coding agent for processing**.

Many chat assistants (for example ChatGPT) can connect directly to a GitHub repository. If you host your vault on GitHub and connect it, you can think out loud in the chat, have the assistant turn your rough thought into a clean, well-worded note, and write that note straight into `01 Inbox/` — without leaving the conversation. Later, in a normal session, your coding agent (Claude Code, Codex, or whichever you use) runs the inbox triage and files those drafts into the right place.

This splits the work along each tool's strength: the chat AI is good at turning a half-formed idea into readable prose and is always a tab away; the coding agent is good at structure, git, and moving files around. Capture with one, process with the other — the vault is just Markdown in git, so it doesn't care which AI touched it, and neither pattern locks out the other. This is the "works with any AI" idea in daily practice: different agents for different jobs, one shared vault underneath.
