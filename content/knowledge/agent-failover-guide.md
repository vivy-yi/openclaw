# OpenClaw Agent 状态感知故障转移教程

**创建日期：** 2026-05-07  
**更新日期：** 2026-05-07  
**相关功能：** PR #78086  
**难度：** 高级

---

## 📖 概述

v2026.5.4 引入 **状态感知故障转移（State-Aware Failover）** 功能，Agent 可以在执行过程中检测到问题并自动切换到备用方案，提升系统稳定性。

---

## 🎯 适用场景

- 多模型部署，某一模型不可用时自动切换
- 长任务执行，途中で接続断続也不中断
- 高可用性要求的生产环境
- Lane 暂停与恢复机制

---

## ⚙️ 核心概念

### Lane（车道）

Agent 执行任务的独立通道，每个 Lane 有独立状态：

```
Lane 1 (Primary)  →  Main Task Execution
Lane 2 (Backup)   →  Failover Target
Lane 3 (Warm)     →  Pre-warmed for quick switch
```

### State-Aware Failover

| 状态 | 说明 | 行为 |
|------|------|------|
| `active` | 正常执行 | 保持当前 Lane |
| `degraded` | 部分问题 | 准备切换 |
| `failed` | 完全失败 | 立即切换至 Backup Lane |
| `paused` | 暂停 | 保存状态，等待恢复 |

---

## 📝 配置步骤

### 1. 配置多模型路由

在 `agents` 配置中设置主备模型：

```yaml
agents:
  defaults:
    model: anthropic/claude-sonnet-4-20250514
  
  failover:
    enabled: true
    primary_lane: lane-1
    backup_lane: lane-2
    health_check_interval: 30s
```

### 2. 启用 Lane 故障转移

```bash
openclaw agents config set failover.enabled true
openclaw agents config set failover.backup_model "openai/gpt-4o"
```

### 3. 配置健康检查

```yaml
health_check:
  endpoint: "/health"
  timeout: 5s
  interval: 30s
  failure_threshold: 3
```

---

## 🛠️ 使用 /steer 命令控制故障转移

### 暂停 Lane

```
/steer lane pause
```

### 恢复 Lane

```
/steer lane resume
```

### 手动切换

```
/steer lane switch --to=lane-2
```

### 查看状态

```
/steer lane status
```

---

## 📊 监控与日志

### 查看当前 Lane 状态

```bash
openclaw agents lane status
```

输出示例：

```
Lane 1 (Primary)   [active] - Model: claude-sonnet-4
Lane 2 (Backup)   [standby] - Model: gpt-4o
Lane 3 (Warm)     [paused] - Model: claude-haiku-3
```

### 故障转移日志

```bash
openclaw logs --grep "failover" --tail 50
```

---

## 🔍 故障排除

### 故障转移未触发

1. 确认 `failover.enabled: true`
2. 检查 `health_check` 配置
3. 查看日志确认失败阈值

### Lane 切换后任务丢失

1. 使用 `/steer lane status` 确认状态
2. 检查 `checkpoint_enabled: true`
3. 查看 session history 恢复上下文

### 模型响应超时

1. 调整 `health_check.timeout`
2. 检查网络连通性
3. 确认 API Key 有效

---

## 📋 相关 Issue

| Issue | 描述 | 状态 |
|-------|------|------|
| #78086 | PR: 状态感知故障转移实现 | ✅ 已合并 |
| #78074 | DM session 继承问题 | ❌ 未解决 |

---

## 🔗 相关资源

- [OpenClaw Agents Documentation](https://docs.openclaw.ai/agents)
- [Multi-Model Routing Guide](../knowledge/multi-model-routing.md)
- [v2026.5.4 发布说明](../releases/v2026.5.4.md)

---

## 💡 最佳实践

1. **始终配置 Backup Lane** - 生产环境必备
2. **设置合理的 health_check_interval** - 太短频繁切换，太长响应慢
3. **测试故障转移流程** - 定期演练
4. **监控 Lane 状态** - 使用 OpenClaw Dashboard

---

*🦦 墨编-内容生成 | 2026-05-07 23:31 CST*
