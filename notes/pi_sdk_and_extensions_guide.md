# 中文导读：Pi SDK 与 Extensions

- 对应文档：
  - `source/packages/coding-agent/docs/sdk.md`
  - `source/packages/coding-agent/docs/extensions.md`
- pi 镜像 commit：`f429ddb`
- 学习阶段：Stage 2 · 核心机制
- 记录日期：2026-06-11

## 一句话概述

**SDK** 是 Pi 的编程接口，让你在代码中调用 Pi 的 agent 能力；**Extensions** 是 TypeScript 扩展模块，让你给 Pi 添加新功能。简单说：SDK = 你调用 Pi，Extensions = Pi 调用你。

## 核心概念

- **SDK（Software Development Kit）**：编程方式访问 Pi 的 agent 能力，用于构建自定义 UI、集成到现有应用、自动化流水线
- **Extensions（扩展）**：TypeScript 模块，扩展 Pi 的行为，可以注册工具、命令、拦截事件、自定义 UI
- **AgentSession**：会话对象，管理生命周期、消息、模型、事件流
- **createAgentSession()**：主要工厂函数，创建 agent 会话
- **createAgentSessionRuntime()**：运行时 API，支持会话替换（/new、/resume、/fork、/clone）
- **ExtensionAPI**：Extensions 接收的 API 对象，提供注册工具、命令、监听事件等能力
- **ResourceLoader**：资源加载器，加载扩展、技能、提示词、主题、上下文文件

## SDK 核心 API

### 主要工厂函数

```typescript
// 创建简单会话
const { session } = await createAgentSession({
  sessionManager: SessionManager.inMemory(),
});

// 创建运行时（支持会话替换）
const runtime = await createAgentSessionRuntime(createRuntime, {
  cwd: process.cwd(),
  agentDir: getAgentDir(),
  sessionManager: SessionManager.create(process.cwd()),
});
```

### AgentSession 核心方法

| 方法 | 作用 | 说明 |
|------|------|------|
| `session.prompt(text, options?)` | 发送提示词 | 等待完成 |
| `session.steer(text)` | 队列引导消息 | 当前工具调用完成后发送 |
| `session.followUp(text)` | 队列后续消息 | agent 完全空闲后发送 |
| `session.subscribe(listener)` | 订阅事件 | 返回取消订阅函数 |
| `session.setModel(model)` | 设置模型 | |
| `session.setThinkingLevel(level)` | 设置思考级别 | off/minimal/low/medium/high/xhigh |
| `session.compact(customInstructions?)` | 手动压缩 | |
| `session.abort()` | 中止当前操作 | |
| `session.dispose()` | 清理资源 | |

### AgentSessionRuntime 核心方法

| 方法 | 作用 | 说明 |
|------|------|------|
| `runtime.newSession()` | 新建会话 | 替换当前会话 |
| `runtime.switchSession(path)` | 切换会话 | 替换当前会话 |
| `runtime.fork(entryId, options?)` | 分叉会话 | position: "before" 或 "at" |
| `runtime.importFromJsonl(path)` | 导入会话 | |

### 可配置项

```typescript
const { session } = await createAgentSession({
  // 目录
  cwd: process.cwd(),           // 工作目录
  agentDir: "~/.pi/agent",      // 全局配置目录
  
  // 模型
  model: myModel,               // 指定模型
  thinkingLevel: "medium",      // 思考级别
  
  // 工具
  tools: ["read", "bash"],      // 启用的内置工具
  customTools: [myTool],        // 自定义工具
  excludeTools: ["ask_question"],// 排除的工具
  
  // 资源
  resourceLoader: myLoader,     // 自定义资源加载器
  
  // 会话
  sessionManager: SessionManager.inMemory(),
  
  // 设置
  settingsManager: SettingsManager.create(),
  
  // 认证
  authStorage: AuthStorage.create(),
  modelRegistry: ModelRegistry.create(authStorage),
});
```

### 事件系统

