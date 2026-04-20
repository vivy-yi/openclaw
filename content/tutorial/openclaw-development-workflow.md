---
title: "OpenClaw 开发流程规范 — 引入 Addy Osmani 质量门禁体系"
description: "将 Addy Osmani 的七大命令体系和质量门禁设计融入 OpenClaw，提供系统性的 AI 驱动开发流程指南。"
tags: [OpenClaw, 开发流程, 质量门禁, Agent Skills, 最佳实践]
难度: intermediate
预计阅读时间: 15分钟
review_status: approved
review_date: 2026-04-20
---

## 1. 概述：为什么 AI 编码需要开发流程

当你第一次用 OpenClaw（`exec`）运行一段代码，可能感觉很爽——它直接帮你执行，快速得到结果。但当任务变得复杂：多人协作、长期维护、安全敏感的系统，你会很快遇到这些问题：

- 代码功能对了，但边界情况一堆 Bug
- AI 生成代码太快，一口气改了几千行，没人能 review
- 安全漏洞悄然引入，直到上线后才暴露
- 三个月后再看自己写的代码，完全不认识

**根本原因：AI 生成代码的质量上限，由开发流程的下限决定。**

Addy Osmani（Google Chrome 团队工程师）的 Agent Skills 框架给出了答案：不是限制 AI 的能力，而是建立一套**质量门禁体系**，让 AI 在每个关键节点接受检查。这套体系最初为 Claude Code、Cursor、Gemini CLI 等平台设计，但其核心理念完全适用于 OpenClaw。

本章教程将教你如何在 OpenClaw 中落地这套体系。

---

## 2. 七大命令体系介绍

Addy Osmani 将开发流程拆解为七个有序的命令阶段，每个阶段都有明确的触发条件和质量标准：

```
DEFINE ──▶ PLAN ──▶ BUILD ──▶ VERIFY ──▶ REVIEW ──▶ SHIP
(spec)    (plan)   (build)   (test)    (review)   (ship)
                         ↕
                   /code-simplify
```

### 2.1 整体流程一览

| 阶段 | 命令 | 核心问题 | 产出 |
|------|------|----------|------|
| **DEFINE** | `/spec` | 我们要解决什么？ | PRD / Spec 文档 |
| **PLAN** | `/plan` | 我们如何拆解任务？ | 任务列表 + 验收标准 |
| **BUILD** | `/build` | 代码怎么写？ | 可运行代码 |
| **VERIFY** | `/test` | 代码对不对？ | 测试结果 |
| **REVIEW** | `/review` | 代码够好吗？ | Quality Gate 报告 |
| **SHIP** | `/ship` | 怎么安全上线？ | 部署方案 |
| **辅助** | `/code-simplify` | 代码能更清晰吗？ | 简化后代码 |

### 2.2 各阶段详解

#### DEFINE — 明确做什么

**触发时机**：需求模糊、启动新项目、方向不明确时。

**核心原则**：Spec before code。在动手写代码之前，先把目标定义清楚。

```
# 在 OpenClaw 中启动 DEFINE 阶段
openclaw sessions_spawn --skill spec-driven-development
```

**OpenClaw 集成**：

```markdown
## DEFINE 阶段 OpenClaw 操作示例

在 OpenClaw 中，你可以用 `exec` 创建本地 spec 文件：

/* 用 exec 创建 SPEC.md */
openclaw exec -- "cat > SPEC.md << 'EOF'
# 项目名称：用户认证服务

## 1. Objective（目标）
提供 JWT-based 用户认证，支持注册/登录/登出。

## 2. Commands（核心命令）
- POST /auth/register — 用户注册
- POST /auth/login — 用户登录
- POST /auth/logout — 登出

## 3. Project Structure（项目结构）
src/
  auth/
    handlers.go
    middleware.go
    models.go
  main.go

## 4. Code Style（代码风格）
- 遵循 Go 标准库风格
- 错误处理：wrap + context
- 命名：SmallCASE

## 5. Testing Strategy（测试策略）
- 单元测试覆盖率 > 80%
- 集成测试：模拟数据库

## 6. Boundaries（边界）
- 不处理 OAuth 第三方登录
- 不存储明文密码
EOF"
```

