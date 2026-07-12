# AI SecondBrain Stack

<p align="center">
  <img src="assets/demo.gif" alt="Paste one prompt and your AI agent scaffolds the whole vault" width="820">
</p>

A git-versioned second brain for AI coding agents — Claude Code, Codex, Gemini CLI, OpenCode. Switch agents anytime; they all follow the same rules.

## Quick Start

1. **Install an AI coding agent.** Any of the following will work:
   - [Claude Code](https://docs.claude.com/en/docs/claude-code)
   - [OpenAI Codex CLI](https://github.com/openai/codex)
   - [Gemini CLI](https://github.com/google-gemini/gemini-cli)
   - [OpenCode](https://opencode.ai/)
2. **Open your agent in an empty, dedicated folder** — a folder you want your second brain to live in. Don't point it at an existing project; give it a clean home of its own.
3. **Paste the Bootstrap Prompt below** into your agent and let it take over from there.

```text
You are setting up AI SecondBrain Stack for me. Clone https://github.com/natscho-hh/ai-secondbrain-stack into a NEW subfolder of the current directory, read SETUP.md from it, and follow SETUP.md exactly, starting with Phase 0 (environment check). Ask me the interview questions one at a time. Do not skip the security checks.
```

Your agent will clone the repo, run the environment checks, interview you about how you work, and assemble a vault that's personalized to you. If you'd rather follow a guided, agent-specific walkthrough instead of pasting a raw prompt, see the landing page at [natscho-hh.github.io/ai-secondbrain-stack](https://natscho-hh.github.io/ai-secondbrain-stack/).

## What you get

- A **PARA-style vault** (`00 Context` through `07 Attachments`) that separates active projects, ongoing areas, reference resources, and a daily log so nothing gets lost between sessions
- A **personal context profile** — five files (about me, audience, offer, writing style, branding) filled from a short interview, so your agent writes in your voice and knows your world from day one
- **Session routines** baked into the rulebook: pull the latest changes, triage the inbox, write a daily note — every time you open the vault
- **Git sync after every change**, so your second brain is versioned, diffable, and recoverable like any codebase
- **Portable skills** that travel with the vault instead of living in a single agent's private config
- **Maintenance built in** — a standing routine for keeping the setup current as agents and tools evolve
- **Works in Obsidian** out of the box — ships with a starter `.obsidian/` config (sensible editor defaults, per-folder graph colors, a theme selection) so the graph view and editor look considered from the first launch, on top of the same plain-text files your agent reads and writes

## How it works

`AGENTS.md` is the single rulebook — every rule about vault structure, session routines, and git sync lives there and nowhere else. `CLAUDE.md` and `GEMINI.md` (and equivalents for other agents) are thin, three-line adapter files that simply point their respective agent at `AGENTS.md`, so the rules never have to be duplicated or kept in sync by hand. Your agent itself is the installer: `SETUP.md` is a script written for an AI agent to execute, not for a human to run manually, and it walks through environment checks, an interview, and vault assembly. Once set up, `MAINTENANCE.md` defines a recurring routine that keeps the rulebook, skills, and adapters up to date as the ecosystem changes.

## Repo map

```text
ai-secondbrain-stack/
├── README.md              # this file
├── LICENSE                # MIT
├── AGENTS.md              # the single rulebook every agent follows
├── CLAUDE.md              # 3-line adapter pointing Claude Code at AGENTS.md
├── GEMINI.md              # 3-line adapter pointing Gemini CLI at AGENTS.md
├── SETUP.md               # agent-executed installer (environment check, interview, assembly)
├── MAINTENANCE.md         # recurring update / health-check routine
├── .gitignore
├── skills/                # core skills, SKILL.md standard
├── manifest/
│   ├── skills.md          # curated skill catalog (name, purpose, source, license)
│   └── mcp.md             # MCP server catalog + per-agent config snippets
├── guides/
│   ├── first-steps.md     # beginner onboarding
│   ├── token-friendly.md  # working efficiently with an AI agent
│   ├── security-basics.md # skill/MCP security, secrets hygiene
│   ├── model-strategy.md  # plan with a strong model, build with a cheaper one
│   └── per-agent-tips.md  # tips per agent (Claude Code, Codex, Gemini CLI, OpenCode)
├── template/               # vault scaffold, copied into the user's new vault
│   ├── 00 Context/         # personal profile and working preferences
│   ├── 01 Inbox/           # unsorted captures, quick notes
│   ├── 02 Projects/        # active work with a concrete goal and end date
│   ├── 03 Areas/           # ongoing responsibilities without an end date
│   ├── 04 Resources/       # reference knowledge and documentation
│   ├── 05 Daily Notes/     # YYYY-MM-DD.md daily log
│   ├── 06 Archive/         # completed projects and inactive areas
│   └── 07 Attachments/     # images, PDFs, and other media
└── docs/                  # GitHub Pages landing page
```

## Requirements

- **Git**, for version control and syncing your vault
- **An AI coding agent** — Claude Code, Codex, Gemini CLI, or OpenCode
- **Obsidian** (recommended) — for browsing and editing your vault with a graph view
- **Node.js** (optional) — needed by some skills
- **A GitHub account** (recommended) — for hosting and syncing your own private copy of the vault

## License

MIT — see [LICENSE](LICENSE).
