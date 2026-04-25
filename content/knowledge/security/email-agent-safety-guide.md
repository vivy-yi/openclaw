---
title: "OpenClaw 邮件 Agent 安全配置指南"
description: "安全使用 OpenClaw 邮件 Agent，防止误操作导致数据丢失。涵盖危险操作分级、二次确认机制、Gmail OAuth 最小权限配置等。"
tags: [OpenClaw, Security, Email, Agent, Safety]
难度: intermediate
预计阅读时间: 8分钟
date: 2026-04-25
---

# OpenClaw 邮件 Agent 安全配置指南

> **适用版本:** v2026.4.23 及以上  
> **安全等级:** 🔴 关键  
> **关联事件:** Meta AI Alignment director 收件箱意外清空（Tom's Hardware, 2026-04-25）

---

## 📋 背景事件

### 发生了什么

2026年4月25日，Tom's Hardware 报道了一起 OpenClaw 用户意外清空 Meta AI Alignment director 收件箱的事件。该事件在 Reddit 获得 **22,136+ 评分**，达到最高级 viral 热度。

**事件经过（推断）:**
1. 用户配置 OpenClaw 处理 Gmail 邮件管理（IMAP / Gmail API）
2. OpenClaw agent 误识别"清空收件箱"指令并执行
3. 缺少二次确认机制（没有 `gateway.unsafeExecutionWhitelist` 保护）
4. 邮件永久删除，无法回滚

### 根因分析

| 因素 | 说明 |
|------|------|
| **权限过大** | Agent 拥有完整邮箱访问权限，包括删除 |
| **缺少防护** | 没有危险操作二次确认机制 |
| **状态不可回滚** | 邮件删除后无法恢复（除非有备份） |

---

## 🚦 危险操作分级

### 🔵 只读操作（低风险）

| 操作 | 风险 | 建议 |
|------|------|------|
| 读取邮件 | 极低 | ✅ 可执行 |
| 搜索邮件 | 极低 | ✅ 可执行 |
| 查看附件列表 | 极低 | ✅ 可执行 |
| 获取邮件头信息 | 极低 | ✅ 可执行 |

### 🟡 草稿操作（中等风险）

| 操作 | 风险 | 建议 |
|------|------|------|
| 写邮件草稿 | 中等 | ⚠️ 建议二次确认 |
| 移动邮件到文件夹 | 中等 | ⚠️ 建议二次确认 |
| 标记已读/未读 | 低 | ⚠️ 可执行，注意分类 |
| 设置提醒 | 低 | ✅ 可执行 |

### 🟠 发送操作（高风险）

| 操作 | 风险 | 建议 |
|------|------|------|
| 发送邮件 | 高 | 🔴 必须二次确认 |
| 回复邮件 | 高 | 🔴 必须二次确认 |
| 转发邮件 | 高 | 🔴 必须二次确认 |
| 添加附件 | 中等 | ⚠️ 确认附件内容 |

### 🔴 危险操作（极高风险）

| 操作 | 风险 | 建议 |
|------|------|------|
| 删除单封邮件 | 极高 | 🔴 必须二次确认 + 延迟执行 |
| 清空收件箱 | 极高 | 🔴 必须双重确认 + 日志记录 |
| 永久删除邮件 | 极高 | 🔴 禁用或仅在明确要求时执行 |
| 删除文件夹/标签 | 极高 | 🔴 禁用或仅在明确要求时执行 |
| 修改邮箱设置 | 高 | 🔴 禁用自动化修改 |

---

## ⚙️ 二次确认机制配置

### 基础配置

在 `openclaw.yaml` 中启用危险操作确认：

```yaml
# openclaw.yaml
gateway:
  # 危险操作确认配置
  unsafeExecutionWhitelist:
    - browser:delete:mail
    - browser:empty:trash
    - exec:delete:file
  
  # 操作延迟（给用户撤销时间）
  unsafeExecutionDelaySeconds: 5
  
  # 日志记录所有危险操作
  logAllOperations: true
  logLevel: debug

# Email 专用配置
email:
  # 启用二次确认的操作类型
  confirmOnDelete: true
  confirmOnSend: true
  confirmOnEmptyTrash: true
  
  # 删除前等待秒数（可撤销）
  deleteDelaySeconds: 10
  
  # 删除操作前强制备份到 [Gmail]/Trash
  softDeleteOnly: true  # true = 移至 Trash，false = 永久删除
```

### 高级配置：自定义确认提示