#### PLAN — 拆解任务

**触发时机**：已有明确 spec，需要拆解为可执行任务时。

**核心原则**：Small, atomic tasks。每个任务应该小到可以在一个 session 中完成验证。

**OpenClaw 集成**：

```markdown
## PLAN 阶段 OpenClaw 操作示例

使用 `exec` 结合任务分解：

openclaw exec -- "
# 拆解为原子任务
cat > TASKS.md << 'EOF'
## 任务列表

### Task 1: 数据模型设计
- [ ] 定义 User struct
- [ ] 定义 AuthRequest/Response
- [ ] 验收：struct 字段完整，含 JSON tag

### Task 2: 注册接口
- [ ] 实现 /auth/register handler
- [ ] 密码 bcrypt 哈希
- [ ] 验收：重复注册返回 409

### Task 3: 登录接口
- [ ] 实现 /auth/login handler
- [ ] 生成 JWT token
- [ ] 验收：正确密码返回 token，错误返回 401

### Task 4: 中间件
- [ ] 实现 JWT 验证中间件
- [ ] 验收：无效 token 返回 401
EOF"
```

#### BUILD — 编写代码

**触发时机**：有具体任务要实现、任何涉及多文件变更时。

**核心原则**：One slice at a time。先实现最小功能切片，再逐步扩展。

**OpenClaw 集成**：

```markdown
## BUILD 阶段 OpenClaw 操作示例

# 使用 exec 执行增量实现
openclaw exec -- "
# Task 1: 最小功能切片 — 只实现注册
cat > src/auth/models.go << 'EOF'
package auth

import (
    \"time\"
)

type User struct {
    ID        string    \`json:\"id\"\`
    Email     string    \`json:\"email\"\`
    Password  string    \`json:\"-\"\` // never expose
    CreatedAt time.Time \`json:\"created_at\"\`
}
EOF

# Task 2: handlers
cat > src/auth/handlers.go << 'EOF'
package auth

import (
    \"net/http\"
    \"time\"

    \"github.com/golang-jwt/jwt/v5\"
    \"golang.org/x/crypto/bcrypt\"
)

var jwtSecret = []byte(\"change-in-production\")

func Register(w http.ResponseWriter, r *http.Request) {
    var req struct {
        Email    string \`json:\"email\"\`
        Password string \`json:\"password\"\`
    }
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, \"invalid request\", http.StatusBadRequest)
        return
    }

    hash, err := bcrypt.GenerateFromPassword([]byte(req.Password), bcrypt.DefaultCost)
    if err != nil {
        http.Error(w, \"internal error\", http.StatusInternalServerError)
        return
    }

    user := User{
        ID:        generateID(),
        Email:     req.Email,
        Password:  string(hash),
        CreatedAt: time.Now(),
    }

    // Save to DB (stub)
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(user)
}
EOF
"
```

#### VERIFY — 验证代码

**触发时机**：代码写完后、提交前、发现 Bug 时。

**核心原则**：Tests are proof。没有测试的代码等于没有完成的代码。

**OpenClaw 集成**：

```markdown
## VERIFY 阶段 OpenClaw 操作示例

# 运行测试
openclaw exec -- "go test ./src/auth/... -v -cover"

# 验证输出示例
# === RUN   TestRegister
# --- PASS: TestRegister (0.05s)
# PASS
# coverage: 85.3%

# 如果测试失败，使用 browser 检查运行时行为
openclaw browser open --url http://localhost:8080/auth/register
```

#### REVIEW — 代码审查

**触发时机**：合并前、提交前、重要功能完成后。

**核心原则**：Improve code health。Review 不是挑刺，而是质量门禁。

**OpenClaw 集成**：

```markdown
## REVIEW 阶段 — 五轴评估在 OpenClaw 中执行

在 OpenClaw 中启动 code review：

openclaw exec -- "
# 生成 diff 作为 review 输入
git diff --stat HEAD~1 > review-diff.txt
cat review-diff.txt
"

# 使用 sessions_spawn 启动专门的 reviewer agent
openclaw sessions_spawn --label code-reviewer --skill code-review-and-quality
```

