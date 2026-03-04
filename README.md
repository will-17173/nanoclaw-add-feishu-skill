# nanoclaw-add-feishu-skill

> [中文文档](README.zh.md)

Add [Feishu (飞书 / Lark)](https://www.feishu.cn) as a messaging channel to [NanoClaw](https://github.com/qwibitai/nanoclaw) — your personal Claude AI assistant.

Uses **WebSocket long connection** — no public URL, no ngrok, no webhook setup required. Works alongside Telegram, WhatsApp, Slack, or as a standalone channel.

## Prerequisites

- [NanoClaw](https://github.com/qwibitai/nanoclaw) installed and running
- A Feishu or [Lark](https://www.larksuite.com) account with permission to create apps

## Install

```bash
npx skills add qwibitai/nanoclaw-add-feishu-skill
```

Claude Code will guide you through the entire setup interactively.

## What This Skill Does

### Code changes (applied automatically)

| File | Action |
|------|--------|
| `src/channels/feishu.ts` | Adds `FeishuChannel` class with self-registration |
| `src/channels/feishu.test.ts` | Adds unit tests |
| `src/channels/index.ts` | Appends `import './feishu.js'` to the channel barrel |
| `package.json` | Installs `@larksuiteoapi/node-sdk` |
| `.env.example` | Adds `FEISHU_APP_ID` and `FEISHU_APP_SECRET` |

### Interactive setup (guided by Claude)

1. **Create a Feishu app** (or use an existing one) and collect App ID + App Secret
2. **Configure `.env`** with your credentials
3. **Register your chat** — direct message or group chat
4. **Verify** the bot is responding

## Manual Setup

If you prefer to set up manually rather than using the skill installer:

### 1. Create a Feishu App

Go to [Feishu Open Platform](https://open.feishu.cn) (or [Lark Open Platform](https://open.larksuite.com) for international):

1. Click **Create App** → **Custom App**
2. Go to **Credentials & Basic Info** — copy your **App ID** and **App Secret**
3. Go to **Event Subscriptions** → **Add Events** → add `im.message.receive_v1`
4. Go to **Permissions & Scopes** → add:
   - `im:message`
   - `im:message:send_as_bot`
   - `im:chat`
   - `im:chat.members:read`
5. Go to **Bot** tab → enable the bot feature
6. Click **Publish** / **Apply for Release**

> **Note:** For enterprise accounts, your IT admin may need to approve the app before it goes live.

### 2. Configure Environment

Add to your NanoClaw `.env`:

```bash
FEISHU_APP_ID=cli_xxxxxxxxxxxxxxxxxx
FEISHU_APP_SECRET=your_app_secret_here
```

Sync to the container environment:

```bash
mkdir -p data/env && cp .env data/env/env
```

### 3. Build and Restart

```bash
npm run build

# macOS
launchctl kickstart -k gui/$(id -u)/com.nanoclaw

# Linux
systemctl --user restart nanoclaw
```

### 4. Register a Chat

Send any message to your bot in Feishu, then find the chat JID in the logs:

```bash
tail -f logs/nanoclaw.log | grep "unregistered Feishu"
```

Or query the database directly:

```bash
sqlite3 store/messages.db "SELECT DISTINCT chat_jid FROM chats WHERE channel = 'feishu'"
```

Then register the chat via Claude Code in your NanoClaw project.

## JID Format

| Chat type | Format | Example |
|-----------|--------|---------|
| Group chat | `feishu:oc_<id>` | `feishu:oc_4e359893776d45f7cd05d40e3ee10f55` |
| Direct (p2p) | `feishu:<open_id>` | `feishu:ou_7a66d6bd1baa3e6e3d7b3df9a8c90000` |

## Troubleshooting

**Bot not receiving messages**
- Confirm `im.message.receive_v1` is added under **Event Subscriptions**
- Confirm the app is published (enterprise apps need admin approval)
- For groups: confirm the bot is added as a group member
- Check that `.env` credentials are synced to `data/env/env`

**Bot not responding in groups**

By default NanoClaw responds in groups only when:
- The bot is @mentioned
- The message contains a trigger keyword
- The message ends with `?` or `？`

To respond to all messages, register the group with `requiresTrigger: false`.

**"Message from unregistered Feishu chat" in logs**

This is expected — the bot is receiving messages but the chat hasn't been registered yet. Follow Step 4 above.

## Removal

```bash
# Remove source files
rm src/channels/feishu.ts src/channels/feishu.test.ts

# Remove the import from src/channels/index.ts
# (delete the line: import './feishu.js';)

# Remove credentials from .env
# (delete FEISHU_APP_ID and FEISHU_APP_SECRET)

# Remove registered chats from database
sqlite3 store/messages.db "DELETE FROM registered_groups WHERE jid LIKE 'feishu:%'"

# Uninstall SDK
npm uninstall @larksuiteoapi/node-sdk

# Rebuild
npm run build
```

## License

MIT
