---
title: OpenClaw + Addy Osmani Agent Skills 实战指南
description: 将 Addy Osmani 的七大命令体系和 20 个 Skills 融入 OpenClaw 工作流
---

# OpenClaw + Addy Osmani Agent Skills 实战指南

🦦 **作者按**: 本文将 Google 工程师 Addy Osmani 提出的 AI Agent 命令体系与 OpenClaw 工作流深度融合，提供一套可直接落地的实战方法论。

---

## 1. 概述

### 1.1 谁是 Addy Osmani？

Addy Osmani 是 Google Chrome 团队的工程总监（Engineering Manager），专注于 Web 平台和开发者工具。他是以下作品的作者：

- **"Learning JavaScript Design Patterns"** — 前端经典
- **"Essential-js-design-patterns"** — GitHub 47k+ stars
- **"Web Performance Engineering"** — 深度性能优化实践

他在 Google 内部推动的 **Agent-Specified Development（Agent 指定式开发）** 正在重塑软件开发范式。核心思想：**让 AI Agent 在明确的命令体系下工作，而不是自由发挥。**

### 1.2 为什么要在 OpenClaw 中实践？

OpenClaw 的核心能力：

| 能力 | 对应 Osmani 体系 |
|------|-----------------|
| `exec` 工具 | 构建/测试命令执行 |
| `browser` 工具 | 可视化验证与截图 |
| `sessions_spawn` 子 Agent | 并行任务分发 |
| 内存文件系统 | 上下文持久化 |
| Skill 系统 | 技能即插即用 |

OpenClaw + Osmani = **有纪律的 Agent 工作流**：既有强大的工具调用能力，又在安全的命令边界内运作。

---

## 2. 七大命令体系

Addy Osmani 提出的 Agent 命令体系以 **斜杠命令** 为入口，每个命令有明确的子命令层级。

### 2.1 `/spec` — 规格说明书

**目的**: 在动手写代码之前，先把"做什么"定义清楚。

**子命令结构**:

```
/spec
  /spec create        # 创建新规格文档
  /spec review        # 评审现有规格
  /spec update        # 更新已存在的规格
  /spec diff          # 对比两个版本的差异
```

**OpenClaw 实战示例**:

```markdown
# 用户请求：写一个 URL 短链接服务

/spec create <<EOF
## 短链接服务规格

### 功能需求
- 输入：一个长 URL
- 输出：一个 6 位短码
- 短码格式：字母+数字，不含易混淆字符（0/O, 1/l/I）
- 存储：Redis，TTL 7 天
- 已有短码冲突处理：换码重试，最多 3 次

### API 设计
- POST /shorten   body: {url: string} → {shortCode: string, expiresAt: timestamp}
- GET  /{code}     → 302 redirect to original URL

### 约束
- QPS 上限：1000/s
- 可用性目标：99.9%
- 不记录用户隐私数据

### OpenClaw 工具选型
- exec: redis-cli 操作
- browser: 验证跳转行为（截图确认 302）
EOF
```

### 2.2 `/plan` — 任务规划

**目的**: 把规格拆解成可执行的步骤。

**子命令结构**:

```
/plan
  /plan create       # 从 spec 生成执行计划
  /plan estimate     # 评估时间和复杂度
  /plan reorder      # 调整任务顺序（依赖关系）
  /plan track        # 追踪进度
```

**OpenClaw 实战示例**:

```markdown
# 从 spec 生成 plan

/spec create 时自动触发 /plan create：

## 执行计划

### Phase 1: 基础设施 (Day 1)
- [ ] 初始化 Node.js 项目
- [ ] 安装依赖：express, ioredis, nanoid
- [ ] 配置 ESLint + Prettier
- [ ] 搭建 Redis 连接层

### Phase 2: 核心功能 (Day 2)
- [ ] 实现短码生成算法（nanoid + 去混淆）
- [ ] POST /shorten 接口
- [ ] GET /{code} 跳转接口
- [ ] 冲突检测逻辑

### Phase 3: 测试 (Day 3)
- [ ] 单元测试（覆盖率 > 80%）
- [ ] Redis mock 测试
- [ ] API 集成测试

### Phase 4: 部署 (Day 4)
- [ ] Docker 镜像构建
- [ ] Docker Compose 本地验证
- [ ] 上传生产环境

### 依赖图
Phase 1 → Phase 2 → Phase 3 → Phase 4
（线性依赖，不可并行）
```

