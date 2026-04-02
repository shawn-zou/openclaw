# OpenClaw 核心概念

## 📖 基础概念

### 1. Gateway（网关）

**Gateway** 是 OpenClaw 的核心控制平面，它是一个长期运行的服务进程。

**关键特性**:
- 监听在 `127.0.0.1:18789`（默认）
- 通过 WebSocket 提供 API
- 管理所有通信通道
- 处理认证和授权
- 维护会话状态

**启动方式**:
```bash
# 前台运行
openclaw gateway

# 指定端口
openclaw gateway --port 18789

# 后台运行（守护进程）
openclaw gateway --daemon
```

### 2. Channel（通道）

**Channel** 是 OpenClaw 与外部通信平台的连接器。

**支持的通道类型**:
```
即时通讯：WhatsApp, Telegram, Signal
团队协作：Slack, Discord, Microsoft Teams
社交平台：LINE, Zalo, Nostr
其他：IRC, Google Chat, WebChat
```

**通道配置示例**:
```json
{
  "channels": {
    "telegram": {
      "botToken": "YOUR_BOT_TOKEN"
    },
    "whatsapp": {
      // WhatsApp 使用 Baileys 库，需要扫码登录
    }
  }
}
```

### 3. Session（会话）

**Session** 代表一次完整的对话过程。

**会话属性**:
- `id`: 唯一标识符
- `active`: 是否活跃
- `history`: 对话历史
- `context`: 上下文信息
- `model`: 使用的 AI 模型

**会话生命周期**:
```
创建 → 活跃 → (可能休眠) → 归档/删除
```

### 4. Agent（智能体）

**Agent** 是执行 AI 推理的核心组件。

**Agent 的工作流程**:
```
接收输入 → 理解意图 → 调用工具 → 生成响应 → 输出结果
```

**思考级别**:
- `off`: 不思考，直接回答
- `minimal`: 最少思考
- `low`: 低强度思考
- `medium`: 中等思考（默认）
- `high`: 高强度思考
- `xhigh`: 极高强度思考

### 5. Tool（工具）

**Tool** 是 Agent 可以调用的功能模块。

**内置工具**:
- `browser`: 浏览器控制
- `canvas`: 画布操作
- `location.get`: 获取位置
- `system.run`: 执行系统命令
- `sessions_send`: 发送消息到其他会话

**工具调用示例**:
```typescript
// Agent 决定调用工具
const toolCall = {
  name: 'browser.navigate',
  arguments: {
    url: 'https://example.com'
  }
};

// 执行工具
const result = await executeTool(toolCall);
```

### 6. Plugin（插件）

**Plugin** 是扩展 OpenClaw 功能的模块化组件。

**插件类型**:
1. **通道插件**: 添加新的通信渠道
2. **Provider 插件**: 添加 AI 模型支持
3. **工具插件**: 添加新工具
4. **节点插件**: 添加设备功能

**插件结构**:
```
my-plugin/
├── package.json
├── openclaw.plugin.json
└── src/
    └── index.ts
```

### 7. Node（节点）

**Node** 是连接到 Gateway 的设备端点。

**节点类型**:
- **macOS**: 桌面应用
- **iOS**: 移动应用
- **Android**: 移动应用
- **Headless**: 无界面设备

**节点能力**:
```typescript
{
  commands: [
    'canvas.*',      // 画布操作
    'camera.*',      // 相机控制
    'screen.record', // 屏幕录制
    'location.get',  // 获取位置
    'system.run'     // 执行命令
  ]
}
```

### 8. Provider（模型提供商）

**Provider** 提供 AI 模型访问服务。

**支持的 Provider**:
```
- OpenAI (GPT 系列)
- Anthropic (Claude 系列)
- Google (Gemini 系列)
- xAI (Grok 系列)
- 智谱 AI
- Moonshot (Kimi)
- 其他...
```

**Provider 配置**:
```json
{
  "providers": {
    "openai": {
      "apiKey": "sk-...",
      "enabled": true
    },
    "anthropic": {
      "apiKey": "sk-ant-...",
      "enabled": true
    }
  }
}
```

## 🔐 安全概念

