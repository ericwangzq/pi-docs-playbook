# 中文导读笔记 (notes)

这个目录用来沉淀「边学 pi 边写」的中文导读笔记，对应 `usage/how-to-use-this-repo.md` 里的 **Lesson Capture** 工作模式。

它解决一个具体问题：`source/` 里的 pi 上游文档是英文，读起来吃力。与其反复重读英文，不如每读完一篇就用中文把要点固定下来。

## 规则

- 只在本目录写笔记，**绝不修改 `source/`**。`source/` 是上游镜像快照。
- 技术术语保留英文：`tool_call`、`session`、`compaction`、`extension`、`skill`、`prompt template` 等，不要翻译，方便和原文对应。
- 每篇笔记都标注它对应的 `source/...` 路径和 pi 镜像 commit（见根目录 `README.md`，当前 `f429ddb`）。
- 笔记是你的理解，不是上游保证。拿不准的地方标 TODO，回头核对原文或最新上游。

## 推荐工作流

1. 用 agent 把文档讲成中文，顺便练 pi：

   ```bash
   pi @source/packages/coding-agent/docs/sdk.md "用中文讲解这篇文档的核心概念，API 名称保留英文"
   ```

2. 复制 `_doc-note-template.md`，重命名为对应文档名（如 `sdk.md` 的笔记叫 `sdk.note.md`）。
3. 按模板填写：一句话概述、核心概念、关键 API、pi 提供 vs 我要自己设计、易踩的坑、给我的项目的启示。
4. 在学习计划对应阶段勾掉「产出中文笔记」这一步。

## 命名约定

- 文档笔记：`<原文件名>.note.md`，例如 `extensions.note.md`、`session-format.note.md`。
- 跨文档的主题总结：`topic-<主题>.md`，例如 `topic-sdk-vs-rpc.md`。
- 模板文件以 `_` 开头（`_doc-note-template.md`），排序时排在前面，也表示「不是正式笔记」。
