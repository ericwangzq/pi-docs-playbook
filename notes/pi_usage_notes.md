# pi 使用学习笔记

## 1. Interactive Mode（交互模式）
- 这是平时直接打开 `pi` 后使用的界面。
- 界面主要分四块：
  - **Startup header**：启动时加载了什么
  - **Messages**：对话区，显示消息、工具调用、报错等
  - **Editor**：输入问题的地方
  - **Footer**：底部状态栏，显示目录、session、token 用量、模型等

### 编辑区常用能力
- `@`：引用文件
- `Tab`：补全路径
- `Shift+Enter`：多行输入
- `!command`：执行命令并把输出发给模型
- `!!command`：执行命令，但不把输出发给模型

## 2. Slash Commands（斜杠命令）
- 在输入框里输入 `/` 打开命令。
- 常见命令：
  - `/login`、`/logout`：登录/退出
  - `/model`：切换模型
  - `/settings`：设置
  - `/resume`：继续旧 session
  - `/new`：新开 session
  - `/tree`、`/fork`、`/clone`：管理 session 分支
  - `/compact`：压缩上下文
  - `/reload`：重新加载配置
  - `/quit`：退出

## 3. Message Queue（消息队列）
- AI 正在工作时，也可以继续发消息。
- `Enter`：发 steering message，当前一轮工具执行完后插入
- `Alt+Enter`：发 follow-up message，等全部做完后再处理
- `Escape`：取消当前输入并恢复
- `Alt+Up`：把队列里的消息取回编辑区

### 简单理解
- **steering**：在当前任务里插一句方向上的调整
- **follow-up**：等它做完后，再补充一件事

## 4. Sessions（会话）
- session 就是一次工作的记录。
- 常见命令：
  - `pi -c`：继续最近一次
  - `pi -r`：挑一个旧 session
  - `--session`：打开指定 session
  - `--fork`：从旧 session 分叉出新 session
  - `--no-session`：不保存

## 5. Context Files（上下文文件）
- pi 启动时会自动读 `AGENTS.md` 和 `CLAUDE.md`。
- 用来告诉 pi：
  - 项目怎么做事
  - 有哪些约束
  - 要跑哪些命令
  - 要遵守哪些规则

## 6. System Prompt Files
- `.pi/SYSTEM.md`：项目级替换系统提示
- `~/.pi/agent/SYSTEM.md`：全局替换
- `APPEND_SYSTEM.md`：在默认提示后追加

## 7. CLI Reference（命令行参数）
### Modes
- 默认：交互模式
- `-p` / `--print`：一次性输出后退出
- `--mode json`：输出 JSON events
- `--mode rpc`：RPC 模式

### Model Options
- `--provider`
- `--model`
- `--thinking`

### Session Options
- `--continue`
- `--resume`
- `--session`
- `--fork`

### Tool Options
- `--tools`
- `--exclude-tools`
- `--no-tools`

### Resource Options
- `--extension`
- `--skill`
- `--prompt-template`
- `--theme`

## 8. Design Principles（设计原则）
- pi 的核心保持小。
- 复杂工作流交给 `extensions`、`skills`、`prompt templates`、`packages`。
- 它不会把所有功能都内置进去。

## 一句话总结
- `pi` 是一个可交互、可脚本化、可嵌入的 AI coding agent。
- 你可以把它当成：**会读文件、跑命令、管理 session 的智能工作助手**。
