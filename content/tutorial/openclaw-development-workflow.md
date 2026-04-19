---
title: "OpenClaw 开发流程规范 — 引入 Addy Osmani 质量门禁体系"
description: "将 Addy Osmani 的七大命令体系和质量门禁设计融入 OpenClaw，提供系统性的 AI 驱动开发流程指南。"
tags: [OpenClaw, 开发流程, 质量门禁, Agent Skills, 最佳实践]
难度: intermediate
预计阅读时间: 15分钟
---

# OpenClaw 开发流程规范 — 引入 Addy Osmani 质量门禁体系

> **背景**：传统 AI 编码依赖"直觉"和"临场判断"，导致代码质量不稳定、流程缺乏规范。本教程将 Addy Osmani 的七大命令体系和质量门禁设计融入 OpenClaw，帮助你建立系统性的 AI 驱动开发流程。

**目标读者**：希望用 OpenClaw 执行系统性开发任务的工程师  
**前置要求**：熟悉 OpenClaw 基础工具（exec、browser、sessions_spawn）

---

## 一、为什么 AI 编码需要开发流程

当你用 OpenClaw 执行编码任务时，是否遇到过以下问题：

- 代码写完了，但边界情况没考虑周全
- 改了一个文件，却意外影响了其他模块
- 提交后才发现有安全问题或性能问题
- 任务做到一半，发现方向跑偏了

这些问题并非工具能力不足，而是**缺乏系统性的流程约束**。

Addy Osmani（Google Chrome 团队工程师）设计了一套完整的 Agent Skills 体系，将资深工程师的开发经验编码为可验证的流程规则。这套体系已经被超过 17,000 个 GitHub Stars 证明其价值。

---

## 二、七大命令体系：AI 编码的完整生命周期

Addy Osmani 的命令体系覆盖了从想法到上线的完整开发周期：

```
DEFINE          PLAN           BUILD          VERIFY         REVIEW          SHIP
 ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐
 │ Idea │ ───▶ │ Spec │ ───▶ │ Code │ ───▶ │ Test │ ───▶ │  QA  │ ───▶ │  Go  │
 │Refine│      │  PRD │      │ Impl │      │Debug │      │ Gate │      │ Live │
 └──────┘      └──────┘      └──────┘      └──────┘      └──────┘      └──────┘
  /spec          /plan          /build        /test         /review       /ship
```

| 命令 | 核心原则 | 使用场景 |
|------|----------|----------|
| `/spec` | Spec before code | 需求模糊、启动新项目 |
| `/plan` | Small, atomic tasks | 已有 spec，拆解任务 |
| `/build` | One slice at a time | 增量实现、任何多文件变更 |
| `/test` | Tests are proof | 编写测试、验证逻辑 |
| `/review` | Improve code health | 合并前质量把控 |
| `/code-simplify` | Clarity over cleverness | 代码可读性/可维护性下降 |
| `/ship` | Faster is safer | 部署上线 |

### 在 OpenClaw 中应用

OpenClaw 不使用斜杠命令，但可以通过 Skill 描述匹配触发类似的工作流：

```markdown
# 启动新任务时的提示示例

我需要开发一个用户认证模块。请先完成以下工作：

1. **Spec 阶段**：描述模块的目标、技术方案、接口设计、边界情况
2. **Plan 阶段**：拆解为 3-5 个小任务，每个任务完成后验证
3. **Build 阶段**：每次只实现一个垂直切片

完成后请按五轴评估进行自检：correctness/readability/architecture/security/performance
```

---

## 三、Spec-Driven Development：需求规范的第一步

### 3.1 什么是 Spec

Spec（规格说明）是比 PRD 更轻量的需求文档。它的核心价值在于：**在写代码之前，先对齐理解和期望**。

### 3.2 Spec 的六核心区域

当你用 OpenClaw 启动新项目或重大功能时，要求模型先输出以下内容：

```markdown
## Spec: [功能名称]

### 1. Objective（目标）
- 这个功能解决什么问题？
- 成功标准是什么？

### 2. Commands（可用的命令/工具）
- 实现过程中可使用哪些工具？
- 哪些操作需要人工批准？

### 3. Project Structure（项目结构）
- 目录如何组织？
- 主要模块的职责是什么？

### 4. Code Style（代码风格）
- 命名规范
- 注释要求
- 禁止的模式

### 5. Testing Strategy（测试策略）
- 单元测试覆盖率目标
- 集成测试场景
- 手动验证清单

### 6. Boundaries（边界）
- Always do：必须执行（如测试通过后才能合并）
- Ask first：需人工批准（如数据库 schema 变更）
- Never do：禁止执行（如直接 commit secrets）
```

### 3.3 OpenClaw 中的 Spec 实操

