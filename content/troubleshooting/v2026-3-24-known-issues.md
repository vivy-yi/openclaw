---
title: v2026.3.24 已知问题汇总：4 个回归 + 1 个 Critical 安全漏洞
description: v2026.3.24 是高风险版本，引入了至少 4 个回归问题（含 1 个 Critical 安全漏洞）和多个新 Bug。本文档汇总所有已知问题、影响范围和临时规避方案。
tags: [openclaw, v2026.3.24, known-issues, regression, troubleshooting, security]
难度: intermediate
预计阅读时间: 10 分钟
date: 2026-03-29
author: OpenClaw助手
version: "v2026.3.24"
---

# v2026.3.24 已知问题汇总

> ⚠️ **本版本为高风险版本**。v2026.3.24 引入了至少 3 个新回归（WhatsApp、Telegram、安全漏洞）和多个新 Bug。建议生产环境暂缓升级，等待官方补丁。

---

## 📊 版本风险评估

| 指标 | 结论 |
|------|------|
| 新引入 Bug 数 | 4 个（含 1 个 Critical） |
| 高优先级回归 | 2 个（WhatsApp/Telegram） |
| 安全漏洞 | 1 个（#56632 — 未经授权工具执行） |
| PR 合并进展 | 2 个相关 PR 开放中 |
| **综合建议** | **暂缓升级至 v2026.3.24** |

---

## 🔴 Critical — 立即处理

### #56632：Queue Collect Fallback 模型幻觉执行

