---
title: "Anthropic 断供 — OpenClaw 紧急迁移指南"
description: "当 Anthropic 切断 OpenClaw API 访问时，如何快速切换到 OpenAI、Google Gemini、Kimi 或 Ollama 等替代 Provider 的完整指南。"
tags: [OpenClaw, Migration, Provider, Anthropic, OpenAI, Gemini, Kimi, Ollama]
难度: intermediate
预计阅读时间: 10分钟
date: 2026-04-25
---

# Anthropic 断供 — OpenClaw 紧急迁移指南

> **紧急程度:** 🔴 高 — 立即行动  
> **适用版本:** v2026.4.23 及以上  
> **最后更新:** 2026-04-25

---

## 📋 执行摘要

| 风险项 | 状态 |
|--------|------|
| Anthropic API 访问被切断 | 🔴 已发生（Business Insider 报道） |
| OpenClaw 多 Provider 切换能力 | 🟢 可用 |
| 迁移所需时间 | ~15 分钟 |
| 数据迁移风险 | 🟢 无数据丢失风险 |

---

## 1. 背景

### 1.1 事件概述

2026 年 4 月 25 日，Anthropic 正式切断了 OpenClaw 的 API 访问权限。据 [Business Insider 报道](https://www.businessinsider.com)，此次断供源于 Anthropic 对 OpenClaw 安全合规性的担忧。同日，Tom's Hardware 曝光了另一起关联事件：OpenClaw 用户意外清空 Meta AI Alignment director 收件箱，Reddit 热度超过 22,000 分。

**时间线：**

| 日期 | 事件 |
|------|------|
| 2026-04-16 | Anthropic 模型默认升级至 Claude Opus 4.7 |
| 2026-04-23 | OpenClaw v2026.4.23 发布，含 20+ 安全补丁 |
| 2026-04-25 | Business Insider 报道 Anthropic 断供 |
| 2026-04-25 | Tom's Hardware 曝光 Meta 收件箱事件 |

### 1.2 为什么现在是最佳迁移时机

OpenClaw 从架构上就是多 Provider 设计，Anthropic 断供并不意味着服务终止。以下因素使得迁移相对平滑：

- ✅ **v2026.4.23 已包含多 Provider 优化**，切换成本低
- ✅ **OpenAI GPT-4o / Google Gemini 2.5 / Kimi Moonshot** 均已通过稳定性验证
- ✅ **Ollama auto-discover（v2026.4.23）** 支持本地模型，无需外部 API
- ✅ **50+ 修复**（含 20+ 安全补丁）已确保基础架构稳定

---

## 2. 影响评估

### 2.1 受影响的功能

| 功能 | 影响程度 | 说明 |
|------|----------|------|
| `claude-*` provider API 调用 | 🔴 完全切断 | 所有 `claude-*` 模型不可用 |
| 默认 model 设为 `anthropic/*` | 🔴 立即失效 | 需手动切换 |
| Claude Code 集成 | 🟡 受限 | 依赖 Anthropic API 的功能受影响 |
| 其他 Provider（OpenAI/Gemini/Kimi/Ollama） | 🟢 不受影响 | 完全正常 |

### 2.2 影响范围自检

运行以下命令检查你是否受影响：

```bash
openclaw status
```

如果你看到以下输出，说明你正在使用 Anthropic provider：

```
🤖 Active Provider: anthropic/claude-sonnet-4
⚠️  WARNING: Anthropic API access may be restricted
```

### 2.3 推荐的 Provider 优先级

| 优先级 | Provider | 推荐原因 |
|--------|----------|----------|
| 🥇 推荐 | **OpenAI (GPT-4o)** | 覆盖最广，延迟最低，工具调用成熟 |
| 🥈 备选 | **Google Gemini 2.5** | 成本低，上下文长（1M tokens） |
| 🥉 中文场景 | **Kimi (Moonshot)** | 中文优化，支持长上下文 |
| 🔒 离线 | **Ollama / Local LLM** | 完全离线，无 API 成本 |

---

## 3. Provider 切换方案

### 3.1 OpenAI (GPT-4o)

**推荐场景：** 通用任务、最佳稳定性、需要 function calling 的复杂场景

**费用对比（Claude Opus 4.7 vs GPT-4o）：**

| 模型 | 输入成本 | 输出成本 | 上下文 |
|------|----------|----------|--------|
| Claude Opus 4.7（已断供） | $15/1M tokens | $75/1M tokens | 200K |
| GPT-4o | $2.5/1M tokens | $10/1M tokens | 128K |
| GPT-4o-mini | $0.15/1M tokens | $0.6/1M tokens | 128K |

**成本节省：** GPT-4o 比 Claude Opus 4.7 便宜 **~6-8 倍**

**配置步骤：**

**Step 1:** 设置 API Key

```bash
export OPENAI_API_KEY="sk-..."
```

**Step 2:** 修改 `openclaw.yaml`

```yaml
# openclaw.yaml

providers:
  openai:
    apiKey: ${OPENAI_API_KEY}
    defaultModel: gpt-4o

agents:
  defaults:
    model: openai/gpt-4o
    provider: openai
```

**Step 3:** 验证配置

```bash
openclaw config validate
openclaw model test openai/gpt-4o --prompt "Hello"
```

---

### 3.2 Google Gemini 2.5

**推荐场景：** 长上下文任务（1M tokens）、成本敏感、需要多模态

**费用对比：**

| 模型 | 输入成本 | 输出成本 | 上下文 |
|------|----------|----------|--------|
| Gemini 2.5 Pro | $0.3/1M tokens | $1.2/1M tokens | 1M |
| Gemini 2.5 Flash | $0.075/1M tokens | $0.3/1M tokens | 1M |

**配置步骤：**

**Step 1:** 设置 API Key

```bash
export GOOGLE_API_KEY="AI..."
```

**Step 2:** 修改 `openclaw.yaml`

```yaml
# openclaw.yaml

providers:
  google:
    apiKey: ${GOOGLE_API_KEY}
    defaultModel: gemini-2.5-pro

agents:
  defaults:
    model: google/gemini-2.5-pro
    provider: google
```

**Step 3:** 验证配置

```bash
openclaw config validate
openclaw model test google/gemini-2.5-pro --prompt "Hello"
```

---

### 3.3 Kimi (Moonshot)

**推荐场景：** 中文任务、长上下文（128K）、国内用户

**费用对比：**

| 模型 | 输入成本 | 输出成本 | 上下文 |
|------|----------|----------|--------|
| Kimi 2.5 | ¥0.1/1M tokens | ¥1/1M tokens | 128K |
| Kimi 2 | ¥0.5/1M tokens | ¥2/1M tokens | 128K |

> 💡 Kimi 使用人民币计费，国内用户成本更低

**配置步骤：**

**Step 1:** 获取 Kimi API Key

前往 [Moonshot 控制台](https://platform.moonshot.cn) 获取 API Key。

**Step 2:** 修改 `openclaw.yaml`

```yaml
# openclaw.yaml

providers:
  kimi:
    apiKey: ${KIMI_API_KEY}
    baseUrl: https://api.moonshot.cn/v1
    defaultModel: moonshot-v1-128k

agents:
  defaults:
    model: kimi/moonshot-v1-128k
    provider: kimi
```

**Step 3:** 验证配置

```bash
openclaw config validate
openclaw model test kimi/moonshot-v1-128k --prompt "你好"
```

---

### 3.4 Ollama / Local LLM

**推荐场景：** 离线环境、隐私敏感、无 API 费用、极低成本部署

**v2026.4.23 新特性：** Ollama auto-discover，无需手动配置 host

**支持的模型：**

| 模型 | 参数量 | 内存需求 | 适用场景 |
|------|--------|----------|----------|
| llama3.2 | 3B | 4GB RAM | 轻量任务 |
| qwen2.5 | 7B | 8GB RAM | 通用任务 |
| deepseek-r1 | 8B | 10GB RAM | 复杂推理 |
| codellama | 7B | 10GB RAM | 代码任务 |

**配置步骤：**

**Step 1:** 安装 Ollama

```bash
# macOS
brew install ollama

# Linux
curl -fsSL https://ollama.com/install.sh | sh

# Windows (PowerShell)
irm https://ollama.com/install.ps1 | iex
```

**Step 2:** 拉取模型

```bash
# 通用任务推荐
ollama pull llama3.2

# 中文任务
ollama pull qwen2.5:7b

# 代码任务
ollama pull codellama:7b
```

**Step 3:** 修改 `openclaw.yaml`

v2026.4.23 支持 auto-discover，自动检测本地 Ollama：

```yaml
# openclaw.yaml

providers:
  ollama:
    autoDiscover: true
    defaultModel: llama3.2

agents:
  defaults:
    model: ollama/llama3.2
    provider: ollama
```

或者手动指定（非 auto-discover 模式）：

```yaml
# openclaw.yaml

providers:
  ollama:
    baseUrl: http://localhost:11434
    defaultModel: llama3.2

agents:
  defaults:
    model: ollama/llama3.2
    provider: ollama
```

**Step 4:** 验证配置

```bash
ollama list  # 确认模型已安装
openclaw config validate
openclaw model test ollama/llama3.2 --prompt "Hello"
```

---

## 4. 迁移步骤

### 完整迁移流程（约 15 分钟）

**Step 1: 确认当前状态**

```bash
openclaw status
openclaw config show
```

**Step 2: 备份当前配置**

```bash
cp ~/.config/openclaw/openclaw.yaml ~/.config/openclaw/openclaw.yaml.backup-$(date +%Y%m%d)
```

**Step 3: 选择目标 Provider**

根据上文第 3 节，选择适合你的 Provider：

- 🥇 **OpenAI (GPT-4o）** — 通用推荐
- 🥈 **Google Gemini 2.5** — 长上下文 / 成本敏感
- 🥉 **Kimi** — 中文场景 / 国内用户
- 🔒 **Ollama** — 离线 / 隐私优先

**Step 4: 修改配置**

按照对应 Provider 的配置示例修改 `openclaw.yaml`。

**Step 5: 验证配置**

```bash
openclaw config validate
```

**Step 6: 测试运行**

```bash
openclaw model test <provider>/<model> --prompt "测试消息"
```

**Step 7: 重启 Gateway**

```bash
openclaw gateway restart
```

**Step 8: 确认状态**

```bash
openclaw status
```

你应该看到新的 Provider 显示为 active：

```
🤖 Active Provider: openai/gpt-4o
✅ Connected
```

---

## 5. 完整配置示例

### 5.1 切换到 OpenAI

```yaml
# openclaw.yaml

version: "2026.4"

providers:
  openai:
    apiKey: ${OPENAI_API_KEY}
    organization: ""  # 可选

  # 保留其他 provider 作为备份
  google:
    apiKey: ${GOOGLE_API_KEY}

  kimi:
    apiKey: ${KIMI_API_KEY}
    baseUrl: https://api.moonshot.cn/v1

  ollama:
    autoDiscover: true

agents:
  defaults:
    model: openai/gpt-4o
    provider: openai
    temperature: 0.7
    maxTokens: 4096

gateway:
  poolSize: 10
  toolTimeout: 30
  logLevel: info
```

### 5.2 多 Provider 备用配置

推荐保留多个 Provider 作为备用，避免单点故障：

```yaml
# openclaw.yaml

version: "2026.4"

providers:
  openai:
    apiKey: ${OPENAI_API_KEY}
    fallbackModel: gpt-4o-mini

  google:
    apiKey: ${GOOGLE_API_KEY}
    fallbackModel: gemini-2.5-flash

  kimi:
    apiKey: ${KIMI_API_KEY}
    baseUrl: https://api.moonshot.cn/v1
    fallbackModel: moonshot-v1-8k

  ollama:
    autoDiscover: true
    fallbackModel: llama3.2

agents:
  defaults:
    model: openai/gpt-4o
    provider: openai
    temperature: 0.7

  # 为特定任务配置备用 agent
  coding:
    model: ollama/codellama:7b
    provider: ollama

  long-context:
    model: google/gemini-2.5-pro
    provider: google
```

---

## 6. 回滚方案

### 6.1 如果需要切回 Anthropic

> ⚠️ **注意：** 如果 Anthropic 持续断供，此方案可能无法使用。建议作为最后手段。

**Step 1:** 恢复备份配置

```bash
cp ~/.config/openclaw/openclaw.yaml.backup-YYYYMMDD ~/.config/openclaw/openclaw.yaml
```

**Step 2:** 检查 Anthropic 连接状态

```bash
openclaw provider status anthropic
```

**Step 3:** 如果连接恢复，验证

```bash
openclaw model test anthropic/claude-sonnet-4 --prompt "test"
```

**Step 4:** 重启 Gateway

```bash
openclaw gateway restart
```

### 6.2 临时禁用 Anthropic（避免报错）

如果暂时无法切换 Provider，可以临时禁用 Anthropic：

```yaml
# openclaw.yaml

providers:
  # 临时禁用 Anthropic
  # anthropic:
  #   enabled: false

  openai:
    apiKey: ${OPENAI_API_KEY}

agents:
  defaults:
    model: openai/gpt-4o
    provider: openai
```

---

## 7. FAQ

### Q1: 我的 OpenClaw 会立即停止工作吗？

**A:** 如果你正在使用 `anthropic/*` 模型作为默认，会立即受到影响。建议**立即切换**到备用 Provider。运行 `openclaw status` 检查你的当前状态。

---

### Q2: 切换 Provider 会丢失我的数据或配置吗？

**A:** 不会。Provider 切换只影响模型调用，不影响你的：
- 对话历史
- 自定义 Agent 配置
- 文件和知识库
- 已安装的 Skills

---

### Q3: 哪个 Provider 最接近 Claude 的使用体验？

**A:** **OpenAI GPT-4o** 与 Claude 最接近：
- 类似的工具调用能力（Function Calling）
- 优秀的代码生成
- 稳定的上下文理解

**Gemini 2.5 Pro** 适合长上下文任务，但工具调用稍弱。

---

### Q4: Kimi 的中文能力如何？

**A:** Kimi 在中文任务上表现优秀，特别是：
- 长文本理解
- 中文对话
- 代码生成（中文注释）

但英文任务的整体质量略低于 GPT-4o。

---

### Q5: Ollama 本地运行性能如何？

**A:** 取决于你的硬件配置：

| 硬件 | 模型 | 延迟 | 推荐场景 |
|------|------|------|----------|
| M1/M2 Mac | llama3.2 | ~30 tokens/s | 开发测试 |
| MacBook Pro M3 | qwen2.5:7b | ~50 tokens/s | 生产轻量任务 |
| 高端 PC (RTX 4090) | deepseek-r1:8b | ~80 tokens/s | 复杂推理 |

---

### Q6: v2026.4.23 安全更新是否与断供有关？

**A:** 时间节点高度重合（4月23日发布安全更新，4月25日断供），推测 OpenClaw 在感知压力后加速了安全修复。20+ 安全补丁可能就是为了应对 Anthropic 的合规要求。建议所有用户升级到 v2026.4.23。

---

### Q7: 如何监控 Provider 状态？

**A:** 使用以下命令：

```bash
# 检查所有 Provider 状态
openclaw provider status all

# 查看详细日志
openclaw logs --level error --provider openai

# 监控实时状态
openclaw status --watch
```

---

### Q8: 切换后遇到 API 错误怎么办？

**A:** 按以下顺序排查：

1. 确认 API Key 正确设置：`echo $OPENAI_API_KEY`
2. 验证配置：`openclaw config validate`
3. 测试连接：`openclaw model test <model> --prompt "test"`
4. 查看日志：`openclaw logs --level debug`

---

## 8. 相关资源

- [OpenClaw 官方文档](https://docs.openclaw.ai)
- [GitHub 仓库](https://github.com/openclaw/openclaw)
- [v2026.4.23 Release Notes](./v2026.4.23.md)
- [Ollama 官方文档](https://ollama.com)
- [OpenAI API 文档](https://platform.openai.com)
- [Google Gemini 文档](https://ai.google.dev)
- [Kimi Moonshot API](https://platform.moonshot.cn)

---

*🦦 墨客 · OpenClaw 知识库 · 紧急迁移指南系列*