```markdown
# 创建新功能时的标准开场白

请先阅读以下规范，然后输出 Spec 再开始实现：

## 规范要求
1. 假设前置：列出你对现有代码库的假设
2. 六区域覆盖：Objective/Commands/Structure/Style/Testing/Boundaries
3. 三层边界：明确 Always do / Ask first / Never do

## 示例输出格式

假设：
- 后端 API 基于 Express.js
- 数据库使用 PostgreSQL
- 前端已有 React 基础组件库

Spec: 用户注册模块
[按照六区域格式输出]
```

---

## 四、五轴 Code Review：在 OpenClaw 中自动质量把控

### 4.1 五轴评估体系

每次代码变更后，用以下五个维度进行自检：

| 轴 | 核心问题 | OpenClaw 检查方法 |
|----|----------|-------------------|
| **Correctness** | 代码是否做了它声称的事？ | 运行测试，检查边界情况 |
| **Readability** | 另一个工程师能否理解？ | 代码审查，检查命名和注释 |
| **Architecture** | 是否引入不必要的复杂性？ | 检查模块边界，是否符合现有架构 |
| **Security** | 是否有安全漏洞？ | 检查输入验证，密钥管理 |
| **Performance** | 是否有性能问题？ | 检查 N+1 查询、无界循环 |

### 4.2 OpenClaw 中的自动化检查

```markdown
# 代码提交前的检查清单

完成实现后，请执行以下检查：

## 1. Correctness
- [ ] 所有测试通过：`exec` 运行测试命令
- [ ] 边界情况覆盖：检查 null/undefined/空数组

## 2. Readability  
- [ ] 命名清晰：变量名、函数名见名知意
- [ ] 注释到位：复杂逻辑有解释

## 3. Architecture
- [ ] 模块边界清晰
- [ ] 没有循环依赖

## 4. Security
- [ ] 用户输入有验证
- [ ] 无硬编码密钥（使用环境变量）
- [ ] SQL 注入防护

## 5. Performance
- [ ] 无 N+1 查询问题
- [ ] 无无界循环
- [ ] 大数据集合有分页

## 五轴自检通过后，再进行 Git commit
```

### 4.3 使用 exec 工具做自动化验证

```javascript
// 示例：OpenClaw exec 命令进行代码质量检查

// 1. 运行测试
exec({ command: "npm test" })

// 2. 检查代码格式
exec({ command: "npm run lint" })

// 3. 检查类型
exec({ command: "npx tsc --noEmit" })

// 4. 安全扫描
exec({ command: "npm audit --audit-level=moderate" })
```

---

## 五、变更管理：100 行原则

### 5.1 为什么变更大小重要

| 变更规模 | 可审查性 | 建议 |
|----------|----------|------|
| ~100 行 | ✅ 一个会话可完成 | 推荐目标 |
| ~300 行 | 🟡 单逻辑变更可接受 | 需要详细说明 |
| ~1000 行 | ❌ 太大 | 必须拆分 |

### 5.2 拆分策略

当变更超过 100 行时，采用以下策略拆分：

| 策略 | 方法 | 适用场景 |
|------|------|----------|
| **Stack** | 提交小变更，基于它开始下一个 | 顺序依赖 |
| **By file group** | 不同变更组对应不同 reviewer | 横切关注点 |
| **Horizontal** | 先创建共享代码/桩，然后是消费者 | 分层架构 |
| **Vertical** | 拆分为更小的全栈功能切片 | 功能工作 |

### 5.3 OpenClaw 中的增量实现

```markdown
# 大任务拆解示例

原始任务：重构整个用户模块

拆解后的执行计划：

## Sprint 1: 数据层
1. 创建 UserRepository 类
2. 添加单元测试
3. 验证测试通过

## Sprint 2: 服务层
1. 创建 UserService 类
2. 依赖 UserRepository
3. 添加单元测试

## Sprint 3: API 层
1. 创建 UserController
2. 挂载路由
3. 集成测试

每个 Sprint 完成时执行五轴自检，确认无问题后再进行下一个。
```

---

## 六、安全边界：三层系统

### 6.1 三层边界定义

| 层级 | 定义 | 示例 |
|------|------|------|
| **Always do** | 必须执行 | 运行测试后再提交、输入验证、使用环境变量存储密钥 |
| **Ask first** | 需人工批准 | 数据库 schema 变更、删除数据、外部 API 密钥配置 |
| **Never do** | 禁止执行 | 直接 commit secrets、硬编码密码、删除日志 |

### 6.2 OpenClaw 安全实践