```yaml
email:
  confirmations:
    delete:
      title: "⚠️ 确认删除邮件"
      message: "即将删除以下邮件：\n{{subject}}\n发件人：{{from}}\n\n此操作将在 {{deleteDelaySeconds}} 秒后执行，可在此期间撤销。"
      requireExplicitConfirm: true
    
    send:
      title: "📤 确认发送邮件"
      message: "即将发送邮件：\n收件人：{{to}}\n主题：{{subject}}\n\n请确认邮件内容正确。"
      requireExplicitConfirm: true
    
    empty_trash:
      title: "🗑️ 确认清空垃圾箱"
      message: "即将永久删除垃圾箱中的所有邮件（共计 {{count}} 封）。此操作不可撤销！"
      requireDoubleConfirm: true  # 需要输入 "DELETE" 确认
```

---

## 🔐 Gmail OAuth 安全范围

### 最小权限原则

创建 Gmail OAuth 应用时，只申请必需权限：

| 权限范围 | 用途 | 风险 |
|---------|------|------|
| `gmail.readonly` | 只读访问 | 低 |
| `gmail.compose` | 撰写草稿 | 中 |
| `gmail.send` | 发送邮件 | 高 |
| `gmail.modify` | 修改邮件 | 高 |
| `gmail.delete` | 删除邮件 | 🔴 极高 |

### 推荐配置

```yaml
# openclaw.yaml
email:
  gmail:
    oauth:
      scopes:
        - https://www.googleapis.com/auth/gmail.readonly    # 只读
        - https://www.googleapis.com/auth/gmail.compose     # 草稿
        - https://www.googleapis.com/auth/gmail.send        # 发送
      
      # 明确禁用危险权限
      disabledScopes:
        - https://www.googleapis.com/auth/gmail.delete
        - https://www.googleapis.com/auth/gmail.settings
    
    # 不使用完整 Gmail API，改为 IMAP（更细粒度控制）
    useImap: true
    imap:
      host: imap.gmail.com
      port: 993
      # IMAP 权限：只读 + 发送，不含删除
      capabilities:
        - READ
        - EXPUNGE  # 软删除（移动到 [Gmail]/Trash）
        - APPEND   # 发送
```

### OAuth 令牌安全

```yaml
email:
  gmail:
    oauth:
      tokenStorage: encrypted  # 加密存储 refresh token
      autoRefresh: true
      # 不保存明文 OAuth credentials
      credentialsEnvVar: GMAIL_OAUTH_CREDENTIALS
      
      # 定期轮转 access token
      rotateAccessToken: true
      accessTokenTtlSeconds: 3600
```

---

## 🛡️ IMAP 安全配置

### 基础 IMAP 配置

```yaml
email:
  imap:
    # 连接安全
    ssl: true
    port: 993
    
    # 权限限制（移除 DELETE 权限模拟）
    permissions:
      read: true
      write: true
      delete: false          # 禁用直接删除
      expunge: false         # 禁用永久删除
    
    # 软删除策略：将删除操作转为移动到 Trash
    softDelete:
      enabled: true
      trashFolder: "[Gmail]/Trash"
      trashRetentionDays: 30  # 30天后自动清理
```

### 使用存档代替删除

```yaml
email:
  imap:
    # 用存档代替删除
    archiveInsteadOfDelete: true
    archiveFolder: "[Gmail]/All Mail"
    
    # 删除操作拦截
    deleteAction: move_to_archive  # move_to_trash / move_to_archive / block
    
    # 清空收件箱转为标记已读 + 移动到 All Mail
    emptyTrashAction: mark_read_and_archive
```

---

## 📊 操作日志与审计

### 启用详细日志

```yaml
gateway:
  # 记录所有邮件操作
  logLevel: debug
  
  # 邮件操作专用日志
  emailAudit:
    enabled: true
    logFile: ./logs/email-audit.log
    logFormat: json
    
    # 记录的操作类型
    logOperations:
      - read
      - send
      - delete
      - move
      - empty_trash
    
    # 日志保留时间
    retentionDays: 90
```

### 日志格式示例

```json
{
  "timestamp": "2026-04-25T14:32:15Z",
  "operation": "delete",
  "target": {
    "type": "email",
    "messageId": "<CAMM_abc123@mail.gmail.com>",
    "subject": "Re: Project Update",
    "from": "alice@example.com"
  },
  "status": "pending_confirm",
  "confirmRequired": true,
  "confirmDeadline": "2026-04-25T14:32:25Z",
  "agentSession": "subagent:abc123"
}
```

### 查看操作日志

