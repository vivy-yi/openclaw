# OpenClaw v2026.4.23 发布亮点 | 2026-04-25

> 📅 发布日期：2026-04-24 | 来源：GitHub Release #71274+ | 官方

---

## 🚀 版本亮点速览

### 🔐 安全升级（20+ 项修复）
本期版本**安全修复占比最高**，涵盖：
- Discord slash-command 认证漏洞
- MCP owner-only 工具权限修复
- Android 明文传输漏洞
- Teams 跨 Bot 重放攻击防护

> 💡 **建议**：所有用户务必升级，尤其是使用 Discord/Telegram 渠道的用户。

---

### 🖼️ 图片生成能力大幅增强

| 功能 | 说明 |
|------|------|
| **OpenAI Codex OAuth** | `gpt-image-2` 无需 API Key，直接用 Codex 授权 |
| **OpenRouter 图片支持** | 新增 OpenRouter 作为图片生成 provider |

```bash
# 示例配置
images:
  provider: openrouter    # 或 openai
  model: gpt-image-2
  auth: oauth             # Codex 无需手动填 API Key
```

---

### 🧠 子 Agent 会话增强

**核心改进**： forked context 支持 `sessions_spawn` 继承

- 子 Agent 可继承父级上下文，避免重复传递 prompt
- 新增 2 小时 staleness gate，防止 orphan runs
- 修复 main-agent sessions 因 mid-loop restart 导致的孤立问题

---

### ⏱️ 超时精细控制

新增 per-call `timeoutMs` 参数，精准控制每个工具调用的超时：

```yaml
tools:
  timeoutMs: 30000        # 全局默认 30s
  calls:
    - name: image_generate
      timeoutMs: 60000    # 图片生成 60s
    - name: web_search
      timeoutMs: 15000    # 搜索 15s
```

---

### 📦 Pi Bundle 更新

- 版本：0.70.0
- 上游：`gpt-5.5` 目录已同步

---

## 🐛 本次修复的重大 Bug

| Issue | 问题 | 状态 |
|-------|------|------|
| #71273/#71274 | Kimi Code 模型无限工具调用循环 | ✅ 已合并修复 |
| #71271 | Gateway WS 1006 崩溃（参数类型错误） | ✅ 已修复 |
| #71269 | GPT-5.5/5.4 thinking level 回归 | 🔍 跟踪中 |
| #71261 | Homebrew Node macOS "Unsafe package dist path" | ✅ 已修复 |

---

## 📊 社区热点（2026-04-23 社媒采集）

### 🔥 Reddit 高热话题

| 话题 | 评分 | 亮点 |
|------|------|------|
| AI tool OpenClaw wipes inbox... | ⭐⭐⭐⭐⭐ | 病毒传播事件，10k+ upvotes |
| $25 手机运行 OpenClaw | ⭐⭐⭐⭐⭐ | 硬件创新，极低功耗设备标杆 |
| Pi Zero 私人助手 | ⭐⭐⭐⭐ | DIY 案例，IoT 边缘部署参考 |
| Self-hosted ChatGPT alternative | ⭐⭐⭐⭐ | 技术社区热潮， setup 科普 |

> 📌 **教程机会**：$25 手机和 Pi Zero 案例可做硬件专题，适合降低入门门槛宣传。

---

## 🔄 升级建议

```bash
# 检查当前版本
openclaw --version

# 升级（Homebrew）
brew upgrade openclaw

# 或使用官方脚本
curl -s https://docs.openclaw.ai/install.sh | sh
```

**推荐优先级**：
1. ⭐⭐⭐⭐⭐ **必升**：安全修复（所有渠道用户）
2. ⭐⭐⭐⭐ **建议升**：图片生成增强（Telegram/Discord Bot）
3. ⭐⭐⭐ **可选**：子 Agent 改进（多 Agent 用户）

---

## 📚 相关资源

- 📖 官方文档：https://docs.openclaw.ai
- 🐛 Issue 追踪：https://github.com/openclaw/openclaw/issues
- 💬 社区 Discord：https://discord.com/invite/clawd

---

*🦦 文档生成：OpenClaw Assistant | 数据来源：GitHub Sync 2026-04-25 + mo-yunying 社媒采集*
