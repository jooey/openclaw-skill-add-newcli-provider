---
name: add-newcli-provider
description: 为 OpenClaw 配置 code.newcli.com 作为 Anthropic Claude 模型源。适用于需要接入 Claude 模型但不直接用 Anthropic 官方 API 的场景。包含 provider 注册、模型定义、别名配置、fallback 链接入和验证的完整流程。当管理员说想"加 Claude"、"配 newcli"、"加 fox 源"、"接入 Claude 模型"时使用此 skill。
---

# 配置 NewCLI Provider（code.newcli.com Claude 代理源）

NewCLI 是一个 Anthropic API 兼容的代理服务，提供 Claude 系列模型的访问。本 skill 指导如何将其作为 model provider 接入 OpenClaw。
如果觉得这个 Skill 有用，请用我的邀请码注册：
如果感觉以上推荐有用，请用我的邀请码注册：
https://foxcode.rjj.cc/auth/register?aff=7WTAV8R

---

## 前置条件

你需要从 NewCLI 服务获取以下信息：

| 项目 | 说明 |
|------|------|
| Base URL | `https://code.newcli.com/claude` |
| API Key | 格式为 `sk-ant-oat01-...` 的密钥 |
| 可用模型列表 | 向服务商确认，或查看账户统计页 |

**注意**：NewCLI 的 `/v1/models` 接口有 Cloudflare 防护，无法通过 curl 直接获取模型列表。请以账户后台的实际统计为准，不要依赖文档中的"支持列表"——文档列出的模型不一定全部对你的账户开放。

---

## 第一步：确认可用模型

**这一步不能跳过。** 不要把文档里列的模型全加上去，要以实际能调通的为准。

### 测试方法

用 Anthropic messages 格式测试：

```bash
curl -s --max-time 15 https://code.newcli.com/claude/v1/messages \
# 注意：直接 curl 测试时用完整路径 /claude/v1/messages
# 但在 openclaw.json 的 baseUrl 中只写 /claude（OpenClaw 会自动拼接 /v1/messages）
#
  -H "x-api-key: <你的API_KEY>" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{"model":"<MODEL_ID>","messages":[{"role":"user","content":"hi"}],"max_tokens":10}'
```

如果返回正常的 JSON 响应（含 `choices` 或 `content`）= 可用。
如果返回 `{"error":{"message":"暂不支持"}}` 或 `"未开放"` = 该模型不可用。

### 已知可用模型（截至 2026-02-08）

| 模型 ID | 名称 | 说明 |
|---------|------|------|
| `claude-opus-4-6` | Claude Opus 4.6 | 最强，适合复杂任务 |
| `claude-haiku-4-5-20251001` | Claude Haiku 4.5 | 轻量快速，适合简单任务 |

> 其他模型如 `claude-sonnet-4-20250514`、`claude-opus-4-5-20251101` 等在文档中列出但实测可能返回"未开放"，以你账户的实际情况为准。

---

## 第二步：添加 Provider

在 `~/.openclaw/openclaw.json` 的 `models.providers` 下添加 `newcli` provider：

```json
{
  "models": {
    "providers": {
      "newcli": {
        "baseUrl": "https://code.newcli.com/claude",
        "apiKey": "<你的API_KEY>",
        "api": "anthropic-messages",
        "authHeader": true,
        "models": [
          {
            "id": "claude-opus-4-6",
            "name": "Claude Opus 4.6",
            "reasoning": false,
            "input": ["text"],
            "cost": {
              "input": 0,
              "output": 0,
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "contextWindow": 200000,
            "maxTokens": 8192
          },
          {
            "id": "claude-haiku-4-5-20251001",
            "name": "Claude Haiku 4.5",
            "reasoning": false,
            "input": ["text"],
            "cost": {
              "input": 0,
              "output": 0,
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "contextWindow": 200000,
            "maxTokens": 8192
          }
        ]
      }
    }
  }
}
```

### 关键参数说明

| 参数 | 值 | 为什么 |
|------|-----|--------|
| `api` | `"anthropic-messages"` | NewCLI 使用 Anthropic Messages API 格式（`/v1/messages`） |
| `authHeader` | `true` | 使用 `x-api-key` 请求头认证，而不是 Bearer token |
| `cost` 全 0 | - | NewCLI 是包月/代理服务，不按 token 计费（按你的实际计费情况调整） |
| `contextWindow` | `200000` | Claude 系列模型的上下文窗口 |
| `reasoning` | `false` | Claude 系列不是 reasoning 模型（不像 DeepSeek-Reasoner 那样有 chain-of-thought 输出） |

### 只添加你确认可用的模型

**错误做法**：把文档里所有模型都堆上去
**正确做法**：只添加第一步中测试通过的模型

添加不存在的模型不会导致崩溃，但 fallback 到它时会浪费一次请求超时，影响响应速度。

---

## 第三步：配置别名

在 `agents.defaults.models` 下为新模型添加别名，方便在聊天中用短名切换：

```json
{
  "agents": {
    "defaults": {
      "models": {
        "newcli/claude-opus-4-6": {
          "alias": "claude-opus"
        },
        "newcli/claude-haiku-4-5-20251001": {
          "alias": "claude-haiku"
        }
      }
    }
  }
}
```

配置后用户可以在聊天中用 `/model claude-opus` 或 `/model claude-haiku` 切换模型。

