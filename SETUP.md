# SETUP.md — Agent-Guided Installer

**You (the AI agent) are the installer.** A user has pasted the Bootstrap Prompt and handed control to you. Work through the phases below in order. One question at a time. Never skip a security check. Everything you create must be reversible (git) — if you can't undo it with a `git` command, don't do it.

This file is a script written for you to execute, not a document for the user to read. Do the work yourself: run the commands, ask the questions, make the decisions the brief delegates to you. The user may be a complete beginner who has never used an AI agent before — explain what you're doing in plain language as you go, and never assume they know a term you haven't defined.

## How to use this file

- Complete each phase fully before moving to the next. Phases 0 → 1 → 2 are always in order. After Phase 2 you branch to **either 3a (new vault) or 3b (migration)**, then continue with 4 → 5 → 6.
- "The vault" means the second-brain folder you are building for the user. "The template" means the `template/` folder inside the cloned `ai-secondbrain-os` repo.
- After every phase, state the **Success criteria** results back to the user in one line so they can see progress.
- If a step fails, stop and fix it before continuing. Do not paper over a failed step.
- Re-running this file later is expected and safe — see [Idempotency rules](#idempotency-rules) at the end. Never overwrite an existing file without asking.

---

## Phase 0 — Environment check

**Goal:** Make sure you are running in a safe, dedicated folder — not somewhere you could touch the user's whole computer — and make the ground rules (permissions, secrets) explicit before creating anything.

**Steps:**

1. Determine the current working directory (e.g. run `pwd`, or the equivalent for your platform) and show it to the user.
2. Judge whether this location is safe to build a vault in. It is **not safe** if it is:
   - the user's home directory (e.g. `~`, `/home/<name>`, `/Users/<name>`, `C:\Users\<name>`),
   - a system folder (e.g. `/`, `/etc`, `C:\Windows`, `C:\Program Files`), or
   - a folder that already contains unrelated files or projects.
   It **is safe** if it is an empty or dedicated folder created specifically for the vault (for example, the fresh subfolder the Bootstrap Prompt asked you to clone into is fine as a base, but the vault itself should live in its own dedicated folder).
3. If the location is not safe, warn the user in these terms: *"From here I can access far more of this computer than needed. Let's give your second brain its own dedicated folder so I only ever work inside it."* Then:
   a. Create a dedicated folder. Suggest `~/Vaults/` as the parent, and confirm the exact path with the user.
   b. Tell the user how to restart the agent in that folder — the exact instruction depends on their agent (for most CLI agents: close the agent, `cd` into the new folder, and start the agent again there). Do **not** try to silently `cd` and keep going; a clean restart in the safe folder is the reliable path.
   c. Wait for the user to confirm they have restarted in the new folder before continuing.
4. Explain the agent's permission mode in one short paragraph, in plain language: describe how your specific agent asks for approval before running commands or editing files (some agents prompt for every action, some run inside a sandbox, some need an explicit "allow" for network or file access). The point the user must understand: **you only ever operate inside this vault folder, and they can see and approve what you do.**
5. State the **secrets rule** clearly and record it as a rule you will follow for the whole setup: *Never put API keys, tokens, or passwords into vault notes or git commits. Secrets only ever go into an agent's own local configuration, never into a file that gets committed.*

**Success criteria:**
- [ ] The working directory is confirmed safe (empty/dedicated), or the user has restarted the agent in a new dedicated folder.
- [ ] The user has heard, in one paragraph, how their agent's permissions work and that you stay inside the vault folder.
- [ ] The secrets rule has been stated and you have committed to it for the rest of setup.

---

## Phase 1 — Dependency check

**Goal:** Confirm the tools the vault needs are present, and make clear which are required and which are optional, installing or linking anything that's missing.

**Steps:**

1. Check **Git** (required): run `git --version`. If it fails, give the user the official install page — <https://git-scm.com/downloads> — wait for them to install it, then re-check. Do not continue past this phase until `git --version` succeeds.
2. Check **Obsidian** (recommended): you can't reliably detect a GUI app from the shell, so **ask the user** whether Obsidian is installed. If not, point them to <https://obsidian.md/download>. It's not required to build the vault (the vault is plain Markdown either way), so continue whether or not they install it — just note that they'll get graph view and a proper editor once they do.
3. Check **Node.js** (optional): run `node --version`. If it's missing, tell the user it's only needed by *some* skills, and that they can install it later from <https://nodejs.org/> if a skill they want requires it. Do not block on this.
4. Check **GitHub access** (recommended): run `gh --version` to see if the GitHub CLI is present, and ask whether the user has a GitHub account. If they want a private synced copy of their vault later (Phase 3a/3b), the GitHub CLI is the smoothest path — point them to <https://cli.github.com/> and the sign-in command `gh auth login`. Not having it is fine; it only affects the optional "push to GitHub" step.
5. For anything the user chooses to install now, wait for them to finish and re-run the check before treating it as present.

**Success criteria:**
- [ ] `git --version` succeeds.
- [ ] The user knows whether Obsidian, Node.js, and GitHub CLI are present, and understands which are optional.
- [ ] Nothing required is missing; any missing optional tool has a clear "install later" path recorded.

---

## Phase 2 — Interview

**Goal:** Understand how *this specific user* works, in their own words, and derive the vault's shape (folders/areas, candidate skills, candidate MCP servers, language, repo choice, experience level, and whether they're migrating) from the conversation. Ask **one question at a time**.

