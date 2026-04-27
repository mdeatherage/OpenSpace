# OpenSpace Session Context
<!-- Last updated: 2026-04-27 -->

## Installation summary

OpenSpace (https://github.com/HKUDS/OpenSpace) is installed and fully operational.

- **Repo**: sparse-cloned to `~/opt/openspace` (assets/ excluded)
- **Venv**: `~/opt/openspace/.venv` (Python 3.14 / Homebrew)
- **Package**: `pip install -e .[macos]` (editable, macOS extras)
- **Binaries**: `openspace`, `openspace-mcp` in `.venv/bin/`

## Warp MCP integration

OpenSpace is registered as a Warp MCP server (added manually via Settings > MCP Servers).
The `~/.warp/.mcp.json` file exists but is superseded by the manually-added entry.

MCP server config (env vars hardcoded due to macOS GUI app env inheritance):
- `ANTHROPIC_API_KEY` — LLM key (hardcoded in Warp MCP settings)
- `OPENAI_API_KEY` — embedding key (hardcoded in Warp MCP settings)
- `OPENSPACE_API_KEY` — cloud skill key (hardcoded in Warp MCP settings)
- `OPENSPACE_MODEL` — `anthropic/claude-sonnet-4-5`
- `OPENSPACE_HOST_SKILL_DIRS` — `/Users/mattd/.warp/skills`
- `OPENSPACE_WORKSPACE` — `/Users/mattd/opt/openspace`

Note: `execute_task` via MCP is cancelled by Warp before completion (Warp timeout).
Use the CLI instead (see below).

## CLI usage

All API keys and the `openspace` alias are defined in `~/.airc` (sourced by `~/.zshrc`):

```zsh
# Run a task
openspace -q "your task here"

# Interactive mode
openspace -i

# With explicit model override
openspace --model anthropic/claude-opus-4-6 -q "your task here"
```

## Skills installed

Global skills (`~/.agents/skills/`, symlinked into `~/.warp/skills/`):

| Skill | Source |
|-------|--------|
| `swift-format-style` | github.com/n0an/Swift-FormatStyle-Agent-Skill |
| `swift-testing-pro` | github.com/twostraws/swift-testing-agent-skill |

Warp host skills (`~/.warp/skills/`):
- `openspace-skill-discovery` — teaches Oz to search OpenSpace's skill registry
- `openspace-delegate-task` — teaches Oz to delegate tasks to OpenSpace
- `apple-diagnostics` — forensic macOS/Apple device diagnosis
- `visual-explainer` — generates HTML visual explanations

Auto-evolved skills (created by OpenSpace during tasks):
- `swift-format-style-enhanced`
- `swift-format-style-verified-enhanced`

## Environment variables (`~/.airc`)

```zsh
export ANTHROPIC_API_KEY="..."
export OPENAI_API_KEY="..."           # enables embedding-based skill ranking
export OPENSPACE_API_KEY="..."        # open-space.cloud community skills
export OPENSPACE_MODEL="anthropic/claude-sonnet-4-5"
export OPENSPACE_HOST_SKILL_DIRS="/Users/mattd/.warp/skills"
export OPENSPACE_WORKSPACE="/Users/mattd/opt/openspace"
alias openspace='/Users/mattd/opt/openspace/.venv/bin/openspace --no-ui'
```

## Updating OpenSpace

```zsh
cd ~/opt/openspace && git pull && .venv/bin/pip install -e .[macos]
```

## Adding skills via skills CLI

```zsh
# Install globally
npx skills add <github-url> --skill <name> --global --yes

# Symlink into ~/.warp/skills so OpenSpace finds them
ln -s ~/.agents/skills/<name> ~/.warp/skills/<name>

# Update all global skills
npx skills update -g
```
