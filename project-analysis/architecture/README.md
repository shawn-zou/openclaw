# OpenClaw 项目架构分析

## 📐 整体架构图

```
┌─────────────────────────────────────────────────────────────┐
│                    用户交互层 (User Interface)                │
├─────────────────────────────────────────────────────────────┤
│  WhatsApp │ Telegram │ Slack │ Discord │ WebChat │ CLI │... │
└───────────┴──────────┴───────┴─────────┴─────────┴─────┴─────┘
       │          │         │        │         │        │
       └──────────┴─────────┴────────┴─────────┴────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────┐
│              WebSocket Gateway (控制平面 Control Plane)        │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  认证授权 │ 会话管理 │ 事件总线 │ 健康检查 │ 日志审计    │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                                    │
            ┌───────────────────────┼───────────────────────┐
            │                       │                       │
            ▼                       ▼                       ▼
┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│   插件系统        │    │    AI Agent      │    │   通道系统        │
│  (Plugin SDK)    │    │   (Pi Runtime)   │    │  (Channels)      │
│                  │    │                  │    │                  │
│ - 技能包管理      │    │ - 模型调用       │    │ - WhatsApp      │
│ - 工具注册        │    │ - 上下文管理     │    │ - Telegram      │
│ - 配置管理        │    │ - 工具调用       │    │ - Slack         │
│ - 生命周期管理    │    │ - 流式响应       │    │ - Discord       │
└──────────────────┘    └──────────────────┘    └──────────────────┘
            │                       │                       │
            └───────────────────────┼───────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────┐
│                    基础设施层 (Infrastructure)                 │
├─────────────────────────────────────────────────────────────┤
│  文件系统 │ 数据库 │ 网络请求 │ 加密模块 │ 进程管理 │ 定时任务  │
└─────────────────────────────────────────────────────────────┘
```

## 🏗️ 核心组件详解

### 1. Gateway（网关）- 核心控制平面

**位置**: `src/gateway/`

**职责**:
- 维护所有通信通道的连接
- 提供 WebSocket API 给客户端
- 管理会话和状态
- 处理认证和授权
- 事件分发和广播

**关键文件**:
```
src/gateway/
├── server.ts              # WebSocket 服务器主逻辑
├── server-http.ts         # HTTP 服务器（Control UI、WebChat）
├── auth.ts                # 认证逻辑
├── client.ts              # 客户端连接管理
├── session-utils.ts       # 会话工具函数
├── hooks-mapping.ts       # Hook 映射和处理
└── protocol/              # 协议定义
```

**工作流程**:
```typescript
// 简化的网关启动流程
async function startGateway() {
  // 1. 加载配置
  const config = await loadConfig();
  
  // 2. 初始化通道
  await initializeChannels(config.channels);
  
  // 3. 启动 WebSocket 服务器
  const wsServer = await createWebSocketServer({
    port: config.gateway.port,
    bind: config.gateway.bind
  });
  
  // 4. 注册处理方法
  wsServer.on('connection', handleClientConnection);
  wsServer.on('message', handleMessage);
  
  // 5. 启动 HTTP 服务器（可选）
  if (config.controlUI.enabled) {
    await startControlUI(config.controlUI.port);
  }
}
```

### 2. Channel System（通道系统）

**位置**: `src/channels/` + `extensions/`

**支持的通道**:
- **即时通讯**: WhatsApp, Telegram, Signal
- **团队协作**: Slack, Discord, Microsoft Teams, Mattermost
- **社交平台**: LINE, Zalo, Nostr, Twitch
- **其他**: IRC, Google Chat, Feishu, WebChat

**通道接口**:
```typescript
interface Channel {
  // 通道唯一标识
  id: string;
  
  // 登录/认证
  login(credentials: Credentials): Promise<void>;
  
  // 发送消息
  sendMessage(target: string, message: Message): Promise<void>;
  
  // 接收消息
  onMessage(callback: (message: Message) => void): void;
  
  // 断开连接
  disconnect(): Promise<void>;
}
```

