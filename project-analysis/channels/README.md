# OpenClaw 通道系统详解

## 📡 什么是通道？

**通道（Channel）** 是 OpenClaw 与外部通信平台之间的桥梁。它让 AI 助手能够通过不同的平台与用户交流。

## 🎯 支持的通道列表

### 即时通讯类
- **WhatsApp** - 全球最流行的 IM
- **Telegram** - 开源、安全的 IM
- **Signal** - 端到端加密的 IM

### 团队协作类
- **Slack** - 企业协作平台
- **Discord** - 游戏/社区聊天
- **Microsoft Teams** - 微软办公协作
- **Mattermost** - 开源团队协作
- **Feishu（飞书）** - 字节办公协作

### 社交平台类
- **LINE** - 亚洲流行 IM
- **Zalo** - 越南流行 IM
- **Nostr** - 去中心化社交协议
- **Twitch** - 游戏直播平台

### 其他
- **IRC** - 老牌聊天协议
- **Google Chat** - 谷歌办公
- **WebChat** - 网页聊天界面
- **BlueBubbles** - iMessage 桥接
- **iMessage** - 苹果消息

## 🔧 通道架构

### 通道接口定义

```typescript
// src/channels/plugins/types.core.ts
interface ChannelPlugin {
  // 基本属性
  readonly id: string;
  readonly name: string;
  readonly description?: string;
  
  // 生命周期
  initialize(config: any): Promise<void>;
  shutdown(): Promise<void>;
  
  // 消息处理
  sendMessage(target: string, message: Message): Promise<void>;
  onMessage(callback: (msg: Message) => void): void;
  
  // 状态管理
  getStatus(): ChannelStatus;
  login(credentials: Credentials): Promise<void>;
  logout(): Promise<void>;
}
```

### 消息类型

```typescript
interface Message {
  id: string;              // 消息唯一 ID
  from: string;            // 发送者
  to: string;              // 接收者
  content: string;         // 消息内容
  timestamp: Date;         // 时间戳
  type: 'text' | 'image' | 'audio' | 'video';
  metadata?: any;          // 额外元数据
}
```

## 📝 通道配置

### 基础配置示例

```json5
// ~/.openclaw/openclaw.json
{
  channels: {
    // Telegram 配置
    telegram: {
      botToken: "123456:ABCDEF",  // Bot Token
      allowFrom: ["*"],           // 允许的用户
      groups: {                   // 群组配置
        "*": {
          requireMention: true    // 需要 @ 机器人
        }
      }
    },
    
    // WhatsApp 配置
    whatsapp: {
      allowFrom: ["+1234567890"], // 允许的号码
      groups: ["group-id"]        // 允许的群组
    },
    
    // Slack 配置
    slack: {
      botToken: "xoxb-...",      // Bot Token
      appToken: "xapp-..."       // App Token
    }
  }
}
```

## 💻 实现示例

### Telegram 通道实现

```typescript
// extensions/telegram/src/index.ts
import { Telegraf } from 'grammY';
import type { ChannelPlugin } from '@openclaw/plugin-sdk';

export class TelegramChannel implements ChannelPlugin {
  id = 'telegram';
  name = 'Telegram';
  
  private bot: Telegraf;
  private messageCallback?: (msg: Message) => void;
  
  constructor(private config: any) {
    this.bot = new Telegraf(config.botToken);
  }
  
  async initialize() {
    // 启动 Bot
    await this.bot.launch();
    console.log('Telegram bot started');
    
    // 监听消息
    this.bot.on('message', async (ctx) => {
      const message: Message = {
        id: ctx.message.message_id.toString(),
        from: ctx.message.from.id.toString(),
        to: ctx.chat?.id.toString() || '',
        content: ctx.message.text || '',
        timestamp: new Date(ctx.message.date * 1000),
        type: 'text'
      };
      
      if (this.messageCallback) {
        this.messageCallback(message);
      }
    });
  }
  
  async sendMessage(target: string, message: Message) {
    await this.bot.telegram.sendMessage(target, message.content);
  }
  
  onMessage(callback: (msg: Message) => void) {
    this.messageCallback = callback;
  }
  
  getStatus() {
    return {
      status: this.bot.isRunning() ? 'connected' : 'disconnected'
    };
  }
  
  async login(credentials: Credentials) {
    // Telegram Bot 不需要额外登录
  }
  
  async logout() {
    await this.bot.stop();
  }
  
  async shutdown() {
    await this.logout();
  }
}
```

### WhatsApp 通道实现

```typescript
// extensions/whatsapp/src/index.ts
import makeWASocket from '@whiskeysockets/baileys';
import type { ChannelPlugin } from '@openclaw/plugin-sdk';

export class WhatsAppChannel implements ChannelPlugin {
  id = 'whatsapp';
  name = 'WhatsApp';
  
  private socket: any;
  private messageCallback?: (msg: Message) => void;
  
  async initialize() {
    // 创建 Socket 连接
    this.socket = makeWASocket({
      printQRInTerminal: true  // 在终端显示二维码
    });
    
    // 监听消息
    this.socket.ev.on('messages.upsert', async ({ messages }) => {
      for (const msg of messages) {
        const message: Message = {
          id: msg.key.id!,
          from: msg.key.remoteJid!,
          to: 'me',
          content: msg.message?.conversation || '',
          timestamp: new Date(msg.messageTimestamp! * 1000),
          type: 'text'
        };
        
        if (this.messageCallback) {
          this.messageCallback(message);
        }
      }
    });
  }
  
  async sendMessage(target: string, message: Message) {
    await this.socket.sendMessage(target, {
      text: message.content
    });
  }
  
  onMessage(callback: (msg: Message) => void) {
    this.messageCallback = callback;
  }
  
  getStatus() {
    return {
      status: this.socket.authState.creds.me ? 'connected' : 'disconnected'
    };
  }
  
  async login() {
    // WhatsApp 通过扫码登录
    console.log('Scan QR code to login');
  }
  
  async shutdown() {
    this.socket.end();
  }
}
```