### 1. Authentication（认证）

**认证方式**:
- Token 认证
- 设备配对
- OAuth（部分通道）

**Token 认证流程**:
```
1. 客户端发送连接请求（包含 token）
2. Gateway 验证 token
3. 验证通过，建立连接
4. 验证失败，拒绝连接
```

### 2. Authorization（授权）

**授权级别**:
- **Admin**: 完全控制
- **Operator**: 基本操作
- **Viewer**: 只读权限

**权限检查**:
```typescript
if (!user.hasPermission('agent.invoke')) {
  throw new Error('Permission denied');
}
```

### 3. Pairing（配对）

**设备配对**用于建立信任关系。

**配对流程**:
```
1. 新设备发起连接
2. Gateway 生成配对码
3. 用户确认配对
4. 建立信任关系
5. 颁发设备令牌
```

## 📡 通信概念

### 1. WebSocket Protocol

**协议特点**:
- 长连接
- 双向通信
- 低延迟

**消息格式**:
```typescript
// 请求
{
  type: 'req',
  id: 'uuid',
  method: 'method_name',
  params: { /* ... */ }
}

// 响应
{
  type: 'res',
  id: 'uuid',
  ok: true,
  payload: { /* ... */ }
}

// 事件推送
{
  type: 'event',
  event: 'event_name',
  payload: { /* ... */ }
}
```

### 2. HTTP API

**用途**:
- Control UI
- WebChat
- Webhook

**端点示例**:
```
GET  /health          # 健康检查
POST /api/send        # 发送消息
GET  /api/sessions    # 获取会话列表
```

### 3. Webhook

**Webhook** 允许外部系统触发 OpenClaw 事件。

**配置示例**:
```json
{
  "webhooks": [
    {
      "url": "/webhook/my-endpoint",
      "handler": "my-handler"
    }
  ]
}
```

## 🧠 AI 相关概念

### 1. Model（模型）

**Model** 是 AI 的大脑。

**模型参数**:
- `temperature`: 创造性（0-1）
- `maxTokens`: 最大输出长度
- `topP`: 核采样参数
- `frequencyPenalty`: 频率惩罚

**模型选择**:
```typescript
const model = config.agent.model;
// 例如："anthropic/claude-opus-4-6"
```

### 2. Context（上下文）

**Context** 是对话的背景信息。

**上下文组成**:
- 系统提示词
- 用户历史消息
- 工具定义
- 当前状态

**上下文管理**:
```typescript
class ContextManager {
  private messages: Message[] = [];
  private maxTokens: number = 4000;
  
  addMessage(msg: Message) {
    this.messages.push(msg);
    this.trimToFit();
  }
  
  private trimToFit() {
    // 移除旧消息以适应 token 限制
    while (this.countTokens() > this.maxTokens) {
      this.messages.shift();
    }
  }
}
```

### 3. Streaming（流式传输）

**Streaming** 允许实时输出 AI 响应。

**流式过程**:
```
开始 → [Token 1] → [Token 2] → ... → [Token N] → 结束
```

**代码示例**:
```typescript
async function* streamResponse(prompt: string) {
  const stream = await model.generateStream(prompt);
  
  for await (const chunk of stream) {
    yield chunk.text;
  }
}
```

### 4. Tool Calling（工具调用）

**Tool Calling** 让 AI 能够执行实际操作。

**调用过程**:
```
1. AI 决定调用工具
2. 返回工具调用请求
3. 执行工具
4. 将结果返回给 AI
5. AI 继续生成响应
```

**示例**:
```typescript
// AI 返回工具调用
{
  role: 'assistant',
  toolCalls: [{
    id: 'call_123',
    type: 'function',
    function: {
      name: 'browser.search',
      arguments: '{"query":"weather"}'
    }
  }]
}

// 执行工具
const result = await browser.search({ query: 'weather' });

// 返回结果给 AI
{
  role: 'tool',
  toolCallId: 'call_123',
  content: result
}
```

## 🗂️ 数据概念

### 1. Configuration（配置）

**配置层次**:
```
1. 默认配置 (代码中)
2. 文件配置 (~/.openclaw/openclaw.json)
3. 环境变量 (process.env)
4. CLI 参数 (命令行)
```