```markdown
# 安全边界示例

在开始任何涉及以下内容的任务时，先确认你已经：

## Always do ✅
- [ ] 密钥存储在环境变量，不硬编码
- [ ] 用户输入进行验证和清理
- [ ] SQL 查询使用参数化语句
- [ ] 敏感操作有日志记录

## Ask first ⚠️
- [ ] 数据库结构变更（先备份）
- [ ] 删除用户数据（需二次确认）
- [ ] 修改认证逻辑（先讨论）

## Never do 🚫
- [ ] 直接 commit .env 或 credentials
- [ ] 在代码中硬编码密码
- [ ] 忽略安全警告继续执行
```

### 6.3 使用 browser 工具时的安全注意

```markdown
# OAuth/第三方集成安全

当使用 browser 工具自动配置 OAuth 时：

1. Ask first：确认 redirect URI 和 scopes
2. 验证：检查最终 redirect 的 URL 是否正确
3. 不要：保存 access token 到代码中
4. 使用：browser 的安全模式处理敏感数据
```

---

## 七、OpenClaw 集成示例

### 7.1 完整开发流程演示

假设任务：为一个现有 Express 项目添加用户注册功能

```markdown
## Step 1: Spec 阶段

启动 OpenClaw 会话，描述：

"我需要为现有 Express 项目添加用户注册功能。请先输出完整的 Spec，包括：
1. 功能目标
2. API 设计（RESTful）
3. 数据模型
4. 测试策略
5. 安全边界（三层系统）"

## Step 2: Plan 阶段

根据 Spec，拆解任务：

1. 创建 User 模型（user.js + user.test.js）
2. 实现 POST /api/users 路由
3. 添加输入验证（email 格式、密码强度）
4. 实现密码哈希（bcrypt）
5. 集成测试

## Step 3: Build + Verify 循环

每个小任务执行：
1. 实现代码
2. 编写/更新测试
3. 运行测试验证
4. 五轴自检
5. 提交（如果变更 ~100 行以内）

## Step 4: Review 阶段

功能完成后，执行完整 review：

- Correctness：测试全部通过？
- Readability：代码可读？
- Architecture：模块边界清晰？
- Security：无安全问题？
- Performance：无性能隐患？

## Step 5: Ship 阶段

检查清单：
- [ ] 所有测试通过
- [ ] 代码已格式化
- [ ] 无 lint 警告
- [ ] 安全扫描通过
- [ ] 已更新文档（如需要）
```

### 7.2 多 Agent 协作场景

当任务复杂时，使用 sessions_spawn 并行处理：

```markdown
# 并行开发示例

任务：需要同时开发前端注册表单和后端 API

主 Agent（你）：
- 负责任务分配和协调
- 执行后端 API 开发
- 最终集成测试

子 Agent（spawned）：
- 执行前端表单开发
- 使用相同的 Spec 和安全边界

主 Agent 发出指令：
"请用 sessions_spawn 启动两个子 Agent：
1. backend-dev：实现 POST /api/users
2. frontend-dev：实现注册表单组件

两者都遵循以下规范：
- Spec: 用户注册模块 v1.0
- 安全边界：Always do / Ask first / Never do
- 变更限制：单个 commit ≤ 100 行"
```

---

## 八、工具速查卡

| 工作流 | OpenClaw 工具 | 关键参数 |
|--------|---------------|----------|
| 运行测试 | `exec` | `command: "npm test"` |
| 代码格式检查 | `exec` | `command: "npm run lint"` |
| 类型检查 | `exec` | `command: "npx tsc --noEmit"` |
| Git 提交 | `exec` | `command: "git add . && git commit"` |
| 安全扫描 | `exec` | `command: "npm audit"` |
| 浏览器验证 | `browser` | `action: "snapshot"` |
| 并行开发 | `sessions_spawn` | `mode: "run"` |
| 状态同步 | `sessions_send` | `sessionKey` |

---

## 九、总结

将 Addy Osmani 的七大命令体系和质量门禁设计融入 OpenClaw，可以显著提升 AI 编码的可靠性和规范性。

**核心要点**：

1. **先 Spec 再代码**：用六区域框架明确需求和边界
2. **小步快跑**：遵循 100 行原则，频繁验证
3. **五轴自检**：Correctness / Readability / Architecture / Security / Performance
4. **三层安全边界**：Always do / Ask first / Never do
5. **善用工具**：exec 做自动化检查，sessions_spawn 做并行开发

---

## 附录：相关资源

- [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) — 官方 Skills 仓库（17k+ Stars）
- [OpenClaw Skills 系统](https://docs.openclaw.ai) — Skill 创建和管理指南
- [OpenClaw 编码代理](../coding-agent/) — OpenClaw 官方编码技能

---

*🦦 本教程由墨客-内容生成基于深度调研自动生成*  
*调研报告：[深度调研报告-AddyOsmani-AgentSkills-2026-04-20.md](../../tmp/深度调研报告-AddyOsmani-AgentSkills-2026-04-20.md)*