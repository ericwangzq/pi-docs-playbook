# 个人助手 Agent：边建边学的学习计划

一条不断生长的主线项目——从零代码塑形，到 skill、extension、SDK、前端、多 agent 逐层叠加。

- 基于：[earendil-works/pi](https://github.com/earendil-works/pi)（commit `f429ddb`，2026-06-01 镜像）
- 规模：8 个阶段，约 15–18 天
- 主线项目：你的个人日常助手 agent（`~/Projects/my-assistant/`）

---

## 项目结构说明

pi 是全局安装的 CLI 工具，不需要 fork。你在自己的项目里使用它：

```
~/.pi/agent/                    ← 全局配置（所有目录生效）
├── AGENTS.md                   ← 全局行为规范
├── settings.json
├── prompts/                    ← 全局 slash 命令
├── skills/                     ← 全局 skill
└── extensions/                 ← 全局 extension

~/Projects/my-assistant/        ← 你的主线项目
├── .pi/                        ← 项目级配置（覆盖全局）
├── AGENTS.md
├── src/                        ← Stage 6 的 SDK 脚本
└── skills/
```

---

## 中文导读工作流（贯穿每个阶段）

解决英文文档吃力的问题，贯穿始终：

1. **用 agent 把文档讲成中文**，顺便练 pi：
   ```bash
   pi @source/packages/coding-agent/docs/sdk.md "用中文讲解核心概念，API 名称保留英文"
   ```
2. **产出中文笔记**：每读完一篇 doc，让 agent 按 `skill-draft/notes/_doc-note-template.md` 格式整理，保存到 `skill-draft/notes/`（绝不写进 `source/`）。
3. **技术术语保留英文**：`tool_call`、`session`、`compaction`、`extension`、`skill` 等不翻译。
4. **先读代码再读散文**：`examples/` 里的代码跨语言、直观，比英文叙述容易入门。

---

## 核心边界（贯穿所有阶段）

> pi 负责：agent loop、tool calling、session、extension、RPC。
> 业务状态机、幂等、审批、审计 log、异常补偿——这些仍必须由你的应用自己设计。

---

## Phase A · 用熟并调成自己的品味（Stage 0–2）

### Stage 0 · 跑通 pi + 配好中文导读（1 天）

**目标**：把 pi 跑起来，理解四种运行模式，并搭好一套能持续用的中文导读工作流。

**需要读的文档**：
- `source/packages/coding-agent/docs/quickstart.md`
- `source/packages/coding-agent/docs/usage.md`
- `source/packages/coding-agent/docs/providers.md`

**主任务**：安装 pi、配置 API key、跑通第一个 session，并建立中文导读笔记目录与模板。

**可玩成果**：在终端里让 pi 总结一个项目，并产出第一份中文导读笔记。

**子任务**：

1. `npm install -g --ignore-scripts @earendil-works/pi-coding-agent`
2. 配置 `ANTHROPIC_API_KEY` 环境变量，或在 pi 内运行 `/login`
3. `cd ~/Projects/my-assistant && pi`，发第一条消息：用中文总结这个仓库
4. 试 `pi -p "list files"`（print 模式）和 `pi --mode json -p "hi"`（json 模式），观察差异
5. 用 `pi @source/packages/coding-agent/docs/usage.md "用中文讲解核心概念，API 名称保留英文"` 生成第一份导读
6. 把导读保存到 `skill-draft/notes/usage.note.md`

**完成标准**：能说清 interactive / print / json / rpc 四种模式各自适用什么场景。

---

### Stage 1 · 架构地图 + 定制策略（1 天）

**目标**：建立 pi 整体架构图，想清楚你打算用哪些方式把 pi 调成自己的助手。

**需要读的文档**：
- `source/packages/coding-agent/README.md`（Philosophy + Customization 部分）
- `source/packages/coding-agent/docs/sdk.md`（Quick Start）
- `source/packages/coding-agent/docs/extensions.md`（Quick Start）
- `source/packages/agent/README.md`

**主任务**：画出层次图，并列出一份「我要怎么把 pi 调成我的」定制清单。

**可玩成果**：一张架构图 + 一份你自己的定制清单（哪些用 skill、哪些用 extension、哪些用 SDK）。

**子任务**：

1. 读 README 的 Philosophy 部分，理解 pi 的最小内核理念
2. 读 README 的 Customization 部分（Extensions / Skills / Prompt Templates / Packages）
3. 读 `sdk.md` 的 Quick Start 和 `extensions.md` 的 Quick Start（各前 60 行）
4. 画出层次图：pi-agent-core → coding-agent → extension / skill / SDK / RPC
5. 用一句话分别回答：extension、skill、SDK、prompt template 各自定位是什么

**完成标准**：能区分 extension（运行时 hook）/ skill（按需注入指令）/ SDK（编程式调用）/ prompt template（slash 快捷内容）。

---

### Stage 2 · 零代码塑形到你的品味（2 天）

**目标**：不写代码，仅靠配置把 pi 的行为、语气、快捷命令、外观调成你的习惯。

**需要读的文档**：
- `source/packages/coding-agent/docs/settings.md`
- `source/packages/coding-agent/docs/prompt-templates.md`
- `source/packages/coding-agent/docs/themes.md`
- `source/packages/coding-agent/docs/keybindings.md`

**主任务**：写自己的 `AGENTS.md`、调 `settings.json`、写 3–5 个日常 prompt 模板、选/改 theme。

**可玩成果**：启动 pi 后行为和输出已经符合你的习惯，且能用 `/你的命令` 触发日常流程。

**子任务**：

1. 写 `~/.pi/agent/AGENTS.md`：你的工作习惯、语气、默认规范（如一律中文回答）
2. 调 `~/.pi/agent/settings.json`：默认模型、compaction、retry 等
3. 在 `~/.pi/agent/prompts/` 写 3–5 个日常 prompt 模板（如 `/周报`、`/整理`、`/翻译`）
4. 选一个 theme，或微调颜色，看 `themes.md`
5. _(进阶)_ 改一两个 keybinding 到顺手

**完成标准**：理解配置的加载优先级：全局 `~/.pi/agent/` vs 项目 `.pi/`，以及谁覆盖谁。

---

## Phase B · 用 coding + skill 做日常任务（Stage 3–5）

### Stage 3 · 第一个 skill（真实日常任务）（2–3 天）

**目标**：把一个你真实会用到的日常任务，按 Agent Skills 规范打包成可复用 skill。

**需要读的文档**：
- `source/packages/coding-agent/docs/skills.md`
- `source/packages/coding-agent/examples/extensions/dynamic-resources/SKILL.md`

**主任务**：选一个真实任务（整理下载目录 / 生成周报 / 批量改文件名等），做成带脚本的 skill。

**可玩成果**：运行 `/skill:你的技能名` 就能完成一个真实日常任务。

**子任务**：

1. 读 `skills.md`：理解 SKILL.md frontmatter（`name`、`description`）和 progressive disclosure
2. 在 `~/.pi/agent/skills/` 下建技能目录，写 `SKILL.md`（description 要具体）
3. 在技能的 `scripts/` 下放一个真正能跑的脚本（bash / node / python 均可）
4. 在 pi 里用 `/skill:名字` 触发，验证 agent 会读 `SKILL.md` 并调用脚本
5. _(进阶)_ 在 `references/` 下放按需加载的详细文档
6. _(进阶)_ 对照 agentskills.io 规范自查命名与字段

**完成标准**：理解 skill 是「按需注入到 context 的指令包」，description 决定何时被加载。

---

### Stage 4 · 第一个 extension（自定义 tool / 守卫）（2–3 天）

**目标**：用 TypeScript 写第一个 extension，给助手加一个自定义工具和一个安全守卫。

**需要读的文档**：
- `source/packages/coding-agent/docs/extensions.md`（全文）
- `source/packages/coding-agent/examples/extensions/README.md`
- `source/packages/agent/docs/hooks.md`

**主任务**：写一个 extension：注册一个自定义 tool + 一个 `tool_call` 守卫 + 一个通知/命令。

**可玩成果**：让 agent 执行危险命令时弹出确认框被拦截；并能调用你注册的自定义 tool。

**子任务**：

1. 读 `extensions.md`（重点：Events、ExtensionContext、Custom Tools、User Interaction）
2. 建 `~/.pi/agent/extensions/my-assistant.ts`
3. 用 `pi.registerTool()` 注册一个对你有用的自定义 tool
4. 订阅 `tool_call`，检测危险命令（如 `rm -rf`），用 `ctx.ui.confirm` 询问，拒绝则 `{ block: true }`
5. 用 `pi.registerCommand()` 注册一个 `/命令`，或用 `ctx.ui.notify` 做提醒
6. _(进阶)_ 用 `/reload` 热重载 extension 并测试

**完成标准**：理解 block / passthrough 机制，以及 `ctx.ui`（confirm / notify / select / input）。

---

### Stage 5 · session / memory + 可恢复任务（2 天）

**目标**：理解 session 树、fork / resume、compaction，并搞清 JSONL 的边界。

**需要读的文档**：
- `source/packages/coding-agent/docs/session-format.md`
- `source/packages/coding-agent/docs/sessions.md`
- `source/packages/coding-agent/docs/compaction.md`
- `source/packages/agent/docs/durable-harness.md`

**主任务**：做一次 fork / resume，手查 JSONL 结构，理解 compaction 触发条件。

**可玩成果**：能在 `/tree` 里看到分支，并在文本编辑器里指出 fork 点的 `parentId`。

**子任务**：

1. 读 `session-format.md`，理解 entry 类型（header / user / assistant / tool_use / branch）
2. 做一段对话，用 `/fork` 分支、`/tree` 看树形结构
3. 用编辑器打开 `session.jsonl`，找到 fork 点的 `parentId`
4. 读 `compaction.md`，记录触发条件与 compaction entry 结构
5. _(进阶)_ 读 `durable-harness.md`，想清楚你的日常任务如何做到可恢复

**完成标准**：能讲清 pi JSONL 是 agent trace，不是你应用的 domain audit log——本质区别在哪。

---

## Phase C · 进阶：编程式 + 前端 + 多 agent（Stage 6–7）

### Stage 6 · SDK 编程式驱动 / headless（2–3 天）

**目标**：用 SDK 以编程方式驱动你的助手，让它能 headless 跑任务（可挂定时）。

**需要读的文档**：
- `source/packages/coding-agent/docs/sdk.md`（全文）
- `source/packages/coding-agent/examples/sdk/README.md`（尤其 01 / 05 / 06 / 11 / 13）

**主任务**：写 TS 脚本，用 `createAgentSession` 让助手无人值守完成一个日常任务。

**可玩成果**：`node/tsx` 跑一个脚本，助手自动完成任务并把结果写到文件。

**子任务**：

1. 新建项目，`npm install @earendil-works/pi-coding-agent`
2. 按 Quick Start 跑通最小示例：订阅 `text_delta` 打印到 stdout
3. 用 `tools` 限制工具集，用 `customTools` / `defineTool` 注入你的工具
4. 把 Stage 4 的能力以编程方式接入，跑一个真实任务
5. 对比 `SessionManager.inMemory()` 与 `create()` 的持久化差异
6. _(进阶)_ 用 cron / launchd 把脚本挂成定时任务

**完成标准**：掌握 `createAgentSession` / `prompt()` / 事件订阅 / 编程式注入 tool 四件套。

---

### Stage 7 · 前端 + 多 agent 体验（3–4 天）

**目标**：给助手包一个简单 Web 前端，并体验多 agent 协作。

**需要读的文档**：
- `source/packages/coding-agent/docs/rpc.md`
- `source/packages/coding-agent/docs/tui.md`
- `source/packages/coding-agent/examples/extensions/subagent/README.md`

**主任务**：前端：用 SDK 或 RPC 包一个简单 Web UI；多 agent：基于 subagent example 配 scout / worker。

**可玩成果**：浏览器里和助手对话；并能让两个 agent 串成 chain 完成一个任务。

**子任务**：

1. 读 `rpc.md` 开头的 Node 提示，决定前端用 SDK（同进程）还是 RPC（进程隔离）
2. 搭一个最简 Web UI：输入框 + 流式输出（SDK 订阅事件 或 RPC 读 stdout）
3. 读 `subagent/README.md`，安装 example 的 extension 和 agents
4. 配置 scout / worker 两个 agent，跑一个 single / chain 工作流
5. _(进阶)_ 把前端和多 agent 接起来：前端触发一个多 agent 任务

**完成标准**：能解释 SDK vs RPC 的选型依据；能让两个 agent 协作完成一个任务。

---

## 主线能力栈总览

```
Phase A（Stage 0-2）：跑通 pi + 用配置塑形成你的品味（零代码）
        ↓
Phase B（Stage 3-5）：用 skill + extension 让它真正会做日常任务（coding）
        ↓
Phase C（Stage 6-7）：SDK headless + 前端 + 多 agent，做成完整体验
```

每个阶段都往同一个「个人助手 agent」上叠加能力，不是孤立的练习。

---

## 参考资源

- 文档入口：`usage/task-reading-matrix.md`
- 中文笔记模板：`skill-draft/notes/_doc-note-template.md`
- 已有笔记：`skill-draft/notes/`
- pi 上游：[earendil-works/pi](https://github.com/earendil-works/pi)