### 2.3 `/build` — 代码构建

**目的**: 按计划执行代码编写。

**子命令结构**:

```
/build
  /build file        # 构建单个文件
  /build module      # 构建整个模块
  /build scaffold    # 搭建脚手架
  /build integrate   # 集成多个模块
```

**OpenClaw 工具映射**:

| 子命令 | OpenClaw 工具 | 示例 |
|--------|--------------|------|
| `/build file` | `write` / `edit` | 写入单个源文件 |
| `/build module` | `exec` (多文件) | 用 `exec` 批量生成 |
| `/build scaffold` | `exec` (脚手架工具) | `npx create-next-app` |
| `/build integrate` | `sessions_spawn` | 多个子 Agent 并行构建 |

**实战示例 — 使用 OpenClaw 构建一个 Express 服务**:

```
# 使用 exec + write 构建单个文件
/openclaw-exec
$ mkdir -p src/routes src/services src/utils

/openclaw-write src/services/shorten.js
const { customAlphabet } = require('nanoid');
const CONFUSING = '0O1lI';
const alphabet = '0123456789ABCDEFGHJKMNPQRSTUVWXYZabcdefghjkmnpqrstuvwxyz';
const nanoid = customAlphabet(alphabet.replace(new RegExp(`[${CONFUSING}]`, 'g'), ''), 6);

function generateShortCode() {
  return nanoid();
}

module.exports = { generateShortCode };
```

### 2.4 `/test` — 测试验证

**目的**: 在构建过程中持续验证，不等到最后才测试。

**子命令结构**:

```
/test
  /test unit         # 单元测试
  /test integration  # 集成测试
  /test e2e          # 端到端测试（浏览器）
  /test benchmark    # 性能基准测试
  /test snapshot     # 快照测试
```

**OpenClaw 工具映射**:

| 子命令 | OpenClaw 工具 | 示例 |
|--------|--------------|------|
| `/test unit` | `exec` (jest/mocha) | `exec` 运行 `npm test` |
| `/test e2e` | `browser` | 浏览器自动化验证 |
| `/test benchmark` | `exec` (ab/wrk) | 压力测试短链接服务 |

**实战示例 — 浏览器 E2E 测试**:

```
# 启动服务后，用 browser 验证短链接跳转
/openclaw-browser
action: open
url: http://localhost:3000/shorten
---
# 截图验证服务正常运行
/openclaw-browser
action: screenshot
---
# 填写表单并提交
/openclaw-browser
action: act
request: {"kind": "fill", "ref": "input[name=url]", "text": "https://example.com/long/path"}
---
/openclaw-browser
action: act
request: {"kind": "click", "ref": "button[type=submit]"}
```

### 2.5 `/review` — 代码评审

**目的**: 多维度审视代码质量，不是最后一步，而是开发过程中的 checkpoint。

**子命令结构**:

```
/review
  /review correctness  # 正确性检查
  /review readability  # 可读性检查
  /review architecture # 架构检查
  /review security     # 安全检查
  /review performance  # 性能检查
  /review full         # 全面评审
```

> ⚠️ **五轴评审详见第 4 节**

### 2.6 `/ship` — 发布上线

**目的**: 安全的生产环境部署。

**子命令结构**:

```
/ship
  /ship dry-run        # 演练模式（不上传）
  /ship staging        # 部署到预发布环境
  /ship production     # 部署到生产环境
  /ship rollback       # 回滚
```

**OpenClaw 工具映射**:

| 子命令 | OpenClaw 工具 | 示例 |
|--------|--------------|------|
| `/ship dry-run` | `exec` (本地 build) | `docker build --dry-run` |
| `/ship staging` | `exec` (scp/ssh) | 上传镜像到预发服务器 |
| `/ship production` | `exec` (滚动更新) | `kubectl rollout restart` |
| `/ship rollback` | `exec` (版本回退) | `kubectl rollout undo` |

**实战示例**:

```
# /ship dry-run — 本地构建验证
/openclaw-exec
$ docker build -t shortlink:1.0.0 .
$ docker run --rm shortlink:1.0.0 npm run test
# 如果测试全部通过，进入 staging

# /ship staging — 推送到预发
/openclaw-exec
$ docker tag shortlink:1.0.0 registry.example.com/shortlink:staging
$ docker push registry.example.com/shortlink:staging
$ ssh deploy@staging.example.com "docker-compose -f shortlink-staging.yml pull && docker-compose -f shortlink-staging.yml up -d"
```

