---
title: "Kimi 模型无限工具调用循环问题修复"
description: "解决 Kimi Code 模型在 OpenClaw 中出现的无限工具调用循环问题（Issue #71273 → #71274）。"
tags: [OpenClaw, Kimi, Troubleshooting, Bug Fix, 工具调用]
难度: intermediate
预计阅读时间: 5分钟
date: 2026-04-23
---

# Kimi 模型无限工具调用循环问题修复

> **问题编号:** [#71273](https://github.com/openclaw/openclaw/issues/71273) → [#71274](https://github.com/openclaw/openclaw/issues/71274)  
> **影响版本:** v2026.4.22 及之前  
> **修复版本:** v2026.4.23  
> **严重程度:** 🔴 高

---

## 问题概述

### 症状

Kimi Code 模型在 OpenClaw 中执行任务时，出现**无限工具调用循环**：

```
工具调用 #1: browser → open(url='https://example.com')
工具调用 #2: browser → snapshot
工具调用 #3: browser → act(click)
工具调用 #4: browser → snapshot
工具调用 #5: browser → act(click)
工具调用 #6: browser → snapshot
... (无限循环，持续到超时)
```

Agent 持续调用工具但无法完成任务，最终超时或资源耗尽。

### 影响范围

- **受影响模型:** Kimi Code (`kimi-code-vision`, `kimi-code-v1`)
- **触发条件:** 复杂多步骤任务，需要多次浏览器操作或文件读写
- **发生概率:** 约 15-20% 的多步骤任务
- **影响版本:** v2026.4.22 及之前的所有版本

---

## 根本原因

分析 Issue #71273 后，确认问题由以下因素共同导致:

### 1. 响应截断问题

Kimi 模型的响应在 token 限制附近被截断，导致 Agent 丢失上下文状态：

```
[截断点]: "...工具调用 #47: browser → act\n  "
                                      ↑
                            丢失后续指令和状态
```

Agent 认为当前任务未完成，继续调用相同工具尝试完成任务。

### 2. 状态同步延迟

Agent 发送工具调用后，立即进行下一次 `snapshot` 检查状态，但此时工具尚未完成执行。Agent 看到的是旧状态，误判为操作失败，从而重复执行。

### 3. 循环检测缺失

Agent 陷入"执行 → 检查 → 再执行"的模式，但没有"终止"判断逻辑。缺乏对重复行为模式的检测和中止机制。

---

## 解决方案

### 方案 A: 升级 OpenClaw（推荐）

```bash
# Homebrew
brew upgrade openclaw

# NPM
npm update -g openclaw

# 验证版本
openclaw --version
# 应显示: v2026.4.23
```

升级后自动启用以下防护：
- `toolCallCount` 计数器（上限 25 次）
- 状态同步等待机制
- 截断响应恢复

### 方案 B: 临时规避措施

如果无法立即升级，在 `openclaw.yaml` 中添加：

```yaml
# openclaw.yaml
kimi:
  maxToolCalls: 20      # 限制最大工具调用次数
  toolCallCooldown: 2000  # 工具调用间隔(毫秒)
  enableLoopDetection: true  # 启用循环检测
```

---

## 核心修复逻辑

### 1. 工具调用计数器

```javascript
// 在 agent runtime 中实现
const toolCallHistory = [];
const MAX_TOOL_CALLS = 25;

function checkLoop(toolCall) {
  toolCallHistory.push({
    name: toolCall.name,
    args: toolCall.arguments,
    timestamp: Date.now()
  });
  
  // 检测模式: 连续 3 次相同工具调用
  const recentCalls = toolCallHistory.slice(-3);
  if (recentCalls.length === 3 && 
      recentCalls.every(t => t.name === toolCall.name)) {
    const result = analyzePattern(toolCallHistory);
    if (result.isLooping) {
      throw new LoopDetectionError(
        `Detected infinite loop: ${toolCall.name} called ${toolCallHistory.length} times consecutively`
      );
    }
  }
  
  // 硬性上限检查
  if (toolCallHistory.length > MAX_TOOL_CALLS) {
    throw new LoopDetectionError(
      `Max tool calls (${MAX_TOOL_CALLS}) exceeded for single task`
    );
  }
}
```

### 2. 状态同步等待

```javascript
// 确保工具执行完成后再继续
async function executeToolCall(toolCall) {
  const result = await toolBridge.execute(toolCall);
  
  // 等待状态稳定 (新增修复)
  if (toolCall.requiresStateSync) {
    await stateManager.waitForStable(toolCall.targetId, {
      timeout: 3000,        // 最大等待 3 秒
      interval: 200,        // 每 200ms 检查一次
      stableCount: 2        // 连续 2 次状态相同才认为稳定
    });
  }
  
  return result;
}
```

### 3. 截断响应恢复

```javascript
// 处理被截断的响应
function handleTruncatedResponse(response) {
  if (response.isTruncated && response.toolCalls) {
    // 记录未完成的工具调用
    const pendingCalls = response.toolCalls.slice(response.completedCalls);
    
    if (pendingCalls.length > 0) {
      // 将其作为下一轮的待执行任务
      session.setContext({
        pendingToolCalls: pendingCalls,
        lastResponse: response,
        isResuming: true
      });
      
      return {
        shouldContinue: true,
        resumeFrom: pendingCalls[0],
        reason: 'Resuming truncated response'
      };
    }
  }
  
  return { shouldContinue: false };
}
```

---

## 验证方法

### 1. 自动化测试

```bash
# 运行循环检测测试套件
openclaw test --suite loop-detection --model kimi-code

# 预期输出:
# ✓ loop-detection/test-1: No false positives
# ✓ loop-detection/test-2: Detects simple loop
# ✓ loop-detection/test-3: Detects nested loop
# ✓ loop-detection/test-4: Handles legitimate repeated calls
# ✓ loop-detection/test-5: Enforces max tool call limit
```

### 2. 手动验证任务

执行以下多步骤任务，观察工具调用行为：

```markdown
# 测试任务
请打开浏览器，访问 https://example.com，截图，分析页面主要内容。
```

**正常行为:** 工具调用 3-5 次后完成
**循环行为:** 工具调用超过 15 次仍未完成（将被自动终止）

---

## 故障排除

### 如果升级后仍有问题

**1. 检查版本:**
```bash
openclaw --version
# 确保是 v2026.4.23
```

**2. 清除缓存:**
```bash
openclaw cache clear
openclaw restart
```

**3. 检查配置:**
```bash
openclaw config show | grep -i kimi
```

**4. 查看日志:**
```bash
openclaw logs --level debug --since 1h | grep -i "loop\|kimi"
```

### 降级方案

如需暂时降级（仅作为临时方案）：

```bash
# Homebrew
brew uninstall openclaw
brew install openclaw@2026.4.22

# NPM
npm install -g openclaw@2026.4.22
```

⚠️ **警告:** 降级后将暴露原有问题，建议仅作为临时过渡方案。

---

## 相关文档

- [OpenClaw v2026.4.23 发布说明](../releases/v2026.4.23.md)
- [Gateway 崩溃问题修复](./gateway-crash.md)
- [子Agent最佳实践](../../tutorial/subagent-best-practices.md)
- [OpenClaw 工具系统文档](https://docs.openclaw.ai/tools)

---

## 相关 Issue

| Issue | 描述 | 状态 |
|-------|------|------|
| [#71273](https://github.com/openclaw/openclaw/issues/71273) | Kimi 无限工具调用循环 (原始报告) | 🟢 已修复 |
| [#71274](https://github.com/openclaw/openclaw/issues/71274) | 循环检测 + 修复补丁 | 🟢 已合并 |

---

*🦦 墨客 · OpenClaw 故障排除系列*
