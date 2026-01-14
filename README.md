# slack-api-skill

Slack Web API skill for AI coding agents. Provides token-efficient, multi-environment Slack integration.

## Installation

### Amp

```bash
amp skill add --global tsenart/slack-api-skill --name using-slack-api --overwrite
```

### Claude Code

```bash
git clone https://github.com/tsenart/slack-api-skill.git ~/.claude/skills/using-slack-api
```

### Manual

Clone or copy to your skills directory and reference in your agent config.

## Features

- **Multi-environment**: Configure multiple Slack workspaces (work, personal, etc.)
- **Token-efficient output**: Compact `key=value` format designed for LLM context windows
- **Full API coverage**: Messages, channels, users, reactions, DND, files, pins, scheduled messages, and more
- **Progressive disclosure**: Core docs in SKILL.md, detailed references loaded on demand

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

### Creating a Slack App

1. Go to [api.slack.com/apps](https://api.slack.com/apps)
2. Click **Create New App** → **From scratch**
3. Name your app (e.g., "AI Agent") and select your workspace
4. Click **Create App**

### Adding Scopes

1. In your app settings, go to **OAuth & Permissions**
2. Scroll to **Scopes** → **Bot Token Scopes**
3. Add scopes based on what you need:

| Operation | Required Scopes |
|-----------|-----------------|
| Send messages | `chat:write`, `chat:write.public` |
| Read channels | `channels:read`, `groups:read` |
| Read history | `channels:history`, `groups:history` |
| Read users | `users:read`, `users:read.email` |
| Join channels | `channels:join` |
| Reactions | `reactions:write`, `reactions:read` |
| Pins | `pins:write`, `pins:read` |
| Files | `files:write`, `files:read` |
| DMs | `im:write`, `mpim:write` |
| User groups | `usergroups:read` |
| DND | `dnd:read`, `dnd:write` |
| Bookmarks | `bookmarks:write`, `bookmarks:read` |

For user-level operations (status, search), add **User Token Scopes** instead:
- `users.profile:write` — Set status
- `search:read` — Search messages

### Installing and Getting the Token

1. Scroll up to **OAuth Tokens** and click **Install to Workspace**
2. Review permissions and click **Allow**
3. Copy the **Bot User OAuth Token** (starts with `xoxb-`)
4. For user tokens, copy the **User OAuth Token** (starts with `xoxp-`)
5. Add to `~/.slack.conf` under the appropriate environment section

### Reinstalling After Scope Changes

When you add new scopes, you must reinstall the app:
1. Go to **OAuth & Permissions**
2. Click **Reinstall to Workspace**
3. Approve the new permissions

### Token Types

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