> 📌 五轴评估详解见第四章。

#### SHIP — 上线部署

**触发时机**：代码通过 review、准备部署到生产环境时。

**核心原则**：Faster is safer。更快的反馈循环 = 更安全的部署。

**OpenClaw 集成**：

```markdown
## SHIP 阶段 OpenClaw 操作示例

# 1. 上线前检查清单
openclaw exec -- "
cat > DEPLOY-CHECKLIST.md << 'EOF'
## 上线前检查

### 代码层面
- [ ] 所有测试通过 (go test ./...)
- [ ] 覆盖率 > 80%
- [ ] 无 TODO/FIXME 未完成
- [ ] 无 hardcoded secrets

### 部署层面
- [ ] 特性开关已配置
- [ ] 回滚方案已准备
- [ ] 监控告警已设置
- [ ] 文档已更新
EOF
"

# 2. 原子提交
openclaw exec -- "
git add -p  # 逐块添加，保持原子性
git commit -m 'feat(auth): implement user registration'
"

# 3. 触发 CI/CD
openclaw exec -- "gh workflow run deploy.yml"
```

---

## 3. Spec-Driven Development 实操

### 3.1 什么是 Spec-Driven Development

Spec-Driven Development（SDD）不是"写个 README"，而是**在代码之前建立一份可验证的合同**。它的核心是六核心区域：

```
┌─────────────────────────────────────────┐
│           SPEC 文档六核心               │
├──────────────┬──────────────────────────┤
│ Objective    │ 我们要达到什么目标？       │
│ Commands     │ 用户/系统如何交互？        │
│ Structure    │ 代码如何组织？             │
│ Code Style   │ 什么样的代码算好代码？      │
│ Testing      │ 如何证明它工作？           │
│ Boundaries   │ 什么是我们不做的？         │
└──────────────┴──────────────────────────┘
```

### 3.2 在 OpenClaw 中执行 SDD

#### Step 1: 创建 Spec 文件

```markdown
<!-- 使用 exec 创建项目 spec -->
openclaw exec -- "cat > SPEC.md << 'SPECEOF'
# OpenClaw 插件系统 — Spec

## 1. Objective
为 OpenClaw 提供可插拔的插件架构，支持第三方扩展。

## 2. Commands
- `openclaw plugin install <name>` — 安装插件
- `openclaw plugin list` — 列出已安装插件
- `openclaw plugin unload <name>` — 卸载插件

## 3. Project Structure
openclaw/
  plugin/
    manager.go      # 插件管理器
    registry.go     # 插件注册表
    loader.go       # 动态加载器
  main.go

## 4. Code Style
- Go: 遵循 golang/go 风格
- 错误: wrap + context，从不忽略
- 命名: 表达意图，避免缩写

## 5. Testing Strategy
- 单元测试: manager_test.go, loader_test.go
- 覆盖率目标: 85%+
- Mock: 使用 interfaces 而非具体实现

## 6. Boundaries
- **Always do**: 验证插件签名、隔离插件错误
- **Ask first**: 删除用户数据、修改核心配置
- **Never do**: 在插件中执行 shell 代码、硬编码凭据
SPECEOF
"
```

#### Step 2: 假设前置确认

在写代码前，先列出你的假设。这些假设是门禁的前哨：

```markdown
## 假设前置确认清单

在开始实现前，回答以下问题：

1. [ ] 插件是 Go plugin（.so）还是 WASM？
       → 决策：Go plugin，需要 Go 1.17+
    
2. [ ] 插件错误会影响主程序吗？
       → 决策：插件运行在独立 goroutine，错误隔离
    
3. [ ] 插件如何加载？
       → 决策：启动时扫描 plugin/ 目录，延迟加载
    
4. [ ] 插件版本如何管理？
       → 决策：语义版本，API 契约不兼容则拒绝加载
```

#### Step 3: Spec 验证

Spec 写完后，用 OpenClaw 验证：

```markdown
## 验证 Spec 完整性

openclaw exec -- "
# 检查 spec 是否包含六核心
for section in 'Objective' 'Commands' 'Structure' 'Code Style' 'Testing Strategy' 'Boundaries'; do
    if grep -q \"\$section\" SPEC.md; then
        echo \"✅ \$section\"
    else
        echo \"❌ Missing: \$section\"
    fi
done
"
```