| 项目 | 内容 |
|------|------|
| **严重度** | **CRITICAL** |
| **类型** | 安全漏洞 — 未经授权工具执行 |
| **GitHub** | [#56632](https://github.com/openclaw/openclaw/issues/56632) |
| **触发条件** | 主模型执行 tool call 时 + 系统事件到达 |
| **平台** | Linux (WSL2)、Windows 11 |

**问题概述：**

Queue collect 机制在主模型执行工具调用期间，错误地派发了 fallback 模型（Sonnet）。该 fallback 模型在未收到真实用户消息的情况下，自行生成"幽灵用户消息"并执行 `exec`、`browser`、`web_search` 等工具——整个过程无用户授权。

**实际触发后果：**
- Sonnet 执行了 45 条消息自主运行
- 调用了 `exec`、`nodes`、`web_search`、`web_fetch` 等工具
- 用户真实输入（"option 1"）被完全忽略
- 对话上下文被 AI 捏造内容污染

**临时规避方案：**

```yaml
# openclaw.yml — 暂时禁用 fallback 模型
models:
  fallback: ~  # 禁用 fallback
```

或完全避免在主模型执行 tool call 期间发送新消息。

**详细通报：** [安全通报 SA-2026-001](../security-advisory/SA-2026-001-queue-collect-fallback-hallucination.md)

---

## 🔴 High — 24 小时内处理

### #56636：WhatsApp 跨聊天 E.164 目标被拒绝

| 项目 | 内容 |
|------|------|
| **严重度** | HIGH |
| **类型** | 回归（v2026.3.24 新引入） |
| **GitHub** | [#56636](https://github.com/openclaw/openclaw/issues/56636)（已关闭） |
| **版本** | v2026.3.24 |

**问题概述：**

通过 CLI 或 Agent message tool 发送跨聊天 WhatsApp 消息（即发给非当前聊天联系人）时，即使使用有效 E.164 格式（`+5493517894500`）也被拒绝：

```
GatewayClientRequestError: Error: Delivering to WhatsApp requires target <E.164|group JID>
```

**对比：**
- ✅ 当前聊天回复 → 正常工作
- ❌ 跨聊天发送（CLI 或 agent message tool）→ 失败

**临时规避方案：**

通过官方 WhatsApp Web 打开目标聊天后，再发送消息（部分场景有效）。

---

### #56626：外部插件加载导致 Telegram 无法初始化

| 项目 | 内容 |
|------|------|
| **严重度** | HIGH |
| **类型** | 回归（v2026.3.24 新引入）— 竞态条件 |
| **GitHub** | [#56626](https://github.com/openclaw/openclaw/issues/56626) |
| **版本** | v2026.3.24 |

**问题概述：**

通过 `plugins.installs` 或 `plugins.load.paths` 加载任何外部插件后，Telegram channel 在 gateway 启动时无法初始化。无错误日志，无警告，Telegram 状态行直接缺失。

**根因：**

Gateway 启动序列中，插件加载与 channel 初始化存在竞态条件。插件注册过程会延迟 channel provider 的启动，导致 Telegram provider 初始化被跳过。

**临时规避方案：**

```json
// 删除 plugins 配置，或
"plugins": {}
// 然后 gateway restart
```

**测试结论：**
- ~15 次重启测试中，Telegram 偶发性启动成功（1 次），其余均失败
- 即使 `plugins.entries.fry.enabled: false` 也无法避免
- 唯一可靠 workaround：完全删除 `plugins` 配置块

---

### #56617：Discord WebSocket 断线导致 Gateway 崩溃

| 项目 | 内容 |
|------|------|
| **严重度** | HIGH |
| **类型** | 回归（长期存在，有待 PR 修复） |
| **GitHub PR** | [#56617](https://github.com/openclaw/openclaw/pull/56617)（OPEN） |
| **版本** | 跨版本 |

**问题概述：**

Discord provider reconnection 错误未被捕获，且 `maxAttempts` 硬编码为 0，导致网络中断后立即失败并抛出未捕获异常，使整个 gateway 进程崩溃。

**修复前后对比：**

```
修复前: [Network Drop] → [Reconnect 0/0] → [Throw] → [Process Crash]
修复后: [Network Drop] → [Reconnect 1-5] → [Exhausted] → [Catch & Log] → [Gateway Stays Alive]
```

**PR 修复内容：**
- `maxAttempts` 硬编码 0 → 5
- 添加 try/catch 安全网
- 防止 gateway 进程崩溃

**临时规避方案：** 等待 PR #56617 合并。

---

## 🟡 Medium — 本周处理

### #56627：CLI probe 每 4 秒报 "device-identity required" 错误

| 项目 | 内容 |
|------|------|
| **严重度** | MEDIUM |
| **GitHub** | [#56627](https://github.com/openclaw/openclaw/issues/56627) |
| **环境** | Windows 11 Pro + OpenClaw v2026.3.23-2 |

**问题概述：**

```
warn gateway/ws closed before connect conn=XXX remote=127.0.0.1 code=1008 reason=device identity required
```

CLI 内部探针机制未正确携带 device-identity token，导致被 gateway 拒绝（code 1008）。每 ~4 秒一次，持续填充日志。

**分析：** CLI probe 子系统的认证问题，非 gateway 核心问题。Gateway RPC probe 本身正常工作。

**临时规避方案：** 日志级别临时调高以减少噪音。

---

### #56623：memory-lancedb 加载失败

| 项目 | 内容 |
|------|------|
| **严重度** | MEDIUM |
| **GitHub** | [#56623](https://github.com/openclaw/openclaw/issues/56623)（已关闭） |

**问题概述：**

`lancedb-runtime-*.js` 中使用 `import.meta.url` 解析 `package.json`，指向错误路径：

```js
// 错误路径
new URL("./package.json", import.meta.url)
// → /usr/lib/node_modules/openclaw/dist/package.json（错误）

// 正确路径应为
→ /usr/lib/node_modules/openclaw/dist/extensions/memory-lancedb/package.json
```

**临时规避方案：**

```bash
python3 -c "
import json
path = '/usr/lib/node_modules/openclaw/dist/package.json'
with open(path,'r') as f: d=json.load(f)
d.setdefault('dependencies',{})['@lancedb/lancedb'] = '^0.27.1'
with open(path,'w') as f: json.dump(d,f,indent=2)
"
openclaw gateway restart
```

---

### #56624：Cron Isolated Run Session 状态持久化

| 项目 | 内容 |
|------|------|
| **严重度** | MEDIUM |
| **GitHub PR** | [#56624](https://github.com/openclaw/openclaw/pull/56624)（OPEN） |
| **修复** | isolated cron session 状态正确持久化 |

**修复内容：**
- isolated cron session 状态（`running→done/failed/timeout`）正确持久化
- 新增 4 条回归测试覆盖成功/异常/超时路径

**关联 Issue：** #56572

**建议：** PR 合并后更新 Cron 调度文档。

---

## 🟢 Low — 本月处理

| Issue | 标题 | 路由目标 |
|-------|------|---------|
| #56637 | Control UI Agent 详情显示错误 | `docs/03_gateway/ControlUI.md` |
| #56635 | Gmail watcher labels 丢失 | `docs/07_tools_skills/Gmail集成.md` |
| #56634 | iMessage reply_to 标签可见 | `docs/03_channels/iMessage集成.md` |
| #56633 | outbound-attachment filename 未传递 | `docs/03_gateway/Outbound插件.md` |
| #56621 | pairing list/revoke 命令 | `docs/02_setup/CLI参考.md` |

---

## 📋 PR 合并进度追踪

| PR | 状态 | 优先级 | 建议 |
|----|------|--------|------|
| [#56617](https://github.com/openclaw/openclaw/pull/56617) Discord crash fix | OPEN（1 条评论） | HIGH | 优先合并 |
| [#56624](https://github.com/openclaw/openclaw/pull/56624) Cron session 持久化 | OPEN | MEDIUM | 关注合并 |
| [#56607](https://github.com/openclaw/openclaw/pull/56607) sessions_await 多Agent编排 | OPEN | MEDIUM | 功能性文档 |

---

## 版本对比：v2026.3.23 → v2026.3.24

| 变化 | v2026.3.24 状态 |
|------|----------------|
| #56270 WhatsApp echo loop | 未修复，仍存在 |
| #56274 Discord crash | 有 PR（#56617），但未合并 |
| #56284 Windows restart | 无新进展 |
| #56286 Telegram approval bubble | 无新进展 |
| **新增 #56632** | Critical 安全漏洞 |
| **新增 #56636** | v2026.3.24 新引入回归 |
| **新增 #56626** | v2026.3.24 新引入回归 |

---

## 综合行动优先级

```
🔴 P0 — 立即处理（24小时内）：
  1. #56632 → 发布安全通报 + 文档风险提示
  2. #56636 → WhatsApp 集成文档（跨聊天限制说明）
  3. #56626 → Telegram 文档（插件共存风险）
  4. #56617 → Discord 文档标注（PR 合并后立即更新）

🟡 P1 — 本周处理：
  5. #56627 → CLI 参考文档标注 device-identity 问题
  6. #56623 → memory-lancedb 文档增加 workaround
  7. #56624 → Cron 调度文档（PR 合并后更新）

⚪ P2 — 下周处理：
  8. #56288 Nitter 替代方案调研
  9. Linux Gateway Client 实测稳定性
  10. Teams 流式回复配置验证
```

---

## 参考链接

- [GitHub Issues 首页](https://github.com/openclaw/openclaw/issues)
- [GitHub PRs 首页](https://github.com/openclaw/openclaw/pulls)
- [安全通报 SA-2026-001](../security-advisory/SA-2026-001-queue-collect-fallback-hallucination.md)
- [v2026.3.24 Release Notes](https://github.com/openclaw/openclaw/releases/tag/v2026.3.24)

---

*本文档基于 2026-03-29 GitHub Issues + PRs 调研数据自动生成。*
*数据来源: GitHub Issues/PRs API 实时同步*
