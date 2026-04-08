# OpenClaw infer CLI 完全指南

> 全面掌握 `openclaw infer` 顶级命令：模型推理、图片生成、语音合成、音视频处理、网络搜索、Embedding

---

## 元信息

| 字段 | 内容 |
|------|------|
| **title** | OpenClaw infer CLI 完全指南 |
| **description** | 详细讲解 v2026.4.5 新增的 `openclaw infer` 命令（list/run/image/audio/video/search/embedding），包含所有子命令语法、选项、与 Agent tool 调用的区别、provider 能力矩阵。 |
| **tags** | OpenClaw, CLI, infer, openclaw infer, TTS, STT, Image Generation, Embedding |
| **难度** | 🟢 初级 |
| **预计阅读时间** | 20 分钟 |
| **date** | 2026-04-08 |
| **author** | 墨客-内容生成专家 |
| **功能版本** | v2026.4.5+ |

---

## 目录

1. [概述](#1-概述)
2. [infer list — 查看可用模型](#2-infer-list--查看可用模型)
3. [infer model — 模型推理](#3-infer-model--模型推理)
4. [infer image — 图片生成](#4-infer-image--图片生成)
5. [infer audio — 语音合成与转写](#5-infer-audio--语音合成与转写)
6. [infer video — 视频生成](#6-infer-video--视频生成)
7. [infer web — 网络搜索](#7-infer-web--网络搜索)
8. [infer embedding — 向量化](#8-infer-embedding--向量化)
9. [Provider 能力矩阵](#9-provider-能力矩阵)
10. [常见问题](#10-常见问题)

---

## 1. 概述

`openclaw infer` 是 v2026.4.5 新增的顶级 CLI 命令，提供对 OpenClaw 所有 AI 能力的直接访问。与 Agent 内部 tool 调用的区别在于：

| 对比项 | `openclaw infer` | Agent tool 调用 |
|--------|-----------------|----------------|
| 使用者 | 人类（CLI） | AI Agent |
| 场景 | 快速测试、单次任务 | 自动化工作流 |
| 控制粒度 | 全手动 | 可编排 |
| 输出格式 | JSON/文本 | 内部消息流 |
| 批处理 | 需循环 | 原生支持 |

### 快速开始

```bash
# 查看所有可用命令
openclaw infer --help

# 查看可用模型
openclaw infer list --json
```

---

## 2. infer list — 查看可用模型

列出所有配置的 AI 模型及其能力。

### 基本用法

```bash
# 列出所有可用模型（格式化输出）
openclaw infer list

# 输出示例：
# MODEL                  PROVIDER         TYPE      CONTEXT
# claude-sonnet-4-6      anthropic        chat      200K
# gpt-4o                 openai           chat      128K
# gpt-4o-mini            openai           chat      128K
# dalle-3                openai           image     -
# turbo-v2b              minimax          image     -
# elevenlabs             elevenlabs       tts       -
# whisper-1              openai           stt       -
```

### JSON 输出

```bash
# 机器可读格式
openclaw infer list --json

# 输出结构：
{
  "models": [
    {
      "name": "claude-sonnet-4-6",
      "provider": "anthropic",
      "type": "chat",
      "contextWindow": 200000,
      "supportsStreaming": true
    }
  ],
  "timestamp": "2026-04-08T16:00:00Z"
}
```

---

## 3. infer model — 模型推理

对指定模型发送文本 prompt 并获取回复。

### 基本用法

```bash
# 基础对话
openclaw infer model run --prompt "Explain quantum entanglement"

# 指定模型
openclaw infer model run \
  --model "anthropic/claude-sonnet-4-6" \
  --prompt "Write a haiku about coding"
```

### 流式输出

```bash
# 实时流式响应（适合长文本）
openclaw infer model run \
  --model "openai/gpt-4o" \
  --prompt "Write a 500-word story" \
  --stream
```

### JSON 输出

```bash
# 结构化 JSON 响应
openclaw infer model run \
  --model "anthropic/claude-sonnet-4-6" \
  --prompt "What is 2+2?" \
  --json

# 输出：
{
  "model": "claude-sonnet-4-6",
  "provider": "anthropic",
  "response": "2 + 2 = 4",
  "usage": {
    "input_tokens": 12,
    "output_tokens": 8,
    "cost": 0.00032
  },
  "latency_ms": 1243
}
```

### 系统消息与参数

```bash
# 带系统消息
openclaw infer model run \
  --model "openai/gpt-4o" \
  --system "You are a helpful Python tutor" \
  --prompt "How do I sort a list in Python?"

# 调整生成参数
openclaw infer model run \
  --model "openai/gpt-4o" \
  --prompt "Generate a password" \
  --temperature 0.9 \
  --max-tokens 50
```

### 完整选项

| 选项 | 描述 | 默认值 |
|------|------|--------|
| `--model` | 模型名称（provider/name 格式） | config default |
| `--prompt` | 用户 prompt | 必需 |
| `--system` | 系统消息 | 可选 |
| `--temperature` | 随机性 0-2 | 0.7 |
| `--max-tokens` | 最大输出 token | 模型默认 |
| `--json` | JSON 格式输出 | false |
| `--stream` | 流式输出 | false |

---

## 4. infer image — 图片生成

### dalle-3/4 图片生成

```bash
# 基础用法
openclaw infer image generate \
  --prompt "A cute otter wearing a top hat"

# 指定尺寸
openclaw infer image generate \
  --prompt "Mountain landscape at sunset" \
  --size 1024x1024

# 质量选项
openclaw infer image generate \
  --prompt "Abstract art" \
  --quality hd
```

### MiniMax Turbo 图片生成

```bash
# 使用 MiniMax Turbo 模型
openclaw infer image generate \
  --model "minimax/turbo-v2b" \
  --prompt "A cozy coffee shop"
```

### 图片变体（Edit/P variations）

```bash
# DALL-E Edit（局部编辑）
openclaw infer image edit \
  --prompt "Add a cat to this image" \
  --image ./photo.jpg

# DALL-E Variations（生成变体）
openclaw infer image variations \
  --image ./generated-image.png \
  --count 4
```

### 完整选项

| 选项 | 描述 | 默认值 |
|------|------|--------|
| `--prompt` | 图片描述 | 必需 |
| `--model` | 图片生成模型 | dalle-3 |
| `--size` | 图片尺寸 | 1024x1024 |
| `--quality` | 质量 (standard/hd) | standard |
| `--style` | 风格 (vivid/natural) | vivid |
| `--output` | 保存路径 | stdout |

---

## 5. infer audio — 语音合成与转写

### TTS（文字转语音）

```bash
# 基础用法
openclaw infer audio tts \
  --text "Hello, welcome to OpenClaw" \
  --output ./hello.mp3

# 指定语音
openclaw infer audio tts \
  --text "Bonjour le monde" \
  --voice "alloy" \
  --output ./bonjour.mp3

# OpenAI TTS
openclaw infer audio tts \
  --provider openai \
  --text "Good morning" \
  --voice "nova" \
  --output ./morning.mp3

# ElevenLabs TTS
openclaw infer audio tts \
  --provider elevenlabs \
  --text "Hello from ElevenLabs" \
  --voice "your-voice-id" \
  --output ./eleven.mp3
```

### STT（语音转文字）

```bash
# 转写音频文件
openclaw infer audio transcribe \
  --file ./memo.m4a \
  --language en

# 指定语言
openclaw infer audio transcribe \
  --file ./meeting.wav \
  --language zh \
  --output ./transcript.txt
```

### Whisper 实时转写

```bash
# 实时录音转写（需要麦克风）
openclaw infer audio transcribe \
  --mode stream \
  --language en
```

### 完整选项

**TTS 选项：**

| 选项 | 描述 | 默认值 |
|------|------|--------|
| `--text` | 要合成的文本 | 必需 |
| `--provider` | TTS provider | openai |
| `--voice` | 语音名称 | alloy |
| `--output` | 输出文件路径 | stdout |
| `--speed` | 语速 (0.5-2.0) | 1.0 |

**STT 选项：**

| 选项 | 描述 | 默认值 |
|------|------|--------|
| `--file` | 音频文件路径 | 必需 |
| `--language` | 源语言 | auto |
| `--model` | Whisper 模型 | whisper-1 |
| `--output` | 输出文件路径 | stdout |

---

## 6. infer video — 视频生成

```bash
# 基础视频生成（需要支持的 provider）
openclaw infer video generate \
  --prompt "A serene lake at sunrise with ducks swimming"

# 指定时长
openclaw infer video generate \
  --prompt "City traffic at night" \
  --duration 10

# 指定分辨率
openclaw infer video generate \
  --prompt "Ocean waves crashing" \
  --resolution 1080p
```

**注意**：视频生成为高级功能，需要配置支持视频的 provider（如 Reel/Runway）。

---

## 7. infer web — 网络搜索

```bash
# 基础搜索
openclaw infer web search \
  --query "OpenClaw v2026.4.5 release notes"

# 限制结果数量
openclaw infer web search \
  --query "OpenClaw tutorial" \
  --count 5

# JSON 输出
openclaw infer web search \
  --query "OpenClaw GitHub" \
  --json
```

### 搜索选项

| 选项 | 描述 | 默认值 |
|------|------|--------|
| `--query` | 搜索关键词 | 必需 |
| `--count` | 返回结果数 | 10 |
| `--provider` | 搜索 provider | tavily |
| `--json` | JSON 格式输出 | false |

---

## 8. infer embedding — 向量化

生成文本的向量表示，用于 RAG、语义搜索等场景。

```bash
# 基础用法
openclaw infer embedding create \
  --text "The quick brown fox jumps over the lazy dog"

# 指定模型
openclaw infer embedding create \
  --text "OpenClaw is awesome" \
  --model "openai/text-embedding-3-small"

# JSON 输出（包含向量）
openclaw infer embedding create \
  --text "Hello world" \
  --json

# 批量处理（多行输入）
echo -e "First text\nSecond text\nThird text" | \
  openclaw infer embedding create
```

### Embedding 选项

| 选项 | 描述 | 默认值 |
|------|------|--------|
| `--text` | 要向量化的文本 | 必需 |
| `--model` | Embedding 模型 | text-embedding-3-small |
| `--dimensions` | 输出维度 | 模型默认 |
| `--json` | JSON 格式输出 | false |

---

## 9. Provider 能力矩阵

| Provider | Chat | Image | TTS | STT | Video | Search | Embedding |
|----------|------|-------|-----|-----|-------|--------|-----------|
| **anthropic** | ✅ claude-sonnet-4-6, claude-opus-4 | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **openai** | ✅ gpt-4o, gpt-4o-mini, gpt-5.2* | ✅ dalle-3 | ✅ | ✅ whisper-1 | ❌ | ✅ | ✅ |
| **minimax** | ✅ MiniMax-M2.7 | ✅ turbo-v2b | ✅ | ❌ | ❌ | ❌ | ❌ |
| **elevenlabs** | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| **tavily** | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| **openai-codex** | ✅ gpt-5.4* | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |

> ⚠️ 注：GPT-5.x 和 openai-codex 在 v2026.4.5 中存在 Bug（见 [Bug Advisory](./v2026-4-5-bug-advisory.md)）

---

## 10. 常见问题

### Q1: `openclaw infer: command not found`

**原因**：v2026.4.5 以下版本不支持 `infer` 命令。

**解决**：升级 OpenClaw：
```bash
npm update -g openclaw
# 或
brew upgrade openclaw
```

### Q2: 模型返回 400 错误

**原因**：可能是 GPT-5.x max_tokens 问题（v2026.4.5 Bug #62130）

**解决**：降级到 GPT-4o，或等待 v2026.4.6 修复

### Q3: TTS 返回 2013 错误

**原因**：MiniMax TTS 参数类型 Bug（v2026.4.5 Bug #62144）

**解决**：切换到 OpenAI TTS：
```bash
openclaw infer audio tts --provider openai --text "Hello" --output hello.mp3
```

### Q4: 如何查看完整的选项帮助？

```bash
# 全局帮助
openclaw infer --help

# 子命令帮助
openclaw infer model --help
openclaw infer image --help
openclaw infer audio --help
openclaw infer web --help
openclaw infer embedding --help
```

### Q5: 如何将输出用于脚本？

```bash
# 获取 JSON 并用 jq 解析
response=$(openclaw infer model run --model "claude-sonnet-4-6" --prompt "Hi" --json)
echo "$response" | jq '.usage'

# 提取文本内容
openclaw infer model run --model "claude-sonnet-4-6" --prompt "Hello" --json | jq -r '.response'
```

---

## 📝 更新日志

| 日期 | 版本 | 更新内容 |
|------|------|---------|
| 2026-04-08 | v1.0 | 初始版本，基于 PR #62129 合并内容 |
| 2026-04-06 | v2026.4.5 | `openclaw infer` CLI 正式发布 |

---

*本文档由墨客内容生成系统自动生成*  
*功能来源：PR #62129（已合并至 v2026.4.5）*  
*生成时间：2026-04-08 16:08 CST*
