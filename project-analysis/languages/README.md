# OpenClaw 项目涉及的语言

## 📋 目录

1. [TypeScript/JavaScript](#typescriptjavascript) - 主要开发语言
2. [Node.js](#nodejs) - 运行时环境
3. [Shell Script](#shell-script) - 自动化脚本
4. [Swift](#swift) - iOS/macOS应用
5. [Kotlin](#kotlin) - Android 应用
6. [YAML/JSON](#yamljson) - 配置文件
7. [Markdown](#markdown) - 文档编写

---

## TypeScript/JavaScript

### 什么是 TypeScript？

**TypeScript** 是 JavaScript 的超集，由微软开发。它在 JavaScript 的基础上添加了**类型系统**。

#### 为什么选择 TypeScript？

```
JavaScript → 灵活但容易出错
TypeScript → 更安全、更易维护、更适合大项目
```

### 基础语法示例

#### 1. 变量声明

```typescript
// 使用 let 和 const（不要用 var）
const name: string = "OpenClaw";  // :string 是类型注解
let age: number = 25;
let isActive: boolean = true;

// 类型推断（可以省略类型注解）
let message = "Hello";  // TypeScript 自动推断为 string
```

#### 2. 函数

```typescript
// 基本函数
function greet(name: string): string {
  return `Hello, ${name}!`;
}

// 箭头函数（更简洁）
const add = (a: number, b: number): number => a + b;

// 可选参数
function sendMessage(text: string, to?: string): void {
  if (to) {
    console.log(`Sending to ${to}: ${text}`);
  } else {
    console.log(text);
  }
}
```

#### 3. 接口（Interface）

```typescript
// 定义对象的形状
interface User {
  id: number;
  name: string;
  email?: string;  // ? 表示可选
  readonly createdAt: Date;  // readonly 表示只读
}

// 使用接口
const user: User = {
  id: 1,
  name: "Alice",
  createdAt: new Date()
};
```

#### 4. 类型别名

```typescript
type ID = string | number;  // 联合类型
type Callback<T> = (data: T) => void;  // 泛型

type Message = {
  id: ID;
  content: string;
  timestamp: Date;
};
```

#### 5. 类（Class）

```typescript
class Gateway {
  private port: number;  // private 私有成员
  public running: boolean = false;  // public 公共成员
  
  constructor(port: number) {
    this.port = port;
  }
  
  async start(): Promise<void> {
    this.running = true;
    console.log(`Gateway started on port ${this.port}`);
  }
}
```

#### 6. 异步编程（重要！）

```typescript
// Promise
async function fetchData(): Promise<string> {
  const response = await fetch('https://api.example.com/data');
  return await response.text();
}

// async/await（现代写法）
async function processUser(userId: string): Promise<void> {
  try {
    const user = await getUserById(userId);
    console.log(user);
  } catch (error) {
    console.error('Error:', error);
  }
}
```

### OpenClaw 中的 TypeScript 模式

#### 导入/导出

```typescript
// 命名导入
import { Gateway, Config } from './gateway';

// 默认导入
import express from 'express';

// 类型导入（只用于类型，不生成运行时代码）
import type { User } from './types';

// 导出
export const VERSION = '1.0.0';
export default Gateway;
```

#### 模块系统

OpenClaw 使用 **ESM（ES Modules）**：

```typescript
// ✅ 正确：ESM 语法
import { something } from './module.js';
export const value = 42;

// ❌ 错误：CommonJS 语法（不要使用）
const something = require('./module');
module.exports = { value };
```

### 实际代码示例

来自 OpenClaw 的真实代码片段：

```typescript
// src/entry.ts - 项目入口文件
#!/usr/bin/env node
import { spawn } from "node:child_process";
import process from "node:process";

// 主函数
async function main() {
  // 环境变量处理
  process.env.OPENCLAW_ENV = "production";
  
  // 启动网关
  const gateway = await import('./gateway');
  await gateway.start();
}

// 执行主函数
main().catch(console.error);
```

### 学习资源

- [TypeScript 官方教程](https://www.typescriptlang.org/docs/handbook/intro.html)
- [TypeScript 入门教程（中文）](https://ts.xcatliu.com/)
- [练习平台](https://www.typescriptlang.org/play)

---

## Node.js

### 什么是 Node.js？

**Node.js** 是一个 JavaScript 运行时，让你可以在服务器端运行 JavaScript/TypeScript 代码。

### 核心概念

#### 1. 包管理器（npm/pnpm/bun）

```bash
# 安装依赖
npm install express
pnpm add typescript
bun install vitest

# 运行脚本
npm run build
pnpm test
bun dev
```

#### 2. 文件系统操作

```typescript
import { readFileSync, writeFileSync } from 'node:fs';
import { join } from 'node:path';

// 读取文件
const content = readFileSync('config.json', 'utf-8');

// 写入文件
writeFileSync('output.txt', 'Hello World');

// 路径拼接（跨平台）
const configPath = join(process.cwd(), 'config', 'default.json');
```

#### 3. 进程管理

```typescript
import { spawn } from 'node:child_process';

// 启动子进程
const child = spawn('node', ['script.js'], {
  stdio: 'inherit'  // 继承父进程的输入输出
});

child.on('exit', (code) => {
  console.log(`Child exited with code ${code}`);
});
```

#### 4. 环境变量

```typescript
import process from 'node:process';

// 读取环境变量
const port = process.env.PORT || '3000';
const isDev = process.env.NODE_ENV === 'development';

// 设置环境变量
process.env.APP_NAME = 'OpenClaw';
```

### OpenClaw 中的 Node.js API

```typescript
// 常用内置模块
import path from 'node:path';      // 路径处理
import fs from 'node:fs';         // 文件系统
import os from 'node:os';         // 操作系统信息
import crypto from 'node:crypto'; // 加密功能
import http from 'node:http';     // HTTP 服务
import events from 'node:events'; // 事件发射器
```

### 学习资源

- [Node.js 官方文档](https://nodejs.org/docs/latest/api/)
- [Node.js 设计模式](https://github.com/nicoespeon/nodejs-design-patterns)

---

## Shell Script

### 什么是 Shell 脚本？

**Shell 脚本**是用于自动化命令行任务的脚本语言。

### 基础语法

```bash
#!/bin/bash
# Shebang - 指定解释器

# 变量
NAME="OpenClaw"
echo "Hello $NAME"

# 条件判断
if [ "$NAME" = "OpenClaw" ]; then
  echo "Correct!"
fi

# 循环
for i in 1 2 3; do
  echo "Number: $i"
done

# 函数
greet() {
  echo "Hello $1"
}

greet "World"
```

### OpenClaw 中的 Shell 脚本示例

```bash
#!/bin/bash
# scripts/restart-mac.sh - macOS 重启脚本

echo "Restarting OpenClaw gateway..."

# 杀死旧进程
pkill -f openclaw-gateway || true

# 等待一下
sleep 1

# 启动新进程
nohup openclaw gateway run > /tmp/openclaw.log 2>&1 &

echo "Gateway restarted!"
```

### 学习资源

- [Bash 脚本入门教程](https://www.shellscript.sh/)
- [菜鸟教程 - Shell 脚本](https://www.runoob.com/linux/linux-shell.html)

---

## Swift（iOS/macOS 应用）

### 什么是 Swift？

**Swift** 是苹果公司开发的编程语言，用于构建 iOS、macOS 应用。

### 基础语法

```swift
// 变量和常量
let name = "OpenClaw"  // 常量
var age = 25          // 变量

// 类型注解
let message: String = "Hello"
let count: Int = 10

// 可选类型（Optional）
var email: String? = nil  // 可能为 nil

// 函数
func greet(name: String) -> String {
  return "Hello, \(name)"
}

// 闭包（类似箭头函数）
let numbers = [1, 2, 3]
let doubled = numbers.map { $0 * 2 }
```

### OpenClaw macOS 应用中的 Swift

```swift
// apps/macos/Sources/OpenClawApp.swift
import SwiftUI

@main
struct OpenClawApp: App {
  @StateObject private var viewModel = AppViewModel()
  
  var body: some Scene {
    MenuBarExtra {
      ContentView(viewModel: viewModel)
    } label: {
      Image(systemName: "brain")
    }
    .menuBarExtraStyle(.window)
  }
}
```

### 学习资源

- [Swift 官方教程](https://www.swift.org/getting-started/)
- [Swift Playgrounds](https://apps.apple.com/app/swift-playgrounds/id908519492)

---

## Kotlin（Android 应用）

### 什么是 Kotlin？

**Kotlin** 是 JetBrains 开发的现代编程语言，是 Android 开发的首选语言。

### 基础语法

```kotlin
// 变量
val name = "OpenClaw"  // 只读（immutable）
var age = 25          // 可变（mutable）

// 类型推断
val message = "Hello"  // 自动推断为 String

// 空安全
var email: String? = null  // 可空
val length = email?.length ?: 0  // Elvis 运算符

// 函数
fun greet(name: String): String {
  return "Hello, $name"
}

// 单表达式函数
fun add(a: Int, b: Int) = a + b

// 数据类
data class User(
  val id: Int,
  val name: String
)
```

### OpenClaw Android 应用中的 Kotlin

```kotlin
// apps/android/app/src/main/java/MainActivity.kt
package com.openclaw

import android.os.Bundle
import androidx.activity.ComponentActivity

class MainActivity : ComponentActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    println("OpenClaw Android starting...")
  }
}
```

### 学习资源

- [Kotlin 官方教程](https://kotlinlang.org/docs/home.html)
- [Kotlin for Android Developers](https://developer.android.com/kotlin)

---

## YAML/JSON

### YAML - 人类友好的配置格式

```yaml
# 基本语法
name: OpenClaw
version: 1.0.0

# 列表
channels:
  - telegram
  - whatsapp
  - discord

# 嵌套对象
config:
  gateway:
    port: 18789
    bind: loopback
  
  agent:
    model: anthropic/claude-opus-4-6
    thinking: high
```

### JSON - 数据交换格式

```json
{
  "name": "OpenClaw",
  "version": "1.0.0",
  "channels": ["telegram", "whatsapp"],
  "config": {
    "gateway": {
      "port": 18789
    }
  }
}
```

### OpenClaw 配置示例

```json5
// ~/.openclaw/openclaw.json
{
  agent: {
    model: "anthropic/claude-opus-4-6",
  },
  channels: {
    telegram: {
      botToken: "123456:ABCDEF",
    },
  },
}
```

### 学习资源

- [YAML 入门教程](https://yaml.org/)
- [JSON 官网](https://www.json.org/json-zh.html)

---

## Markdown

### 什么是 Markdown？

**Markdown** 是一种轻量级标记语言，用于编写文档。

### 基础语法

```markdown
# 一级标题
## 二级标题
### 三级标题

**粗体文本**
*斜体文本*
~~删除线~~

- 无序列表项 1
- 无序列表项 2

1. 有序列表项 1
2. 有序列表项 2

[链接文本](https://example.com)
![图片描述](image.png)

`行内代码`

```typescript
// 代码块
function hello() {
  console.log("Hello World");
}
```

> 引用文本

| 列 1 | 列 2 | 列 3 |
|------|------|------|
| A    | B    | C    |
```

### 学习资源

- [Markdown 官方指南](https://www.markdownguide.org/)
- [Markdown 中文教程](https://markdown-zh.readthedocs.io/)

---

## 📚 推荐学习顺序

### 零基础学习者

1. **第 1-2 周**：Markdown（写笔记）
2. **第 3-8 周**：TypeScript 基础
3. **第 9-10 周**：Node.js 基础
4. **第 11-12 周**：Shell 脚本基础
5. **后续**：根据兴趣选学 Swift 或 Kotlin

### 有编程经验者

1. **第 1 周**：快速过一遍 TypeScript
2. **第 2 周**：Node.js + 项目工具链
3. **第 3 周**：开始阅读 OpenClaw 源码

---

**记住**：学习编程语言最好的方式是**动手实践**！不要只看教程，要亲自写代码。

*最后更新：2026 年 4 月 2 日*