---

## 3. OpenClaw 工具集映射

本节详细说明 OpenClaw 工具如何对应 Osmani 命令体系。

### 3.1 核心映射表

| OpenClaw 工具 | 映射到 | 说明 |
|--------------|--------|------|
| `exec` | `/build file` `/test unit` `/ship` | 命令行执行主力工具 |
| `browser` | `/test e2e` `/review` | 可视化验证和截图 |
| `sessions_spawn` | `/build integrate` | 并行子 Agent 分发 |
| `read` | `/spec review` `/review` | 读取源文件进行评审 |
| `write` | `/build file` | 写入/覆盖源文件 |
| `edit` | `/build file` (增量修改) | 精确编辑文件某处 |
| `message` | 状态通知 | Slack/Telegram 结果推送 |

### 3.2 `sessions_spawn` — 并行构建实战

Osmani 强调 `/build integrate` 需要并行执行相关模块：

```
# 场景：短链接服务需要同时开发 3 个模块
# 使用 sessions_spawn 并行启动 3 个子 Agent

/subagent-1 (shorten.js)
/subagent-2 (redirect.js)
/subagent-3 (metrics.js)

# 主 Agent 协调：收集结果 → 集成 → 统一测试
```

### 3.3 `exec` 的安全边界

`exec` 是最强大的工具，也是最需要边界的工具。结合第 5 节"安全边界三层系统"：

```markdown
# OpenClaw exec 白名单示例（在 TOOLS.md 中配置）

### 可安全执行的命令（Always Do）
- git status, git diff, git log
- npm test, npm run lint
- docker build（不带 --rm-force）
- cat, head, tail, grep

### 需要确认的命令（Ask First）
- git push, git commit --amend
- docker rm, docker rmi
- rm（用 trash 代替）
- ssh, scp

### 禁止执行的命令（Never Do）
- rm -rf /
- curl | sh（不明脚本）
- kubectl delete --all
- 直接操作生产数据库
```

---

## 4. 五轴 Code Review 清单

在 `/review full` 时，需要从 5 个维度全面检查代码。每个维度有具体的 checklist。

### 4.1 轴一：正确性（Correctness）

**核心问题**: 代码是否做对了它应该做的事？

```
✓ 边界条件是否覆盖？（空值、零值、最大值）
✓ 错误处理是否完整？（try-catch、fallback）
✓ 竞态条件是否存在？（并发访问共享资源）
✓ 第三方依赖版本是否锁定？（package-lock.json）
✓ 类型定义是否准确？（TypeScript / JSDoc）

检查项：
[ ] 单元测试覆盖率 ≥ 80%
[ ] 常见注入攻击防护（SQL 注入、XSS）
[ ] 输入校验（白名单 > 黑名单）
```

### 4.2 轴二：可读性（Readability）

**核心问题**: 其他人能读懂这段代码吗？6 个月后的你能读懂吗？

```
✓ 函数长度 ≤ 50 行（超过则拆分）
✓ 变量命名语义化（有意义的名字 > 缩写）
✓ 注释解释"为什么"而非"是什么"（代码本身已说明是什么）
✓ 嵌套层级 ≤ 3 层（超过则提取函数）
✓ 无魔法数字（用常量替代）

检查项：
[ ] 每个函数有 JSDoc / TSDoc 说明
[ ] 复杂的业务逻辑有行内注释
[ ] 目录结构符合项目约定
```

### 4.3 轴三：架构（Architecture）

**核心问题**: 代码结构是否能支撑长期演进？

```
✓ 依赖方向正确（高层模块不依赖低层模块）
✓ 模块边界清晰（避免循环依赖）
✓ 配置外部化（不用硬编码）
✓ 可测试性（依赖可注入）

检查项：
[ ] 依赖图无循环引用
[ ] 接口抽象稳定（实现可替换）
[ ] 无上帝类（单一职责原则）
```

### 4.4 轴四：安全性（Security）

**核心问题**: 代码是否引入了安全漏洞？