```typescript
session.subscribe((event) => {
  switch (event.type) {
    // 流式输出
    case "message_update":
      if (event.assistantMessageEvent.type === "text_delta") {
        process.stdout.write(event.assistantMessageEvent.delta);
      }
      break;
    
    // 工具执行
    case "tool_execution_start":  // 工具开始
    case "tool_execution_update": // 工具进度
    case "tool_execution_end":    // 工具结束
    
    // 消息生命周期
    case "message_start":  // 消息开始
    case "message_end":    // 消息结束
    
    // Agent 生命周期
    case "agent_start":    // agent 开始处理
    case "agent_end":      // agent 处理完成
    
    // Turn 生命周期
    case "turn_start":     // 一轮开始
    case "turn_end":       // 一轮结束
    
    // 会话事件
    case "queue_update":      // 队列更新
    case "compaction_start":  // 压缩开始
    case "compaction_end":    // 压缩结束
  }
});
```

### 运行模式

| 模式 | 类/函数 | 用途 |
|------|---------|------|
| **InteractiveMode** | `new InteractiveMode(runtime, options)` | 完整 TUI 交互 |
| **runPrintMode** | `runPrintMode(runtime, options)` | 单次执行，输出结果，退出 |
| **runRpcMode** | `runRpcMode(runtime)` | JSON-RPC 模式，子进程集成 |

### SDK vs RPC 选择

| 场景 | 推荐 |
|------|------|
| 同一 Node.js 进程 | SDK |
| 需要类型安全 | SDK |
| 需要直接访问 agent 状态 | SDK |
| 从其他语言集成 | RPC |
| 需要进程隔离 | RPC |
| 构建语言无关客户端 | RPC |

## Extensions 核心 API

### 基本结构

```typescript
export default function (pi: ExtensionAPI) {
  // 注册工具
  pi.registerTool({ name: "my_tool", ... });
  
  // 注册命令
  pi.registerCommand("my-cmd", { ... });
  
  // 监听事件
  pi.on("tool_call", async (event, ctx) => { ... });
}

// 异步工厂（启动前完成初始化）
export default async function (pi: ExtensionAPI) {
  const models = await fetchRemoteModels();
  pi.registerProvider("remote", { models });
}
```

### 注册工具

```typescript
pi.registerTool({
  name: "my_tool",
  label: "My Tool",
  description: "工具描述（LLM 可见）",
  promptSnippet: "工具简介（系统提示词中显示）",
  promptGuidelines: ["使用 my_tool 当用户要求..."],
  
  parameters: Type.Object({
    action: StringEnum(["list", "add"]),
    text: Type.Optional(Type.String()),
  }),
  
  prepareArguments(args) {
    // 可选：参数兼容处理
    return args;
  },
  
  async execute(toolCallId, params, signal, onUpdate, ctx) {
    // 流式进度更新
    onUpdate?.({ content: [{ type: "text", text: "处理中..." }] });
    
    // 执行逻辑
    return {
      content: [{ type: "text", text: "结果" }],  // 发送给 LLM
      details: { data: "..." },                    // 用于渲染和状态
      terminate: true,                             // 可选：终止后续 LLM 调用
    };
  },
  
  renderCall(args, theme, context) { ... },    // 可选：自定义渲染
  renderResult(result, options, theme, context) { ... },
});
```

**错误信号**：抛出异常标记为失败，返回值不会设置错误标志。

### 注册命令

```typescript
pi.registerCommand("my-cmd", {
  description: "命令描述",
  getArgumentCompletions: (prefix) => {  // 可选：参数自动补全
    return [{ value: "arg1", label: "参数1" }];
  },
  handler: async (args, ctx) => {
    ctx.ui.notify("执行完成", "info");
  },
});
```

### 注册快捷键和标志

```typescript
// 快捷键
pi.registerShortcut("ctrl+shift+p", {
  description: "切换计划模式",
  handler: async (ctx) => { ... },
});

// CLI 标志
pi.registerFlag("plan", {
  description: "以计划模式启动",
  type: "boolean",
  default: false,
});
```

