# 中文导读：Pi 最小内核理念

- 对应文档：`source/packages/coding-agent/README.md` (Philosophy 部分)
- pi 镜像 commit：`f429ddb`
- 学习阶段：Stage 1 · 概念入门
- 记录日期：2026-06-11

## 一句话概述

Pi 的设计哲学是保持**最小内核**，通过极度可扩展的 extension 系统让用户按需构建功能，而不是把常见功能内置到核心中。

## 核心概念

- **Aggressively Extensible（极度可扩展）**：Pi 刻意不内置常见功能，而是提供强大的扩展机制，让用户自由选择和构建
- **Minimal Kernel（最小内核）**：核心只包含最基础的能力（read/write/edit/bash 四个工具），其他一切通过扩展实现
- **Extension System（扩展系统）**：TypeScript 模块，可以注册工具、命令、快捷键、拦截事件、自定义 UI 等
- **Skills（技能）**：按需加载的能力包，遵循 Agent Skills 标准
- **Pi Packages（Pi 包）**：通过 npm 或 git 分享的扩展、技能、提示词模板和主题的集合

## 关键 API / 配置

| 名称（保留英文） | 作用 | 备注 |
| --- | --- | --- |
| `pi.registerTool()` | 注册自定义工具供 LLM 调用 | 扩展系统的核心 API |
| `pi.registerCommand()` | 注册自定义斜杠命令 | 如 `/mycommand` |
| `pi.on()` | 订阅生命周期事件 | 可拦截和修改工具调用 |
| `pi.registerShortcut()` | 注册自定义键盘快捷键 | 增强交互体验 |

## Pi 刻意不内置的功能

| 功能 | Pi 的态度 | 推荐替代方案 |
|------|----------|-------------|
| **MCP** | ❌ 不内置 | 用 CLI 工具 + README（Skills），或写 extension 添加 MCP 支持 |
| **子代理 (Sub-agents)** | ❌ 不内置 | 通过 tmux 启动多个 Pi 实例，或用 extension 实现 |
| **权限弹窗** | ❌ 不内置 | 在容器中运行，或用 extension 构建确认流程 |
| **计划模式 (Plan mode)** | ❌ 不内置 | 把计划写到文件里，或用 extension 实现 |
| **内置待办 (To-dos)** | ❌ 不内置 | 用 TODO.md 文件，或用 extension 实现 |
| **后台 Bash** | ❌ 不内置 | 使用 tmux，获得完全的可观测性 |

## pi 提供 vs 我要自己设计

- **pi 提供**：
  - 最小终端编码工具（read/write/edit/bash）
  - 强大的扩展系统（TypeScript extensions）
  - 会话管理和持久化（JSONL 格式）
  - 多种运行模式（交互式、打印、JSON、RPC、SDK）
  - 热重载扩展、技能、提示词、主题
  - 通过 Pi Packages 分享扩展

- **我要自己设计**：
  - 业务状态机和工作流
  - 审批流程和权限控制
  - 幂等性保证
  - 审计日志（pi session JSONL 是 agent trace，不是业务审计）
  - 异常补偿机制
  - 领域特定的验证规则

## 易踩的坑

- **不要期待内置功能**：如果你习惯了其他工具的内置功能（如子代理、计划模式），在 Pi 中需要自己构建
- **扩展有完全系统权限**：安装第三方扩展前务必审查代码，它们可以执行任意代码
- **Compaction 是有损的**：长时间会话的压缩会丢失细节，完整历史仍在 JSONL 文件中，需要用 `/tree` 回溯
- **Session JSONL 不是业务审计**：它是 agent trace，不适合直接作为业务操作的审计记录

## 设计哲学背后的原因

1. **灵活性优先**：不同开发者/团队的工作流程差异很大，内置固定功能会迫使所有人适应同一套方式
2. **核心精简**：更少的代码 = 更少的 bug = 更容易维护和理解
3. **社区驱动**：通过 Pi Packages 机制，社区可以共享构建的扩展
4. **Unix 哲学**：做好"最小终端编码工具"这一件事，其他通过可组合的扩展实现

## 对「我的个人助手 agent」的启示

- **明确边界**：Pi 负责 agent loop、tool calling、session 管理等底层能力；业务逻辑需要自己设计
- **从 extension 开始**：大多数定制需求都可以通过写 extension 解决
- **利用社区**：先看看 npm 上是否有现成的 Pi Package，避免重复造轮子
- **保持轻量**：只安装需要的扩展，保持系统的简洁和可维护性

## 待核对 (TODO)

- [ ] 确认当前 Pi 版本是否仍然保持这个最小内核理念（可能随版本更新有变化）
- [ ] 检查是否有新的内置功能被添加（查看 CHANGELOG）
- [ ] 了解 extension 系统的性能影响，特别是在加载多个扩展时