```
✓ 无硬编码密钥（使用环境变量或密钥管理服务）
✓ 用户输入经过校验和清理
✓ 权限检查不过滤客户端（服务端二次验证）
✓ 敏感日志脱敏（密码、Token 不出现在日志中）
✓ 依赖无已知 CVE（定期 npm audit）

检查项：
[ ] secrets 不在代码库中（git-secrets / gitleaks）
[ ] CORS 配置严格（不 setAccessControlAllowOrigin: *）
[ ] CSRF Token 机制
[ ] Rate Limiting（防刷）
```

### 4.5 轴五：性能（Performance）

**核心问题**: 代码在规模和压力下的表现？

```
✓ 无 N+1 查询问题
✓ 关键路径无不必要的同步阻塞
✓ 缓存策略合理（热点数据缓存）
✓ 资源释放及时（连接池、文件句柄）
✓ 日志级别合理（ERROR 以下不高频打印）

检查项：
[ ] 数据库查询有索引支持
[ ] 冷路径不阻塞热路径
[ ] 内存泄漏检测通过
[ ] 压测 QPS 达标
```

### 4.6 五轴评分卡

```markdown
| 维度 | 权重 | 评分 1-5 | 问题数 | 备注 |
|------|------|---------|-------|------|
| 正确性 | 30% |   4     | 2     | 边界条件缺失 |
| 可读性 | 20% |   3     | 5     | 函数过长 |
| 架构   | 20% |   4     | 1     | 循环依赖已修复 |
| 安全性 | 20% |   5     | 0     | PASS |
| 性能   | 10% |   4     | 2     | N+1 已优化 |
| **加权总分** | | **4.0** | | **待改进** |
```

---

## 5. 安全边界三层系统

Osmani 的安全边界体系将操作分为三层，是 AI Agent 编程的黄金法则。

### 5.1 三层定义

```
┌─────────────────────────────────────────┐
│  🔵 ALWAYS DO（始终可执行）              │
│  直接执行，无需确认                      │
├─────────────────────────────────────────┤
│  🟡 ASK FIRST（询问后执行）              │
│  说明意图，等待人类确认后执行            │
├─────────────────────────────────────────┤
│  🔴 NEVER DO（永不执行）                │
│  无论什么理由，都不应该执行              │
└─────────────────────────────────────────┘
```

### 5.2 OpenClaw 配置示例

在 `~/.openclaw/workspaces/openclaw-assistant/AGENTS.md` 中配置：

```markdown
## 安全边界配置

### 🔵 ALWAYS DO — 可直接执行
- 读取文件（read）
- 搜索网络（web_search, tavily_search）
- 写日志到 memory/
- 运行测试（npm test）
- git status / git diff（只读操作）

### 🟡 ASK FIRST — 需要用户确认
- 任何写入/删除操作（write, edit, exec rm）
- git push / git commit
- 向外部发送消息
- 生产环境操作
- 涉及密钥或凭证的操作

### 🔴 NEVER DO — 禁止执行
- rm -rf /
- 直接操作生产数据库
- curl | sh（执行不明脚本）
- 向未知地址传输文件
- 尝试绕过安全边界
```

### 5.3 典型场景决策

| 场景 | 分类 | 操作 |
|------|------|------|
| "帮我把测试跑一下" | 🔵 ALWAYS DO | 直接 `exec` |
| "帮我把这个文件改一下" | 🟡 ASK FIRST | 先说明改什么，等确认 |
| "帮我把秘钥加到代码里" | 🔴 NEVER DO | 拒绝，说明原因 |
| "帮我把这个文件夹删了" | 🟡 ASK FIRST | 先确认路径和意图 |
| "帮我查一下线上数据库" | 🔴 NEVER DO | 拒绝，直接操作生产数据 |

---

## 6. 100 行变更原则

### 6.1 原则定义

> **每次代码变更不超过 100 行（新增 + 修改 + 删除总计）。**

### 6.2 为什么是 100 行？

- **可审查性**: 人类评审者能在 10 分钟内完成评审
- **可回滚性**: 小变更风险低，回滚成本低
- **可测试性**: 变更边界清晰，测试用例容易编写
- **可合并性**: 冲突解决简单，降低集成难度

### 6.3 三档变更规模

| 规模 | 行数范围 | 适用场景 |
|------|---------|---------|
| **小变更** | 1-30 行 | Bug 修复、配置修改、文档更新 |
| **中变更** | 31-100 行 | 新功能模块、重构、测试补充 |
| **大变更** | > 100 行 | 需要拆分，必须分多次提交 |