**示例 - Telegram 通道**:
```typescript
// extensions/telegram/index.ts
import { Telegraf } from 'grammY';

export class TelegramChannel implements Channel {
  private bot: Telegraf;
  
  constructor(private token: string) {
    this.bot = new Telegraf(token);
  }
  
  async login() {
    await this.bot.launch();
    console.log('Telegram bot started');
  }
  
  async sendMessage(chatId: string, message: Message) {
    await this.bot.telegram.sendMessage(chatId, message.content);
  }
  
  onMessage(callback: (msg: Message) => void) {
    this.bot.on('message', (ctx) => {
      callback({
        from: ctx.message.from.id.toString(),
        content: ctx.message.text,
        timestamp: new Date()
      });
    });
  }
}
```

### 3. Plugin System（插件系统）

**位置**: `src/plugin-sdk/` + `extensions/`

**插件类型**:
1. **通道插件**: 添加新的通信渠道
2. **Provider 插件**: 添加新的 AI 模型提供商
3. **工具插件**: 添加新工具和技能
4. **节点插件**: 添加设备功能

**插件结构**:
```
extension-name/
├── package.json           # 插件元信息
├── openclaw.plugin.json   # OpenClaw 插件配置
├── src/
│   ├── index.ts          # 入口文件
│   ├── api.ts            # 公开 API
│   └── ...               # 实现细节
└── README.md             # 使用说明
```

**插件示例 - OpenAI Provider**:
```typescript
// extensions/openai/src/index.ts
import { ProviderPlugin } from '@openclaw/plugin-sdk';

export default class OpenAIProvider implements ProviderPlugin {
  id = 'openai';
  name = 'OpenAI';
  
  async authenticate(apiKey: string) {
    // 验证 API Key
    const client = new OpenAI({ apiKey });
    await client.models.list();
  }
  
  async chat(messages: Message[], model: string) {
    const client = new OpenAI();
    const response = await client.chat.completions.create({
      model,
      messages
    });
    return response.choices[0].message;
  }
}
```

### 4. AI Agent（智能体）

**位置**: `src/agents/`

**核心功能**:
- 理解用户意图
- 调用工具执行任务
- 管理对话上下文
- 流式输出响应

**Agent Loop**:
```typescript
async function agentLoop(session: Session) {
  while (session.active) {
    // 1. 获取用户输入
    const userInput = await session.getInput();
    
    // 2. 构建提示词（包含历史、工具、上下文）
    const prompt = buildPrompt({
      history: session.history,
      tools: availableTools,
      userMessage: userInput
    });
    
    // 3. 调用 AI 模型
    const response = await callModel(prompt);
    
    // 4. 处理工具调用
    if (response.toolCalls) {
      for (const toolCall of response.toolCalls) {
        const result = await executeTool(toolCall);
        session.addToolResult(result);
      }
    }
    
    // 5. 返回响应给用户
    await session.sendResponse(response.content);
    
    // 6. 更新会话历史
    session.addMessage(response);
  }
}
```

### 5. CLI（命令行界面）

**位置**: `src/cli/`

**主要命令**:
```bash
openclaw gateway          # 启动网关
openclaw agent            # 与 AI 代理交互
openclaw message send     # 发送消息
openclaw channels status  # 查看通道状态
openclaw plugins list     # 列出已安装插件
openclaw config set       # 修改配置
openclaw onboard          # 新手引导
```

**CLI 架构**:
```typescript
// src/cli/program.ts
import { Command } from 'commander';

const program = new Command();

program
  .name('openclaw')
  .description('Personal AI Assistant CLI');

program
  .command('gateway')
  .description('Start the gateway server')
  .option('-p, --port <number>', 'Port number', '18789')
  .action((options) => {
    startGateway(options);
  });

program
  .command('agent')
  .description('Interact with AI agent')
  .option('-m, --message <text>', 'Initial message')
  .action((options) => {
    startAgentSession(options);
  });

program.parse(process.argv);
```

## 🔄 数据流示例

### 场景：用户通过 WhatsApp 发送消息

