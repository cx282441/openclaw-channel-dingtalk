# DingTalk Channel for OpenClaw

An internal enterprise bot **channel plugin** for DingTalk using **Stream mode** (no public IP required).

## Features

- ✅ **Stream Mode** — WebSocket long-lived connection; no public IP or webhook required
- ✅ **Direct Messages** — chat with the bot 1:1
- ✅ **Group Chats** — @mention the bot in a group
- ✅ **Multiple Message Types** — text, images, audio (with built-in recognition), video, files
- ✅ **Markdown Replies** — rich-text formatted responses
- ✅ **Interactive Cards** — streaming updates, ideal for real-time AI output
- ✅ **End-to-End AI Chat** — integrated with the Clawdbot message processing pipeline

## Installation

### Method A: Install from remote repository (recommended)

Run the OpenClaw plugin install command. OpenClaw will handle download, dependency installation, and registration automatically:

```bash
openclaw plugins install https://github.com/soimy/clawdbot-channel-dingtalk.git
```

### Method B: Install from local source

If you want to customize the plugin, clone the repository first:

```bash
# 1. Clone the repo
git clone https://github.com/soimy/openclaw-channel-dingtalk.git
cd openclaw-channel-dingtalk

# 2. Install dependencies (required)
npm install

# 3. Install in link mode (changes take effect immediately)
openclaw plugins install -l .
```

### Method C: Manual installation

1. Download/copy this directory to `~/.openclaw/extensions/dingtalk`.
2. Ensure it contains `index.ts`, `openclaw.plugin.json`, and `package.json`.
3. Run `openclaw plugins list` and confirm `dingtalk` is listed.

## Update

```bash
openclaw plugins update dingtalk
```

## Configuration

### 1. Create a DingTalk app

