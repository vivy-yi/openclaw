---
title: OpenClaw 完整使用指南
description: 从安装配置到高级用法，全面掌握自托管 AI 网关 OpenClaw
tags: [openclaw, 入门, 教程, 配置, 工具系统]
难度: beginner
预计阅读时间: 25 分钟
date: 2026-04-29
author: OpenClaw助手
version: latest
---

# OpenClaw 完整使用指南

## 概述

OpenClaw 是一个**自托管网关**，将您喜欢的聊天应用和频道连接到 AI 编码代理。您可以在自己的机器上运行单个网关进程，它成为消息应用与全天候可用的 AI 助手之间的桥梁。

### 核心特点

- **自托管**: 运行在您的硬件上，您的规则
- **多渠道**: 一个网关同时服务内置渠道和第三方插件
- **原生代理**: 为编码代理构建，支持工具使用、会话、记忆和多代理路由
- **开源**: MIT 许可，社区驱动

---

## 第一章：安装与快速开始

### 系统要求

- Node 24 (推荐) 或 Node 22 LTS (22.14+)
- API 密钥（来自您选择的提供商）
- 5 分钟即可完成安装

### 安装步骤

```bash
# 1. 安装 OpenClaw
npm install -g openclaw@latest

# 2. 引导设置并安装服务
openclaw onboard --install-daemon

# 3. 打开控制 UI 并发送消息
openclaw dashboard
```

### 控制 UI 地址

- 本地默认: http://127.0.0.1:18789/
- 远程访问: Web surfaces 和 Tailscale

---

## 第二章：配置文件详解

配置文件位于 `~/.openclaw/openclaw.json`

### 基础配置示例

```json
{
  "channels": {
    "whatsapp": {
      "allowFrom": ["+15555550123"],
      "groups": {
        "*": { "requireMention": true }
      }
    }
  },
  "messages": {
    "groupChat": {
      "mentionPatterns": ["@openclaw"]
    }
  }
}
```

### 渠道配置要点

| 配置项 | 说明 |
|--------|------|
| `channels` | 定义支持的渠道（telegram, discord, whatsapp 等） |
| `allowFrom` | 允许访问的白名单号码/用户 ID |
| `groups` | 群组配置，`requireMention` 控制是否需要 @ 提及 |

---

## 第三章：工具系统架构

OpenClaw 有三个层次的工具系统，理解它们对于高效使用至关重要。

### 3.1 三个层次概览

| 层次 | 说明 | 示例 |
|------|------|------|
| **工具 (Tools)** | 代理调用什么 | `exec`, `browser`, `web_search` |
| **技能 (Skills)** | 教会代理何时及如何使用 | SKILL.md 文件 |
| **插件 (Plugins)** | 打包一切 | 渠道、模型提供商、工具包 |

### 3.2 内置工具一览

| 工具 | 功能 |
|------|------|
| `exec` / `process` | 运行 shell 命令，管理后台进程 |
| `browser` | 控制 Chromium 浏览器 |
| `web_search` | 搜索网页 |
| `read` / `write` / `edit` | 工作区文件 I/O |
| `message` | 跨渠道发送消息 |
| `image` / `image_generate` | 分析或生成图像 |
| `music_generate` | 生成音乐曲目 |
| `video_generate` | 生成视频 |
| `tts` | 文本转语音 |
| `sessions_*` | 会话管理和子代理编排 |
| `cron` / `gateway` | 计划任务和网关管理 |

### 3.3 工具组

OpenClaw 将工具分组便于配置权限：

| 组 | 包含工具 |
|----|---------|
| `group:runtime` | exec, process, code_execution |
| `group:fs` | read, write, edit, apply_patch |
| `group:sessions` | sessions_list, sessions_history, sessions_send 等 |
| `group:memory` | memory_search, memory_get |
| `group:web` | web_search, x_search, web_fetch |
| `group:ui` | browser, canvas |
| `group:automation` | cron, gateway |
| `group:messaging` | message |
| `group:media` | image, image_generate, music_generate, video_generate, tts |

### 3.4 工具配置

通过 `tools.allow` / `tools.deny` 控制代理可以调用哪些工具。**拒绝始终优先于允许**。

```json
{
  "tools": {
    "allow": ["group:fs", "browser", "web_search"],
    "deny": ["exec"]
  }
}
```

### 3.5 预定义工具配置文件

| 配置 | 适用场景 |
|------|---------|
| `full` | 无限制，适用于广泛的命令/控制访问 |
| `coding` | 编码代理，包含 fs, runtime, web, sessions, memory, cron, 媒体生成 |
| `messaging` | 消息场景，包含 messaging, sessions_list, sessions_history, sessions_send |
| `minimal` | 仅 session_status，最小权限 |

---

