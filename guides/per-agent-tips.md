# Per-Agent Tips

Practical notes for each of the four agents this vault supports. Facts below were checked against each vendor's own docs; where something changes quickly (exact model names, fast-moving CLI flags), that's flagged so you know to double-check against the agent's own help output instead of trusting this file blindly.

## Compatibility matrix

| | Claude Code | Codex CLI | Gemini CLI | OpenCode |
|---|---|---|---|---|
| **Rulebook entry** | `CLAUDE.md` adapter | `AGENTS.md` (native) | `GEMINI.md` adapter | `AGENTS.md` (native) |
| **Skill discovery** | `.claude/skills/` (native — mirror of `skills/`) | `.agents/skills/` (native — mirror of `skills/`) | reads `skills/` via the `GEMINI.md` adapter | reads `skills/` via `AGENTS.md`, plus a native `skill` tool |
| **Session hooks** | `.claude/settings.json` | `hooks.json` (global `~/.codex/` or project `.codex/`) | hooks configured per the Gemini CLI hooks reference | plugin event subscriptions (`"plugin": [...]` in `opencode.json`) |
| **MCP config** | `.mcp.json` / `~/.claude.json` | `~/.codex/config.toml` (`mcp_servers`, TOML) | `~/.gemini/settings.json` (`mcpServers`) | `opencode.json` (`mcp` key) |
| **Plan mode** | `Shift+Tab` / `/plan` | `/plan` | `Shift+Tab` / `/plan` / `--approval-mode=plan` | `Tab` toggle |
| **Model switch** | `/model` | `/model` | `-m` flag / `/model` | `/models` |

The rows are the portable/vendor-specific split in one view: the rulebook, skills, guides, and vault content are shared; hook wiring and MCP config files are per-agent and never leave the agent's own config.

## Windows notes

**Point tools at the real binary, not the `.cmd` shim.** The security fix for CVE-2024-27980 made patched Node reject `.cmd` and `.bat` files passed directly to `child_process.spawn()` or `spawnSync()` with `shell: false` — you get a bare `EINVAL`. Anything that launches an agent CLI as a child process hits this if it targets the npm-generated shim. Point it at the real executable, or at `node.exe` with the package's JavaScript entrypoint. `shell: true` also works, but only use it with fully controlled arguments: it re-opens exactly the command-injection class the fix closed.

**`.DELETE.<hash>` leftovers mean an interrupted npm operation.** Files with that suffix under `node_modules` are the remains of a rename or cleanup that npm did not finish. Treat them as a diagnosis, not a diagnosis-and-cure: check the npm log and `npm ls` first. If platform-specific optional packages are genuinely missing, `npm install --include=optional` refetches them. If the tree is inconsistent in other ways, delete `node_modules` and reinstall — slower, but it actually resolves the state.

**Path length and synced folders.** Windows truncates at 260 characters unless long paths are enabled, and nested `node_modules` inside a synced folder reaches that quickly. Keep vaults and repos out of sync-client folders anyway: a sync client rewriting files under a running tool is its own category of bug.

## Claude Code