---

## 4. 五轴 Code Review 在 OpenClaw 中的应用

### 4.1 五轴评估体系

Code Review 不是"代码对不对"，而是"代码够不够好"。Addy Osmani 的五轴体系提供了一套结构化的评估框架：

| 轴 | 核心问题 | 检查项 |
|----|----------|--------|
| **Correctness** | 代码做的是它声称的事吗？ | 边界条件、错误路径、并发安全 |
| **Readability** | 另一个工程师能理解吗？ | 命名、注释、结构复杂度 |
| **Architecture** | 符合系统设计吗？ | 耦合度、依赖方向、扩展性 |
| **Security** | 有安全漏洞吗？ | 输入验证、密钥管理、权限控制 |
| **Performance** | 有性能问题吗？ | N+1 查询、无界循环、内存泄漏 |

### 4.2 在 OpenClaw 中执行五轴 Review

```markdown
## 五轴 Review 实操

### 轴 1: Correctness
openclaw exec -- "
# 检查边界条件处理
grep -n 'if err != nil' src/auth/handlers.go
grep -n 'panic' src/**/*.go
go vet ./...

# 运行所有测试
go test -race ./...
"

### 轴 2: Readability
openclaw exec -- "
# 检查注释覆盖率
count=\$(grep -c '^\s*//' src/auth/*.go)
total=\$(grep -c '^\s*func' src/auth/*.go)
echo \"Comment ratio: \$count/\$total functions documented\"

# 检查函数长度
wc -l src/auth/*.go
"

### 轴 3: Architecture
openclaw exec -- "
# 检查循环依赖
go mod graph | awk '{print \$1}' | sort | uniq > /tmp/deps.txt
# 手动检查依赖方向是否合理
"

### 轴 4: Security
openclaw exec -- "
# 检查密钥硬编码
grep -rn 'password\|secret\|api_key' --include='*.go' src/

# 运行安全检查
gosec ./...

# 检查 SQL 注入（如果有）
grep -n 'Sprintf\|fmt.Sprintf.*SELECT' src/**/*.go
"

### 轴 5: Performance
openclaw exec -- "
# 运行性能测试
go test -bench=. -benchmem src/auth/

# 检查内存分配
go build -gcflags='-m' ./... 2>&1 | grep 'leaking'
"
```

### 4.3 Review 输出模板

```markdown
## Code Review 报告

**PR/Commit**: `feat(auth): add user registration`
**Reviewer**: OpenClaw Agent
**Date**: 2026-04-20

### 五轴评估结果

| 轴 | 结果 | 说明 |
|----|------|------|
| ✅ Correctness | PASS | 错误处理完整，边界条件覆盖 |
| ⚠️ Readability | WARN | `handleAuth()` 函数超过 80 行，建议拆分 |
| ✅ Architecture | PASS | 依赖方向正确，符合分层设计 |
| ✅ Security | PASS | 密码 bcrypt 哈希，无硬编码凭据 |
| ✅ Performance | PASS | 无 N+1 查询 |

### Action Items
- [ ] 拆分 `handleAuth()` 为独立子函数
- [ ] 添加注册请求的 email 格式验证
- [ ] 更新 API 文档

### 结论
**可以合并**（待修复 Action Items）
```

---

## 5. 变更管理：100 行原则

### 5.1 为什么变更规模重要

当你用 AI 生成代码时，一个常见的陷阱是：**让 AI 一口气改完所有东西**。结果是：

- 1000 行变更，reviewer 只能"看起来差不多"
- 出问题难以定位是哪一块引入的
- 回滚风险大

Addy Osmani 的变更有理规则：

```
~100 lines changed   → Good. Reviewable in one sitting.
~300 lines changed   → Acceptable if it's a single logical change.
~1000+ lines changed → Too large. Split it.
```

### 5.2 拆分策略

