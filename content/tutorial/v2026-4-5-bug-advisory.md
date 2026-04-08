# OpenClaw v2026.4.5 紧急 Bug 通报与 Workaround 指南

> v2026.4.5 发布后 24 小时内集中爆发的 6 个 Bug 分析与临时解决方案

---

## 元信息

| 字段 | 内容 |
|------|------|
| **title** | OpenClaw v2026.4.5 紧急 Bug 通报与 Workaround 指南 |
| **description** | 详尽分析 v2026.4.5 发布后的 6 个 Critical/High Bug（Anthropic 503 重试循环、MiniMax TTS 参数类型、MiniMax TTS 整数参数、Codex Cloudflare 403、GPT-5 max_tokens 废弃、launchd token drift），提供经过验证的临时解决方案。 |
| **tags** | OpenClaw, Bug Advisory, v2026.4.5, Troubleshooting, Workaround |
| **难度** | 🔴 高 |
| **预计阅读时间** | 15 分钟 |
| **date** | 2026-04-08 |
| **author** | 墨客-内容生成专家 |
| **紧急度** | 🔴 Critical — 立即阅读 |

---

## 目录

1. [概述](#1-概述)
2. [🔴 Critical Bug](#2-critical-bug)
3. [🟠 High Priority Bug](#3-high-priority-bug)
4. [Bug Workaround 速查表](#4-bug-workaround-速查表)
5. [修复状态追踪](#5-修复状态追踪)

---

## 1. 概述

v2026.4.5 于 **2026-04-06** 发布后，24 小时内集中涌现 6 个 Bug（3 Critical + 3 High），本通报旨在：

1. 帮助受影响的用户快速找到临时解决方案
2. 为开发者提供问题根因分析
3. 追踪官方修复进度

**影响范围评估：**

| Bug | 影响用户比例 | 阻断程度 |
|-----|------------|---------|
| #62141 Anthropic 503 | 高（所有 Anthropic 用户）| 🔴 完全卡死 |
| #62144 MiniMax TTS | 中（MiniMax TTS 用户）| 🔴 功能不可用 |
| #62142 Codex Cloudflare 403 | 低（Codex 用户）| 🔴 功能不可用 |
| #62130 GPT-5.x 400 | 中（GPT-5.x 用户）| 🔴 完全阻断 |
| #62140 launchd token | 低（v2026.4.5 升级用户）| 🟠 维护窗口 |
| #62137 exec crash | 低（多 Agent 并发用户）| 🟠 随机崩溃 |

---

## 2. 🔴 Critical Bug

### 🟥 Bug #62141 — Anthropic overloaded_error 无限重试

**严重度**: Critical  
**Issue**: [#62141](https://github.com/openclaw/openclaw/issues/62141)  
**状态**: OPEN  
**影响**: 所有使用 Claude Sonnet/Opus 的用户，Anthropic API 503 时完全卡死

#### 问题本质

OpenClaw 2026.4.5 的模型降级逻辑在收到 `overloaded_error` (503) 时，不能正确识别该错误为"临时过载"，而是将其当作永久失败，导致对同一 provider 的无限重试循环。正常应该立即切换到下一个 fallback 模型。

#### 问题验证

```bash
# 检查是否遇到 503 overloaded_error
grep -i "overloaded_error\|503" ~/.openclaw/logs/gateway.log | tail -20
```

#### Workaround ✅ 有效

**方案一：临时切换 Primary Model（推荐）**

```yaml
# ~/.openclaw/config.yaml
models:
  default: "openai/gpt-4.1"
  # 暂时移除 Anthropic 作为 primary
```

**方案二：修改 Fallback 列表**

```yaml
# ~/.openclaw/config.yaml
models:
  fallback:
    - "openai/gpt-4.1"
    # 暂时从 fallback 中移除 Anthropic
    # 等待官方修复后再恢复
```

**方案三：使用环境变量**

```bash
export OPENCLAW_DEFAULT_MODEL="openai/gpt-4.1"
```

#### 修复建议

fallback 决策引擎应将 `overloaded_error` (503) 归类为"可恢复的临时错误"，立即触发 fallback，而不是重试当前模型。

---

### 🟥 Bug #62144 — MiniMax TTS: API 2013 错误（float vs int）

**严重度**: Critical（对使用 MiniMax TTS 的用户）  
**Issue**: [#62144](https://github.com/openclaw/openclaw/issues/62144)  
**状态**: OPEN  
**影响**: MiniMax TTS 完全不可用（100% 失败率）

#### 问题本质

OpenClaw 向 MiniMax TTS API 发送的 `voice_setting` 参数中，`speed`、`vol`、`pitch` 为浮点数（如 `1.0`、`0.0`），但 MiniMax API 要求整数（`1`、`0`）。API 返回 2013 invalid params 错误。

#### 问题验证

```bash
# 测试 MiniMax TTS（会失败）
openclaw tts convert --text "Hello" --provider minimax --voice "Fox"
# 预期错误：2013 invalid params
```

#### Workaround ✅ 有效

**切换到 OpenAI TTS（推荐）：**

```yaml
# ~/.openclaw/config.yaml
messages:
  tts:
    provider: "openai"
    # 而非 minimax
```

**或使用 ElevenLabs：**

```yaml
messages:
  tts:
    provider: "elevenlabs"
    voice: "your-voice-id"
```

#### 修复建议

在 minimax TTS provider 代码中，将 `speed`/`vol`/`pitch` 强制转为整数：

```javascript
voice_setting: {
  speed: parseInt(speed),  // 1.0 → 1
  vol: parseInt(vol),       // 1.0 → 1  
  pitch: parseInt(pitch)    // 0.0 → 0
}
```

---

### 🟥 Bug #62142 — openai-codex Cloudflare 403（headless HTTP）

**严重度**: Critical（对使用 openai-codex 的用户）  
**Issue**: [#62142](https://github.com/openclaw/openclaw/issues/62142)  
**状态**: OPEN  
**影响**: openai-codex/gpt-5.4 和 gpt-5.4-mini 完全不可用

#### 问题本质

v2026.4.5 升级后，openai-codex provider 的 HTTP 请求头（`originator: pi` + `User-Agent: pi (linux; x86_64)`）被 Cloudflare 识别为 headless 爬虫，触发 403 JS 挑战页。

**额外问题**:
1. OAuth 认证流程会写入无效字段 `token`/`refreshToken`，导致 schema 校验失败
2. 每次重新认证会创建重复 profile（email-based），导致双重超时

#### 问题验证

```bash
# 测试 Codex（会触发 403）
openclaw infer model run --model "openai-codex/gpt-5.4" --prompt "test"
# 预期：403 Forbidden
```

#### Workaround ⚠️ 部分有效

**方案一：切换 Primary Model（推荐）**

```yaml
# ~/.openclaw/config.yaml
models:
  default: "anthropic/claude-sonnet-4-6"
  # 而非 openai-codex
```

**方案二：修复 OAuth Schema 问题**

```bash
# 删除重复 profile
rm ~/.openclaw/agents/<name>/agent/auth-profiles.json
# 重新认证
openclaw auth login
```

**方案三：每次 re-auth 后运行 doctor**

```bash
openclaw doctor --fix
# 注意：下次 re-auth 又会重新出现同样问题
```

---

## 3. 🟠 High Priority Bug

### 🟠 Bug #62130 — max_tokens 参数废弃导致 GPT-5.x 全部 400 错误

**严重度**: High  
**Issue**: [#62130](https://github.com/openclaw/openclaw/issues/62130)  
**状态**: OPEN  
**影响**: 所有 GPT-5.2、GPT-5.4 用户无法使用 OpenAI 模型

#### 问题本质

OpenAI GPT-5.x API 不再接受 `max_tokens` 参数，需要改用 `max_completion_tokens`。OpenClaw 2026.4.5 仍在发送旧参数，导致所有 GPT-5.x 请求 400 报错。

#### 问题验证

```bash
# 测试 GPT-5.x（会返回 400）
openclaw infer model run --model "openai/gpt-5.4" --prompt "Hello"
# 预期：400 Bad Request
```

#### Workaround ❌ 不工作

用户尝试在配置中添加兼容参数但 OpenClaw 2026.4.5 完全忽略该配置：

```yaml
# 此配置在 v2026.4.5 中无效
models:
  providers:
    openai:
      models:
        - name: "gpt-5.4"
          compat:
            maxTokensField: "max_completion_tokens"
```

**建议**：等待官方修复，或降级至 GPT-4.x：

```yaml
models:
  default: "openai/gpt-4o"
```

---

### 🟠 Bug #62140 — launchd gateway token drift

**严重度**: High  
**Issue**: [#62140](https://github.com/openclaw/openclaw/issues/62140)  
**状态**: OPEN  
**影响**: v2026.4.5 更新后，config migration 需要 doctor repair，但修复后又引入 launchd metadata token 漂移

#### 问题本质

`openclaw gateway install` 和 `openclaw doctor --fix` 在 `gateway.auth.token` 已通过 env file 或 SecretRef 管理的情况下，仍会将 `OPENCLAW_GATEWAY_TOKEN` 嵌入 LaunchAgent plist，造成双重 token 来源，导致下次修复时再次触发维护窗口。

#### 问题验证

```bash
# 检查 plist 中的 token
cat ~/Library/LaunchAgents/ai.openclaw.gateway.plist | grep -A2 OPENCLAW_GATEWAY_TOKEN
```

#### Workaround ✅ 有效

**手动清理 plist token：**

```bash
# 1. 检查当前 token 来源
cat ~/.openclaw/config.yaml | grep token

# 2. 如果 runtime token 来自 env file，删除 plist 中的重复嵌入
# 编辑 plist 移除 OPENCLAW_GATEWAY_TOKEN 环境变量

# 3. 重载 launchd
launchctl unload ~/Library/LaunchAgents/ai.openclaw.gateway.plist
launchctl load ~/Library/LaunchAgents/ai.openclaw.gateway.plist
```

---

### 🟠 Bug #62137 — Unhandled promise rejection 导致 gateway 崩溃

**严重度**: High  
**Issue**: [#62137](https://github.com/openclaw/openclaw/issues/62137)  
**状态**: OPEN  
**影响**: 频繁运行 background exec 的多 Agent 用户（4+ Discord bot），gateway 会随机崩溃

#### 问题本质

Agent run 完成后，background exec session 仍在 PTY 中运行。当它 emit stdout 时，`onUpdate` 回调触发 `Agent.processEvents`，但此时已经没有 active run，导致 unhandled promise rejection，gateway 进程崩溃。

**触发条件**: 
- 多个 agent 并发运行
- heartbeat-driven cron spawn exec sessions
- exec 在 agent run 结束后继续产生输出

#### 问题验证

```bash
# 检查 gateway crash 日志
grep -i "unhandled promise rejection\|ECONNRESET" ~/.openclaw/logs/gateway.log | tail -10
```

#### Workaround ❌ 无有效 workaround

只能手动重启 gateway。systemd 无法可靠自动重启：

```bash
# 手动重启
openclaw gateway restart

# 检查是否再次崩溃
tail -f ~/.openclaw/logs/gateway.log
```

---

## 4. Bug Workaround 速查表

| Bug | 受影响功能 | Workaround | 有效性 | 推荐度 |
|-----|----------|-----------|--------|--------|
| #62141 Anthropic 503 | Claude Sonnet/Opus | 切换 primary model 到 GPT-4.1 | ✅ 有效 | ⭐⭐⭐⭐⭐ |
| #62144 MiniMax TTS | MiniMax TTS | 切换到 OpenAI/ElevenLabs TTS | ✅ 有效 | ⭐⭐⭐⭐ |
| #62142 Codex Cloudflare 403 | openai-codex | 切换到 claude-sonnet-4-6 | ⚠️ 部分有效 | ⭐⭐⭐ |
| #62130 GPT-5.x 400 | GPT-5.2/5.4 | 降级到 GPT-4o | ❌ 不工作 | ⭐ |
| #62140 launchd token | Gateway | 手动清理 plist token | ✅ 有效 | ⭐⭐⭐⭐ |
| #62137 exec crash | Gateway | 手动重启 | ❌ 临时 | ⭐⭐ |

---

## 5. 修复状态追踪

| Bug | 官方状态 | 预计修复版本 | 追踪链接 |
|-----|---------|------------|---------|
| #62141 Anthropic 503 | 🟡 确认中 | v2026.4.6 | [GitHub #62141](https://github.com/openclaw/openclaw/issues/62141) |
| #62144 MiniMax TTS | 🟡 确认中 | v2026.4.6 | [GitHub #62144](https://github.com/openclaw/openclaw/issues/62144) |
| #62142 Codex Cloudflare 403 | 🟡 确认中 | 待定 | [GitHub #62142](https://github.com/openclaw/openclaw/issues/62142) |
| #62130 GPT-5.x 400 | 🟡 确认中 | 待定 | [GitHub #62130](https://github.com/openclaw/openclaw/issues/62130) |
| #62140 launchd token | 🟡 确认中 | 待定 | [GitHub #62140](https://github.com/openclaw/openclaw/issues/62140) |
| #62137 exec crash | 🟡 确认中 | 待定 | [GitHub #62137](https://github.com/openclaw/openclaw/issues/62137) |

---

## 📝 更新日志

| 日期 | 版本 | 更新内容 |
|------|------|---------|
| 2026-04-08 | v1.0 | 初始版本，涵盖 v2026.4.5 发布后 6 个 Bug |
| 2026-04-06 | v2026.4.5 | 官方发布（引入上述 Bug） |

---

*本文档由墨客内容生成系统基于 GitHub Issues 调研自动生成*  
*数据来源：GitHub Issues #62141, #62144, #62142, #62130, #62140, #62137*  
*生成时间：2026-04-08 16:08 CST*
