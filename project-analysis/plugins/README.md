# OpenClaw 插件系统详解

## 🧩 什么是插件？

**插件（Plugin）** 是 OpenClaw 的扩展机制，允许开发者添加新功能而无需修改核心代码。

## 🎯 插件类型

### 1. 通道插件（Channel Plugin）

用于添加新的通信渠道。

**示例**:
- Telegram
- WhatsApp
- Slack
- Discord

**接口**:
```typescript
interface ChannelPlugin {
  id: string;
  name: string;
  initialize(config: any): Promise<void>;
  sendMessage(target: string, message: Message): Promise<void>;
  onMessage(callback: (msg: Message) => void): void;
}
```

### 2. Provider 插件（模型提供商）

用于添加新的 AI 模型支持。

**示例**:
- OpenAI (GPT)
- Anthropic (Claude)
- Google (Gemini)
- xAI (Grok)

**接口**:
```typescript
interface ProviderPlugin {
  id: string;
  name: string;
  authenticate(credentials: Credentials): Promise<void>;
  chat(messages: Message[], model: string): Promise<Response>;
  stream(messages: Message[], model: string): AsyncIterable<Chunk>;
}
```

### 3. 工具插件（Tool Plugin）

用于添加新工具和技能。

**示例**:
- Browser（浏览器控制）
- Canvas（画布）
- Location（位置服务）
- System（系统命令）

**接口**:
```typescript
interface ToolPlugin {
  id: string;
  name: string;
  description: string;
  parameters: ParameterSchema;
  execute(args: any): Promise<any>;
}
```

### 4. 节点插件（Node Plugin）

用于添加设备功能。

**示例**:
- iOS 设备功能
- Android 设备功能
- macOS 设备功能

## 📦 插件结构

### 标准插件目录

```
my-plugin/
├── package.json              # npm 包配置
├── openclaw.plugin.json      # OpenClaw 插件配置
├── src/
│   ├── index.ts             # 主入口
│   ├── api.ts               # 公开 API
│   ├── runtime-api.ts       # 运行时 API（可选）
│   └── ...                  # 其他实现
├── README.md                 # 使用说明
└── test/                     # 测试（可选）
    └── index.test.ts
```

### package.json 示例

```json
{
  "name": "@openclaw/my-plugin",
  "version": "1.0.0",
  "type": "module",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "dependencies": {
    "@openclaw/plugin-sdk": "workspace:*"
  },
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```

### openclaw.plugin.json 示例

```json
{
  "id": "my-plugin",
  "name": "My Plugin",
  "version": "1.0.0",
  "description": "A sample plugin",
  "author": "Your Name",
  "license": "MIT",
  "main": "./dist/index.js",
  "exports": {
    ".": "./dist/index.js",
    "./api": "./dist/api.js"
  },
  "dependencies": {
    "@openclaw/plugin-sdk": "workspace:*"
  }
}
```

## 🔨 开发插件

### 步骤 1: 初始化项目

```bash
# 创建目录
mkdir my-plugin
cd my-plugin

# 初始化 npm 项目
npm init -y

# 安装 SDK
pnpm add @openclaw/plugin-sdk
```

### 步骤 2: 创建 TypeScript 配置

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "lib": ["ES2022"],
    "moduleResolution": "bundler",
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "strict": true,
    "esModuleInterop": true
  },
  "include": ["src/**/*"]
}
```

### 步骤 3: 实现插件

#### 示例：天气 Provider 插件

```typescript
// src/index.ts
import type { ProviderPlugin, Message } from '@openclaw/plugin-sdk';

export default class WeatherProvider implements ProviderPlugin {
  readonly id = 'weather';
  readonly name = 'Weather Provider';
  
  private apiKey: string = '';
  
  async authenticate(credentials: any) {
    this.apiKey = credentials.apiKey;
    // 验证 API Key
    const response = await fetch('https://api.weather.com/validate', {
      headers: { 'Authorization': `Bearer ${this.apiKey}` }
    });
    
    if (!response.ok) {
      throw new Error('Invalid API key');
    }
  }
  
