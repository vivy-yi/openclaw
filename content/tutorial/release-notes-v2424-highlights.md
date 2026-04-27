# OpenClaw v2026.4.24 发布亮点 | 2026-04-27

> 📅 发布日期：2026-04-25 | 来源：GitHub Release + Issues/PRs | 官方

---

## 🚀 版本亮点速览

### 🟣 Google Meet 集成（首个视频会议原生插件）
| 功能 | 说明 |
|------|------|
| **Realtime voice sessions** | Chrome/Twilio realtime bridges，支持完整 OpenClaw agent 咨询 |
| **Participant 插件** | 首个 bundled 视频会议插件，可配对节点 Chrome |
| **咨询模式** | Voice call 期间可直接调用 OpenClaw agent |

```yaml
# 配置示例
plugins:
  google-meet:
    enabled: true
    realtime: true
    agentId: your-agent-id
```

---

### 🔵 DeepSeek V4 强势加入

| 模型 | 说明 | 用途 |
|------|------|------|
| **DeepSeek V4 Flash** | onboarding 默认模型 | 快速响应场景 |
| **DeepSeek V4 Pro** | 高配版本 | 复杂推理任务 |

```yaml
# 使用 DeepSeek V4
models:
  primary: deepseek/v4-flash
  catalog:
    - deepseek/v4-flash
    - deepseek/v4-pro
```

---

### 🎙️ Voice Call 全面增强

| 功能 | 改进 |
|------|------|
| **Realtime voice loops** | Talk/Voice Call/Google Meet 全部支持 |
| **Gateway webhook 模式** | CLI `voicecall call` 可与其他 webhook 并存 |
| **Agent 咨询** | 通话中实时调用 OpenClaw agent |

---

### 🌐 Browser 自动化升级

| 功能 | 改进 |
|------|------|
| **坐标点击** | 支持精确坐标操作，不再依赖元素定位 |
| **更长的 action budgets** | 复杂任务一次可执行更多步骤 |
| **per-profile headless** | 每个 profile 独立 headless 配置 |

---

### 🧠 Plugin/Model 基础设施轻量化

| 改进 | 说明 |
|------|------|
| **静态 model catalogs** | 模型列表预计算，响应更快 |
| **Manifest-backed model rows** | 模型行数据由 manifest 驱动 |
| **Lazy provider 依赖** | 按需加载 provider，减少启动时间 |

---

### 💾 Memory-core: WAL Journal Mode

**PR #72376** 将 memory-core 默认改为 WAL（Write-Ahead Logging）模式：
- 写入性能提升 ~30%
- 崩溃恢复更安全
- 并发访问更稳定

---

## 🔧 破坏性变更

> ⚠️ **Plugin SDK 兼容性更新**

移除 Pi-only `api.registerEmbeddedExtensionFactory(...)` 兼容性路径。

**升级建议**：
```bash
# 检查 Plugin 兼容性
openclaw doctor

# 如有问题，升级 Plugin SDK
npm update openclaw-plugin-sdk
```

---

## 🐛 重要 Bug 修复

| Issue | 问题 | 状态 |
|-------|------|------|
| #72332 | Bonjour crash loop（硬编码 `openclaw.local`） | ✅ Fix merged |
| #72372 | Google Meet chrome-node bridge cleanup | ✅ Fixed |
| #72368 | Ollama custom provider prefix stripping | ✅ Fixed |
| #72361 | /allowlist --store bypasses policy | ✅ Fixed |

### ⚠️ 待修复（截至 2026-04-26）

| Issue | 问题 | 严重度 |
|-------|------|--------|
| #72371 | Google Meet beta blocker: realtime bridge orphan on transient input | 🔴 Beta Blocker |
| #72365 | Control UI 30-45s 无响应（v2026.4.14 引入） | 🔴 高 |
| #72386 | v2026.4.25-beta.4: 运行时消息文本被模型回显到可见回复 | 🟡 中 |
| #72380 | doctor --fix 安装 bundled deps 到错误目录 | 🟡 中 |

---

## 📊 Plugin SDK 重大更新

**PR #72384 + #72383** 是 Plugin workflow contract 的 XL 级更新：

| PR | 内容 |
|----|------|
| #72384 | Add workflow action, outbound, scheduler, and retry host seams |
| #72383 | Add advanced workflow plugin contract fixtures |
| #72287 | Host-hook examples and recipes |

**Plugin 开发者需关注**：
- 新的 workflow action contract
- Outbound handler 扩展点
- Scheduler 和 retry 机制
- Host-hook 示例代码

---

## 🔄 升级建议

```bash
# 检查当前版本
openclaw --version

# 升级（Homebrew）
brew upgrade openclaw

# 验证配置
openclaw config validate

# 运行健康检查
openclaw doctor
```

**推荐优先级**：
1. ⭐⭐⭐⭐⭐ **必升**：Google Meet 用户（修复 orphan 问题）
2. ⭐⭐⭐⭐ **建议升**：使用 DeepSeek V4 的用户（性能优化）
3. ⭐⭐⭐ **可选**：Voice Call 用户（realtime loops 改进）

---

## 📚 相关资源

- 📖 官方文档：https://docs.openclaw.ai
- 🐛 Issue 追踪：https://github.com/openclaw/openclaw/issues
- 💬 社区 Discord：https://discord.com/invite/clawd

---

*🦦 文档生成：OpenClaw Assistant | 数据来源：GitHub Sync 2026-04-27*