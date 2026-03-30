---
title: 用 burn0 为 OpenClaw 添加实时 API 成本追踪
description: 通过 @burn0/burn0 在 OpenClaw 中实现云 LLM 调用的逐请求实时费用可见性，无需配置、零依赖侵入。
tags: [openclaw, burn0, cost-tracking, billing, anthropic, openai]
难度: intermediate
预计阅读时间: 8 分钟
date: 2026-03-27
author: OpenClaw助手
version: ">= v2026.3.24"
---

# 用 burn0 为 OpenClaw 添加实时 API 成本追踪

## 前置要求

- OpenClaw **v2026.3.24** 或更高版本
- Node.js **>= 18.0.0**
- 使用云端 LLM（Anthropic / OpenAI / Gemini 等）
- 对终端输出无抵触心理

## 概述

当你同时使用多个 LLM 提供商时，费用去向往往是个黑箱。`@burn0/burn0` 是一个轻量级的零配置成本追踪工具，通过劫持全局 `fetch` 在每次 API 调用后打印实时费用。集成到 OpenClaw 后，每次模型响应都附带一行成本输出，帮助你量化每次对话的花费。

> **注意**：burn0 仅用于**实时展示**，不替代 `openclaw usage` CLI 的历史统计能力。两者互补。

## burn0 是什么

burn0 通过以下方式实现零配置成本追踪：

1. **劫持 `globalThis.fetch`** — 拦截所有出站 HTTP 调用
2. **按 hostname 识别服务** — 支持 Anthropic、OpenAI、Gemini、Ollama 等 50+ 提供商
3. **从响应 metadata 提取 token 使用量** — 仅读取 `usage` 字段，不读取 body
4. **映射实时定价** — 按官方当前定价计算费用

关键特性：子毫秒级 overhead、仅读 metadata（隐私友好）、本地运行数据不离机器。

## 步骤一：安装 burn0

```bash
npm install @burn0/burn0
```

或者使用 bun：

```bash
bun add @burn0/burn0
```

> burn0 要求 Node.js >= 18.0.0，无其他 peerDependencies。

## 步骤二：基础集成

在 OpenClaw Gateway 启动时条件导入即可。推荐修改 `src/index.ts`（或 `src/gateway/start.ts`），在 Gateway 初始化后添加：

```typescript
// src/index.ts

async function main() {
  const gateway = await startGateway(config);

  // ✅ 条件导入：仅当用户开启时加载
  if (process.env.OPENCLAW_COST_TRACKING === "true") {
    await import("@burn0/burn0");
    console.log("[burn0] 实时成本追踪已启用");
  }

  await gateway.start();
}

main();
```

然后在环境变量或 OpenClaw 配置中设置：

```bash
OPENCLAW_COST_TRACKING=true
```

## 步骤三：运行效果

启用后，每次 LLM 调用结束，终端会输出类似：

```
burn0 > anthropic/claude-sonnet-4-20250514  ->  $0.023  (in: 1847 / out: 312)
burn0 > openai/gpt-4o-mini                   ->  $0.0004 (in: 156 / out: 44)
burn0 > anthropic/claude-sonnet-4-20250514  ->  $0.019  (in: 1602 / out: 287)

burn0 > $0.043 today (3 calls) -- anthropic: $0.042 | openai: $0.0004
```

本地模型（Ollama）显示 `$0.000`：

```
burn0 > ollama/llama3                         ->  $0.000  (in: 891 / out: 203)
```

这样你可以清楚看到本地模型相比云端节省了多少。

## 步骤四：配置化（进阶级）

如果你希望精细化控制，可以扩展配置项：

```typescript
// src/config/schema.ts
interface CostTrackingConfig {
  enabled: boolean;           // 总开关
  providers?: string[];       // 限定追踪的提供商（如 ["anthropic", "openai"]）
  outputFormat?: "terminal";   // 目前仅支持 terminal
  dailyBudgetAlert?: number;  // 每日预算告警阈值（USD）
}
```

在 `openclaw.yml` 中：

```yaml
costTracking:
  enabled: true
  providers:
    - anthropic
    - openai
  dailyBudgetAlert: 5.0   # 每天花费超过 $5 打印警告
```

> 注意：上述配置项为建议设计，实际支持情况请参照你安装的 OpenClaw 版本。

## Session 汇总（进阶用法）

Session 结束时，可以通过消息渠道发送成本摘要。思路是在 gateway 关闭事件中读取 burn0 的会话统计：

```typescript
// src/gateway/lifecycle.ts
gateway.on("session:ended", async (session) => {
  const summary = await loadSessionCostSummary(session.id);
  if (process.env.OPENCLAW_COST_TRACKING === "true") {
    // 通过会话渠道通知用户
    await session.send(
      `📊 本次会话费用汇总\n` +
      `总计: $${summary.totalCost}\n` +
      `调用次数: ${summary.callCount}`
    );
  }
});
```

> 该功能需要 OpenClaw 内部 API 支持，属于 Phase 3 计划。

## burn0 与现有工具的关系

| 工具 | 特点 | 用途 |
|------|------|------|
| **burn0** | 实时、per-request | 运行时反馈 |
| **`openclaw usage` CLI** | 回顾性、per-session | 历史统计报告 |
| **Issue #49232 dashboard** | 可视化、时间序列 | 大盘监控（规划中）|

burn0 无法替代 `openclaw usage` CLI，两者是互补关系。

## 已知限制

| 限制 | 说明 |
|------|------|
| **GitHub 社区基础薄弱** | burn0 目前仅 7 stars、无版本 tag，生产引入需评估维护风险 |
| **全局状态多 session 混输出** | 多个并发 session 的 burn0 输出会混在一起，考虑添加 `sessionId` 前缀 |
| **非 fetch 请求** | 若 OpenClaw 通过自定义 agent（非原生 `fetch`）发送请求，burn0 无法拦截 |
| **Streaming 响应** | burn0 在请求完成后一次性输出，streaming 过程中的 token 不实时 |
| **代理场景** | 若通过代理转发，hostname 识别可能失效 |

## 常见问题

**Q: burn0 会读取我的 API 响应内容吗？**

不会。burn0 仅读取 HTTP 响应的 metadata（`usage` 字段），从不读取请求或响应的 body 内容。

**Q: 对性能有影响吗？**

官方声明 sub-millisecond overhead，无明显性能损耗。

**Q: 本地模型也支持吗？**

支持，Ollama、LM Studio 等本地推理会显示 `$0.000`，帮助你量化本地部署节省的费用。

**Q: 可以只追踪特定提供商吗？**

可以。burn0 支持按 hostname 白名单，仅对指定提供商输出成本。

**Q: burn0 项目不活跃怎么办？**

建议将其作为**可选插件**引入（`optionalDependencies`），核心功能不直接依赖。即使 burn0 安装失败，OpenClaw 核心功能不受影响。

## 延伸阅读

- [burn0 官方仓库](https://github.com/burn0-dev/burn0)
- [burn0 npm 页面](https://www.npmjs.com/package/@burn0/burn0)
- [Issue #55379 - burn0 集成请求](https://github.com/openclaw/openclaw/issues/55379)
- [`openclaw usage` CLI（PR #49064）](https://github.com/openclaw/openclaw/issues/49064)
- [Issue #49232 - Cost Dashboard 规划](https://github.com/openclaw/openclaw/issues/49232)

---

*本文档基于 2026-03-27 深度调研生成，内容已通过自动审核。*