  async chat(messages: Message[], model: string) {
    // 获取最后一条用户消息
    const lastMessage = messages[messages.length - 1];
    
    // 调用天气 API
    const weather = await this.getWeather(lastMessage.content);
    
    return {
      role: 'assistant',
      content: `当前天气：${weather.condition}, 温度：${weather.temp}°C`
    };
  }
  
  async *stream(messages: Message[], model: string) {
    const response = await this.chat(messages, model);
    
    // 流式输出
    for (const char of response.content) {
      yield { text: char };
    }
  }
  
  private async getWeather(location: string) {
    const response = await fetch(
      `https://api.weather.com/current?location=${encodeURIComponent(location)}`,
      {
        headers: { 'Authorization': `Bearer ${this.apiKey}` }
      }
    );
    return await response.json();
  }
}
```

#### 示例：计算器工具插件

```typescript
// src/index.ts
import type { ToolPlugin } from '@openclaw/plugin-sdk';

export default class CalculatorTool implements ToolPlugin {
  readonly id = 'calculator';
  readonly name = 'Calculator';
  readonly description = 'Perform mathematical calculations';
  
  readonly parameters = {
    type: 'object',
    properties: {
      expression: {
        type: 'string',
        description: 'Mathematical expression to evaluate'
      }
    },
    required: ['expression']
  };
  
  async execute(args: { expression: string }) {
    try {
      // 安全计算（不要使用 eval！）
      const result = this.safeEvaluate(args.expression);
      return { success: true, result };
    } catch (error) {
      return { 
        success: false, 
        error: error instanceof Error ? error.message : 'Unknown error'
      };
    }
  }
  
  private safeEvaluate(expression: string): number {
    // 只允许数字和基本运算符
    if (!/^[\d+\-*/().\s]+$/.test(expression)) {
      throw new Error('Invalid expression');
    }
    // 使用安全的数学计算库
    return Function('"use strict";return (' + expression + ')')();
  }
}
```

### 步骤 4: 导出 API

```typescript
// src/api.ts
export { default as WeatherProvider } from './index';
export { default as CalculatorTool } from './calculator';

// 辅助函数
export function formatTemperature(celsius: number): string {
  return `${celsius}°C (${celsius * 9/5 + 32}°F)`;
}
```

### 步骤 5: 构建插件

```bash
# 编译 TypeScript
pnpm tsc

# 或使用构建脚本
pnpm build
```

## 🚀 安装和使用

### 安装方式

```bash
# 从本地安装
openclaw plugins install ./my-plugin

# 从 npm 安装
openclaw plugins install @openclaw/my-plugin

# 从 GitHub 安装
openclaw plugins install github:user/repo
```

### 启用插件

```json
// ~/.openclaw/openclaw.json
{
  "plugins": {
    "enabled": ["my-plugin", "weather-provider"]
  }
}
```

### 使用插件

```typescript
// 导入插件 API
import { WeatherProvider } from '@openclaw/my-plugin/api';

// 创建实例
const provider = new WeatherProvider();

// 认证
await provider.authenticate({ apiKey: 'xxx' });

// 使用
const response = await provider.chat(messages, 'weather-model');
```

## 🔌 插件 SDK API

### 核心接口

```typescript
// 从 '@openclaw/plugin-sdk' 导入
import {
  type ChannelPlugin,
  type ProviderPlugin,
  type ToolPlugin,
  type NodePlugin
} from '@openclaw/plugin-sdk';
```

### 辅助工具

```typescript
// HTTP 请求
import { fetchWithRetry } from '@openclaw/plugin-sdk';

// 日志
import { logger } from '@openclaw/plugin-sdk';
logger.info('Plugin initialized');

// 配置管理
import { getConfig } from '@openclaw/plugin-sdk';
const config = getConfig('my-plugin');

// Secret 管理
import { getSecret } from '@openclaw/plugin-sdk';
const apiKey = await getSecret('weather-api-key');
```

## 🧪 测试插件

### 单元测试示例

```typescript
// test/weather.test.ts
import { describe, it, expect, vi } from 'vitest';
import WeatherProvider from '../src/index';

