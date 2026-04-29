# OpenClaw v2026.4.26 发布亮点 | 2026-04-28

> 📅 发布日期：2026-04-28 | 来源：GitHub Release v2026.4.26 | 官方

---

## 🚀 版本亮点速览

### 🆕 Cerebras 官方支持

本期新增 **Cerebras 作为捆绑插件**，提供静态模型目录和完整的接入文档：

| 特性 | 说明 |
|------|------|
| **捆绑插件** | 开箱即用，无需手动安装 |
| **静态目录** | 官方维护模型列表 |
| **接入文档** | 完整配置指南 |

```yaml
# 示例配置
providers:
  cerebras:
    apiKey: ${CEREBRAS_API_KEY}
    # Cerebras 官方模型自动可用
```

---

### 🌐 Control UI/Talk 实时传输升级

**Google Live 浏览器会话支持**，新增 WebSocket 传输合同：

- Google Live 浏览器 Talk 会话支持 constrained ephemeral tokens
- Gateway relay 支持后端实时语音插件
- Control UI 实时传输稳定性提升

> 💡 **适用场景**：需要实时语音交互的用户，Google Meet 集成更稳定。

---

### 📊 内存检索能力增强

**新增非对称嵌入端点支持**：

```yaml
memorySearch:
  inputType: query        # 查询类型
  queryInputType: query  # 查询输入类型
  documentInputType: document  # 文档输入类型
```

**Ollama 嵌入优化**：
- `nomic-embed-text` 查询前缀
- `qwen3-embedding` 查询前缀
- `mxbai-embed-large` 查询前缀

---

### 🔐 Matrix E2EE 加密设置简化

新增 `openclaw matrix encryption setup` 命令，一站式完成：

1. Matrix 加密启用
2. 恢复引导
3. 验证状态检查

```bash
# 一键设置 Matrix 加密
openclaw matrix encryption setup
```

---

### 🔄 Claude 导入工具

**新增 Claude 导入器**，支持从 Claude Code 和 Claude Desktop 迁移：

- MCP servers 配置
- Skills 配置
- Command prompts
- Safe archive 状态

```bash
# 预览导入内容
openclaw migrate claude --dry-run

# 执行导入
openclaw migrate claude --apply
```

---

### 🐛 本次修复的重大 Bug

| Issue | 问题 | 状态 |
|-------|------|------|
| #72720 | Gateway 启动挂起（低内存 Linux/Node 24） | ✅ 已修复 |
| #72753 | WebSocket 僵尸客户端问题 | ✅ 已修复 |
| #72739 | Anthropic 扩展思考 prefill 问题 | ✅ 已修复 |
| #72715 | CLI/update 自动更新挂起 | ✅ 已修复 |
| #72665 | 更新后 1006 断开级联 | ✅ 已修复 |
| #72827 | subagent 权限绕过 | ✅ 已修复 |
| #72780 | 压缩重复用户输入问题 | ✅ 已修复 |
| #72774 | SQLite WAL 文件无限增长 | ✅ 已修复 |

---

### 🛠️ 本地模型支持改进

**Ollama 大量优化**：

| 改进项 | 说明 |
|--------|------|
| 上下文窗口 | 尊重自定义 Modelfile `num_ctx` |
| Thinking 控制 | 暴露原生思考级别 |
| 工具调用 | 保留非安全整数参数 |
| 本地发现 | 可选启用，避免意外探测 |
| 请求超时 | 可配置，防止无限等待 |

**LM Studio 改进**：
- 支持无 API key 本地服务器
- LAN/loopback 端点信任默认开启
- Gemma 4 reasoning 正确处理

---

### 🐳 Docker 改进

| 改进 | 说明 |
|------|------|
| 证书安装 | slim 镜像现包含 CA 证书 |
| LM Studio | 通过 `host.docker.internal` 路由 |
| Ollama | Linux 主机网关映射已添加 |

---

### 📈 Cron 任务稳定性

多项 Cron 相关修复：

- `maxConcurrentRuns` 应用于独立 agent-turn lane
- 隔离运行成功判定改进
- 失败提醒目标解析优化
- `failureAlert.includeSkipped` 新增

---

### 🔧 其他重要改进

| 类别 | 改进 |
|------|------|
| **Feishu** | 发送原生卡片按钮，修复 Windows ESM 加载问题 |
| **WhatsApp** | 支持 `HTTPS_PROXY` / `HTTP_PROXY` |
| **Bonjour** | 默认使用系统 hostname，避免冲突 |
| **Browser** | 自动启动，Chrome 启动失败电路保护 |
| **Discord** | 消息并发问题修复 |
| **Google Meet** | 媒体权限改进，音频设备配置 |
| **Telegram** | 修复长期预览流回复时间戳 |
| **Mattermost** | 修复直接消息回复根问题 |

---

## 📝 升级建议

### 🔴 高优先级

- **所有 WebSocket 用户**：#72753 修复，建议升级
- **Docker 用户**：TLS 设置修复必需升级
- **Node 24 + Linux 用户**：#72720 修复必需

### 🟡 建议升级

- **Ollama 用户**：大量改进，强烈建议
- **Matrix E2EE 用户**：新命令简化设置
- **Cron 用户**：稳定性和失败通知改进

---

## 📚 相关文档

- [Ollama 完整配置指南](./openclaw-ollama-setup-guide.md)（待更新）
- [Docker 部署指南](./openclaw-docker-deployment.md)
- [Matrix 加密设置](./openclaw-matrix-e2ee-setup.md)（待创建）
- [本地模型配置](./openclaw-local-models-guide.md)

---

## 🐛 已知问题

| Issue | 问题 | 状态 |
|-------|------|------|
| #72640 | Bedrock 空白文本验证 | 🔍 修复中 |
| #72622 | Bedrock 空白用户消息 | 🔍 修复中 |

---

## 📊 统计

| 指标 | 数量 |
|------|------|
| **总修复项** | 200+ |
| **新功能** | 10+ |
| **Bug 修复** | 190+ |
| **贡献者** | 100+ |

---

*本版本由 OpenClaw 核心团队和社区贡献者共同完成*
*查看完整发布说明：https://github.com/openclaw/openclaw/releases/tag/v2026.4.26*