```bash
# 查看最近的危险操作日志
openclaw logs --level debug --since 1h | grep -E "(delete|empty_trash|send)" | jq .

# 导出审计报告
openclaw audit --operation delete --since 2026-04-01 --format csv > email-delete-audit.csv
```

---

## 🔄 紧急回滚方案

### 场景 1: 邮件已删除但未超过 30 天

Gmail 的"垃圾箱"保留 30 天，可在设置中恢复：

1. 登录 Gmail 网页版
2. 进入"垃圾箱"文件夹
3. 选择要恢复的邮件 → 点击"删除前移至收件箱"
4. 或使用 Gmail API 批量恢复

### 场景 2: 已超过垃圾箱保留期

```yaml
# 提前配置邮件备份
email:
  backup:
    enabled: true
    schedule: "0 2 * * *"  # 每天凌晨 2 点
    
    # 备份到本地
    localBackup:
      enabled: true
      path: ./backups/emails
      format: eml  # 单封邮件格式
      compress: true
    
    # 或备份到云存储
    cloudBackup:
      enabled: true
      provider: google_drive
      folder: OpenClaw-Email-Backup
```

### 场景 3: 意外清空收件箱后

```bash
# 检查是否有备份
ls ./backups/emails/

# 查看备份中是否有清空操作的记录
grep -r "empty_trash" ./logs/email-audit.log

# 联系 Gmail 支持（数据恢复请求）
# https://support.google.com/mail/troubleshooter/9602689
```

---

## ✅ 配置清单

### 快速检查表

| 配置项 | 推荐值 | 检查 |
|--------|--------|------|
| `gateway.unsafeExecutionWhitelist` | 启用 | ☐ |
| `email.confirmOnDelete` | `true` | ☐ |
| `email.confirmOnSend` | `true` | ☐ |
| `email.softDeleteOnly` | `true` | ☐ |
| `email.deleteDelaySeconds` | `10` | ☐ |
| `email.gmail.useImap` | `true` | ☐ |
| `gateway.logAllOperations` | `true` | ☐ |

### 完整配置示例

```yaml
# openclaw.yaml — 邮件安全配置
gateway:
  unsafeExecutionWhitelist:
    - email:delete
    - email:empty_trash
    - email:send
  
  unsafeExecutionDelaySeconds: 5
  logAllOperations: true
  logLevel: debug

email:
  confirmOnDelete: true
  confirmOnSend: true
  confirmOnEmptyTrash: true
  deleteDelaySeconds: 10
  softDeleteOnly: true
  
  gmail:
    oauth:
      scopes:
        - https://www.googleapis.com/auth/gmail.readonly
        - https://www.googleapis.com/auth/gmail.compose
        - https://www.googleapis.com/auth/gmail.send
      disabledScopes:
        - https://www.googleapis.com/auth/gmail.delete
    useImap: true
    imap:
      permissions:
        read: true
        write: true
        delete: false
      softDelete:
        enabled: true
        trashFolder: "[Gmail]/Trash"
  
  backup:
    enabled: true
    localBackup:
      enabled: true
      path: ./backups/emails
```

---

## ❓ FAQ

**Q: 为什么不能直接授予完整邮箱权限？**  
A: 最小权限原则降低数据泄露风险。即使 OpenClaw 被入侵，攻击者也无法直接删除或导出邮件。

**Q: 误删邮件后多久能恢复？**  
A: Gmail 垃圾箱保留 30 天。超过后 Google 不保证恢复。强烈建议开启自动备份。

**Q: v2026.4.23 的 20+ 安全补丁与此有何关联？**  
A: 此次更新包含 `gateway.unsafeExecutionWhitelist` 和 `exec` 权限收紧，可能与此类邮件误操作事件有关。建议升级后检查配置。

**Q: 可以完全禁用邮件删除操作吗？**  
A: 可以。将 `email.confirmOnDelete` 和 `email.confirmOnEmptyTrash` 均设为 `true`，并使用 `softDeleteOnly: true`，可实现删除前必须二次确认且只移至垃圾箱。

**Q: OpenClaw 邮件 Agent 支持哪些邮件服务商？**  
A: Gmail（推荐）、Outlook、Yahoo、QQ邮箱、企业邮箱（IMAP）。

---

## 📚 相关资源

- [OpenClaw v2026.4.23 发布说明](../releases/v2026.4.23.md)
- [Anthropic 断供迁移指南](../migrations/anthropic-shutdown-migration-guide.md)
- [OpenClaw 安全配置指南](https://docs.openclaw.ai/security)
- [Gmail API 权限文档](https://developers.google.com/gmail/api/reference/scopes)

---

*🦦 墨客 · OpenClaw 安全配置系列*