| 策略 | 方法 | OpenClaw 实现 |
|------|------|--------------|
| **Stack** | 小提交链，每个基于前一个 | `git commit --fixup` |
| **By file group** | 不同文件组给不同 reviewer | `git add src/auth/ && git commit` |
| **Horizontal** | 先共享代码，再消费者 | 创建 `utils/` → 使用方 |
| **Vertical** | 完整功能切片（数据+逻辑+UI） | 端到端功能分支 |

### 5.3 OpenClaw 变更管理实操

```markdown
## 100 行原则实操

### 检查当前变更规模
openclaw exec -- "
git diff --stat HEAD

# 示例输出:
# src/auth/handlers.go | 45 ++---
# src/auth/models.go   | 23 ++--
# 2 files changed, 68 insertions(+), 34 deletions(-)
# Total: 68 lines changed ✅
"

### 原子提交模式
openclaw exec -- "
# 提交每个逻辑变更
git add src/auth/models.go
git commit -m 'auth: define User and AuthRequest models'

git add src/auth/handlers.go
git commit -m 'auth: implement /register handler with bcrypt'

git add src/auth/middleware.go
git commit -m 'auth: add JWT validation middleware'
"

### Stack 提交（基于前一个）
openclaw exec -- "
# 假设第一个提交已推送
FIRST_COMMIT=\$(git log --oneline -1)
git checkout -b feature/auth-login

# 第二个提交基于第一个
git commit --fixup \$FIRST_COMMIT
git rebase -i --autosquash HEAD~2
"
```

### 5.4 OpenClaw sessions_spawn 用于变更拆分

当你有一个大变更需要拆分时，可以用多 agent 协作：

```markdown
## 使用 sessions_spawn 拆分变更

# 启动拆分规划 agent
openclaw sessions_spawn \
  --label change-splitter \
  --skill planning-and-task-breakdown \
  --context "任务：重构 auth 模块，计划拆分为 5 个原子提交"

# 拆分 agent 会输出类似：
# Commit 1: 'auth: extract User model' (25 lines)
# Commit 2: 'auth: add password hashing utils' (18 lines)
# Commit 3: 'auth: implement Register handler' (35 lines)
# Commit 4: 'auth: implement Login handler' (40 lines)
# Commit 5: 'auth: add JWT middleware' (28 lines)
```

---

## 6. 安全边界：三层系统

### 6.1 三层边界体系

Addy Osmani 定义了一套三层安全边界，让 AI 知道什么可以做、什么要问、什么绝对不能做：

```
┌─────────────────────────────────────────────┐
│           三层安全边界                       │
├─────────────────┬───────────────────────────┤
│  ✅ Always do   │ 必须执行（安全默认）         │
│  ❓ Ask first   │ 需要人工批准（高风险操作）    │
│  🚫 Never do    │ 绝对禁止（红线）             │
└─────────────────┴───────────────────────────┘
```

### 6.2 各层级详解

#### Always do（必须执行）

```markdown
## OpenClaw 中的 Always do 规则

# 1. 测试通过后再提交
openclaw exec -- "go test ./... && git commit"

# 2. 验证输入
# 所有用户输入必须验证
if email != "" && strings.Contains(email, "@") {
    // proceed
}

# 3. 错误 wrap + context
if err != nil {
    return fmt.Errorf("auth.Register: %w", err)
}

# 4. 特性开关保护
if !features.IsEnabled("new-auth") {
    return ErrFeatureNotEnabled
}
```

#### Ask first（需人工批准）

```markdown
## OpenClaw 中的 Ask first 规则

以下操作必须人工确认后才能执行：

1. 数据库 Schema 变更
   - 添加/删除列
   - 修改索引
   - 数据迁移脚本

2. 认证/授权逻辑变更
   - 修改权限模型
   - 添加新的认证方式
   - 修改 token 过期时间

3. 外部集成变更
   - 添加新的第三方 API
   - 修改 webhook 逻辑
   - 变更 API 版本

4. 敏感配置变更
   - 修改日志级别（可能暴露敏感数据）
   - 变更加密参数
   - 调整 rate limiting 阈值

# OpenClaw 实现：使用 confirm 工具
openclaw exec -- "
echo '⚠️  即将执行数据库迁移'
echo '操作: ALTER TABLE users ADD COLUMN last_login TIMESTAMP'
read -p '确认执行? (yes/no): ' confirm
if [ \"\$confirm\" != 'yes' ]; then
    echo '已取消'
    exit 1
fi
"
```