**Steps:**

1. **Open with examples, then ask the open question.** Show 3–4 concrete usage examples so the user understands the range, for instance:
   - *Content creation* — drafting posts, scripts, newsletters in a consistent voice.
   - *Coding projects* — tracking features, decisions, and knowledge across repos.
   - *Research* — collecting sources, studies, and notes on a topic over time.
   - *Health / habit tracking* — training, nutrition, routines, and progress logs.

   Then ask the **open** question and let them answer freely:

   > "What do you want to build or manage with your second brain? Just tell me in your own words."

2. **Derive, don't interrogate.** From their answer, work out *yourself*:
   - which **areas and projects** the vault should start with (map their words onto `03 Areas` and `02 Projects`),
   - which **skills** might help (you'll match these against `manifest/skills.md` and community directories in Phase 4),
   - which **MCP servers** might help (you'll match these against `manifest/mcp.md` in Phase 5).

   Only ask follow-up questions where something is **genuinely unclear** — don't re-ask what they already told you. Summarize back what you inferred and let them correct it.
3. **Ask the setup basics** (one at a time):
   - **Vault language** — what language should the vault and its notes be in? (This decides whether folder names and the rulebook stay English or get translated in Phase 3a.)
   - **Vault name** — what should the vault folder be called?
   - **Private GitHub repo** — do they want a private GitHub repo created for the vault (recommended for backup and sync), yes or no?
4. **Ask experience level** with AI agents: *beginner*, *intermediate*, or *pro*. Explain that this only steers how much hand-holding Phase 6 (onboarding) gives them — it changes nothing about the vault itself.
5. **Ask the migration question** — this decides the next phase:

   > "Do you already use an Obsidian vault or another note collection you'd want to bring in?"

   - If **yes** → go to **Phase 3b (Migration)**.
   - If **no** → go to **Phase 3a (Scaffold)**.

**Success criteria:**
- [ ] The user has answered the open question, and you have summarized back the areas/projects, candidate skills, and candidate MCPs you derived — and they've confirmed it.
- [ ] Vault language, vault name, private-repo choice, and experience level are all recorded.
- [ ] The migration question is answered, and you know whether the next phase is 3a or 3b.

---

## Phase 3a — Scaffold (new vault)

**Goal:** Build a fresh, personalized vault from the template, translated if needed, initialized as a git repo, and optionally pushed to a private GitHub repo.

**Steps:**

1. Create the vault folder using the **vault name** from the interview, and copy the **contents of `template/`** into it — all nine folders (`00 Context` through `07 Attachments`, plus `99 Templates`) with their example content, the root-level `template/todos.md` priority board, the root-level `template/.maintenance-log.md` file (lands at the vault root as `.maintenance-log.md`), and the `template/.obsidian/` folder (starter Obsidian settings — editor defaults, per-folder graph colors, theme selection; lands at the vault root as `.obsidian/`). **Copy hidden dotfiles and dotfolders too** — a naive "copy everything" can silently skip names starting with a dot, so verify `.maintenance-log.md` and `.obsidian/` actually made it across.
2. **If the vault language is not English:** translate the nine folder names and the generated rule file **consistently** into the chosen language (e.g. `00 Context` → `00 Kontext`). Keep the numeric prefixes and the two-digit ordering unchanged. Apply the same translated names everywhere they appear — in the folders themselves, in every reference inside `AGENTS.md`, **and inside the copied `.obsidian/` settings** (the `attachmentFolderPath`/`newFileFolderPath` in `.obsidian/app.json` and the per-folder `query` values in `.obsidian/graph.json` all name folders by their English canon — retarget them to the translated names, or the graph colors and default paths will point at folders that no longer exist).

   Also translate the **example starter notes** so the vault reads natively from the first launch: the body text of `00 Context/About me`, `01 Inbox/Welcome`, and `02 Projects/Example Project`, plus the three note templates in `99 Templates/` and the root `todos.md` board (translate the file's headings and body; keep the `{{date}}` / `{{title}}` placeholders in the templates unchanged — Obsidian fills them in). Localize their **frontmatter *values*** too — the tag words (`context`, `inbox`, `project`, …) and the `status` value (`active`) — into the chosen language, using the **same vocabulary your translated `AGENTS.md` uses** so tags and statuses match across the vault. Keep the frontmatter **field keys** (`tags`, `status`, `date`) as they are — those are conventional YAML keys, not prose. **Leave `skills/` and `guides/` in English** — they're reference material, not vault content, and many community skills are English-only anyway; the `CLAUDE.md`/`GEMINI.md` adapters also stay English (see step 4). If the language is English, skip this whole step.
3. Write a **personalized `AGENTS.md`** into the vault: copy it from the repo root as the base, then adjust the structure section, rules, and language to match the interview results (their real areas/projects, their language). **Keep the version comment on line 1 UNCHANGED** — it must stay exactly as it is in the repo root's `AGENTS.md` (currently `<!-- asbos-template-version: 0.2.0 -->`) so later maintenance can tell which template version this vault came from.
4. Write both **adapters** next to it: `CLAUDE.md` and `GEMINI.md`, each a thin file that points at `AGENTS.md` (copy the three-line adapters from the repo root). These are agent entry files and stay in English regardless of vault language, because they only redirect the agent to `AGENTS.md`.
5. Initialize version control inside the vault folder: `git init`, then make the **first commit** with everything staged (`git add -A && git commit -m "chore: initial vault from ai-secondbrain-os template"`).
6. **If the user wanted a private GitHub repo** (from the interview) and the GitHub CLI is available: create and push it with
   ```bash
   gh repo create <vault-name> --private --source . --push
   ```
   If `gh` isn't available, tell the user how to create the private repo manually in the GitHub web UI and add it as a remote, or skip and note it as a "later" step. Never make the repo public.

**Success criteria:**
- [ ] The vault folder exists, named after the vault, containing all nine folders with example content, plus `todos.md`, `.maintenance-log.md`, and the `.obsidian/` settings folder at the vault root.
- [ ] If a non-English language was chosen, folder names, `AGENTS.md`, the `.obsidian/` settings, and the three example notes (body + frontmatter tag/status values) are fully and consistently translated; `skills/` and `guides/` are left in English; version comment line is untouched.
- [ ] `AGENTS.md` + `CLAUDE.md` + `GEMINI.md` are present in the vault, personalized where appropriate.
- [ ] `git init` and a first commit are done; if requested and possible, a private GitHub repo exists and has been pushed to.

---

## Phase 3b — Migration (existing vault)

**Goal:** Bring the AI SecondBrain OS rulebook and structure to a vault the user **already has**, strictly non-destructively. Nothing the user already built gets deleted, renamed, or reorganized without their explicit, item-by-item confirmation. When in doubt, leave it as it is.

**Steps:**

1. **Commit first.** Before touching anything, make the existing vault recoverable. If it's already a git repo, commit any uncommitted changes so there's a clean checkpoint (`git add -A && git commit -m "chore: checkpoint before ai-secondbrain-os migration"`). If it is **not** yet a git repo, run `git init` and make an initial commit of the current state. This checkpoint is what makes every later step reversible.
2. **Analyze the existing structure.** Read the vault as it actually is: its folders, how notes are organized, naming conventions, and — importantly — any **installed plugins or skills** already in use (Obsidian plugins, an existing `skills/` folder, agent config files). Build an accurate picture of what's there before proposing anything.
3. **Present a mapping table — as a recommendation, not a command.** Show the user a table comparing their vault to the AI SecondBrain OS layout, with columns like: *existing folder → recommended place*, *what's missing* (parts of the PARA layout they don't have yet), *what already fits* (folders that already map cleanly), and *what stays different on purpose* (structure of theirs you'd deliberately keep as-is). **Recommend a change only where it adds real value.** If their structure already works, say so and leave it. Make clear nothing happens until they choose.
4. **Apply ONLY what the user confirms, item by item.** Go through the mapping one row at a time and act only on the rows the user approves. Existing notes, their `[[wikilinks]]`, and their installed skills **stay intact** — you do not rewrite links or move notes wholesale. Any skills you detected in step 2 are **registered** (recorded in the vault's manifest/rulebook so agents know they exist), **not reinstalled** — don't re-download or overwrite something they already have. **Never overwrite the user's existing `.obsidian/` settings** — those are their own appearance, hotkeys, and plugin configuration. If they don't have per-folder graph colors yet and want the template's, you may *offer* to add the `colorGroups` from `template/.obsidian/graph.json` (retargeted to their real folder names) to their existing `graph.json` — but only additively, and only if they say yes.
5. **Write `AGENTS.md` + adapters adapted to the REAL structure.** Generate `AGENTS.md` (plus the `CLAUDE.md` and `GEMINI.md` adapters) describing the vault **as it actually is after the confirmed changes** — the user's real folder names and organization — not the idealized template layout. Keep the version comment line `<!-- asbos-template-version: 0.1.0 -->` on line 1 so maintenance can track the template version. Commit the result.
6. **Create `.maintenance-log.md` at the vault root**, if it doesn't already exist — the migration path copies no `template/` content, so this file (which `AGENTS.md`'s Maintenance section and `MAINTENANCE.md` both expect to find) is never created otherwise. Give it the same initial shape as the template's: a `last-check:` line dated to today, plus the comment line `<!-- updated by the maintenance routine -->`. Commit the result.

**Success criteria:**
- [ ] A git checkpoint of the pre-migration state exists (commit made, or repo initialized + committed).
- [ ] The user has seen a mapping table framed as recommendations, and only approved rows were applied.
- [ ] All existing notes, wikilinks, and installed skills are intact; detected skills are registered, not reinstalled.
- [ ] `AGENTS.md` + adapters describe the vault's real structure and carry the unchanged version comment; changes are committed.
- [ ] `.maintenance-log.md` exists at the vault root with a `last-check` date and the maintenance-routine comment line.

---

## Phase 4 — Skills

**Goal:** Give the vault the skills that match how the user works — the core skills that ship with the template, curated matches from the manifest, and vetted community skills — installing third-party skills only after a security check and explicit approval.

**Steps:**

1. **Copy the core skills.** Copy the skills from the repo's `skills/` folder into the vault's `skills/` folder. Each skill is its own folder with a `SKILL.md` entry point; keep that structure intact.
2. **Match the manifest.** Read `manifest/skills.md` (the curated catalog: name, purpose, source, license) and compare its entries against the goals you derived in Phase 2. **Propose** the matching entries to the user, saying in one line what each one is for. Note that `manifest/skills.md`'s Install column shows each tool's upstream default (often a single agent's own private config directory) — when you actually install a third-party skill, also place its folder into the vault's `skills/<name>/` (adapting the upstream command's destination), so it's portable across whichever agent the user is running, not just the one the upstream default targets.
3. **Search community directories.** For goals the manifest doesn't cover, additionally search community skill directories (for example the awesome-claude-skills list) for skills that fit the user's stated goals, and surface promising ones.
4. **Run the security gate before EVERY third-party skill.** Before installing any skill that isn't a template core skill, run this check and say it to the user in these exact words:

   > Before installing, I check: (1) Is the source a public repo with real usage? (2) I read the skill's files for commands that touch things outside the vault (network calls, deletions, credential access). (3) I summarize what it does in one sentence and wait for your OK.

5. **Install only after confirmation.** Actually install a third-party skill only once the user says OK for that specific skill. If any step of the gate raises a concern (private/unknown source, no real usage, commands that reach outside the vault without a clear reason), tell the user plainly and let them decide with that information in hand.

**Success criteria:**
- [ ] Core skills from `skills/` are copied into the vault with their `SKILL.md` structure intact.
- [ ] Manifest matches and any community finds were proposed with a one-line purpose each.
- [ ] The security gate was run verbatim before every third-party skill, and nothing third-party was installed without explicit user approval.

---

## Phase 5 — MCP (optional)

**Goal:** Offer the user MCP servers that match their goals, and configure only the ones they want — for only the agent(s) they actually use — keeping every API key out of the vault.

**Steps:**

1. From the goals derived in Phase 2, read `manifest/mcp.md` and **offer the matching MCP servers**, explaining in one line what each adds. MCP is optional — make clear the vault works fully without any of it.
2. Configure a server only after the user opts in. **Write MCP config only for the agent(s) the user actually uses** — don't scatter config files for agents they've never installed. `manifest/mcp.md` contains the per-agent config snippets; use the one that matches their agent.
3. For any server that needs an **API key**: ask the user for it, and store it **only in that agent's own local configuration** — the exact location depends on the agent. **NEVER** write the key into a vault note, and NEVER let it enter a git commit. Re-state the secrets rule from Phase 0 if the user seems unsure.
4. After writing config, tell the user how to verify the server loaded (typically by restarting the agent) and remind them that keys live only in their local agent config.

**Success criteria:**
- [ ] Matching MCP servers were offered with a one-line purpose each, and only opted-in servers were configured.
- [ ] Config was written only for the agent(s) the user actually uses.
- [ ] Any API key is stored only in the agent's local config — never in the vault, never in a commit.

---

## Phase 6 — Verify + onboarding

**Goal:** Prove the vault works end to end with a real session-start run, create the user's first daily note, and hand off in a way that matches their experience level — then tell them how to keep it current and how to add more later.

**Steps:**

1. **Run one full session-start routine as a test.** Follow the session-start routine from the vault's `AGENTS.md` (pull latest, show new commits, check `01 Inbox/` for notes to triage) so both you and the user see it work. If the vault has no remote yet, note that the pull step is a no-op for now and will start working once GitHub is connected.
2. **Create the first daily note** at `05 Daily Notes/YYYY-MM-DD.md` (using today's date), so the daily log exists from day one. Add a short first entry noting that setup is complete.
3. **Onboard according to experience level** (from Phase 2):
   - **Beginner:** walk through `guides/first-steps.md` **interactively**, doing the steps together rather than just linking it. While you do, explain **token-friendly habits** (from `guides/token-friendly.md`) and the **model strategy** (plan with a strong model, build with a cheaper one — `guides/model-strategy.md`) in context, as they come up.
   - **Intermediate / pro:** hand over the `guides/` folder as reference and point out the specific guides most relevant to their stated goals, without walking through them step by step.
4. **Explain how to keep it current and how to extend it.** Tell the user:
   - how to **update later** — the recurring health-check routine defined in `MAINTENANCE.md`, which keeps the rulebook, skills, and adapters current as agents and tools evolve.
   - how to **re-run setup to add modules** later — re-running this `SETUP.md` is **idempotent**: it never overwrites what's already there, it only adds. To add skills or folders later, they just ask the agent to re-run the relevant phase.

**Success criteria:**
- [ ] A full session-start routine has been run successfully as a test.
- [ ] Today's daily note exists in `05 Daily Notes/` with a first entry.
- [ ] The user has been onboarded at the right depth for their experience level (beginner = interactive walkthrough incl. token-friendly + model-strategy habits; intermediate/pro = guides handed over as reference).
- [ ] The user knows how to update later (`MAINTENANCE.md`) and that re-running setup only adds, never overwrites.

---

## Idempotency rules

Setup is safe to run again — a user may re-run it months later to add a skill, a folder, or an MCP server. Follow these rules every time:

- **Never overwrite an existing file without asking.** If a file you're about to write already exists, stop and ask the user whether to keep, merge, or replace it — default to keeping what's there.
- **Adding things later = re-run the relevant phase only.** To add skills, re-run Phase 4. To add an MCP server, re-run Phase 5. To add or reshape folders, re-run the relevant part of Phase 3. You do not re-run the whole installer from Phase 0 to make a small addition.
- **Re-running never destroys.** Every run only adds or, with explicit confirmation, changes a specific item. Nothing the user already has is removed as a side effect of running setup again — and because every change is committed to git, anything can be undone.