## 🔄 消息流转过程

### 接收消息流程

```
1. 用户在 WhatsApp 发送消息
   ↓
2. WhatsApp 服务器推送消息到 Baileys Socket
   ↓
3. WhatsApp 通道监听到消息
   ↓
4. 转换为标准 Message 格式
   ↓
5. 发送到 Gateway
   ↓
6. Gateway 路由到对应 Session
   ↓
7. Agent 处理消息
   ↓
8. 生成回复
```

### 发送消息流程

```
1. Agent 生成回复
   ↓
2. Gateway 接收回复
   ↓
3. 查找目标通道
   ↓
4. 调用通道的 sendMessage 方法
   ↓
5. 通道格式化消息
   ↓
6. 发送到外部平台（WhatsApp/Telegram 等）
   ↓
7. 用户收到消息
```

## 🛠️ 开发新通道

### 步骤 1: 创建插件结构

```bash
my-channel/
├── package.json
├── openclaw.plugin.json
├── src/
│   ├── index.ts        # 主入口
│   ├── api.ts          # 公开 API
│   └── types.ts        # 类型定义
└── README.md
```

### 步骤 2: 实现通道接口

```typescript
// src/index.ts
import type { ChannelPlugin } from '@openclaw/plugin-sdk';

export default class MyChannel implements ChannelPlugin {
  id = 'my-channel';
  name = 'My Channel';
  
  async initialize(config: any) {
    // 初始化逻辑
  }
  
  async sendMessage(target: string, message: Message) {
    // 发送逻辑
  }
  
  onMessage(callback: (msg: Message) => void) {
    // 注册消息监听
  }
  
  // ... 其他方法
}
```

### 步骤 3: 配置插件元信息

```json
// openclaw.plugin.json
{
  "id": "my-channel",
  "name": "My Channel Plugin",
  "version": "1.0.0",
  "description": "A new channel plugin",
  "main": "./dist/index.js",
  "dependencies": {
    "@openclaw/plugin-sdk": "workspace:*"
  }
}
```

### 步骤 4: 安装和测试

```bash
# 安装插件
openclaw plugins install ./my-channel

# 启用通道
openclaw channels enable my-channel

# 查看状态
openclaw channels status my-channel
```

## 🔍 调试技巧

### 1. 启用详细日志

```json
{
  "logging": {
    "level": "debug",
    "modules": ["channel:*"]
  }
}
```

### 2. 使用 CLI 工具

```bash
# 查看通道状态
openclaw channels status

# 测试发送
openclaw message send --to +1234567890 --message "Test"

# 查看日志
openclaw logs --follow
```

### 3. 检查连接

```typescript
// 健康检查
const status = channel.getStatus();
console.log('Channel status:', status);
```

## ⚠️ 常见问题

### 问题 1: 通道无法连接

**原因**:
- 凭证错误
- 网络问题
- API 限制

**解决**:
```bash
# 重新登录
openclaw channels login --channel telegram

# 检查配置
openclaw config get channels.telegram
```

### 问题 2: 消息发送失败

**排查步骤**:
1. 检查通道状态
2. 验证目标地址
3. 查看错误日志
4. 测试网络连接

### 问题 3: 收不到消息

**可能原因**:
- Webhook 未正确配置
- 权限不足
- 消息过滤规则

**解决方法**:
```json
{
  "channels": {
    "telegram": {
      "allowFrom": ["*"],  // 允许所有用户
      "webhookUrl": "https://your-domain.com/webhook"
    }
  }
}
```

## 📊 性能优化

### 1. 连接池

复用连接，避免频繁创建销毁：

```typescript
class ConnectionPool {
  private connections: Map<string, Connection> = new Map();
  
  getConnection(channelId: string): Connection {
    if (!this.connections.has(channelId)) {
      this.connections.set(channelId, new Connection());
    }
    return this.connections.get(channelId)!;
  }
}
```

### 2. 消息队列

控制发送速率：

```typescript
class MessageQueue {
  private queue: Message[] = [];
  private rateLimit = 10; // 每秒 10 条
  
  async process() {
    while (this.queue.length > 0) {
      const batch = this.queue.splice(0, this.rateLimit);
      await Promise.all(batch.map(msg => this.send(msg)));
      await sleep(1000);
    }
  }
}
```

### 3. 错误重试

```typescript
async function sendWithRetry(
  channel: Channel,
  target: string,
  message: Message,
  maxRetries = 3
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      await channel.sendMessage(target, message);
      return;
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await sleep(1000 * (i + 1)); // 指数退避
    }
  }
}
```

---

**总结**: 通道系统是 OpenClaw 与外部世界沟通的桥梁。理解通道的工作原理对于开发自定义集成至关重要。

*最后更新：2026 年 4 月 2 日*