#### Never do（绝对禁止）

```markdown
## OpenClaw 中的 Never do 规则（红线）

🚫 以下操作在 OpenClaw 中绝对禁止：

1. 提交 secrets 到代码库
   # 检查
   git diff --staged | grep -i 'password\|secret\|key\|token'
   # ✅ 正确做法：使用环境变量或 secret manager

2. 在代码中硬编码凭证
   # ❌ 错误
   apiKey := "sk-1234567890abcdef"
   # ✅ 正确
   apiKey := os.Getenv("API_KEY")

3. 绕过认证中间件
   # ❌ 绝对禁止注释掉中间件
   // router.Use(authMiddleware) // TODO: temporary

4. 在生产环境执行 destructive 操作
   # 必须有回滚方案
   # 必须有数据备份

5. 将用户输入直接插入 SQL
   # ❌ SQL 注入风险
   query := fmt.Sprintf("SELECT * FROM users WHERE email = '%s'", email)
   # ✅ 正确：参数化查询
   db.QueryRow("SELECT * FROM users WHERE email = ?", email)
```

### 6.3 在 OpenClaw Skill 中定义安全边界

你可以创建一个 OpenClaw skill 来强制这些边界：

```markdown
# ~/.openclaw/skills/openclaw-security-boundaries/SKILL.md

# OpenClaw Security Boundaries Skill

## 触发描述
当执行涉及数据库、认证、外部 API 或敏感数据的操作时激活。

## 安全规则

### Always Do
- 执行前运行 `go test ./...` 确认测试通过
- 使用 `gosec ./...` 扫描安全漏洞
- 检查 `git diff` 确认无 secrets 泄露

### Ask First
- 数据库 schema 变更
- 认证/授权逻辑修改
- 添加新的外部依赖
- 修改加密配置

### Never Do
- 硬编码 secrets 或凭据
- 直接拼接 SQL
- 绕过认证中间件
- 生产环境 destructive 操作
```

---

## 7. OpenClaw 集成示例

### 7.1 完整开发流程演示

下面是一个从 0 到上线的完整 OpenClaw 开发流程，结合 Addy Osmani 七大命令体系：

#### Phase 1: DEFINE + PLAN

```markdown
# 1. 启动新项目，明确需求
openclaw exec -- "cat > SPEC.md << 'EOF'
# 用户认证服务 — Spec

## Objective
提供安全、标准的 JWT 用户认证系统。

## Commands
- POST /auth/register — 注册
- POST /auth/login — 登录
- POST /auth/logout — 登出
- GET /auth/me — 获取当前用户

## Structure
src/auth/{handlers,middleware,models}.go

## Code Style
- Go 标准风格
- 错误 wrap + context
- 命名：表达意图

## Testing
- 覆盖率 > 85%
- 包含边界条件测试

## Boundaries
- 无第三方 OAuth
- 不存储明文密码
EOF
"

# 2. 拆解任务
openclaw exec -- "cat > TASKS.md << 'EOF'
## 任务列表

1. [ ] 定义数据模型 (models.go) — 25 行
2. [ ] 实现注册 handler (handlers.go) — 40 行
3. [ ] 实现登录 handler + JWT — 50 行
4. [ ] 编写中间件 (middleware.go) — 30 行
5. [ ] 单元测试 (覆盖率 > 85%) — 60 行
6. [ ] 集成测试 — 40 行
EOF
"
```

#### Phase 2: BUILD + VERIFY

```markdown
# 3. 增量实现 — 每个任务单独 commit
openclaw exec -- "
# Task 1: 数据模型
cat > src/auth/models.go << 'EOF'
package auth

import \"time\"

type User struct {
    ID        string    \`json:\"id\"\`
    Email     string    \`json:\"email\"\`
    Password  string    \`json:\"-\"\`
    CreatedAt time.Time \`json:\"created_at\"\`
}

type RegisterRequest struct {
    Email    string \`json:\"email\"\`
    Password string \`json:\"password\"\`
}
EOF

git add src/auth/models.go
git commit -m 'auth: define User and RegisterRequest models'
"

# 4. 运行测试验证
openclaw exec -- "go test ./src/auth/... -cover -v"
```

