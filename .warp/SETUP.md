# OpenSpace + Warp Setup Guide

This document records how OpenSpace was installed and configured for use with Warp/Oz on this machine.

## Prerequisites

- macOS 13+
- Python 3.12+ (this install uses Python 3.14 via Homebrew/zerobrew)
- Node.js (for `npx skills`)
- Warp (with MCP support)

## 1. Clone the repo

Sparse-clone to skip the large `assets/` directory (~50MB):

```zsh
git clone --filter=blob:none --sparse ~/opt/openspace https://github.com/HKUDS/OpenSpace
cd ~/opt/openspace
git sparse-checkout init --no-cone
git sparse-checkout set '/*' '!assets'
```

## 2. Create a venv and install

```zsh
python3 -m venv ~/opt/openspace/.venv
~/opt/openspace/.venv/bin/pip install -e '.[macos]'
```

Verify:

```zsh
~/opt/openspace/.venv/bin/openspace-mcp --help
~/opt/openspace/.venv/bin/openspace --help
```

## 3. Configure environment variables

Create `~/.airc` and source it from `~/.zshrc`:

```zsh
# ~/.airc
export ANTHROPIC_API_KEY="sk-ant-..."        # required — powers OpenSpace's LLM
export OPENAI_API_KEY="sk-..."               # optional — enables embedding-based skill ranking
export OPENSPACE_API_KEY="sk-..."            # optional — open-space.cloud community skills

export OPENSPACE_MODEL="anthropic/claude-sonnet-4-5"
export OPENSPACE_HOST_SKILL_DIRS="/Users/mattd/.warp/skills"
export OPENSPACE_WORKSPACE="/Users/mattd/opt/openspace"
alias openspace='/Users/mattd/opt/openspace/.venv/bin/openspace --no-ui'
```

Add to `~/.zshrc`:

```zsh
[[ -f ~/.airc ]] && source ~/.airc
```

## 4. Register as a Warp MCP server

In Warp, go to **Settings > MCP Servers > + Add** and paste:

```json
{
  "mcpServers": {
    "openspace": {
      "command": "/Users/mattd/opt/openspace/.venv/bin/openspace-mcp",
      "args": [],
      "env": {
        "OPENSPACE_HOST_SKILL_DIRS": "/Users/mattd/.warp/skills",
        "OPENSPACE_WORKSPACE": "/Users/mattd/opt/openspace",
        "OPENSPACE_MODEL": "anthropic/claude-sonnet-4-5",
        "ANTHROPIC_API_KEY": "<your-key>",
        "OPENAI_API_KEY": "<your-key>",
        "OPENSPACE_API_KEY": "<your-key>"
      }
    }
  }
}
```

> **Note**: Hardcode key values directly (do not use `${VAR}` syntax) — macOS GUI apps
> do not inherit shell environment variables from `~/.zshrc`.

> **Note**: Warp cancels long-running MCP tool calls before `execute_task` completes.
> Use the CLI (`openspace -q "..."`) for tasks instead.

## 5. Install host skills (teaches Warp/Oz how to use OpenSpace)

The host skills are stored in `~/.warp/skills/`. They were fetched from the OpenSpace
repo and are already in place on this machine:

- `openspace-skill-discovery/SKILL.md`
- `openspace-delegate-task/SKILL.md`

## 6. Install Swift skills via skills CLI

```zsh
# swift-format-style
npx skills add https://github.com/n0an/Swift-FormatStyle-Agent-Skill --skill swift-format-style --global --yes
ln -s ~/.agents/skills/swift-format-style ~/.warp/skills/swift-format-style

# swift-testing-pro
npx skills add https://github.com/twostraws/swift-testing-agent-skill --skill swift-testing-pro --global --yes
ln -s ~/.agents/skills/swift-testing-pro ~/.warp/skills/swift-testing-pro
```

## Using OpenSpace

```zsh
# One-shot task (CLI)
openspace -q "Refactor Foo.swift to use modern async/await"

# Interactive REPL
openspace -i

# Verify skills available
npx skills ls -g
```

## Updating

```zsh
# Update OpenSpace itself
cd ~/opt/openspace && git pull && .venv/bin/pip install -e .[macos]

# Update all globally installed skills
npx skills update -g
```
