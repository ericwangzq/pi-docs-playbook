# pi Quickstart 学习笔记

## 核心概念

- **Install**：先安装 `@earendil-works/pi-coding-agent`
- **Uninstall**：用对应包管理器卸载，但会保留设置、sessions 等数据在 `~/.pi/agent/`
- **Authenticate**：启动前要登录，或配置 API key
- **First session**：进入项目目录后直接运行 `pi`，开始提需求
- **Tools**：默认给模型 4 个工具：`read`、`write`、`edit`、`bash`
- **AGENTS.md**：项目指令文件，告诉 pi 这个项目怎么工作
- **@files**：把文件作为上下文传给 pi
- **!command / !!command**：在对话里跑 shell 命令
- **Session**：pi 会自动保存会话，方便继续
- **Non-interactive mode**：用 `-p` 一次性提问；`--mode json` 和 `--mode rpc` 用于程序集成

## 一句话总结

`quickstart` 就是在教你：**安装 → 登录 → 进项目 → 提问 → 用文件/命令辅助 → 保存并继续 session**。