**配置合并规则**:
```
CLI 参数 > 环境变量 > 文件配置 > 默认配置
```

### 2. Credentials（凭证）

**Credentials** 是敏感信息（API Key、密码等）。

**存储位置**:
```
~/.openclaw/credentials/
```

**凭证类型**:
```typescript
interface Credentials {
  type: 'api_key' | 'oauth' | 'password';
  provider: string;
  value: SecretRef;  // 加密存储
}
```

### 3. State（状态）

**State** 是运行时信息。

**状态持久化**:
```typescript
// 保存状态
await stateStore.save('session:123', sessionData);

// 加载状态
const data = await stateStore.load('session:123');
```

## ⚙️ 运行概念

### 1. Process（进程）

**进程模式**:
- **Foreground**: 前台运行，输出到终端
- **Background**: 后台运行，作为守护进程

**进程管理**:
```bash
# 启动
openclaw gateway

# 停止
openclaw gateway stop

# 重启
openclaw gateway restart

# 查看状态
openclaw gateway status
```

### 2. Logging（日志）

**日志级别**:
```
DEBUG < INFO < WARN < ERROR
```

**日志输出**:
```typescript
logger.debug('调试信息');
logger.info('一般信息');
logger.warn('警告信息');
logger.error('错误信息');
```

**日志文件位置**:
```
macOS: ~/Library/Logs/OpenClaw/
Linux: ~/.local/share/openclaw/logs/
Windows: %APPDATA%/OpenClaw/logs/
```

### 3. Health Check（健康检查）

**健康检查端点**:
```
WebSocket: health
HTTP: GET /health
```

**健康信息**:
```typescript
{
  status: 'ok',
  uptime: 3600,      // 运行时间（秒）
  channels: {        // 通道状态
    telegram: 'connected',
    whatsapp: 'connected'
  },
  memory: 256000000  // 内存使用
}
```

## 🎭 用户概念

### 1. Account（账户）

**Account** 代表一个用户身份。

**账户信息**:
```typescript
interface Account {
  id: string;
  platform: string;  // 'telegram', 'whatsapp', etc.
  username: string;
  displayName?: string;
}
```

### 2. Group（群组）

**Group** 是多人群聊。

**群组激活模式**:
- `mention`: 仅在被 @ 时响应
- `always`: 总是响应
- `never`: 从不响应

**群组配置**:
```json
{
  "groups": {
    "*": {
      "requireMention": true
    }
  }
}
```

### 3. Presence（在线状态）

**Presence** 表示用户的在线状态。

**状态类型**:
```typescript
type PresenceStatus = 'online' | 'offline' | 'away' | 'busy';
```

**状态更新**:
```typescript
gateway.emit('presence', {
  userId: 'user123',
  status: 'online',
  timestamp: Date.now()
});
```

## 🔄 高级概念

### 1. Hook（钩子）

**Hook** 允许在特定事件发生时执行自定义逻辑。

**Hook 类型**:
- `beforeAgent`: Agent 执行前
- `afterAgent`: Agent 执行后
- `beforeSend`: 发送消息前
- `afterSend`: 发送消息后

**Hook 配置**:
```json
{
  "hooks": {
    "afterAgent": {
      "script": "~/scripts/log-agent-response.js"
    }
  }
}
```

### 2. Cron Job（定时任务）

**Cron Job** 允许定时执行任务。

**配置示例**:
```json
{
  "cron": [
    {
      "schedule": "0 8 * * *",  // 每天早上 8 点
      "action": "send",
      "params": {
        "to": "+1234567890",
        "message": "Good morning!"
      }
    }
  ]
}
```

### 3. Skill（技能）

**Skill** 是预定义的自动化流程。

**技能示例**:
```markdown
# SKILL.md

## Name: Weather Reporter

## Trigger:
When user asks about weather

## Steps:
1. Extract location from query
2. Call weather API
3. Format response
4. Send to user
```

---

**掌握这些核心概念**是理解 OpenClaw 的关键。建议结合实际代码和使用场景来加深理解。

*最后更新：2026 年 4 月 2 日*
