# OpenClaw 项目学习指南

## 📚 项目概述

### 什么是 OpenClaw？

OpenClaw 是一个**个人 AI 助手系统**，你可以把它部署在自己的设备上运行。它的核心理念是：

> **本地优先、多通道集成、完全可控的个人 AI**

想象一下，你有一个私人的 AI 助手，它：
- 运行在你自己的电脑上（而不是云端）
- 可以通过微信、Telegram、Slack、Discord 等各种聊天工具与它交流
- 能帮你执行任务、搜索信息、控制浏览器
- 完全由你控制，数据不经过第三方

### 项目的核心特点

1. **多通道支持**：支持 20+ 种通信渠道（WhatsApp、Telegram、Slack、Discord、微信等）
2. **插件化架构**：可以像手机 App 一样安装各种技能包
3. **本地优先**：数据和处理都在本地，保护隐私
4. **可扩展性强**：基于 TypeScript，易于开发和贡献

## 🎯 适合谁学习？

- **编程初学者**：想学习现代 TypeScript 项目开发
- **AI 爱好者**：想了解如何构建 AI 助手系统
- **全栈开发者**：想学习微服务、WebSocket、插件化架构
- **隐私倡导者**：想搭建自己的私人 AI 助手

## 📖 如何使用这份学习指南

本指南按照**由浅入深**的顺序组织，建议按以下路径学习：

### 第一阶段：基础准备（1-2 周）
1. 先学习 [`languages/`](./languages/README.md) 中的编程语言知识
2. 了解项目使用的技术栈和工具

### 第二阶段：理解架构（2-3 周）
1. 阅读 [`architecture/`](./architecture/README.md) 了解整体设计
2. 学习核心概念 [`core-concepts/`](./core-concepts/README.md)

### 第三阶段：深入模块（3-4 周）
1. 学习通道系统 [`channels/`](./channels/README.md)
2. 学习插件系统 [`plugins/`](./plugins/README.md)

### 第四阶段：实战练习（持续）
1. 跟着 [`learning-path/`](./learning-path/README.md) 的学习路径实践
2. 尝试修改代码、添加功能

## 🔗 快速链接

- [官方文档](https://docs.openclaw.ai)
- [GitHub 仓库](https://github.com/openclaw/openclaw)
- [Discord 社区](https://discord.gg/clawd)
- [愿景文档](../VISION.md)

## 💡 学习建议

1. **不要急于求成**：这是一个大型项目，慢慢来
2. **动手实践**：光看不练假把式，一定要动手改代码
3. **提问是好习惯**：遇到问题先思考，再查文档，最后问社区
4. **记录笔记**：好记性不如烂笔头

---

**开始你的 OpenClaw 学习之旅吧！🦞**

*最后更新：2026 年 4 月 2 日*