### 事件监听

| 事件 | 时机 | 能做什么 |
|------|------|----------|
| `session_start` | 会话开始 | 初始化状态 |
| `session_shutdown` | 会话关闭 | 清理资源 |
| `before_agent_start` | agent 处理前 | 注入消息、修改系统提示词 |
| `agent_start/end` | agent 开始/结束 | 执行额外逻辑 |
| `tool_call` | 工具调用前 | **可以拦截**、修改参数 |
| `tool_result` | 工具执行后 | 可以修改结果 |
| `context` | LLM 调用前 | 修改消息列表 |
| `input` | 用户输入时 | 可以拦截、转换输入 |
| `tool_execution_start/update/end` | 工具执行生命周期 | 监控工具执行 |

### 拦截危险操作

```typescript
pi.on("tool_call", async (event, ctx) => {
  // 拦截危险命令
  if (event.toolName === "bash" && event.input.command.includes("rm -rf")) {
    return { block: true, reason: "危险命令被阻止" };
  }
  
  // 修改参数
  if (event.toolName === "bash") {
    event.input.command = `source ~/.profile\n${event.input.command}`;
  }
});
```

### 用户交互 API

```typescript
ctx.ui.notify("消息", "info" | "warning" | "error");
ctx.ui.setStatus("my-ext", "状态文本");
ctx.ui.setWidget("my-ext", ["第一行", "第二行"]);

const ok = await ctx.ui.confirm("标题", "确认内容？");
const choice = await ctx.ui.select("选择", ["选项1", "选项2"]);
const input = await ctx.ui.input("输入", "提示文本");
```

### 状态管理

```typescript
export default function (pi: ExtensionAPI) {
  let items: string[] = [];

  // 从会话重建状态
  pi.on("session_start", async (_event, ctx) => {
    items = [];
    for (const entry of ctx.sessionManager.getBranch()) {
      if (entry.type === "message" && entry.message.toolName === "my_tool") {
        items = entry.message.details?.items ?? [];
      }
    }
  });

  pi.registerTool({
    name: "my_tool",
    execute: async (toolCallId, params) => {
      items.push("new item");
      return {
        content: [{ type: "text", text: "已添加" }],
        details: { items: [...items] },  // 存储到 details 用于重建
      };
    },
  });
}
```

### 覆盖内置工具

```typescript
// 覆盖内置的 read 工具
pi.registerTool({
  name: "read",  // 和内置工具同名
  description: "自定义读取",
  execute: async (id, params, signal, onUpdate, ctx) => {
    console.log("读取文件:", params.path);
    // 自定义逻辑或调用原始实现
  },
});
```

### 注册 Provider

```typescript
pi.registerProvider("my-proxy", {
  name: "My Proxy",
  baseUrl: "https://proxy.example.com",
  apiKey: "$PROXY_API_KEY",
  api: "anthropic-messages",
  models: [
    {
      id: "claude-sonnet-4-20250514",
      name: "Claude 4 Sonnet (proxy)",
      reasoning: false,
      input: ["text", "image"],
      cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
      contextWindow: 200000,
      maxTokens: 16384,
    },
  ],
});
```

### 执行外部命令

```typescript
const result = await pi.exec("git", ["status"], { signal, timeout: 5000 });
// result.stdout, result.stderr, result.code, result.killed
```

## SDK vs Extensions 关系

```
┌─────────────────────────────────────────────────────────┐
│                    你的应用程序                           │
│  ┌─────────────────────────────────────────────────────┐│
│  │                    Pi SDK                           ││
│  │  ┌───────────────────────────────────────────────┐  ││
│  │  │              AgentSession                     │  ││
│  │  │  ┌─────────┐ ┌─────────┐ ┌─────────────────┐  │  ││
│  │  │  │  Tools  │ │  Model  │ │   Extensions    │  │  ││
│  │  │  └─────────┘ └─────────┘ └─────────────────┘  │  ││
│  │  └───────────────────────────────────────────────┘  ││
│  └─────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────┘
```