## 第四章：技能系统

### 什么是技能？

技能是 markdown 文件 (SKILL.md)，注入到系统提示中，为代理提供：
- 上下文
- 约束
- 分步指导

### 技能位置

- 工作区内 (`skills/` 目录)
- 共享文件夹
- 插件内打包

### 技能配置

- 技能路径配置
- 斜杠命令集成
- ClawHub 技能市场

---

## 第五章：自动化与任务

### 5.1 自动化功能

| 功能 | 说明 |
|------|------|
| **计划任务 (Cron)** | 按时间表执行任务 |
| **后台任务** | 后台处理长时间运行的作业 |
| **任务流** | 工作流编排 |
| **常驻命令** | 持久化命令 |
| **钩子** | 事件触发 |

### 5.2 Cron 任务示例

Cron 任务允许您按计划执行自动化工作流，适合：
- 定时数据同步
- 定期健康检查
- 自动化报告生成
- 定时内容发布

---

## 第六章：故障排除

### 6.1 首要 60 秒诊断

按顺序运行以下命令：

```bash
openclaw status
openclaw status --all
openclaw gateway probe
openclaw gateway status
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

### 6.2 诊断结果解读

| 命令 | 期望输出 |
|------|---------|
| `openclaw status` | 显示已配置的渠道，无明显认证错误 |
| `openclaw status --all` | 完整报告可共享 |
| `openclaw gateway probe` | 预期网关目标可访问 (`Reachable: yes`) |
| `openclaw gateway status` | `Runtime: running`, `Connectivity probe: ok` |
| `openclaw doctor` | 无阻塞配置/服务错误 |
| `openclaw channels status --probe` | 可达网关返回每个账户的实时传输状态 |
| `openclaw logs --follow` | 稳定活动，无重复致命错误 |

### 6.3 常见问题解决

#### 问题 1: Anthropic 长上下文 429

**错误**: `HTTP 429: rate_limit_error: Extra usage is required for long context requests`

**解决**: 访问 `/gateway/troubleshooting#anthropic-429-extra-usage-required-for-long-context`

#### 问题 2: 本地 OpenAI 兼容后端连接失败

**可能原因和解决方案**:

1. 如果错误提到 `messages[].content` 期望字符串，设置：
```json
"models.providers.<provider>.models[].compat.requiresStringContent": true
```

2. 如果后端仍然仅在 OpenClaw 代理调用时失败，设置：
```json
"models.providers.<provider>.models[].compat.supportsTools": false
```

#### 问题 3: 插件安装失败 - 缺少 openclaw extensions

如果安装失败并显示 `package.json missing openclaw.extensions`，说明插件包使用的是旧格式。

**修复方法**:

1. 在 `package.json` 中添加 `openclaw.extensions`
2. 指向构建的运行时文件（通常是 `./dist/index.js`）
3. 重新发布插件并再次运行 `openclaw plugins install <package>`

**示例**:
```json
{
  "name": "@openclaw/my-plugin",
  "version": "1.2.3",
  "openclaw": {
    "extensions": ["./dist/index.js"]
  }
}
```

### 6.4 问题排查决策树

当遇到问题时，按以下路径排查：

| 症状 | 检查项 |
|------|--------|
| 无回复 | 网络连接、API 密钥配置 |
| 控制 UI 无法连接 | 网关状态、防火墙设置 |
| 网关无法启动 | Node 版本、端口占用 |
| 渠道连接但消息不流动 | 渠道配置、webhook 设置 |
| Cron 或心跳未触发 | 调度器配置 |
| 节点工具失败 | 节点连接状态 |
| Exec 突然要求批准 | exec 安全配置 |
| 浏览器工具失败 | 浏览器依赖、权限设置 |

### 6.5 相关资源链接

- [FAQ](https://docs.openclaw.ai/help/faq) — 常见问题
- [Gateway Troubleshooting](https://docs.openclaw.ai/gateway/troubleshooting) — 网关特定问题
- [Doctor](https://docs.openclaw.ai/gateway/doctor) — 自动健康检查和修复
- [Channel Troubleshooting](https://docs.openclaw.ai/channels/troubleshooting) — 渠道连接问题
- [Automation Troubleshooting](https://docs.openclaw.ai/automation/cron-jobs#troubleshooting) — Cron 和心跳问题

---

## 总结

本文档涵盖了 OpenClaw 的核心概念、安装配置、工具系统、技能机制、自动化功能和故障排除方案。

### 下一步

- 深入阅读 [官方文档](https://docs.openclaw.ai)
- 加入 [Discord 社区](https://discord.com/invite/clawd) 交流经验
- 探索 [ClawHub](https://clawhub.openclaw.ai) 技能市场

---

*本文档基于 OpenClaw 官方文档生成*
