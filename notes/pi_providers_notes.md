# pi Providers 学习笔记

## 核心概念

- **Provider**：模型来源，比如 `OpenAI`、`Anthropic`、`Google`、`GitHub Copilot` 等
- **Subscription**：订阅登录方式，用 `/login`，适合 `ChatGPT Plus/Pro`、`Claude Pro/Max`、`GitHub Copilot`
- **API key**：用环境变量或 `auth.json` 提供密钥
- **Auth file**：`~/.pi/agent/auth.json`，pi 会把凭证存在这里
- **Cloud providers**：像 `Azure OpenAI`、`Amazon Bedrock`、`Cloudflare AI Gateway` 这类云平台接入
- **Custom providers**：如果官方没直接支持，可以通过 `models.json` 或 `extension` 接入
- **Resolution Order**：pi 找凭证的优先级

## 怎么选 Provider

- 想"登录就能用" → 用 **Subscription**
- 想自己放密钥 → 用 **API key**
- 想接企业云平台 → 看 **Cloud providers**
- 想接特殊后端 → 看 **Custom providers**

## Resolution Order（凭证优先级）

1. CLI `--api-key` flag
2. `auth.json` entry（API key 或 OAuth token）
3. Environment variable
4. Custom provider keys from `models.json`

## 重要规则

- `auth.json` 里的凭证优先级高于环境变量
- `/login` 不只是登录订阅，也能存 API key
- pi 会识别每个 provider 可用的 model 列表
- `auth.json` 文件权限是 `0600`（仅用户可读写）
