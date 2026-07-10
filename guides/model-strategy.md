# Model Strategy

## The core rule

**Plan with the strongest model available, in your agent's plan mode. Build with a cheaper one.**

Planning is where mistakes are expensive to make and cheap to catch — a wrong assumption caught in a reviewable plan costs a few sentences to fix; the same mistake caught after files have already changed costs a revert and a re-do. Use your agent's strongest available model, in plan mode, for that stage. Once you've approved the plan, the actual editing — applying steps you've already reviewed — is mechanical work a cheaper, faster model handles just as well, at a fraction of the cost. This mirrors the "prefer plan mode for big asks" and "use cheaper models for mechanical work" habits in `guides/token-friendly.md`.

## Per-agent reference

| Agent | Plan mode | Model-switch command | Typical pairing |
|---|---|---|---|
| **Claude Code** | Press `Shift+Tab` twice to cycle into plan mode (shown as "⏸ plan mode on"), or use `/plan` (v2.1+) | `/model` (e.g. `/model opus`, `/model sonnet`) — the menu also offers a combined option: "Use Opus in plan mode, Sonnet otherwise" | Opus for planning → Sonnet for building |
| **Codex CLI** | `/plan [goal]` slash command switches the session into plan mode; pair with `--path` or `--model` flags for more control | `/model` | Highest-reasoning Codex model for planning → a faster Codex model for building |
| **Gemini CLI** | `Shift+Tab` cycles approval modes (Default → Auto-Edit → Plan), or `/plan [goal]`, or launch directly into it with `--approval-mode=plan`; Plan Mode is genuinely read-only until you approve | `-m` / `--model` flag at launch, or `/model` interactively | Gemini Pro for planning → Gemini Flash for building |
| **OpenCode** | Press `Tab` to toggle between Build and Plan mode (indicator in the lower-right corner); Plan mode is read-only until you switch back | `/models` interactive picker, or the `"model": "provider/model-name"` default in `opencode.json` | Strongest available model for planning → a faster model for building |

Exact model names (which one is "strongest" or "cheapest") change often as providers ship new versions — check your agent's own model picker or `/model` list for what's current rather than assuming a name from this table is still accurate.

## Automated

You don't have to remember any of this yourself. `AGENTS.md`'s "Model strategy" section makes it part of the vault's rulebook: once a plan is approved, your agent is instructed to propose the switch itself and name the exact command for your specific agent. The habit is built into the vault, not into your memory.