describe('WeatherProvider', () => {
  it('should authenticate with valid API key', async () => {
    const provider = new WeatherProvider();
    
    // Mock fetch
    global.fetch = vi.fn(() => 
      Promise.resolve({ ok: true })
    );
    
    await expect(
      provider.authenticate({ apiKey: 'valid-key' })
    ).resolves.not.toThrow();
  });
  
  it('should reject invalid API key', async () => {
    const provider = new WeatherProvider();
    
    global.fetch = vi.fn(() => 
      Promise.resolve({ ok: false })
    );
    
    await expect(
      provider.authenticate({ apiKey: 'invalid-key' })
    ).rejects.toThrow('Invalid API key');
  });
});
```

### 运行测试

```bash
# 运行测试
pnpm test

# 带覆盖率
pnpm test:coverage
```

## 📚 最佳实践

### 1. 错误处理

```typescript
try {
  await riskyOperation();
} catch (error) {
  logger.error('Operation failed', { error });
  throw new PluginError(
    'OPERATION_FAILED',
    'Failed to complete operation',
    { cause: error }
  );
}
```

### 2. 日志记录

```typescript
logger.debug('Debug info');
logger.info('General info');
logger.warn('Warning');
logger.error('Error occurred');
```

### 3. 资源清理

```typescript
async shutdown() {
  // 关闭连接
  await this.connection.close();
  
  // 清除定时器
  clearInterval(this.timer);
  
  // 删除临时文件
  await fs.rm(this.tempDir, { recursive: true });
}
```

### 4. 配置验证

```typescript
import { z } from 'zod';

const configSchema = z.object({
  apiKey: z.string().min(1),
  baseUrl: z.string().url(),
  timeout: z.number().positive().optional()
});

function validateConfig(config: any) {
  return configSchema.parse(config);
}
```

## 🔍 调试技巧

### 1. 启用调试模式

```json
{
  "logging": {
    "level": "debug",
    "modules": ["plugin:*", "my-plugin"]
  }
}
```

### 2. 使用开发者工具

```bash
# 查看已安装插件
openclaw plugins list

# 查看插件详情
openclaw plugins info my-plugin

# 禁用插件
openclaw plugins disable my-plugin

# 重新加载插件
openclaw plugins reload my-plugin
```

### 3. 检查插件状态

```typescript
// 在插件中暴露状态
export function getStatus() {
  return {
    connected: this.isConnected,
    lastUpdate: this.lastUpdate,
    errorCount: this.errors.length
  };
}
```

## ⚠️ 常见问题

### 问题 1: 插件无法加载

**原因**:
- 路径错误
- 依赖缺失
- 版本不兼容

**解决**:
```bash
# 检查插件信息
openclaw plugins info my-plugin

# 重新安装
openclaw plugins reinstall my-plugin
```

### 问题 2: API 调用失败

**排查**:
1. 检查认证信息
2. 验证网络连接
3. 查看错误日志
4. 测试 API 端点

### 问题 3: 内存泄漏

**预防**:
```typescript
// 及时清理资源
onDisconnect() {
  clearInterval(this.interval);
  this.listeners.clear();
  this.cache.clear();
}
```

## 🎓 进阶主题

### 1. 插件间通信

```typescript
// 使用事件总线
import { eventBus } from '@openclaw/plugin-sdk';

eventBus.on('message:sent', (msg) => {
  console.log('Message sent:', msg);
});

eventBus.emit('message:sent', { id: '123', content: 'Hello' });
```

### 2. 共享状态

```typescript
// 使用全局存储
import { globalState } from '@openclaw/plugin-sdk';

globalState.set('my-plugin:data', { key: 'value' });
const data = globalState.get('my-plugin:data');
```

### 3. 动态加载

```typescript
// 懒加载模块
async function loadModule(name: string) {
  return await import(`./modules/${name}.js`);
}
```

---

**总结**: 插件系统是 OpenClaw 扩展性的核心。通过开发插件，你可以为社区贡献新功能，或定制专属自己的 AI 助手。

*最后更新：2026 年 4 月 2 日*