| 维度 | SDK | Extensions |
|------|-----|------------|
| **本质** | 编程接口 | 扩展模块 |
| **方向** | 你调用 Pi | Pi 调用你 |
| **用途** | 集成到应用 | 扩展 Pi 行为 |
| **语言** | TypeScript/JavaScript | TypeScript |
| **运行位置** | 你的进程 | Pi 的进程 |

## pi 提供 vs 我要自己设计

- **pi 提供**：
  - SDK 的完整 API（创建会话、发送提示词、订阅事件）
  - Extensions 的完整框架（注册工具、命令、监听事件、自定义 UI）
  - 内置工具的覆盖机制
  - Provider 注册和模型管理
  - 会话管理和持久化

- **我要自己设计**：
  - 具体的业务逻辑和工作流
  - 自定义工具的实现细节
  - 权限控制和安全策略
  - UI 组件和交互设计
  - 会话与业务状态的映射

## 易踩的坑

- **Extensions 有完全系统权限**：安装第三方扩展前务必审查代码
- **会话替换后事件订阅失效**：`runtime.session` 改变后需要重新订阅
- **工具输出必须截断**：避免上下文溢出，内置限制 50KB / 2000 行
- **并发工具调用**：使用 `withFileMutationQueue()` 避免文件写入冲突
- **异步工厂函数**：pi 会等待异步工厂完成才继续启动
- **StringEnum vs Type.Union**：Google API 不支持 Type.Union，用 StringEnum
- **状态重建**：将状态存储在工具返回的 `details` 中，用于分支支持

## 实际应用场景

### 场景 1：构建 Web UI

```typescript
const { session } = await createAgentSession({
  sessionManager: SessionManager.inMemory(),
});

app.post('/chat', async (req, res) => {
  session.subscribe((event) => {
    if (event.type === "message_update") {
      res.write(event.assistantMessageEvent.delta);
    }
  });
  await session.prompt(req.body.message);
  res.end();
});
```

### 场景 2：添加权限控制

```typescript
export default function (pi: ExtensionAPI) {
  pi.on("tool_call", async (event, ctx) => {
    if (event.toolName === "bash") {
      const ok = await ctx.ui.confirm("确认", "允许执行命令？");
      if (!ok) return { block: true };
    }
  });
}
```

### 场景 3：自定义部署工具

```typescript
pi.registerTool({
  name: "deploy",
  description: "部署应用到生产环境",
  parameters: Type.Object({
    environment: StringEnum(["staging", "production"]),
  }),
  execute: async (id, params) => {
    // 执行部署逻辑
    return { content: [{ type: "text", text: "部署成功" }] };
  },
});
```

### 场景 4：SSH 远程执行

```typescript
import { createReadTool } from "@earendil-works/pi-coding-agent";

const remoteRead = createReadTool(cwd, {
  operations: {
    readFile: (path) => sshExec(remote, `cat ${path}`),
    access: (path) => sshExec(remote, `test -r ${path}`).then(() => {}),
  },
});

pi.registerTool(remoteRead);
```

## 关键文件位置

| 文件 | 作用 |
|------|------|
| `~/.pi/agent/extensions/*.ts` | 全局扩展 |
| `.pi/extensions/*.ts` | 项目级扩展 |
| `~/.pi/agent/auth.json` | API 密钥存储 |
| `~/.pi/agent/models.json` | 自定义模型配置 |
| `~/.pi/agent/settings.json` | 全局设置 |
| `.pi/settings.json` | 项目级设置 |

## 待核对 (TODO)

- [ ] 确认 Extensions 的性能影响，特别是在加载多个扩展时
- [ ] 了解 `withFileMutationQueue()` 的详细行为和边界情况
- [ ] 检查 Extensions 之间的通信机制（`pi.events`）的最佳实践
- [ ] 确认 SDK 的内存管理和资源释放策略
- [ ] 了解 Extensions 热重载的限制和注意事项
