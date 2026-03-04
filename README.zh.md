# nanoclaw-add-feishu-skill

> [English Documentation](README.md)

为 [NanoClaw](https://github.com/qwibitai/nanoclaw)（你的个人 Claude AI 助手）添加[飞书 (Feishu / Lark)](https://www.feishu.cn) 消息频道。

使用 **WebSocket 长连接** — 无需公网 URL、无需 ngrok、无需配置 Webhook。可与 Telegram、WhatsApp、Slack 同时运行，也可单独作为唯一频道使用。

## 前置条件

- 已安装并运行 [NanoClaw](https://github.com/qwibitai/nanoclaw)
- 拥有飞书或 [Lark](https://www.larksuite.com) 账号，且有权限创建应用

## 安装

```bash
npx skills add will-17173/nanoclaw-add-feishu-skill
```

Claude Code 会全程交互式引导你完成配置。

## 技能做了什么

### 代码变更（自动应用）

| 文件 | 操作 |
|------|------|
| `src/channels/feishu.ts` | 新增 `FeishuChannel` 类（含自动注册） |
| `src/channels/feishu.test.ts` | 新增单元测试 |
| `src/channels/index.ts` | 追加 `import './feishu.js'` |
| `package.json` | 安装 `@larksuiteoapi/node-sdk` |
| `.env.example` | 添加 `FEISHU_APP_ID` 和 `FEISHU_APP_SECRET` |

### 交互式配置（由 Claude 引导）

1. **创建飞书应用**（或使用已有应用），获取 App ID 和 App Secret
2. **配置 `.env`** 填入凭证
3. **注册聊天**——单聊或群聊均可
4. **验证**机器人是否正常响应

## 手动配置

如果不想使用技能安装器，也可以手动配置：

### 1. 创建飞书应用

前往[飞书开放平台](https://open.feishu.cn)（国际版前往 [Lark Open Platform](https://open.larksuite.com)）：

1. 点击 **创建应用** → **自建应用**
2. 进入 **凭证与基础信息**，复制 **App ID** 和 **App Secret**
3. 进入 **事件订阅** → **添加事件** → 搜索并添加 `im.message.receive_v1`
4. 进入 **权限管理** → 添加以下权限：
   - `im:message`
   - `im:message:send_as_bot`
   - `im:chat`
   - `im:chat.members:read`
5. 进入 **机器人** 选项卡 → 开启机器人功能
6. 点击 **发布** / **申请发布**

> **企业版注意：** 企业自建应用发布需要管理员审批后才能生效。

### 2. 配置环境变量

在 NanoClaw 的 `.env` 文件中添加：

```bash
FEISHU_APP_ID=cli_xxxxxxxxxxxxxxxxxx
FEISHU_APP_SECRET=your_app_secret_here
```

同步到容器环境：

```bash
mkdir -p data/env && cp .env data/env/env
```

### 3. 构建并重启

```bash
npm run build

# macOS
launchctl kickstart -k gui/$(id -u)/com.nanoclaw

# Linux
systemctl --user restart nanoclaw
```

### 4. 注册聊天

在飞书中向机器人发送任意消息，然后在日志中找到聊天 JID：

```bash
tail -f logs/nanoclaw.log | grep "unregistered Feishu"
```

或直接查询数据库：

```bash
sqlite3 store/messages.db "SELECT DISTINCT chat_jid FROM chats WHERE channel = 'feishu'"
```

然后在 NanoClaw 项目中通过 Claude Code 注册该聊天。

## JID 格式

| 聊天类型 | 格式 | 示例 |
|---------|------|------|
| 群聊 | `feishu:oc_<id>` | `feishu:oc_4e359893776d45f7cd05d40e3ee10f55` |
| 单聊 | `feishu:<open_id>` | `feishu:ou_7a66d6bd1baa3e6e3d7b3df9a8c90000` |

## 常见问题

**机器人收不到消息**
- 确认 **事件订阅** 中已添加 `im.message.receive_v1`
- 确认应用已发布（企业版需管理员审批）
- 群聊场景：确认机器人已被添加为群成员
- 检查 `.env` 中的凭证是否已同步到 `data/env/env`

**机器人在群里不响应**

NanoClaw 在群聊中默认只在以下情况响应：
- 消息中 @了机器人
- 消息包含触发关键词
- 消息以 `?` 或 `？` 结尾

如需响应所有消息，注册群聊时设置 `requiresTrigger: false`。

**日志中出现 "Message from unregistered Feishu chat"**

这是正常现象，说明机器人已收到消息，但该聊天尚未注册。按照第 4 步完成注册即可。

## 卸载

```bash
# 删除源文件
rm src/channels/feishu.ts src/channels/feishu.test.ts

# 从 src/channels/index.ts 中删除这一行：
# import './feishu.js';

# 从 .env 中删除：
# FEISHU_APP_ID 和 FEISHU_APP_SECRET

# 从数据库中删除已注册的飞书聊天
sqlite3 store/messages.db "DELETE FROM registered_groups WHERE jid LIKE 'feishu:%'"

# 卸载 SDK
npm uninstall @larksuiteoapi/node-sdk

# 重新构建
npm run build
```

## License

MIT
