# 中文导读：Pi 定制化体系（Customization）

- 对应文档：`source/packages/coding-agent/README.md` (Customization 部分)
- pi 镜像 commit：`f429ddb`
- 学习阶段：Stage 1 · 概念入门
- 记录日期：2026-06-11

## 一句话概述

Pi 提供五个层次的定制能力：Prompt Templates（提示词模板）、Skills（技能）、Extensions（扩展）、Themes（主题）、Pi Packages（打包分享），从简单配置到完整编程，渐进式满足不同复杂度的需求。

## 核心概念

- **Prompt Templates（提示词模板）**：可复用的 Markdown 文件，用 `/name` 展开，适合重复性提示词
- **Skills（技能）**：按需加载的能力包，遵循 Agent Skills 标准，用 `/skill:name` 调用
- **Extensions（扩展）**：TypeScript 模块，可以注册工具、命令、拦截事件、自定义 UI，是最强大的定制方式
- **Themes（主题）**：视觉外观定制，内置 dark/light，支持热重载
- **Pi Packages（Pi 包）**：打包分享机制，将扩展、技能、提示词、主题组合并通过 npm/git 分享

## 五个定制层次对比

| 层次 | 复杂度 | 能力 | 格式 | 分享方式 |
|------|--------|------|------|----------|
| **Prompt Templates** | ⭐ | 可复用的提示词文本 | Markdown | 文件/Pi Package |
| **Skills** | ⭐⭐ | 按需加载的能力包 | Markdown (SKILL.md) | 文件/Pi Package |
| **Extensions** | ⭐⭐⭐ | 完整的 TypeScript 编程能力 | TypeScript | 文件/Pi Package |
| **Themes** | ⭐ | 视觉外观定制 | JSON | 文件/Pi Package |
| **Pi Packages** | - | 打包和分发机制 | package.json + 资源 | npm/git |

## Prompt Templates 详解

**最简单的定制方式**，就是可复用的 Markdown 文件。

```markdown
<!-- ~/.pi/agent/prompts/review.md -->
Review this code for bugs, security issues, and performance problems.
Focus on: {{focus}}
```

**存放位置：**
- `~/.pi/agent/prompts/`（全局）
- `.pi/prompts/`（项目级）
- Pi Package 中（可分享）

**使用方式：** 在编辑器中输入 `/name` 即可展开

## Skills 详解

