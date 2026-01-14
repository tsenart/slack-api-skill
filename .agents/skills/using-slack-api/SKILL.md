---
name: using-slack-api
description: Interacts with Slack workspaces via the Web API. Sends messages, lists channels/users, uploads files, sets status, manages DND. Use when asked to post to Slack, query Slack data, or automate Slack operations.
---

# Using Slack API

Direct Slack API access with multi-environment support.

## Security Rules

**NEVER read, cat, or open `~/.slack.conf`** — it contains secrets. Instead use:
- `scripts/slack-envs` — List configured environments (safe, no secrets)
- `scripts/slack` — Make API calls (reads config internally)

**NEVER expose tokens.** Do not:
- Print, log, or display tokens
- Include tokens in error messages or debug output
- Store tokens in files other than `~/.slack.conf`

## MANDATORY First Step: Discover Environments

**⚠️ ALWAYS run this BEFORE any Slack API call. NEVER assume environment names like "work" or "personal" exist.**

```bash
scripts/slack-envs
```

This lists the actual configured environment names. Use ONLY the names returned by this command. Environment names are user-defined and vary per configuration—do not guess or assume.

## Setup

Create `~/.slack.conf`:
```ini
[work]
token = xoxb-your-bot-token

[personal]
token = xoxp-your-user-token
```

Protect it: `chmod 600 ~/.slack.conf`

Get tokens: https://api.slack.com/apps → OAuth & Permissions

## Quick Start

```bash
scripts/slack work auth.test                               # Verify token
scripts/slack work conversations.list types=public_channel # List channels
scripts/slack work users.list                              # List users
scripts/slack work chat.postMessage channel=C1234 text="Hello"
```

## The `slack` Script

```bash
scripts/slack <env> <method> [key=value...] [--raw|--full]
```

- `<env>` — Environment name from config (e.g., `work`, `personal`)
- `--raw` — Original JSON output
- `--full` — No string truncation

Output is compact `key=value` format, one line per item.

## Common Operations

### Channels
```bash
scripts/slack work conversations.list types=public_channel,private_channel
scripts/slack work conversations.list types=im              # DMs
scripts/slack work conversations.info channel=C1234
scripts/slack work conversations.history channel=C1234 limit=20
scripts/slack work conversations.create name=new-channel is_private=false
```

### Messages
```bash
scripts/slack work chat.postMessage channel=C1234 text="Hello"
scripts/slack work chat.postMessage channel=C1234 text="Reply" thread_ts=1234567890.123
scripts/slack work chat.update channel=C1234 ts=MSG_TS text="Updated"
scripts/slack work chat.delete channel=C1234 ts=MSG_TS
```

### Users
```bash
scripts/slack work users.list
scripts/slack work users.info user=U1234
scripts/slack work users.lookupByEmail email=user@example.com
```

### Status (requires user token xoxp-)
```bash
scripts/slack personal users.profile.set profile='{"status_text":"In meeting","status_emoji":":calendar:"}'
scripts/slack personal users.profile.set profile='{"status_text":"","status_emoji":""}'  # Clear
```

### DND / Snooze
```bash
scripts/slack work dnd.setSnooze num_minutes=60
scripts/slack work dnd.endSnooze
scripts/slack work dnd.info
```

### Reactions
```bash
scripts/slack work reactions.add channel=C1234 timestamp=MSG_TS name=thumbsup
scripts/slack work reactions.remove channel=C1234 timestamp=MSG_TS name=thumbsup
```

### Pins
```bash
scripts/slack work pins.add channel=C1234 timestamp=MSG_TS
scripts/slack work pins.remove channel=C1234 timestamp=MSG_TS
scripts/slack work pins.list channel=C1234
```

### Scheduled Messages
```bash
scripts/slack work chat.scheduleMessage channel=C1234 text="Hello" post_at=UNIX_TS
scripts/slack work chat.scheduledMessages.list channel=C1234
scripts/slack work chat.deleteScheduledMessage channel=C1234 scheduled_message_id=Q1234
```

### Direct Messages
```bash
scripts/slack work conversations.open users=U1234            # Open DM, get channel ID
scripts/slack work conversations.open users=U1234,U5678      # Group DM
scripts/slack work chat.postMessage channel=D1234 text="Hi"  # Send to DM channel
```

### User Groups
```bash
scripts/slack work usergroups.list                           # List @-mention groups
```

### File Upload (3-step)
```bash
# 1. Get upload URL
scripts/slack work files.getUploadURLExternal filename=doc.txt length=1024

# 2. Upload content (use curl)
curl -s -X POST "$UPLOAD_URL" -F "file=@local-file.txt"

# 3. Complete upload and share
scripts/slack work files.completeUploadExternal 'files=[{"id":"F1234","title":"My Doc"}]' channel_id=C1234
```

### Search (user token only)
```bash
scripts/slack personal search.messages query="keyword" count=20
```

## Output Format

Compact, one line per item:
```
# 15 channels (more avail)
C01234567 general
C01234568 random
C01234569 team-backend [priv]
```

```
# message posted
ts=1234567890.123456 channel=C01234567
```

## Token Types

| Prefix | Type | Use for |
|--------|------|---------|
| `xoxb-` | Bot | Messages, reactions, most operations |
| `xoxp-` | User | Status, profile, search, user-scoped ops |

## Required Scopes

| Operation | Scopes |
|-----------|--------|
| Messages | `chat:write` (+`chat:write.public` for any channel) |
| Channels | `channels:read`, `groups:read` |
| History | `channels:history`, `groups:history` |
| Users | `users:read`, `users:read.email` |
| Status | `users.profile:write` (user token) |
| Reactions | `reactions:write` |
| DND | `dnd:write` |
| Pins | `pins:write`, `pins:read` |
| Files | `files:write`, `files:read` |
| DMs | `im:write`, `mpim:write` |
| User Groups | `usergroups:read` |
| Bookmarks | `bookmarks:write` |
| Search | `search:read` (user token) |

## References

- `references/api-methods.md` — Full method reference
- `references/blocks.md` — Block Kit formatting
