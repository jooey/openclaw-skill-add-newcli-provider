# add-newcli-provider

[中文](#中文) | [English](#english)

---

## 中文

### 用最低的成本在 OpenClaw 中使用 Claude

[NewCLI](https://code.newcli.com) 是一个 Anthropic API 兼容的 Claude 代理服务，价格远低于官方 API 直连。本 Skill 让你一键将 NewCLI 接入 OpenClaw，立刻获得 Claude Opus 4.6、Claude Haiku 4.5 等模型的访问能力。

### 为什么选 NewCLI？

- **便宜** — 包月制/低价代理，不按 token 计费，适合高频使用场景
- **兼容** — 完全兼容 Anthropic Messages API，无需改代码
- **稳定** — 支持 Claude 最新模型，持续更新

### 这个 Skill 做了什么？

一个完整的 OpenClaw provider 配置指南，包含：

- Provider 注册与模型定义
- 别名配置（聊天中用 `/model claude-opus` 快速切换）
- Fallback 链接入（主力模型挂了自动切到 Claude）
- 验证与排障流程（踩过的坑都写进去了）

### 支持的模型

| 模型 ID | 名称 | 适用场景 |
|---------|------|----------|
| `claude-opus-4-6` | Claude Opus 4.6 | 复杂推理、长文写作、代码生成 |
| `claude-haiku-4-5-20251001` | Claude Haiku 4.5 | 轻量对话、快速响应 |

> 实际可用模型以你的 NewCLI 账户为准。

### 使用方法

将 `SKILL.md` 放入 OpenClaw 的 skills 目录，对你的 agent 说「配 newcli」或「接入 Claude 模型」即可。

### 注册 NewCLI

**使用我的邀请链接注册：**
https://foxcode.rjj.cc/auth/register?aff=7WTAV8R

---

## English

### Use Claude on OpenClaw at a Fraction of the Cost

[NewCLI](https://code.newcli.com) is an Anthropic API-compatible Claude proxy service, priced significantly lower than the official API. This Skill lets you integrate NewCLI into OpenClaw with minimal effort, giving you instant access to Claude Opus 4.6, Claude Haiku 4.5, and more.

### Why NewCLI?

- **Affordable** — Flat-rate / low-cost proxy pricing instead of per-token billing, ideal for heavy usage
- **Compatible** — Fully compatible with the Anthropic Messages API, no code changes needed
- **Up-to-date** — Supports the latest Claude models with continuous updates

### What Does This Skill Do?

A complete OpenClaw provider configuration guide, covering:

- Provider registration and model definitions
- Alias setup (switch models with `/model claude-opus` in chat)
- Fallback chain integration (auto-fallback to Claude when your primary model is down)
- Validation and troubleshooting (lessons learned from real-world deployment)

### Supported Models

| Model ID | Name | Best For |
|----------|------|----------|
| `claude-opus-4-6` | Claude Opus 4.6 | Complex reasoning, long-form writing, code generation |
| `claude-haiku-4-5-20251001` | Claude Haiku 4.5 | Lightweight conversation, fast responses |

> Actual availability depends on your NewCLI account.

### Quick Start

Place `SKILL.md` in your OpenClaw skills directory and tell your agent to "configure newcli" or "add Claude models".

### Sign Up for NewCLI

**Register with my referral link:**
https://foxcode.rjj.cc/auth/register?aff=7WTAV8R

---

## License

MIT