```
1. WhatsApp 用户发送 "Hello"
   ↓
2. WhatsApp 通道接收到消息
   ↓
3. 通道将消息转发给 Gateway
   ↓
4. Gateway 进行认证和权限检查
   ↓
5. Gateway 查找对应的 Session
   ↓
6. Gateway 将消息传递给 AI Agent
   ↓
7. Agent 处理消息并生成回复
   ↓
8. Agent 可能调用工具（如搜索、计算等）
   ↓
9. Agent 将回复流式传输回 Gateway
   ↓
10. Gateway 通过 WhatsApp 通道发送回复
   ↓
11. 用户收到回复 "Hello! How can I help you?"
```

**代码流程**:
```typescript
// 1. WhatsApp 接收消息
whatsapp.on('message', async (msg) => {
  // 2. 转发到 Gateway
  await gateway.receiveMessage({
    channel: 'whatsapp',
    from: msg.from,
    content: msg.body
  });
});

// 3. Gateway 处理
gateway.on('message', async (event) => {
  // 4. 找到会话
  const session = await sessions.getOrCreate(event.from);
  
  // 5. 添加到历史
  session.addUserMessage(event.content);
  
  // 6. 调用 Agent
  const response = await agent.process(session);
  
  // 7. 发送回复
  await whatsapp.sendMessage(event.from, response);
});
```

## 📦 目录结构详解

```
openclaw/
├── src/                          # 源代码
│   ├── gateway/                  # WebSocket 网关
│   │   ├── server.ts            # WS 服务器
│   │   ├── server-http.ts       # HTTP 服务器
│   │   ├── auth.ts              # 认证
│   │   ├── client.ts            # 客户端管理
│   │   └── protocol/            # 协议定义
│   │
│   ├── agents/                   # AI Agent
│   │   ├── agent-loop.ts        # Agent 循环
│   │   ├── context.ts           # 上下文管理
│   │   └── tools/               # 工具集
│   │
│   ├── cli/                      # 命令行工具
│   │   ├── program.ts           # Commander 配置
│   │   ├── commands/            # 命令实现
│   │   └── argv.ts              # 参数解析
│   │
│   ├── channels/                 # 通道核心
│   │   ├── types.ts             # 类型定义
│   │   └── plugins/             # 通道插件
│   │
│   ├── plugin-sdk/               # 插件 SDK
│   │   ├── core.ts              # 核心接口
│   │   ├── provider-entry.ts    # Provider 入口
│   │   └── channel-contract.ts  # 通道契约
│   │
│   ├── config/                   # 配置系统
│   │   ├── schema.ts            # Schema 定义
│   │   └── loader.ts            # 配置加载
│   │
│   ├── infra/                    # 基础设施
│   │   ├── fs.ts                # 文件系统
│   │   ├── net.ts               # 网络
│   │   └── process.ts           # 进程管理
│   │
│   └── entry.ts                  # 程序入口
│
├── extensions/                   # 插件目录
│   ├── telegram/                 # Telegram 插件
│   ├── whatsapp/                 # WhatsApp 插件
│   ├── openai/                   # OpenAI Provider
│   ├── anthropic/                # Anthropic Provider
│   └── ...                       # 更多插件
│
├── apps/                         # 应用程序
│   ├── macos/                    # macOS 应用
│   ├── ios/                      # iOS 应用
│   └── android/                  # Android 应用
│
├── docs/                         # 文档
│   ├── concepts/                 # 概念说明
│   ├── channels/                 # 通道文档
│   ├── plugins/                  # 插件文档
│   └── install/                  # 安装指南
│
├── test/                         # 测试
│   ├── unit/                     # 单元测试
│   ├── integration/              # 集成测试
│   └── e2e/                      # 端到端测试
│
└── scripts/                      # 构建脚本
    ├── build.ts                  # 构建脚本
    ├── test.ts                   # 测试脚本
    └── release.ts                # 发布脚本
```

## 🔧 关键技术点

### 1. WebSocket 通信协议

**连接建立**:
```typescript
// 客户端连接
const ws = new WebSocket('ws://127.0.0.1:18789');

ws.onopen = () => {
  // 发送握手消息
  ws.send(JSON.stringify({
    type: 'connect',
    params: {
      device: 'my-device',
      platform: 'macos',
      auth: { token: 'xxx' }
    }
  }));
};
```