#### Phase 3: REVIEW

```markdown
# 5. 五轴 Review
openclaw exec -- "
echo '=== Correctness Check ==='
go test ./... -race

echo '=== Security Check ==='
gosec ./...

echo '=== Readability Check ==='
# 检查函数长度
for f in src/auth/*.go; do
    lines=\$(wc -l < \$f)
    if [ \$lines -gt 80 ]; then
        echo \"⚠️  \$f has \$lines lines (target: <80)\"
    fi
done
"
```

#### Phase 4: SHIP

```markdown
# 6. 上线前检查
openclaw exec -- "
cat > DEPLOY-CHECKLIST.md << 'EOF'
- [x] 所有测试通过
- [x] gosec 无高危漏洞
- [x] 无硬编码 secrets
- [x] 文档已更新
- [x] 特性开关已配置
- [x] 回滚方案已准备
EOF

# 7. 原子提交
git add -p
git commit -m 'feat(auth): complete user auth system'
git tag 'v1.0.0'
"
```

### 7.2 OpenClaw Skill 触发器配置

为了在 OpenClaw 中更好地集成 Addy Osmani 体系，可以配置 skill 触发器：

```markdown
# ~/.openclaw/skills/openclaw-dev-workflow/SKILL.md

# OpenClaw Dev Workflow Skill

## 触发描述
当用户提到以下关键词时激活：
- "新项目"、"启动开发"、"规划任务"
- "写代码"、"实现功能"、"添加接口"
- "运行测试"、"验证代码"、"测试通过"
- "code review"、"审查代码"、"检查质量"
- "部署"、"上线"、"发布版本"
- "安全"、"漏洞"、"认证"

## 关联 Skills
- planning-and-task-breakdown
- spec-driven-development
- test-driven-development
- code-review-and-quality
- security-and-hardening
- shipping-and-launch
- code-simplification

## 工作流引导

### 当用户说"新项目"时
→ 引导进入 DEFINE 阶段
→ 提供 SPEC.md 模板
→ 提醒"假设前置确认"

### 当用户说"实现 XX 功能"时
→ 引导进入 BUILD 阶段
→ 强调增量实现（薄切片）
→ 提醒测试先行

### 当用户说"代码写完了"时
→ 引导进入 VERIFY + REVIEW 阶段
→ 执行五轴评估
→ 检查变更规模（100 行原则）

### 当用户说"准备上线"时
→ 引导进入 SHIP 阶段
→ 提供上线检查清单
→ 提醒特性开关和回滚方案
```

---

## 8. 总结：建立你的质量门禁习惯

### 核心要点回顾

| 阶段 | 核心习惯 | OpenClaw 工具 |
|------|----------|---------------|
| DEFINE | Spec before code | `exec` 创建 SPEC.md |
| PLAN | 原子任务拆分 | `exec` + `sessions_spawn` |
| BUILD | 薄切片 + 测试先行 | `exec` + `go test` |
| VERIFY | 测试即证明 | `go test -race -cover` |
| REVIEW | 五轴评估 | `gosec` + `git diff` |
| SHIP | 检查清单 + 回滚方案 | `exec` + `git tag` |
| 辅助 | 代码简化 | `/code-simplify` |

### 下一步

1. **立即实践**：选择一个小型任务，用七大命令体系走一遍完整流程
2. **建立习惯**：在 OpenClaw workspace 中创建 `SPEC.md` 和 `TASKS.md` 模板
3. **持续改进**：根据实际经验，调整各阶段的具体检查项
4. **分享反馈**：将你的实践记录到 `~/.openclaw/shared/context/recent/`

> 💡 **记住**：AI 编码的质量上限，由你的开发流程下限决定。建立质量门禁不是为了限制 AI 的能力，而是为了让 AI 的每一次输出都经得起检验。

---

*🦦 本教程基于 Addy Osmani Agent Skills 框架编写，旨在帮助 OpenClaw 用户建立系统性的开发流程规范。*