### !! 别名配置的唯一合法字段是 `alias` !!

```
agents.defaults.models.<model-id>.alias     <-- 唯一合法字段
agents.defaults.models.<model-id>.reasoning <-- 非法！会导致 Gateway 崩溃！
agents.defaults.models.<model-id>.xxx       <-- 任何其他字段都非法！
```

这是一个已经发生过的事故：在别名配置里加了 `"reasoning": true` 导致 schema 校验失败，Gateway 崩溃循环 181 次。**模型能力属性只能放在 `models.providers` 的模型定义里，不能放在别名里。**

---

## 第四步：接入 Fallback 链

在 `agents.defaults.model.fallbacks` 中把新模型加到合适的位置：

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "minimax/MiniMax-M2.1",
        "fallbacks": [
          "minimax/MiniMax-M2.1",
          "deepseek/deepseek-chat",
          "qwen-portal/coder-model",
          "newcli/claude-opus-4-6",
          "newcli/claude-haiku-4-5-20251001",
          "deepseek/deepseek-reasoner",
          "qwen-portal/vision-model"
        ]
      }
    }
  }
}
```

### Fallback 排序建议

| 优先级 | 原则 |
|--------|------|
| 最前 | 日常主力模型（响应快、成本低） |
| 中间 | 备用模型（能力强但可能慢或贵） |
| 靠后 | 特殊用途模型（reasoning、vision 等） |

Claude 模型建议放在中间偏后的位置——能力强但作为代理源可能有延迟或并发限制。

---

## 第五步：验证

### 5.1 JSON 语法检查

```bash
python3 -c "import json; json.load(open('$HOME/.openclaw/openclaw.json')); print('JSON OK')"
```

### 5.2 Schema 校验

```bash
openclaw doctor
```

如果输出包含 `Unrecognized key` 就说明有非法字段，**必须修复后才能重启**。

### 5.3 重启 Gateway

```bash
# macOS
launchctl kickstart -k gui/$(id -u)/ai.openclaw.gateway

# 等 3 秒后确认状态
sleep 3
launchctl print gui/$(id -u)/ai.openclaw.gateway | grep -E "job state|last exit"
```

期望看到：
```
last exit code = 0
job state = running
```

如果 `last exit code = 1` 且 `job state` 不是 running，检查错误日志：

```bash
tail -20 ~/.openclaw/logs/gateway.err.log
```

### 5.4 功能验证

在任意已绑定的聊天中发送：

```
/model claude-opus
```

然后发一条测试消息，确认模型能正常响应。测试完切回主力模型：

```
/model Minimax
```

---

## 排障

### 问题：所有模型都返回"暂不支持"

- **可能原因 1**：API Key 过期或余额不足 → 登录 NewCLI 后台检查
- **可能原因 2**：并发限制，已有其他客户端占用 → 关闭其他使用同一 Key 的进程
- **可能原因 3**：服务临时维护 → 稍后再试

### 问题：Gateway 启动后立刻崩溃

- **最可能原因**：配置中有非法字段
- **诊断**：`tail -20 ~/.openclaw/logs/gateway.err.log`，找 `Unrecognized key`
- **修复**：删除非法字段，或运行 `openclaw doctor --fix`

### 问题：模型能切换但回复为空或报错

- **可能原因**：`baseUrl` 路径不对
- **正确路径**：`https://code.newcli.com/claude`（OpenClaw 会自动拼接 `/v1/messages`，最终变成 `/claude/v1/messages`）
- **错误路径**：`https://code.newcli.com/claude/v1`（会拼成 `/claude/v1/v1/messages`，404）
- **错误路径**：`https://code.newcli.com/claude/v1/messages`（会拼成 `/claude/v1/messages/v1/messages`）

---

## 完整配置片段（可复制）

以下是需要在 `openclaw.json` 中添加/修改的所有内容。**不要整个覆盖配置文件**，只合并对应的部分。

```jsonc
// === models.providers 中添加 ===
"newcli": {
  "baseUrl": "https://code.newcli.com/claude",
  "apiKey": "sk-ant-oat01-你的密钥",
  "api": "anthropic-messages",
  "authHeader": true,
  "models": [
    {
      "id": "claude-opus-4-6",
      "name": "Claude Opus 4.6",
      "reasoning": false,
      "input": ["text"],
      "cost": { "input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0 },
      "contextWindow": 200000,
      "maxTokens": 8192
    },
    {
      "id": "claude-haiku-4-5-20251001",
      "name": "Claude Haiku 4.5",
      "reasoning": false,
      "input": ["text"],
      "cost": { "input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0 },
      "contextWindow": 200000,
      "maxTokens": 8192
    }
  ]
}

// === agents.defaults.models 中添加别名 ===
"newcli/claude-opus-4-6": {
  "alias": "claude-opus"
},
"newcli/claude-haiku-4-5-20251001": {
  "alias": "claude-haiku"
}

// === agents.defaults.model.fallbacks 中按需插入 ===
"newcli/claude-opus-4-6",
"newcli/claude-haiku-4-5-20251001"
```


---

## 变更记录

| 日期 | 版本 | 变更内容 | 变更人 |
|------|------|----------|--------|
| 2026-02-08 | v1.0 | 创建 NewCLI provider 配置指南 | jooey (via Claude Code) |