**按需加载的能力包**，遵循 [Agent Skills 标准](https://agentskills.io)。

```markdown
<!-- ~/.pi/agent/skills/my-skill/SKILL.md -->
# My Skill
Use this skill when the user asks about X.

## Steps
1. Do this
2. Then that
```

**存放位置：**
- `~/.pi/agent/skills/` 或 `~/.agents/skills/`（全局）
- `.pi/skills/` 或 `.agents/skills/`（项目级，从 cwd 向上搜索）
- Pi Package 中（可分享）

**使用方式：** `/skill:name` 或让 agent 自动加载

## Extensions 详解

**最强大的定制方式**，TypeScript 模块，可以做任何事情。

### 基本结构

```typescript
export default function (pi: ExtensionAPI) {
  // 注册工具
  pi.registerTool({ name: "deploy", execute: ... });
  
  // 注册命令
  pi.registerCommand("stats", { handler: ... });
  
  // 监听事件
  pi.on("tool_call", async (event, ctx) => { ... });
}
```

### 能做什么

- 自定义工具（甚至替换内置工具）
- 子代理和计划模式
- 自定义压缩和摘要
- 权限门控和路径保护
- 自定义编辑器和 UI 组件
- 状态栏、页眉、页脚
- Git 检查点和自动提交
- SSH 和沙箱执行
- MCP 服务器集成
- 让 pi 看起来像 Claude Code
- 等待时玩游戏（是的，Doom 可以运行）

### 关键特性

- 支持同步和异步工厂函数
- 异步工厂会在启动前完成初始化
- 可以注册工具、命令、快捷键、标志
- 可以拦截和修改事件
- 热重载支持（`/reload`）

### 存放位置

- `~/.pi/agent/extensions/`（全局）
- `.pi/extensions/`（项目级）
- Pi Package 中（可分享）

## Themes 详解

**视觉定制**，内置 `dark` 和 `light` 两种主题。

- **热重载**：修改主题文件后立即生效
- **存放位置**：
  - `~/.pi/agent/themes/`（全局）
  - `.pi/themes/`（项目级）
  - Pi Package 中（可分享）

## Pi Packages 详解

**分享和分发的机制**，将扩展、技能、提示词模板和主题打包在一起。

### 类比理解

Pi Packages = **npm 包 + 插件商店** 的结合体

```
普通 npm 包：提供代码库供你 import
Pi Package：提供 pi 相关资源供 pi 加载使用
```

### 包结构示例

```
my-code-review-package/
├── package.json          # 声明这是 Pi Package
├── extensions/
│   └── review-tools.ts   # 自定义审查工具
├── skills/
│   └── review/SKILL.md   # 审查技能
├── prompts/
│   └── review.md         # 审查提示词模板
└── themes/
    └── review-dark.json  # 专用主题
```

### package.json 声明

```json
{
  "name": "my-code-review-package",
  "keywords": ["pi-package"],
  "pi": {
    "extensions": ["./extensions"],
    "skills": ["./skills"],
    "prompts": ["./prompts"],
    "themes": ["./themes"]
  }
}
```

### 安装方式

```bash
pi install npm:@foo/pi-tools          # npm 包
pi install git:github.com/user/repo   # git 仓库
pi install https://github.com/user/repo  # URL
```

### 管理命令

```bash
pi list                      # 列出已安装的包
pi update                    # 更新包
pi config                    # 启用/禁用包资源
pi remove npm:@foo/pi-tools  # 删除包
```

### 安装位置

- `~/.pi/agent/npm/`（npm 包）
- `~/.pi/agent/git/`（git 包）
- `.pi/npm/` 或 `.pi/git/`（项目级，用 `-l` 参数）

## TypeScript 简介（Extension 的实现语言）

### 是什么

TypeScript = JavaScript + 类型系统

```javascript
// JavaScript（没有类型）
function add(a, b) { return a + b; }
add("hello", 1)  // 不报错，运行时出问题

// TypeScript（有类型）
function add(a: number, b: number): number { return a + b; }
add("hello", 1)  // 编译时报错，提前发现问题
```

### 核心特点

| 特点 | 说明 |
|------|------|
| **超集** | 任何合法的 JavaScript 都是合法的 TypeScript |
| **可选类型** | 可以加类型，也可以不加（渐进式） |
| **编译时检查** | 代码运行前就能发现错误 |
| **编辑器支持** | 自动补全、跳转定义、重构 |
| **运行时仍是 JS** | 编译后变成普通 JavaScript 执行 |

### 为什么 Pi 选择 TypeScript

1. **Node.js 生态天然语言**：Pi 运行在 Node.js 上，TypeScript 是 Node.js 社区的主流选择
2. **类型安全 = 更少的 bug**：Extension 系统需要处理复杂的事件和 API，类型能提前发现问题
3. **编辑器智能提示**：写 Extension 时，编辑器能告诉你有哪些 API 可用、参数类型、返回值
4. **渐进式采用**：不需要一开始就写完美的类型
5. **跨平台一致性**：TypeScript 编译成 JavaScript，可以在任何有 JS 运行时的地方跑

## pi 提供 vs 我要自己设计

- **pi 提供**：
  - 五个层次的定制框架
  - Extension 系统的完整 API（注册工具、命令、事件监听）
  - 热重载机制
  - Pi Packages 的安装和管理
  - TypeScript 运行时环境

- **我要自己设计**：
  - 具体的业务逻辑和工作流
  - 自定义工具的实现
  - 权限控制和安全策略
  - UI 组件和交互设计
  - 打包和分享自己的 Pi Package

## 易踩的坑

- **Extension 有完全系统权限**：安装第三方扩展前务必审查代码，它们可以执行任意代码
- **Pi Packages 安全风险**：Extensions 执行任意代码，Skills 可以指示模型执行任何操作包括运行可执行文件
- **TypeScript 学习曲线**：如果不熟悉 TypeScript，写 Extension 会有一定门槛
- **异步工厂函数**：Extension 工厂可以是异步的，pi 会等待它完成才继续启动
- **热重载的边界**：不是所有修改都能热重载，有些需要重启 pi

## 对「我的个人助手 agent」的启示

- **从 Prompt Templates 开始**：最简单的定制方式，先积累常用的提示词模板
- **Skills 封装工作流**：把重复性的工作流程封装成 Skills
- **Extensions 实现复杂逻辑**：需要自定义工具、拦截事件、修改 UI 时用 Extensions
- **利用社区 Pi Packages**：先看看 npm 上是否有现成的解决方案
- **渐进式学习 TypeScript**：可以先用 `any` 类型，慢慢加上严格的类型

## 待核对 (TODO)

- [ ] 确认 Pi Packages 的安全机制是否有改进（当前文档提到有安全风险）
- [ ] 了解 Extensions 的性能影响，特别是在加载多个扩展时
- [ ] 检查是否有 TypeScript 类型定义的最佳实践文档
- [ ] 确认 Pi Packages 的版本管理和更新策略
