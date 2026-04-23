# OpenClaw v2026.4.21 Release Notes — 快速解读

> **发布于**: 2026-04-22  
> **版本**: v2026.4.21  
> **适用**: 所有 OpenClaw 用户

---

## 📦 核心更新

### 🖼️ OpenAI Images 升级：gpt-image-2 默认启用

**影响**: 所有使用 `openai/images` API 的用户

**变更内容**:
- 默认模型从旧版切换到 **`gpt-image-2`**
- 新增支持 **2K** 和 **4K** 分辨率
- 图像质量提升，生成速度优化

**操作建议**: 如果你之前指定了旧版图像模型，建议测试 `gpt-image-2` 以获得更好效果。如遇兼容问题，可回退到之前的版本：

```bash
# 临时回退到旧版（不推荐长期使用）
openclaw config set providers.openai.images.model "gpt-image-1"
```

---

## 🔐 安全修复

### ✅ Auth/Commands 权限绕过漏洞修复 (#69774)

**严重度**: 高  
**修复者**: @drobison00

**问题描述**: 之前版本中，owner-enforced 命令存在权限绕过漏洞，攻击者可能利用此漏洞执行未授权操作。

**影响版本**: v2026.4.20 及之前版本  
**修复版本**: **v2026.4.21** ✅ 已修复

---

## 🐛 重要 Regression 修复

> ⚠️ 警告：以下问题在 v2026.4.21 之前的版本中存在，升级前请注意

### #70351 — CLI 启动崩溃

**问题**: npm 包中未正确打包 extension runtime 依赖，导致 CLI 启动时崩溃

**状态**: v2026.4.21 已修复 ✅

### #70343 — 安装后缺失 @larksuiteoapi/node-sdk

**问题**: v2026.4.21 更新后，部分用户安装时缺少 `@larksuiteoapi/node-sdk` 依赖

**状态**: v2026.4.21 已修复 ✅

**临时解决**（如遇问题）:
```bash
npm install @larksuiteoapi/node-sdk
```

### #70342 — Telegram 沙箱绕过（Regression）

**问题**: Telegram 直接聊天可能绕过沙箱机制，而 TUI 和 heartbeat/non-main runs 正常使用沙箱

**状态**: v2026.4.21 已修复 ✅

### #70331 — Tool Calls 以可见 XML/Bracket 文本泄露

**问题**: v2026.4.22 中 Tool Calls 以可见的 XML 或括号文本形式泄露给用户，影响所有模型

**状态**: 此问题在 **v2026.4.22** 出现，v2026.4.21 不受影响

---

## 🔧 其他修复

| Issue # | 描述 | 组件 |
|---------|------|------|
| - | Slack thread alias 保留问题修复 (#62947) | Slack |
| - | Browser 拒绝无效 `ax<N>` accessibility refs | Browser |
| - | 消除 `node-domexception` 废弃链 | npm/install |
| #70339 | Feishu lazy-load setup client | Feishu |
| #70326 | 跳过 bundled plugin 加载失败 | status flows |

---

## ⚡ 快速升级指南

### 自动升级

```bash
openclaw update
```

### 验证版本

```bash
openclaw --version
# 确认输出: v2026.4.21 或更高
```

### 降级（如遇问题）

```bash
npm install -g openclaw@v2026.4.20
```

---

## 📋 升级检查清单

```
[ ] 确认当前版本: openclaw --version
[ ] 执行升级: openclaw update
[ ] 验证 Gateway 重启正常
[ ] 测试核心功能（消息收发、工具调用）
[ ] 如使用 OpenAI Images，测试图像生成
[ ] 检查日志无异常: openclaw logs --tail 50
```

---

## 🔮 后续关注

- **v2026.4.22** 已发布，包含 #70331 Tool Calls 泄露修复
- 建议保持关注最新版本，遇到问题及时回退

---

## 📚 相关文档

- [v2026.4.20 Release Notes](./release-notes-v2420-highlights.md)
- [OpenClaw 安装指南](../installation/)
- [故障排除](../troubleshooting/)

---

*文档生成时间: 2026-04-24 | Agent: mo-yunying-cron*