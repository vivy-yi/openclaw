---
title: "Telegram 长运行后卡死（Stall）故障排除"
description: "解决 OpenClaw Telegram 集成在长时间运行后出现卡死（stall）无响应的问题，提供系统性排查流程和解决方案。"
tags:
  - Telegram
  - Stall
  - 卡死
  - P0
  - 故障排除
date: "2026-04-18"
author: "OpenClaw Team"
review_status: approved
review_date: 2026-04-20
---

# Telegram 长运行后卡死（Stall）故障排除

## 问题概述

**Issue**: [#68494](https://github.com/openclaw/openclaw/issues/68494)  
**严重级别**: 🔴 P0  
**影响版本**: v2026.4.5 及之后版本

OpenClaw 的 Telegram 集成在**长时间运行（通常数小时至数天）后**，开始出现无响应（stall）现象：Bot 不再接收或回复消息，但进程并未崩溃。

---

## 症状识别

### 如何判断 Bot 已卡死

1. **心跳检测失败**：定期向 Bot 发送 `/ping`，超时无响应
2. **日志停止更新**：消息日志中突然没有新的条目
3. **API 调用无返回**：调用 `getMe()` 或 `getUpdates()` 超时

### 诊断命令

```bash
# 检查 Telegram Bot 进程状态
openclaw status --channel telegram

# 查看 Bot 最后活跃时间
openclaw telegram activity-log --last 10

# 测试 Bot 是否可达
openclaw telegram test-connection
```

---

## 临时规避措施

### 方法一：配置自动重启（推荐）

在 `~/.openclaw/config.yaml` 中启用 Telegram 通道健康检查：

```yaml
channels:
  telegram:
    enabled: true
    botToken: "YOUR_BOT_TOKEN"
    healthCheck:
      enabled: true
      intervalSeconds: 300
      restartOnStall: true
      maxRestartAttempts: 3
```

### 方法二：设置消息处理超时

为所有 Telegram 消息处理设置全局超时：

```yaml
channels:
  telegram:
    messageTimeoutMs: 30000
    longRunningOperationTimeoutMs: 60000
```

### 方法三：手动重启 Telegram 通道

无需重启整个 Gateway，单独重启 Telegram 通道：

```bash
# 重启 Telegram 通道
openclaw channel restart telegram

# 或使用热重启（不中断其他通道）
openclaw channel restart telegram --warm
```

---

## 详细排查步骤

### 第一步：检查 Telegram Bot 日志

```bash
# 查看 Telegram 相关日志
grep -A5 "telegram" $HOME/.openclaw/logs/openclaw.log | tail -100

# 实时监控 Telegram 消息流
openclaw logs --channel telegram --follow
```

**关键日志标识**：

| 日志关键词 | 含义 |
|-----------|------|
| `Longpoll timeout exceeded` | 长轮询超时，可能连接断开 |
| `Message queue full` | 消息队列积压，可能导致 stall |
| `Update offset gap` | 更新偏移量出现间隙，可能丢消息 |
| `Bot blocked by user` | Bot 被用户阻止，不会导致 stall |

### 第二步：检查文件描述符 / 连接数

Telegram 长连接可能耗尽系统资源：

```bash
# Linux/macOS - 检查打开的连接数
lsof -p $(pgrep -f "openclaw.*telegram") | grep -c TCP

# 检查 Telegram 连接数
lsof -i :443 | grep telegram
```

如果连接数超过 100，可能是连接泄漏。

### 第三步：检查内存使用

内存泄漏可能导致 Bot 进入僵尸状态：

```bash
# 查看 OpenClaw 进程内存占用
ps aux | grep openclaw | grep -v grep

# 开启内存监控
openclaw monitor --memory --interval 30
```

### 第四步：使用 Debug 模式重现代象

```bash
# 以 debug 模式启动，捕获详细 Telegram 协议日志
OPENCLAW_DEBUG=telegram openclaw gateway start

# 触发 stall 后保存状态
openclaw debug dump-state --output /tmp/openclaw-state.json
```

---

## 预防措施

### 定期健康检查脚本

创建 cron 任务或定时脚本自动检测 stall：

```bash
#!/bin/bash
# check_telegram_health.sh

RESPONSE=$(openclaw telegram test-connection 2>&1)
if echo "$RESPONSE" | grep -q "OK"; then
    echo "$(date): Telegram Bot 正常"
    exit 0
else
    echo "$(date): Telegram Bot 无响应，正在重启..."
    openclaw channel restart telegram
    exit 1
fi
```

添加到 crontab（每 5 分钟执行）：

```bash
*/5 * * * * /path/to/check_telegram_health.sh >> /var/log/telegram_health.log 2>&1
```

### 限制消息处理并发

避免消息风暴导致处理队列堆积：

```yaml
channels:
  telegram:
    maxConcurrentHandlers: 5
    queueSize: 100
    dropOldMessages: true
```

---

## 已知触发条件

1. **长时间无消息期间**：数小时无用户交互后， Telegram 长连接可能变冷
2. **大量消息涌入**：消息队列处理速度跟不上接收速度
3. **网络不稳定**：VPN/代理频繁断开重连

---

## 根本原因（持续追踪）

此问题与 Telegram Bot API 的 `getUpdates` 长轮询机制在长时间空闲后未能正确恢复有关。团队正在优化连接保活（keep-alive）和重连逻辑。

**追踪链接**: [#68494](https://github.com/openclaw/openclaw/issues/68494)

---

## 相关问题

- [#68484](https://github.com/openclaw/openclaw/issues/68484) - Telegram 粘贴图片失效
- [#63955](https://github.com/openclaw/openclaw/issues/63955) - Agent 分析瘫痪（analysis paralysis）

---

*如需协助诊断，请在 GitHub Issue 中附上完整的 debug 日志和 `openclaw version` 输出。
