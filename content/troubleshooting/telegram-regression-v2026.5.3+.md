# OpenClaw Telegram Regression 综合故障排除指南

**创建日期：** 2026-05-07  
**更新日期：** 2026-05-07  
**版本：** v2026.5.3 及以上  
**优先级：** 🔴 高

---

## ⚠️ 概述

OpenClaw v2026.5.3 起，Telegram 渠道出现多个 Regression（回归问题）。本指南汇总所有已知 Telegram 相关问题及解决方案。

---

## 🔴 高优先级问题

### 1. Telegram 发送消息后 3-6 分钟延迟

| 属性 | 值 |
|------|-----|
| Issue # | #78079 |
| 严重程度 | 🔴 高 |
| 首次出现 | v2026.5.3 |
| 状态 | ❌ 未解决 |

**症状：**
- 发送消息后 3-6 分钟才收到回复
- 问题为间歇性，但发生频率高
- 影响所有 Telegram Bot 用户

**可能原因：**
- Telegram API 速率限制配置不当
- 消息队列处理延迟
- Webhook/Long Polling 切换问题

**临时解决方案：**
1. 重启 Gateway：`openclaw gateway restart`
2. 切换至 Long Polling 模式（Web UI → Settings → Telegram）
3. 检查 Telegram Bot Token 是否有效

**完整 Issue：** https://github.com/openclaw/openclaw/issues/78079

---

### 2. Telegram /verbose 不再流式推送 thinking/tool 进度

| 属性 | 值 |
|------|-----|
| Issue # | #78070 |
| 严重程度 | 🟡 中 |
| 首次出现 | v2026.5.4 |
| 状态 | ❌ 未解决 |

**症状：**
- 使用 `/verbose` 命令时，不再显示 AI thinking 过程
- Tool 调用进度不再实时显示
- 回复直接显示完整内容，无中间过程

**复现步骤：**
```
1. 在 Telegram 中发送 /verbose on
2. 发送任何问题触发 AI 响应
3. 观察是否显示 thinking... 进度
```

**影响版本：** v2026.5.4 及以上

**完整 Issue：** https://github.com/openclaw/openclaw/issues/78070

---

## 🟡 相关问题

### 3. 入站 DM 创建新 session 继承问题

| 属性 | 值 |
|------|-----|
| Issue # | #78074 |
| 严重程度 | 🟡 中 |
| 首次出现 | v2026.5.3 |
| 状态 | ❌ 未解决 |

**症状：**
- 入站 DM 创建新 session 时，继承已关闭 cron 的 sessionKey
- 导致 cron 任务异常终止
- 历史会话上下文丢失

**Workaround：**
- 使用 `/session new` 强制创建新 session
- 定期清理旧 session

**完整 Issue：** https://github.com/openclaw/openclaw/issues/78074

---

## 📋 问题汇总表

| 问题 | Issue # | 版本 | 状态 | 优先级 |
|------|---------|------|------|--------|
| 消息 3-6 分钟延迟 | #78079 | v2026.5.3+ | ❌ 未解决 | 🔴 高 |
| /verbose 不流式 | #78070 | v2026.5.4+ | ❌ 未解决 | 🟡 中 |
| DM session 继承 | #78074 | v2026.5.3+ | ❌ 未解决 | 🟡 中 |

---

## 🔧 通用排查步骤

1. **检查 Gateway 状态**
   ```bash
   openclaw gateway status
   ```

2. **重启 Gateway**
   ```bash
   openclaw gateway restart
   ```

3. **检查日志**
   ```bash
   openclaw logs --tail 100 --channel telegram
   ```

4. **验证 Bot Token**
   - 访问 https://t.me/BotFather
   - 检查 Token 是否有效
   - 确认 Bot 已启动

5. **切换 Webhook 模式**
   ```bash
   openclaw config set telegram.webhook.enabled true
   openclaw config set telegram.webhook.url YOUR_WEBHOOK_URL
   ```

---

## 📞 报告问题

如遇到以上未列出的 Telegram 问题，请前往：
- GitHub Issues: https://github.com/openclaw/openclaw/issues
- Discord #support: https://discord.com/invite/clawd

---

## 🔗 相关资源

- [Telegram Integration Docs](https://docs.openclaw.ai/channels/telegram)
- [v2026.5.4 发布说明](../releases/v2026.5.4.md)
- [v2026.5.3 Issues](../releases/v2026.5.3-issues.md)

---

*🦦 墨编-内容生成 | 2026-05-07 23:30 CST*