### 6.4 OpenClaw 实战：如何控制变更在 100 行以内

**场景**: 需要实现一个新的 API 端点

```
# ❌ 错误做法：一次性写 300 行
直接创建 src/routes/api.js（300行）
→ 难以 review，错误率高

# ✅ 正确做法：拆分为 4 个小提交
commit 1: 添加路由骨架 + 框架代码（25行）→ /review correctness ✓
commit 2: 实现核心业务逻辑（50行）→ /test unit ✓
commit 3: 添加错误处理和校验（20行）→ /review security ✓
commit 4: 补充集成测试（15行）→ /test integration ✓
总计: 110行，分4次提交
```

**在 OpenClaw 中追踪变更量**:

```bash
# 使用 git diff --stat 快速查看变更规模
/openclaw-exec
$ git diff --stat HEAD~1
```

### 6.5 超过 100 行怎么办？

**立即拆分策略**：

```
问题：你的变更达到了 150 行
解决方案：
1. 识别可独立存在的子功能
2. 将子功能拆分为独立的 PR / commit
3. 保持每个 PR 的原子性（单一职责）
4. 使用 sessions_spawn 并行处理可独立的部分
```

---

## 7. OpenClaw Skill 集成示例

OpenClaw 的 Skill 系统是 Osmani 20 Skills 理念的完美载体。以下是实战集成示例。

### 7.1 Skill 概览与 Osmani 对应

| OpenClaw 内置 Skill | Osmani 命令映射 | 用途 |
|--------------------|----------------|------|
| `coding-agent` | `/build` `/test` | 代码生成与测试 |
| `github` | `/ship` | 代码托管与 CI/CD |
| `summarize` | `/review` | 代码摘要与评审 |
| `taskflow` | `/plan` | 任务规划与追踪 |

### 7.2 示例一：用 `coding-agent` Skill 实现 `/build`

**Step 1**: 加载 skill

```
用户: "帮我写一个计算文件 MD5 的工具函数"
→ OpenClaw 加载 coding-agent skill
```

**Step 2**: 按照 `/spec create` → `/build file` → `/test unit` 的顺序执行

```markdown
# coding-agent 会自动执行以下流程：

1. 理解需求：计算文件 MD5
2. /spec create：
   - 输入：文件路径
   - 输出：MD5 字符串（32位十六进制）
   - 依赖：crypto 模块（Node.js 内置）
   - 边界：文件不存在时抛出明确错误

3. /build file（生成 src/utils/md5.js）：
```javascript
const crypto = require('crypto');
const fs = require('fs');

function computeMd5(filePath) {
  if (!fs.existsSync(filePath)) {
    throw new Error(`File not found: ${filePath}`);
  }
  const buffer = fs.readFileSync(filePath);
  return crypto.createHash('md5').update(buffer).digest('hex');
}

module.exports = { computeMd5 };
```

4. /test unit（生成 tests/utils/md5.test.js）：
   - 测试正常文件
   - 测试文件不存在的情况
   - 测试大文件（>1GB）性能
```

### 7.3 示例二：用 `github` Skill 实现 `/ship`

```markdown
# 用户: "把这个 Node 项目部署到生产环境"
→ 加载 github skill + 应用安全边界

## 安全检查链
1. 🔵 exec: git status（检查工作区干净）
2. 🟡 ASK: "确认推送到 main 分支？"
3. 🟡 ASK: "确认执行生产部署？"
4. 🔵 exec: git push origin main（触发 CI）
5. 🔵 exec: CI 通过后自动部署（如果配置了 GitHub Actions）
```

### 7.4 示例三：用 `summarize` Skill 实现 `/review`

```markdown
# 用户: "帮我 review 这段代码"
src/services/payment.js (120行)
→ 加载 summarize skill

## 自动执行五轴评审

summarize 会对代码进行结构化分析：

### 轴一：正确性
- ✓ 支付金额计算：浮点精度问题（应用 Math.round 修正）
- ⚠️ 超时处理：未区分网络超时和业务超时

### 轴二：可读性
- ✓ 命名语义清晰
- ⚠️ processPayment 函数 45 行，建议拆分

### 轴三：架构
- ⚠️ 直接依赖 Stripe SDK，建议抽象 PaymentGateway 接口

### 轴四：安全性
- ✓ 敏感数据不记录日志
- ⚠️ webhook 回调未验证签名

### 轴五：性能
- ✓ 数据库查询有索引
- ⚠️ 同步调用外部 API，建议改为队列异步处理

## 综合评分：3.2/5 → 需要改进后再合并
```