**消息格式**:
```typescript
// 请求
{
  type: 'req',
  id: 'uuid-123',
  method: 'agent',
  params: { /* ... */ }
}

// 响应
{
  type: 'res',
  id: 'uuid-123',
  ok: true,
  payload: { /* ... */ }
}

// 事件
{
  type: 'event',
  event: 'agent',
  payload: { /* ... */ }
}
```

### 2. 配置系统

**配置加载顺序**:
```typescript
// 1. 默认配置
const defaults = { ... };

// 2. 文件配置 (~/.openclaw/openclaw.json)
const fileConfig = await loadFromFile();

// 3. 环境变量
const envConfig = loadFromEnv();

// 4. CLI 参数
const cliConfig = parseCliArgs();

// 合并配置
const finalConfig = merge(
  defaults,
  fileConfig,
  envConfig,
  cliConfig
);
```

### 3. 错误处理

**统一错误类型**:
```typescript
class OpenClawError extends Error {
  constructor(
    public code: string,
    message: string,
    public details?: any
  ) {
    super(message);
  }
}

// 使用示例
try {
  await channel.sendMessage(to, message);
} catch (error) {
  throw new OpenClawError(
    'CHANNEL_SEND_FAILED',
    'Failed to send message via channel',
    { channel: 'whatsapp', error }
  );
}
```

### 4. 日志系统

**日志级别**:
```typescript
logger.debug('Detailed debugging info');
logger.info('General information');
logger.warn('Warning message');
logger.error('Error occurred');
```

**日志输出**:
```typescript
// 结构化日志
{
  level: 'info',
  timestamp: '2026-04-02T09:00:00Z',
  module: 'gateway',
  message: 'Server started',
  data: { port: 18789 }
}
```

## 🎯 设计模式

### 1. 观察者模式（Observer Pattern）

用于事件系统：
```typescript
class EventEmitter {
  private listeners: Map<string, Function[]> = new Map();
  
  on(event: string, callback: Function) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event)!.push(callback);
  }
  
  emit(event: string, data: any) {
    this.listeners.get(event)?.forEach(cb => cb(data));
  }
}
```

### 2. 工厂模式（Factory Pattern）

用于创建通道实例：
```typescript
class ChannelFactory {
  static create(type: string, config: any): Channel {
    switch (type) {
      case 'telegram':
        return new TelegramChannel(config);
      case 'whatsapp':
        return new WhatsAppChannel(config);
      // ...
      default:
        throw new Error(`Unknown channel: ${type}`);
    }
  }
}
```

### 3. 单例模式（Singleton Pattern）

用于全局服务：
```typescript
class Gateway {
  private static instance: Gateway;
  
  private constructor() {}
  
  static getInstance(): Gateway {
    if (!Gateway.instance) {
      Gateway.instance = new Gateway();
    }
    return Gateway.instance;
  }
}
```

## 📊 性能优化

### 1. 连接池

复用 WebSocket 连接，避免频繁创建销毁。

### 2. 消息队列

使用队列管理消息发送，避免并发过高：
```typescript
class MessageQueue {
  private queue: Message[] = [];
  private processing = false;
  
  async add(message: Message) {
    this.queue.push(message);
    if (!this.processing) {
      this.process();
    }
  }
  
  private async process() {
    this.processing = true;
    while (this.queue.length > 0) {
      const msg = this.queue.shift()!;
      await this.send(msg);
    }
    this.processing = false;
  }
}
```

### 3. 缓存策略

缓存常用数据，减少重复计算：
```typescript
const cache = new Map<string, any>();

async function getConfig(key: string) {
  if (cache.has(key)) {
    return cache.get(key);
  }
  const value = await loadConfig(key);
  cache.set(key, value);
  return value;
}
```

---

**总结**: OpenClaw 采用分层架构，通过 WebSocket 网关作为核心控制平面，连接各种通信通道、AI 模型和客户端应用。插件化设计使其易于扩展，TypeScript 保证了代码质量和可维护性。

*最后更新：2026 年 4 月 2 日*