- **Install:** native installer — `curl -fsSL https://claude.ai/install.sh | bash` — is the current recommended path (auto-updating, no Node dependency). `npm install -g @anthropic-ai/claude-code` remains a fully documented alternative, alongside Homebrew, WinGet, and Linux package managers (apt/dnf/apk). Docs: <https://code.claude.com/docs>.
- **Loads the rulebook via:** the `CLAUDE.md` adapter in your vault root, which points to `AGENTS.md`.
- **Plan mode:** press `Shift+Tab` twice to cycle into it (status bar shows "⏸ plan mode on"); `/plan` is also available from v2.1 onward.
- **Model switch:** `/model` — e.g. `/model opus`, `/model sonnet`, `/model haiku`; the menu also has a combined "Opus in plan mode, Sonnet otherwise" option.
- **MCP config:** `.mcp.json` in the vault root (project-scoped, shareable via git) or `~/.claude.json` (user-scoped, managed with `claude mcp add --scope user`).
- **Quirk:** Claude Code discovers skills natively only from `.claude/skills/`, not the vault's `skills/` folder directly — `skills/` is the canonical source, `.claude/skills/` a generated mirror (set up in `SETUP.md` Phase 4). Codex works the same way, from its own `.agents/skills/` mirror (see the Codex section). Gemini CLI and OpenCode have no native project-skill directory and need to be told to check `skills/` manually, per the "Skill reflex" section of `AGENTS.md`.
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
  echo "Agent branches not merged into main:"
  git branch -r --no-merged origin/main | grep -v 'origin/HEAD'
  ```

  The last two lines are session-start step 4 from `AGENTS.md`. Unattended agents write to `agent/<topic>` branches, and a hook that only reads `main` never shows their work.

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

  On Windows, write the same two scripts in PowerShell and call them with `powershell -NoProfile -ExecutionPolicy Bypass -File`. Codex has its own hook system with the same JSON shape (`~/.codex/hooks.json` or project-local `.codex/hooks.json`; note Codex expects JSON on stdout — plain-text output is discarded as failed, and project-local hooks must be trusted once via `/hooks`). Gemini CLI now has a hooks system too (`SessionStart` / `SessionEnd` events configured in `settings.json` — see the [Gemini CLI hooks reference](https://geminicli.com/docs/hooks/reference/)), but both events are advisory only: the CLI never waits on them and ignores their `continue`/`decision` fields, so they can't block startup or exit the way the git-sync guard above needs. OpenCode's plugin system (`"plugin": [...]` in `opencode.json` — see the [OpenCode plugin docs](https://opencode.ai/docs/plugins/)) can veto actions, such as a tool call, from inside a `tool.execute.before` hook, but its session events (`session.created`, `session.idle`, and so on) track chat-session lifecycle, not the CLI process starting or exiting, so there's no equivalent of the Stop guard there either. For both agents the routines stay conversational, which is exactly why they're written down in `AGENTS.md`.

- **Context fill level in the status line (optional, recommended):** Claude Code passes a JSON object on stdin to the status-line command on every refresh. Its `context_window.used_percentage` field is the fill level of the current context window, and `model.display_name` names the model. A few lines turn that into a gauge you see all the time instead of a number you have to ask for with `/context`:

  ```json
  { "statusLine": { "type": "command", "command": "sh .claude/statusline.sh" } }
  ```

  ```sh
  #!/bin/sh
  # Green below 50 %, yellow from 50 %, red from 75 %.
  jq -r '
    (.context_window.used_percentage // empty | floor) as $p
    | (if $p >= 75 then "196" elif $p >= 50 then "214" else "114" end) as $c
    | "\u001b[38;5;\($c)mctx \($p)%\u001b[0m \(.model.display_name // "")"'
  ```

  Put the setting in `~/.claude/settings.json` if you want it in every project, or in the vault's `.claude/settings.json` for the vault alone. On Windows, the same logic in PowerShell reads stdin with `[Console]::In.ReadToEnd()` and parses it with `ConvertFrom-Json`. If you already run another status-line script, call it from the same process rather than starting a second shell — on Windows each extra PowerShell start costs around 400 ms per refresh.

- **Measure your base context before optimizing it.** Everything loaded before your first word — system prompt, rulebook, memory index, skill descriptions, MCP tool names, hook output — is paid again on every cache rewrite. One headless call shows how much it is:

  ```sh
  claude -p "Reply with the word OK." --output-format json
  ```

  The `usage.cache_creation_input_tokens` value of that first reply is your base context. Run it once more with `--strict-mcp-config` (no MCP servers from config files) and once with `--disable-slash-commands` (no skills), and the differences tell you what each layer costs. Measured in the vault this template comes from: roughly 98,000 tokens in total, about 17,000 of them for MCP servers even though most of their tools were only loaded by name, and just 1,200 for all skill descriptions together — skills are cheap, connectors are not. The two savings did not simply add up, so measure the combination too rather than summing. In the same vault, identical rule text cost **1.5 times as many tokens in German as in English**; that is worth knowing, and not a reason to write your rules in a language you do not read.

- **Tool router on `UserPromptSubmit` (optional, measured, not recommended by default):** the skill-reflex rule in `AGENTS.md` asks the agent to check the available tools before every task. A tempting way to enforce that is a machine-readable tool register plus a `UserPromptSubmit` hook that injects only the matching lines, so a long tool table stops being loaded on every session. It works — it was built and measured in the vault this template comes from. The numbers are worth knowing before copying the idea:

  - Removing a 37-row tool table saved **2,418 bytes** from the always-loaded rulebook.
  - The hook cost **~300 ms on every prompt**, of which only **2 ms** was the actual matching. The rest is process startup, so simplifying the matcher saves nothing.
  - After the other rules from the same rework were added, the rulebook ended up **27 bytes smaller than before it started**. Net zero.
  - Injected lines land in the conversation and get re-sent with every later request, while the saving from a smaller rulebook applies once per request. Over a long session the ledger can tip the wrong way.
  - Naive substring matching produced absurd hits — in German, the word for *voice* is a substring of the word for *agree*, so "can you agree with that?" pulled the text-to-speech tool. **Match on word boundaries**, and drop any trigger shorter than four characters.

  The durable half of the idea is the **register, not the hook**: keep the tool list as JSON rather than as a markdown table, so every agent can parse it without guessing. Two independent reviewers failed to parse the equivalent markdown table reliably — struck-through cells, prose in a path column, several values in one cell. Ship the register; add the hook only once you have measured that your rulebook is genuinely the bottleneck.

## Codex CLI

- **Install:** standalone installer — `curl -fsSL https://chatgpt.com/codex/install.sh | sh` (macOS/Linux; the same docs page has Windows and Homebrew variants) — is the primary documented path. `npm install -g @openai/codex` (requires Node.js) remains a supported alternative. Docs: <https://learn.chatgpt.com/docs/codex/cli>.
- **Loads the rulebook via:** `AGENTS.md` directly — Codex reads it natively, no adapter file needed.
- **Plan mode:** the `/plan` slash command switches the session into plan mode, optionally with an inline prompt (e.g. `/plan propose a migration plan for this service`). Keyboard shortcuts and any extra flags have moved between recent releases — check `codex --help` or `/plan` in-session for what your version supports.
- **Model switch:** `/model` — also lets you adjust reasoning level, not just the model itself.
- **MCP config:** `~/.codex/config.toml`, with servers under `[mcp_servers.<name>]` — note this is TOML, and the key is `mcp_servers` (underscore), not the `mcpServers` (camelCase) used by the other JSON-based agents.
- **Skill discovery (native):** Codex scans `.agents/skills/` — repo-local (vault root, parent folder, repo root) and global (`~/.agents/skills/`) — and follows symlinks/junctions. It does **not** scan the vault's `skills/` folder, which is why setup mirrors `skills/` into `.agents/skills/`. Two gotchas: the `SKILL.md` frontmatter must start on line 1 — even a leading UTF-8 BOM makes Codex reject the skill — and while Codex detects new or changed skills automatically, a restart is the documented fallback when one doesn't show up.
- **Session-routine hooks (optional, recommended):** Codex has a hook system with the same JSON shape as Claude Code's — put this in `.codex/hooks.json` in your vault:

  ```json
  {
    "hooks": {
      "SessionStart": [
        { "hooks": [{ "type": "command", "command": "sh .codex/hooks/session-start-codex.sh" }] }
      ],
      "Stop": [
        { "hooks": [{ "type": "command", "command": "sh .claude/hooks/stop-gitsync.sh" }] }
      ]
    }
  }
  ```

  Two Codex-specific differences: **stdout must be JSON** (plain text is discarded as a failed hook), and **project-local hooks run only after you trust them once** — start `codex` in the vault, run `/hooks`, and approve them. The Stop guard from the Claude Code section works as-is (exit code 2 + stderr is the same blocking convention). For SessionStart, wrap the pull-and-inbox output as `hookSpecificOutput.additionalContext` — that's the field that reaches the **model's context**, exactly like plain stdout does in Claude Code (`systemMessage`, by contrast, is only shown to the user in the UI). `jq` handles the escaping:

  ```sh
  #!/bin/sh
  out=$({ git pull --no-edit origin main 2>&1; echo "Inbox:"; ls "01 Inbox"; })
  printf '{"hookSpecificOutput": {"hookEventName": "SessionStart", "additionalContext": %s}}' \
    "$(printf '%s' "$out" | jq -Rs .)"
  ```

- **Quirk:** it's the one agent here configured in TOML rather than JSON, so a config snippet copied from Claude Code or Gemini CLI needs reformatting, not just a key rename.

## Gemini CLI

- **Install:** `npm install -g @google/gemini-cli` (requires Node.js 18+), or run it without installing via `npx`. Docs: <https://geminicli.com/docs/get-started/installation/>.
- **Loads the rulebook via:** the `GEMINI.md` adapter in your vault root, which points to `AGENTS.md`.
- **Plan mode:** enabled by default; `Shift+Tab` cycles approval modes (Default → Auto-Edit → Plan), `/plan [goal]` switches into it directly, or launch straight into it with `gemini --approval-mode=plan`.
- **Model switch:** `-m` / `--model` flag at launch, or `/model` to change models interactively mid-session.
- **MCP config:** `~/.gemini/settings.json`, servers nested under `mcpServers`.
- **Quirk:** Gemini CLI's plan mode is stricter than the others' — it's genuinely read-only (write tools are blocked) until you explicitly approve the plan and switch to an editing mode, rather than just asking before each edit.

## OpenCode

- **Install:** `curl -fsSL https://opencode.ai/install | bash`, or `npm install -g opencode-ai` (requires Node.js). Docs: <https://opencode.ai/docs/>.
- **Loads the rulebook via:** `AGENTS.md` directly — OpenCode searches upward from the current directory for a local `AGENTS.md` (or `CLAUDE.md`, used only when no `AGENTS.md` exists) and a global `~/.config/opencode/AGENTS.md`, falling back further to `~/.claude/CLAUDE.md` for Claude Code compatibility.
- **Plan mode:** press `Tab` to toggle between Build and Plan mode — the indicator sits in the lower-right corner of the TUI; Plan mode is read-only until you switch back.
- **Model switch:** `/models` opens the interactive model picker; a default lives in `opencode.json` as `"model": "provider/model-name"`.
- **MCP config:** `opencode.json` (project root) or `~/.config/opencode/opencode.json` (global), servers under the `mcp` key with a `type` of `local` or `remote`.
- **Quirk:** its MCP config differs structurally from the others — the key is `mcp` (not `mcpServers`), each server declares `type: "local"`/`"remote"`, and a local server's `command` is an **array** (`["npx", "-y", "some-mcp"]`), not a command string plus `args`. OpenCode also has a native `skill` tool and a plugin system (`"plugin": [...]` in `opencode.json`).