1. Visit the [DingTalk Developer Console](https://open-dev.dingtalk.com/)
2. Create an internal enterprise application
3. Add the "Bot" capability
4. Set message receiving mode to **Stream mode**
5. Publish the app

### 2. Configure permissions

On the app's permission page, enable the following:

- ✅ **Card.Instance.Write** — create and deliver card instances
- ✅ **Card.Streaming.Write** — stream updates to cards

Steps:

1. Open the app -> Permissions
2. Search for permissions related to "Card"
3. Enable the two permissions above
4. Save

### 3. Create a card template

Steps:

1. Visit the [DingTalk Card Platform](https://open-dev.dingtalk.com/fe/card)
2. Go to "My Templates"
3. Click "Create Template"
4. Choose **"AI Card"** as the scenario
5. Design the layout, then save and publish
6. Note the content field name defined in the template
7. Copy the template ID (format: `xxxxx-xxxxx-xxxxx.schema`)
8. Set `cardTemplateId` in `openclaw.json`
9. Or set it in OpenClaw Console -> Channels -> DingTalk -> Card Template Id
10. Set `cardTemplateKey` in `openclaw.json`
11. Or set it in OpenClaw Console -> Channels -> DingTalk -> Card Template Key

Notes:

- With DingTalk's official AI Card template, `cardTemplateKey` defaults to `msgContent`.
- If you create a custom template, make sure the template includes the content field you intend to update, and set `cardTemplateKey` to that field name.

Template config example:

```json5
{
  channels: {
    dingtalk: {
      messageType: 'card',
      cardTemplateId: '<your-template-id>',
      cardTemplateKey: '<your-template-content-key>',
    },
  },
}
```

### 4. Get credentials

From the developer console:

- **Client ID** (AppKey)
- **Client Secret** (AppSecret)
- **Robot Code** (same as Client ID)
- **Corp ID** (enterprise ID)
- **Agent ID** (application ID)

### 5. Configure OpenClaw

Add the following under `channels` in `~/.openclaw/openclaw.json`:

```json5
{
  channels: {
    dingtalk: {
      enabled: true,
      clientId: 'dingxxxxxx',
      clientSecret: 'your-app-secret',
      robotCode: 'dingxxxxxx',
      corpId: 'dingxxxxxx',
      agentId: '123456789',
      dmPolicy: 'open',
      groupPolicy: 'open',
      messageType: 'markdown',
      debug: false,
    },
  },
}
```

### 6. Restart Gateway

```bash
openclaw gateway restart
```

## Configuration Options

| Option                  | Type     | Default      | Description                                       |
| ----------------------- | -------- | ------------ | ------------------------------------------------- |
| `enabled`               | boolean  | `true`       | Enable/disable                                    |
| `clientId`              | string   | required     | App AppKey                                        |
| `clientSecret`          | string   | required     | App AppSecret                                     |
| `robotCode`             | string   | -            | Robot code (media download and cards)             |
| `corpId`                | string   | -            | Corp ID                                           |
| `agentId`               | string   | -            | Agent/App ID                                      |
| `dmPolicy`              | string   | `"open"`     | DM policy: open/pairing/allowlist                 |
| `groupPolicy`           | string   | `"open"`     | Group policy: open/allowlist                      |
| `allowFrom`             | string[] | `[]`         | Allowed sender ID list                            |
| `messageType`           | string   | `"markdown"` | Message type: markdown/card                       |
| `cardTemplateId`        | string   | -            | AI card template ID (messageType=card only)       |
| `cardTemplateKey`       | string   | `"content"`  | Card template content key (messageType=card only) |
| `debug`                 | boolean  | `false`      | Enable debug logs                                 |
| `maxConnectionAttempts` | number   | `10`         | Max connection attempts                           |
| `initialReconnectDelay` | number   | `1000`       | Initial reconnect delay (ms)                      |
| `maxReconnectDelay`     | number   | `60000`      | Max reconnect delay (ms)                          |
| `reconnectJitter`       | number   | `0.3`        | Reconnect jitter factor (0-1)                     |

### Connection robustness

Advanced settings to improve stability:

- **maxConnectionAttempts**: maximum retries after failures; stops and alerts after exceeding.
- **initialReconnectDelay**: initial delay for the first reconnect (ms); subsequent reconnects grow exponentially.
- **maxReconnectDelay**: reconnect delay cap (ms) to prevent very long waits.
- **reconnectJitter**: adds randomness to the delay (+/- 30% by default) to avoid thundering herds.

Reconnect delay formula: `delay = min(initialDelay * 2^attempt, maxDelay) * (1 +/- jitter)`

Example delays (defaults): ~1s, ~2s, ~4s, ~8s, ~16s, ~32s, ~60s (capped)

See [CONNECTION_ROBUSTNESS.md](./CONNECTION_ROBUSTNESS.md) for details.

## Security Policies

### Direct message policy (dmPolicy)

- `open` - anyone can DM the bot
- `pairing` - new users must verify via a pairing code
- `allowlist` - only users in `allowFrom` can use it

### Group policy (groupPolicy)

- `open` - any group can @mention the bot
- `allowlist` - only configured groups can use it

## Supported Message Types

### Receiving

| Type      | Supported | Notes                            |
| --------- | --------- | -------------------------------- |
| Text      | ✅        | Full support                     |
| Rich text | ✅        | Extracts text content            |
| Image     | ✅        | Downloads and passes to AI       |
| Audio     | ✅        | Uses DingTalk speech recognition |
| Video     | ✅        | Downloads and passes to AI       |
| File      | ✅        | Downloads and passes to AI       |

### Sending

| Type             | Supported | Notes                                     |
| ---------------- | --------- | ----------------------------------------- |
| Text             | ✅        | Full support                              |
| Markdown         | ✅        | Auto-detect or manual                     |
| Interactive card | ✅        | Streaming updates for real-time AI output |
| Image            | ⏳        | Requires the media upload API             |

## API Cost Notes

### Text/Markdown mode

| Operation    | API calls | Notes                                                                        |
| ------------ | --------- | ---------------------------------------------------------------------------- |
| Get token    | 1         | Shared/cached (checks expiry once every 60 seconds)                          |
| Send message | 1         | Uses `/v1.0/robot/oToMessages/batchSend` or `/v1.0/robot/groupMessages/send` |
| **Total**    | **2**     | Per reply                                                                    |

### Card (AI interactive card) mode

| Stage              | API calls               | Notes                                            |
| ------------------ | ----------------------- | ------------------------------------------------ |
| **Create card**    | 1                       | `POST /v1.0/card/instances/createAndDeliver`     |
| **Stream updates** | M                       | M = number of chunks; `PUT /v1.0/card/streaming` |
| **Finish card**    | Included in last update | Mark with `isFinalize=true`                      |
| **Total**          | **1 + M**               | M = number of chunks produced by the agent       |

### Typical scenario comparison

| Scenario                | Text/Markdown | Card | Savings |
| ----------------------- | ------------- | ---- | ------- |
| Short reply (1 chunk)   | 2             | 2    | Same    |
| Medium reply (5 chunks) | 6             | 6    | Same    |
| Long reply (10 chunks)  | 12            | 11   | 1 call  |

### Optimization strategies

Ways to reduce API calls:

1. Merge chunks - adjust agent output to reduce chunk count
2. Token caching - tokens are cached automatically (60 seconds)
3. Buffer mode - use `dispatchReplyWithBufferedBlockDispatcher` to merge small chunks

Recommendation:

- Card mode is recommended: better streaming UX, similar or lower cost than Text/Markdown.
- Monitor quotas if you call APIs frequently; check usage in DingTalk developer console.

## Choosing Message Type

The plugin supports two reply types via `messageType`:

### 1. `markdown` (default)

- Rich formatting (headings, bold, lists)
- Auto-detects Markdown
- Suitable for most scenarios

### 2. `card` (AI interactive card)

- True streaming updates (real-time AI output)
- Better visual presentation and interaction
- Renders Markdown
- Template via `cardTemplateId`
- Content field via `cardTemplateKey`
- Best for AI chat scenarios

When `messageType: "card"` is configured:

1. Create + deliver via `/v1.0/card/instances/createAndDeliver`
2. Stream via `/v1.0/card/streaming`
3. Automatic state management (PROCESSING -> INPUTING -> FINISHED)
4. Stable streaming experience without manual throttling

Config example:

```json5
{
  messageType: 'card',
  cardTemplateId: '382e4302-551d-4880-bf29-a30acfab2e71.schema',
  cardTemplateKey: 'msgContent',
}
```

Note: `cardTemplateKey` must match your template field name. `msgContent` works with DingTalk's official AI Card template.

## Usage

After configuration:

1. DM the bot - find the bot and send a message
2. In a group chat - @mention the bot name and send your message

## Troubleshooting

### Not receiving messages

1. Confirm the app is published
2. Confirm message receiving mode is Stream
3. Check Gateway logs: `openclaw logs | grep dingtalk`

### No response in group chats

1. Confirm the bot has been added to the group
2. Confirm you @mentioned the bot correctly (use the bot name)
3. Confirm the group is an internal enterprise group

### Connection failed

1. Check `clientId` and `clientSecret`
2. Confirm the network can access the DingTalk API

## Development

### First-time setup

1. Clone the repo and install dependencies

```bash
git clone https://github.com/soimy/openclaw-channel-dingtalk.git
cd openclaw-channel-dingtalk
npm install
```

2. Verify your dev environment

```bash
npm run type-check
npm run lint
```

### Common commands

| Command              | Description           |
| -------------------- | --------------------- |
| `npm run type-check` | TypeScript type check |
| `npm run lint`       | ESLint checks         |
| `npm run lint:fix`   | Auto-fix formatting   |

### Project structure

```
src/
  channel.ts           - plugin definition and helpers
  runtime.ts           - runtime management
  types.ts             - type definitions

index.ts              - plugin registration
utils.ts              - utility functions

openclaw.plugin.json  - plugin manifest
package.json          - project configuration
README.md             - this file
```

### Types

Core types live in `src/types.ts`, including:

```ts
// Config
DingTalkConfig;
DingTalkChannelConfig;

// Message processing
DingTalkInboundMessage;
MessageContent;
HandleDingTalkMessageParams;

// AI interactive cards
AICardInstance;
AICardCreateAndDeliverRequest;
AICardStreamingRequest;
AICardStatus;

// Utilities
Logger;
RetryOptions;
MediaFile;
```

### Public API

The plugin exports low-level APIs for custom integrations:

```ts
// Text/Markdown
sendBySession(config, sessionWebhook, text, options);

// AI interactive cards
createAICard(config, conversationId, data, log);
streamAICard(card, content, finished, log);
finishAICard(card, content, log);

// Auto selection
sendMessage(config, conversationId, text, options);

// Auth
getAccessToken(config, log);
```

Example:

```ts
import { createAICard, streamAICard, finishAICard } from './src/channel';

const card = await createAICard(config, conversationId, messageData, log);

for (const chunk of aiResponseChunks) {
  await streamAICard(card, currentText + chunk, false, log);
}

await finishAICard(card, finalText, log);
```

## License

MIT