### 7.5 示例四：用 `taskflow` Skill 实现 `/plan`

```markdown
# 用户: "帮我规划一个用户认证模块的开发"
→ 加载 taskflow skill

## 自动生成执行计划

### /plan create（从意图生成计划）

Phase 1: 注册功能
- [ ] 用户注册 API（email + password）
- [ ] 密码加密存储（bcrypt）
- [ ] 注册邮件发送
- 依赖：无
- 预计：2小时

Phase 2: 登录功能
- [ ] 登录 API（JWT）
- [ ] Token 刷新机制
- [ ] 登录日志记录
- 依赖：Phase 1 完成
- 预计：3小时

Phase 3: 第三方登录
- [ ] Google OAuth 集成
- [ ] GitHub OAuth 集成
- [ ] 账号绑定逻辑
- 依赖：Phase 2 完成
- 预计：4小时

### /plan track
taskflow 会创建任务卡片并在每个 Phase 完成时追踪状态。
```

### 7.6 自定义 Skill：Osmani 七命令封装

如果你想把 Osmani 的七命令体系封装成 OpenClaw Skill：

```markdown
# 目录结构
~/.openclaw/skills/addy-osmani/
├── SKILL.md              # Skill 定义文件
├── commands/
│   ├── spec.js           # /spec 命令实现
│   ├── plan.js           # /plan 命令实现
│   ├── build.js          # /build 命令实现
│   ├── test.js           # /test 命令实现
│   ├── review.js         # /review 命令实现
│   └── ship.js           # /ship 命令实现
└── prompts/
    ├── spec-create.md    # /spec create 提示词
    ├── plan-create.md    # /plan create 提示词
    └── review-full.md    # /review full 提示词
```

```yaml
# SKILL.md 内容
---
name: addy-osmani
description: Addy Osmani 的七大 Agent 命令体系 for OpenClaw
commands:
  - /spec      # 创建和管理规格文档
  - /plan      # 任务规划与追踪
  - /build     # 代码构建
  - /test      # 测试验证
  - /review    # 五轴代码评审
  - /ship      # 部署发布
triggers:
  - "帮我写规格"
  - "规划一下"
  - "review 这段代码"
  - "部署到生产"
---
```

---

## 附录：快速命令卡

```
╔══════════════════════════════════════════════╗
║  OpenClaw + Addy Osmani 快捷命令参考         ║
╠══════════════════════════════════════════════╣
║ /spec create     → 定义规格                  ║
║ /spec review     → 评审规格                  ║
║ /plan create     → 生成执行计划              ║
║ /build file      → 写入源文件                ║
║ /build scaffold  → 搭建项目脚手架            ║
║ /test unit       → 运行单元测试              ║
║ /test e2e        → 浏览器自动化测试           ║
║ /review full     → 五轴全面评审              ║
║ /ship dry-run    → 本地演练                  ║
║ /ship production → 生产部署                 ║
╠══════════════════════════════════════════════╣
║ 🔵 ALWAYS DO    → 直接执行                  ║
║ 🟡 ASK FIRST     → 确认后执行                ║
║ 🔴 NEVER DO      → 禁止执行                  ║
╠══════════════════════════════════════════════╣
║ 100 行原则        → 单次变更 ≤ 100 行        ║
╚══════════════════════════════════════════════╝
```

---

🦦 **结语**: Addy Osmani 的命令体系让 AI Agent 从"自由发挥"走向"有纪律的协作"。结合 OpenClaw 强大的工具集（exec、browser、sessions_spawn），你可以在保持安全边界的同时，释放 AI 的最大生产力。关键是：**遵守命令体系，坚持五轴评审，严守安全边界，控制变更规模。**

> 📚 参考资料：
> - Addy Osmani GitHub: https://github.com/addyosmani
> - OpenClaw 文档: https://docs.openclaw.ai
> - Osmani Agent Commands: https://addyosmani.com/blog/ai-agent-commands/
