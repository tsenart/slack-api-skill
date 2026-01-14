# slack-api-skill

An [Agent Skill](https://agentskills.io) for interacting with the Slack Web API. Provides token-efficient, multi-environment Slack integration for AI agents.

## Features

- **Multi-environment**: Configure multiple Slack workspaces (work, personal, etc.)
- **Token-efficient output**: Compact `key=value` format designed for LLM context windows
- **Full API coverage**: Messages, channels, users, reactions, DND, files, and more
- **Progressive disclosure**: Core docs in SKILL.md, detailed references loaded on demand

## Installation

### For Agent Skills-compatible tools

Copy to your skills directory:

```bash
# Global installation
cp -r . ~/.config/agents/skills/using-slack-api

# Or project-specific
cp -r . .agents/skills/using-slack-api
```

### Standalone usage

The `scripts/slack` wrapper works independently:

```bash
./scripts/slack <env> <method> [params...]
```

## Configuration

Create `~/.slack.conf` with your Slack tokens:

```ini
[work]
token = xoxb-your-bot-token

[personal]
token = xoxp-your-user-token
```

Protect it:

```bash
chmod 600 ~/.slack.conf
```

### Getting tokens

1. Go to [api.slack.com/apps](https://api.slack.com/apps)
2. Create or select an app
3. Navigate to **OAuth & Permissions**
4. Add required scopes (see below)
5. Install to workspace and copy the token

### Token types

| Prefix | Type | Use for |
|--------|------|---------|
| `xoxb-` | Bot token | Messages, reactions, most operations |
| `xoxp-` | User token | Status, profile, search, user-scoped operations |

## Usage

```bash
# Verify token
scripts/slack work auth.test

# List channels
scripts/slack work conversations.list types=public_channel

# Send message
scripts/slack work chat.postMessage channel=C1234567 text="Hello!"

# Set status (user token)
scripts/slack personal users.profile.set profile='{"status_text":"Away","status_emoji":":palm_tree:"}'
```

### Output format

Compact, one line per item:

```
# 15 channels (more avail)
C01234567 general
C01234568 random
C01234569 team-backend [priv]
```

### Flags

- `--raw` — Output original JSON
- `--full` — Disable string truncation

## Required scopes

| Operation | Scopes |
|-----------|--------|
| Messages | `chat:write`, `chat:write.public` |
| Channels | `channels:read`, `groups:read` |
| History | `channels:history`, `groups:history` |
| Users | `users:read`, `users:read.email` |
| Join channels | `channels:join` |
| Status | `users.profile:write` (user token) |
| Reactions | `reactions:write` |
| DND | `dnd:read`, `dnd:write` |
| Search | `search:read` (user token) |

## File structure

```
slack-api-skill/
├── SKILL.md              # Agent skill definition
├── README.md             # This file
├── scripts/
│   ├── slack             # Main CLI wrapper
│   └── slack-fmt         # Output formatter
└── references/
    ├── api-methods.md    # Complete API reference
    └── blocks.md         # Block Kit formatting guide
```

## Requirements

- `bash` 4.0+
- `curl`
- `jq`

## License

MIT

## Related

- [Agent Skills specification](https://agentskills.io/specification)
- [Slack Web API documentation](https://api.slack.com/methods)
- [Block Kit Builder](https://app.slack.com/block-kit-builder)
