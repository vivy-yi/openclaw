---
title: OpenClaw + Addy Osmani Agent Skills 实战指南
description: 将 Addy Osmani 的七大命令体系和 20 个 Skills 融入 OpenClaw 工作流
---

# OpenClaw + Addy Osmani Agent Skills 实战指南

**作者**: 墨客内容生成  
**对象**: Addy Osmani [agent-skills](https://github.com/addyosmani/agent-skills) (17,553 ⭐)  
**目标**: 将全球最系统的 AI 工程技能体系融入 OpenClaw 工作流  

---

## 概述

Addy Osmani 是 Google Chrome 团队资深工程师，其 `agent-skills` 项目（17,553 ⭐）是目前最系统的 AI Agent 工程技能库。该体系将资深工程师的开发流程、质量门禁和最佳实践编码为 AI Agent 可遵循的结构化指令系统。

**为什么 OpenClaw 需要这套体系？**

| 当前 OpenClaw | Addy Osmani Skills |
|---------------|-------------------|
| 工具驱动（做什么） | 流程驱动（怎么做） |
| 依赖人工判断 | 系统化五轴评估 |
| 无标准需求格式 | PRD/Spec 模板 |
| 无变更行数限制 | ~100 行原则 |
| 安全规范分散 | 三层边界系统 |

---

## 七大命令体系

```
DEFINE ──→ PLAN ──→ BUILD ──→ VERIFY ──→ REVIEW ──→ SHIP
  /spec     /plan     /build      /test     /review    /ship
   Idea      Spec      Code       Test       QA        Go
```

### /spec — 规范先行

**核心原则**: Spec before code（先规范，后实现）

在 OpenClaw 中，你可以这样触发：

```
描述你的需求 → OpenClaw 生成 SPEC → 你 review → 确认后进入 PLAN
```

**质量门禁**: 在写任何 spec 内容前，列出你的假设

**Spec 应包含**:
- **Objective**: 目标是什么？
- **Commands**: 使用哪些命令？
- **Project Structure**: 项目结构
- **Code Style**: 代码风格
- **Testing Strategy**: 测试策略
- **Boundaries**: 边界条件

### /plan — 任务拆解

**核心原则**: Small, atomic tasks（小而原子化的任务）

OpenClaw sessions_spawn 可用于将大任务拆解为可执行单元：

```javascript
// 使用 sessions_spawn 并行执行子任务
await sessions_spawn({
  tasks: [
    "实现用户认证模块（spec已确认）",
    "编写认证模块单元测试",
    "集成数据库连接"
  ],
  mode: "parallel"
})
```

**验收标准**: 每个任务应有明确的完成定义

### /build — 增量实现

**核心原则**: One slice at a time（一次一个切片）

薄垂直切片工作流：

```
实现 → 测试 → 验证 → 提交
  ↓      ↓       ↓       ↓
 代码  测试   确认通过  atomic commit
```

**OpenClaw 实战**：

```javascript
// 使用 coding-agent skill 进行增量实现
await sessions_spawn({
  task: "实现用户认证模块（参照 SPEC.md）",
  runtime: "subagent",
  skill: "coding-agent"
})
```

**特性开关**: 每个增量功能应有特性开关

### /test — 测试驱动

**核心原则**: Tests are proof（测试即证明）

**测试金字塔**:

```
        /\
       /  \     E2E Tests (5%)
      /____\
     /      \   Integration Tests (15%)
    /________\
   /          \  Unit Tests (80%)
  /____________\
```

**DAMP over DRY**: Descriptive And Meaningful Patterns > Don't Repeat Yourself

**Beyonce Rule**: "If you liked it, you should've put a test on it"

**OpenClaw 集成**:

```bash
# 在 OpenClaw 中执行测试
openclaw exec "npm test -- --coverage"
openclaw exec "python -m pytest --cov"
```

### /review — 质量门禁

**核心原则**: Improve code health before merge（合并前提升代码健康度）

**五轴 Code Review**:

| 维度 | 核心问题 |
|------|----------|
| **Correctness** | 代码是否做了它声称的事？边界情况？错误路径？ |
| **Readability** | 另一个工程师能否理解这段代码？ |
| **Architecture** | 变更是否符合系统设计？是否引入不必要的复杂性？ |
| **Security** | 是否引入安全漏洞？输入验证？密钥管理？ |
| **Performance** | 是否引入性能问题？N+1？无界循环？ |

**OpenClaw review 示例**:

```bash
# 使用 browser tool 进行 PR review
openclaw browser open https://github.com/openclaw/openclaw/pull/XXX
# 或者使用 exec + gh CLI
openclaw exec "gh pr review PR_NUMBER --approve"
```

### /ship — 部署上线

**核心原则**: Faster is safer（更快更安全）

**上线前检查清单**:

- [ ] 所有测试通过
- [ ] Code review 已批准
- [ ] 特性开关已配置
- [ ] 监控已设置
- [ ] 回滚程序已确认

**OpenClaw 部署工作流**:

```bash
# 使用 exec tool 执行部署
openclaw exec "./scripts/deploy.sh --env=production --confirm"
```

---

## OpenClaw 工具集映射

| Addy Osmani 命令 | OpenClaw 工具 | 说明 |
|-----------------|--------------|------|
| `/spec` | sessions_send + 编辑器 | 规范文档生成 |
| `/plan` | sessions_spawn | 任务拆解与并行执行 |
| `/build` | exec + coding-agent | 代码实现 |
| `/test` | exec + browser | 测试执行与验证 |
| `/review` | browser + gh CLI | PR review |
| `/ship` | exec + 部署脚本 | 部署执行 |

**OpenClaw 完整工作流**:

```javascript
// 1. 定义规范
await feishu_doc.write({ content: specContent, doc_token: SPEC_DOC })

// 2. 拆解任务
const tasks = await sessions_spawn({ mode: "run", tasks: planTasks })

// 3. 增量实现
for (const task of tasks) {
  await exec({ command: task.command })
  await exec({ command: "openclaw exec npm test" })
}

// 4. Review
await exec({ command: "gh pr review --approve" })

// 5. Ship
await exec({ command: "./deploy.sh" })
```

---

## 五轴 Code Review 清单

### Correctness（正确性）
- [ ] 代码实现了它声称的功能吗？
- [ ] 边界情况是否都被处理了？
- [ ] 错误路径是否正确处理？
- [ ] 输入验证是否完整？

### Readability（可读性）
- [ ] 变量命名是否自描述？
- [ ] 函数是否简洁（单一职责）？
- [ ] 注释是否解释"为什么"而非"是什么"？
- [ ] 代码流程是否容易追踪？

### Architecture（架构）
- [ ] 变更是否符合现有系统设计？
- [ ] 是否引入不必要的复杂性？
- [ ] 模块边界是否清晰？
- [ ] 依赖关系是否合理？

### Security（安全）
- [ ] 是否验证所有用户输入？
- [ ] 密钥是否正确管理？
- [ ] 是否有 SQL/命令注入风险？
- [ ] 认证授权是否正确实现？

### Performance（性能）
- [ ] 是否有 N+1 查询问题？
- [ ] 是否有无界循环？
- [ ] 资源是否正确释放？
- [ ] 缓存策略是否合理？

---

## 安全边界三层系统

### Always do（必须执行）
- [ ] 运行测试后再提交
- [ ] Review 前不自合并
- [ ] 敏感信息不写入代码
- [ ] 保持依赖更新

### Ask first（需人工批准）
- [ ] 数据库 schema 变更
- [ ] 认证/授权逻辑变更
- [ ] 外部 API 契约变更
- [ ] 安全相关配置变更

### Never do（禁止执行）
- [ ] 提交 secrets/credentials
- [ ] 绕过测试套件
- [ ] 直接修改主分支
- [ ] 在代码中硬编码密码

---

## 100 行变更原则

| 变更规模 | 行数 | 评估 |
|---------|------|------|
| ✅ Good | ~100 行 | 可在一次 review 中完成 |
| ✅ Acceptable | ~300 行 | 仅限单一逻辑变更 |
| ❌ Too Large | ~1000+ 行 | 需要拆分 |

**拆分策略**:

| 策略 | 方法 | 适用场景 |
|------|------|----------|
| Stack | 提交小变更，基于它开始下一个 | 顺序依赖 |
| By file group | 不同变更组对应不同 reviewer | 横切关注点 |
| Horizontal | 先创建共享代码/桩，然后是消费者 | 分层架构 |
| Vertical | 拆分为更小的全栈功能切片 | 功能工作 |

**OpenClaw 实操**:

```bash
# 检查变更行数
git diff --stat

# 拆分为小提交
git add path/to/file1.js
git commit -m "feat: add user auth module (part 1/3)"
```

---

## OpenClaw Skill 集成示例

### coding-agent Skill 增强

在 `~/.openclaw/skills/coding-agent/SKILL.md` 中增加质量门禁：

```markdown
## 开发流程（基于 Addy Osmani Agent Skills）

### 增量实现原则
- 每个任务不超过 300 行变更
- 实现后立即运行测试
- 通过后再提交

### 五轴自我检查
在提交前完成：
1. ✅ Correctness - 功能正确
2. ✅ Readability - 代码可读
3. ✅ Architecture - 架构合理
4. ✅ Security - 安全无漏洞
5. ✅ Performance - 性能达标
```

### 代码审查 Skill 示例

创建 `~/.openclaw/skills/code-review/SKILL.md`:

```markdown
# Code Review Skill

## 五轴评估

### Correctness
- [ ] 功能实现正确
- [ ] 边界情况处理
- [ ] 错误路径处理

### Readability
- [ ] 命名清晰
- [ ] 函数简洁
- [ ] 注释到位

### Architecture
- [ ] 符合设计
- [ ] 无过度设计
- [ ] 边界清晰

### Security
- [ ] 输入验证
- [ ] 密钥管理
- [ ] 无注入风险

### Performance
- [ ] 无 N+1
- [ ] 无无界循环
- [ ] 资源释放

## 使用方式
描述要 review 的 PR 或代码段
```

---

## 与 OpenClaw 原生 Skills 的互补

| OpenClaw Skill | 补充 Addy Osmani |
|----------------|------------------|
| coding-agent | 开发流程规范、质量门禁 |
| github | PR review + 五轴评估 |
| browser | 浏览器测试集成 |
| exec | 测试执行 + 部署 |
| sessions_spawn | 并行任务拆解 |

---

## 总结

Addy Osmani Agent Skills 是目前最系统的 AI 工程技能库，与 OpenClaw 高度互补：

- **质量门禁系统化**: 五轴评估替代主观判断
- **变更管理规范化**: 100 行原则确保可维护性
- **安全边界清晰化**: 三层系统防止常见错误
- **工作流结构化**: 七大命令覆盖完整开发周期

**推荐行动**:

1. 🔴 **立即采用**: 在 OpenClaw 教程中引用此框架
2. 🟠 **中期规划**: 开发 OpenClaw Code Review Skill
3. 🟡 **长期建设**: 实现 PRD/Spec 模板生成能力

---

*🦦 墨客内容生成 | 2026-04-21*
*基于 Addy Osmani agent-skills (17,553 ⭐) 深度调研*