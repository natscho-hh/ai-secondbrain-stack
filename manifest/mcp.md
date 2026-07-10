# MCP Manifest

The Model Context Protocol (MCP) is an open standard that lets an AI agent call out to external tools and services — web scraping, live documentation lookup, databases, and more — through a small local or remote server process. API keys and other credentials for these servers stay in your agent's own config file (or your shell environment) and are never written into the vault or committed to git.

None of these servers are installed by the template itself. Phase 5 of `SETUP.md` reads this file, offers matching servers based on what the user said they want to do in Phase 2, and configures a server only after the user opts in — and only for the agent(s) they actually use.

## Config file locations

| Agent | Config file path |
|---|---|
| Claude Code | `.mcp.json` in the project root (project-scoped, shareable via git) — or `~/.claude.json` (user-scoped, applies across all projects; managed via `claude mcp add --scope user`, since this file also holds other Claude Code state) |
| Codex CLI | `~/.codex/config.toml` |
| Gemini CLI | `~/.gemini/settings.json` |
| OpenCode | `opencode.json` in the project root (project-scoped) or `~/.config/opencode/opencode.json` (global) |

Note the key-name trap: Claude Code and Gemini CLI nest servers under `mcpServers`, Codex uses `mcp_servers` in TOML, and OpenCode uses a top-level `mcp` key with a different per-server shape (`type: "local"`/`"remote"`, and `command` as an array instead of command + `args`) — copying a snippet between agents without adjusting key names and structure is the most common mistake.

## firecrawl

Web scraping, crawling, and search — turns a URL or a search query into clean Markdown/structured data. Requires a `FIRECRAWL_API_KEY` from [firecrawl.dev](https://www.firecrawl.dev/app/api-keys). Package: `firecrawl-mcp` on npm.

**Claude Code** (`.mcp.json`, project-scoped):
```json
{
  "mcpServers": {
    "firecrawl": {
      "command": "npx",
      "args": ["-y", "firecrawl-mcp"],
      "env": {
        "FIRECRAWL_API_KEY": "YOUR_API_KEY"
      }
    }
  }
}
```

**Codex CLI** (`~/.codex/config.toml`):
```toml
[mcp_servers.firecrawl]
command = "npx"
args = ["-y", "firecrawl-mcp"]

[mcp_servers.firecrawl.env]
FIRECRAWL_API_KEY = "YOUR_API_KEY"
```

**Gemini CLI** (`~/.gemini/settings.json`):
```json
{
  "mcpServers": {
    "firecrawl": {
      "command": "npx",
      "args": ["-y", "firecrawl-mcp"],
      "env": {
        "FIRECRAWL_API_KEY": "YOUR_API_KEY"
      }
    }
  }
}
```

**OpenCode** (`opencode.json` — note the `mcp` key and the `command` array):
```json
{
  "mcp": {
    "firecrawl": {
      "type": "local",
      "command": ["npx", "-y", "firecrawl-mcp"],
      "enabled": true,
      "environment": {
        "FIRECRAWL_API_KEY": "YOUR_API_KEY"
      }
    }
  }
}
```

## context7

Up-to-date, version-specific documentation for libraries, frameworks, and APIs — fetched live instead of relying on the model's training data. Works without an API key at basic rate limits; an optional key from [context7.com/dashboard](https://context7.com/dashboard) raises the limit. Package: `@upstash/context7-mcp` on npm. (These snippets use the local stdio server; context7 also offers a remote HTTP endpoint at `https://mcp.context7.com/mcp` if you'd rather not run `npx` locally — see [context7.com/docs/resources/all-clients](https://context7.com/docs/resources/all-clients).)

**Claude Code** (`.mcp.json`, project-scoped):
```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp", "--api-key", "YOUR_API_KEY"]
    }
  }
}
```

**Codex CLI** (`~/.codex/config.toml`):
```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp", "--api-key", "YOUR_API_KEY"]
```

**Gemini CLI** (`~/.gemini/settings.json`):
```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp", "--api-key", "YOUR_API_KEY"]
    }
  }
}
```

**OpenCode** (`opencode.json` — note the `mcp` key and the `command` array):
```json
{
  "mcp": {
    "context7": {
      "type": "local",
      "command": ["npx", "-y", "@upstash/context7-mcp", "--api-key", "YOUR_API_KEY"],
      "enabled": true
    }
  }
}
```

If no API key is available or wanted, drop the `"--api-key", "YOUR_API_KEY"` pair from `args` in any snippet above — context7 still works, just at a lower rate limit.

## Adding another server

To add any MCP server not listed here:

1. Find the server's own docs (README, npm page, or vendor site) for its exact `command`, `args`, and required environment variables — don't guess at the values from memory, config syntax is easy to get subtly wrong.
2. Copy the snippet above for the target agent, keeping that agent's key name (`mcpServers` / `mcp_servers` / `mcp`) and structure.
3. Replace the server name, `command`, `args`, and `env` entries with the new server's values.
4. Put any real API key only in the local config file (or a local `.env` it references) — never in a vault note, and never committed to git.

Confirm the file this manifest points to still matches reality before trusting a snippet verbatim: config formats change, and this file was last verified July 2026.
