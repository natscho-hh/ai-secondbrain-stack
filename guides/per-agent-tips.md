# Per-Agent Tips

Practical notes for each of the five agents this vault supports. Facts below were checked against each vendor's own docs; where something changes quickly (exact model names, fast-moving CLI flags), that's flagged so you know to double-check against the agent's own help output instead of trusting this file blindly.

## Claude Code

- **Install:** native installer — `curl -fsSL https://claude.ai/install.sh | bash` — is the current recommended path (auto-updating, no Node dependency). The older `npm install -g @anthropic-ai/claude-code` still works but is being phased out in favor of the native installer. Docs: <https://docs.claude.com/en/docs/claude-code>.
- **Loads the rulebook via:** the `CLAUDE.md` adapter in your vault root, which points to `AGENTS.md`.
- **Plan mode:** press `Shift+Tab` twice to cycle into it (status bar shows "⏸ plan mode on"); `/plan` is also available from v2.1 onward.
- **Model switch:** `/model` — e.g. `/model opus`, `/model sonnet`, `/model haiku`; the menu also has a combined "Opus in plan mode, Sonnet otherwise" option.
- **MCP config:** `.mcp.json` in the vault root (project-scoped, shareable via git) or `~/.claude.json` (user-scoped, managed with `claude mcp add --scope user`).
- **Quirk:** Claude Code is the only one of these five agents that reads the vault's `skills/` folder natively — every other agent needs to be told to check it manually, per the "Skill reflex" section of `AGENTS.md`.
- **Session-routine hooks (optional, recommended):** the session routines in `AGENTS.md` are prose — they work only as long as the agent remembers them. Claude Code can enforce the two critical ones mechanically with hooks in `.claude/settings.json` inside your vault:

  ```json
  {
    "hooks": {
      "SessionStart": [
        { "matcher": "startup", "hooks": [{ "type": "command", "command": "sh .claude/hooks/session-start.sh" }] }
      ],
      "Stop": [
        { "hooks": [{ "type": "command", "command": "sh .claude/hooks/stop-gitsync.sh" }] }
      ]
    }
  }
  ```

  `session-start.sh` runs the session-start routine for real (pull + inbox report; its stdout lands in Claude's context):

  ```sh
  #!/bin/sh
  git pull --no-edit origin main
  echo "Inbox:"
  ls "01 Inbox"
  ```

  `stop-gitsync.sh` is a git-sync guard: exit code 2 blocks the session from ending and shows the message to Claude, so unsynced changes can't slip through. The `stop_hook_active` check prevents an infinite loop:

  ```sh
  #!/bin/sh
  payload=$(cat)
  case "$payload" in *'"stop_hook_active":true'*) exit 0 ;; esac
  if [ -n "$(git status --porcelain)" ]; then
    echo "Uncommitted vault changes - run the git sync (add/commit/push) before ending." >&2
    exit 2
  fi
  ```

  On Windows, write the same two scripts in PowerShell and call them with `powershell -NoProfile -ExecutionPolicy Bypass -File`. The other four agents have no hook system for this — for them the routines stay conversational, which is exactly why they're written down in `AGENTS.md`.

## Codex CLI

- **Install:** `npm install -g @openai/codex` (requires Node.js). Docs: <https://developers.openai.com/codex/cli>.
- **Loads the rulebook via:** `AGENTS.md` directly — Codex reads it natively, no adapter file needed.
- **Plan mode:** the `/plan [goal]` slash command switches the session into plan mode; you can pair it with `--path` (target a specific directory) or `--model` flags.
- **Model switch:** `/model` — also lets you adjust reasoning level, not just the model itself.
- **MCP config:** `~/.codex/config.toml`, with servers under `[mcp_servers.<name>]` — note this is TOML, and the key is `mcp_servers` (underscore), not the `mcpServers` (camelCase) used by the other JSON-based agents.
- **Quirk:** it's the one agent here configured in TOML rather than JSON, so a config snippet copied from Claude Code or Cursor needs reformatting, not just a key rename.

## Gemini CLI

- **Install:** `npm install -g @google/gemini-cli` (requires Node.js 18+), or run it without installing via `npx`. Docs: <https://geminicli.com/docs/get-started/installation/>.
- **Loads the rulebook via:** the `GEMINI.md` adapter in your vault root, which points to `AGENTS.md`.
- **Plan mode:** enabled by default; `Shift+Tab` cycles approval modes (Default → Auto-Edit → Plan), `/plan [goal]` switches into it directly, or launch straight into it with `gemini --approval-mode=plan`.
- **Model switch:** `-m` / `--model` flag at launch, or `/model` to change models interactively mid-session.
- **MCP config:** `~/.gemini/settings.json`, servers nested under `mcpServers`.
- **Quirk:** Gemini CLI's plan mode is stricter than the others' — it's genuinely read-only (write tools are blocked) until you explicitly approve the plan and switch to an editing mode, rather than just asking before each edit.

## Cursor

- **Install:** the editor is a download from <https://cursor.com/>; the standalone CLI installs with `curl https://cursor.com/install -fsS | bash` and runs as the `agent` command.
- **Loads the rulebook via:** `AGENTS.md` directly, read natively at the project root by both the editor and the CLI — alongside `.cursor/rules` if you have any, no adapter needed.
- **Plan mode:** `Shift+Tab` in the agent input (editor); a plan opens as an editable Markdown file you can revise before approving, and can be saved to `.cursor/plans/`.
- **Model switch:** the model picker dropdown in the editor's agent panel; on the CLI, the `--model` flag at launch or `/model` interactively.
- **MCP config:** `~/.cursor/mcp.json` (global, all projects) or `.cursor/mcp.json` in the project root (project-scoped).
- **Quirk:** Cursor can fan a single prompt out to several different models at once, each running in its own git worktree, so you can compare their results side by side before picking one to keep.

## GitHub Copilot

- **Install:** the "GitHub Copilot" and "GitHub Copilot Chat" extensions from the VS Code Marketplace (requires an active Copilot subscription/license). Docs: <https://code.visualstudio.com/docs/copilot/agents/overview>.
- **Loads the rulebook via:** `AGENTS.md` directly — Copilot reads it natively (the nearest `AGENTS.md` in the directory tree wins), and can also pick up `.github/copilot-instructions.md` alongside it.
- **Plan mode:** the built-in "Plan" agent in Copilot Chat's agent picker explores the codebase, asks clarifying questions, and produces a reviewable step-by-step plan before touching any files.
- **Model switch:** the model picker dropdown in the Chat view — there's no slash command for this; custom agents can also pin a model via the `model:` property in a `.agent.md` file.
- **MCP config:** `.vscode/mcp.json` in the project root (workspace-scoped).
- **Quirk:** Copilot's MCP config uses a top-level `servers` key instead of the `mcpServers` / `mcp_servers` key every other agent here uses — copying a snippet from Claude Code, Gemini, or Cursor without renaming that key is the most common mistake (also called out in `manifest/mcp.md`).
