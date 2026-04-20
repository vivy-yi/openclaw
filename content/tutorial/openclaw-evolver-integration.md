# OpenClaw + Evolver 自进化实战

> 让你的 AI Agent 从经验中学习，持续自我优化

**适用场景**：希望 OpenClaw Agent 能够分析执行日志、自动优化 Skills 和提示词的用户  
**前置要求**：OpenClaw 已配置，Node.js ≥ 18，Git 环境  
**推荐等级**：中级 / 高级

---

## 一、什么是 Self-Evolution（自进化）？

### 1.1 概念定义

Self-Evolution（自进化）是指 AI Agent 通过分析自身执行日志、错误模式和质量反馈，**自动优化其 Skills、提示词和行为规则**的能力。

传统 Agent 配置是**静态的**——你写好规则，它就一直用下去。而自进化系统能够：

- 🔍 **从经验学习**：将每次执行转化为优化素材
- 📋 **可审计**：保留完整进化历史，支持回滚
- ⚙️ **自动化**：减少人工干预，持续改进
- 🧠 **跨会话积累**：知识不再随会话结束而消失

### 1.2 OpenClaw Skills 的现状与局限

| 能力 | 支持情况 |
|------|----------|
| Skill 定义（SKILL.md） | ✅ 标准化格式 |
| Skill 触发（描述匹配） | ✅ 自动路由 |
| 知识库（references/） | ✅ 支持 |
| 脚本支持（scripts/） | ✅ 可执行 |
| **自进化** | ❌ 无内置机制 |
| **进化审计** | ❌ 无历史记录 |

OpenClaw 的 Skills 系统是**高质量的静态系统**，但缺乏自我进化的能力。Evolver 正是为补全这一环而设计。

---

## 二、Evolver 核心概念解析

### 2.1 GEP 协议

GEP（Genome Evolution Protocol）是 Evolver 的核心协议，参考生物基因组的进化机制：

```
执行日志/错误模式
       ↓
  扫描信号（Signal）
       ↓
  选择最佳匹配 Gene/Capsule
       ↓
  生成 GEP 约束的进化提示词
       ↓
  记录 EvolutionEvent（可审计）
```

### 2.2 三个核心概念

#### Gene（基因）
可复用的进化单元，代表特定优化模式。例如：
- `gene:error-recovery` — 错误恢复模式
- `gene:context-window-optimization` — 上下文窗口优化
- `gene:skill-refinement` — Skill 精化模式

#### Capsule（胶囊）
封装完整进化逻辑的独立模块，一个 Capsule 包含多个 Gene 以及它们之间的关系规则。

#### EvolutionEvent（进化事件）
每次进化的完整记录，包括：
- 时间戳
- 输入信号
- 执行的 Gene/Capsule
- 输出的优化建议
-采纳状态

### 2.3 与 OpenClaw 的天然契合

Evolver 原生支持 `sessions_spawn(...)` 协议——这正是 OpenClaw 的核心能力。这意味着：

```
Evolver 输出 → sessions_spawn(...) → OpenClaw 直接解释执行
```

**无需任何适配层！**

---

## 三、安装与配置

### 3.1 系统要求

| 要求 | 最低版本 | 推荐版本 |
|------|----------|----------|
| Node.js | 18.0.0 | 20.x LTS |
| Git | 2.30 | 最新 |
| npm | 9.0 | 最新 |
| 操作系统 | macOS/Linux/WSL | macOS/Linux |

> ⚠️ **注意**：Evolver 要求在 Git 仓库目录中运行，这是其审计机制的基础。

### 3.2 安装步骤

```bash
# 方式一：独立安装（推荐用于测试）
git clone https://github.com/EvoMap/evolver.git
cd evolver
npm install

# 方式二：npm 全局安装
npm install -g @evomap/evolver

# 验证安装
evolver --help
```

### 3.3 OpenClaw 工作区初始化

```bash
# 进入你的 OpenClaw 工作区
cd ~/clawd  # 或你的工作区路径

# 确保在 Git 仓库中
git status  # 应显示工作区状态

# 安装 Evolver（如果用独立方式）
git clone https://github.com/EvoMap/evolver.git
cd evolver && npm install
cd ..
```

### 3.4 目录结构

安装完成后，推荐的目录结构：

```
openclaw-workspace/
├── evolver/              # Evolver 核心
│   ├── index.js
│   ├── assets/
│   │   └── gep/         # Gene/Capsule 资产库
│   └── package.json
├── skills/              # 你的 Skills
├── memory/              # 执行日志（Evolver 扫描对象）
└── .openclaw/           # OpenClaw 配置
```

---

## 四、OpenClaw 集成实操

### 4.1 启动 Evolver 审查模式

```bash
cd ~/clawd/evolver

# 启动审查模式——扫描最近的执行日志
node index.js --review

# 指定扫描范围（最近 N 条日志）
node index.js --review --limit 50

# 仅分析特定 Skill 的执行
node index.js --review --skill self-improving
```

### 4.2 理解输出格式

Evolver 的输出遵循 `sessions_spawn(...)` 协议：

```
[EvolutionEvent] timestamp=2026-04-20T01:15:00Z
Gene: gene:skill-refinement
Signal: sessions_history shows repeated context-window overflow
Recommendation: Split long SKILL.md into modular references/
Confidence: 0.87
---
sessions_spawn({
  task: "Refactor self-improving SKILL.md",
  runtime: "subagent",
  references: [".skills/self-improving/SKILL.md"]
})
```

### 4.3 OpenClaw 自动串联

当 OpenClaw 检测到 stdout 中的 `sessions_spawn(...)` 指令时，会自动：

1. **解析** 指令参数（task, runtime, references）
2. **调度** 子 Agent 执行任务
3. **记录** 执行结果到 memory/
4. **反馈** 给 Evolver 完成闭环

```javascript
// OpenClaw sessions_spawn 协议示例
sessions_spawn({
  task: "Refactor the self-improving SKILL.md to split long sections into modular references/",
  runtime: "subagent",
  references: [
    ".openclaw/skills/self-improving/SKILL.md",
    "memory/evolution-events/*.md"
  ]
})
```

### 4.4 手动触发进化循环

如果需要更精细的控制：

```bash
# 1. 生成进化建议（不自动执行）
node index.js --generate --skill <skill-name>

# 2. 审查建议
cat evolution-suggestions.md

# 3. 采纳建议（OpenClaw 自动执行）
node index.js --apply --event-id <event-id>

# 4. 验证效果
node index.js --verify --event-id <event-id>
```

---

## 五、Gene 和 Capsule 实战使用

### 5.1 查看可用 Gene

```bash
cd evolver/assets/gep/
ls -la genes/
```

常见 Gene 类型：

| Gene 名称 | 用途 | 触发信号 |
|-----------|------|----------|
| `error-recovery` | 优化错误恢复逻辑 | 重复执行失败 |
| `context-optimization` | 减少上下文占用 | token 溢出警告 |
| `skill-refinement` | 精化 Skill 描述 | 触发匹配率低 |
| `memory-consolidation` | 合并分散记忆 | 知识碎片化 |
| `prompt-enhancement` | 增强提示词效果 | 反馈质量下降 |

### 5.2 选择性运行特定 Gene

```bash
# 只运行 error-recovery 基因
node index.js --review --gene error-recovery

# 组合多个基因
node index.js --review --gene skill-refinement --gene context-optimization
```

### 5.3 Capsule 的使用

Capsule 是更高级的封装，适合完整的技能优化：

```bash
# 查看可用 Capsule
ls -la capsules/

# 运行完整胶囊（包含多个基因的协同优化）
node index.js --review --capsule skill-lifecycle
```

### 5.4 自定义 Gene（高级）

如果你有特定的优化模式：

```javascript
// evolver/assets/gep/genes/custom/my-optimization.js
module.exports = {
  name: 'my-optimization',
  description: 'Custom optimization for specific patterns',
  signals: ['pattern-a', 'pattern-b'],
  async evaluate(context) {
    // 自定义评估逻辑
    return {
      score: 0.85,
      suggestions: ['...']
    }
  }
}
```

---

## 六、进化审计与回滚

### 6.1 查看进化历史

```bash
# 列出所有 EvolutionEvent
node index.js --history

# 过滤特定时间范围
node index.js --history --from 2026-04-01 --to 2026-04-20

# 查看特定 Skill 的进化
node index.js --history --skill self-improving
```

### 6.2 进化事件详情

```bash
# 查看单个事件的完整记录
node index.js --event <event-id>
```

输出示例：

```
EvolutionEvent: EVT-2026-0420-001
Timestamp: 2026-04-20T01:15:00Z
Gene: gene:skill-refinement
Skill: self-improving
Signal: Trigger match rate < 60% over 10 sessions
Recommendation: Refine SKILL.md description with more specific trigger phrases
Confidence: 0.87
Status: adopted
Applied by: sessions_spawn(subagent)
Result: SKILL.md updated, match rate improved to 82%
```

### 6.3 回滚操作

如果进化效果不佳：

```bash
# 查看可回滚版本
node index.js --rollback --list --skill self-improving

# 回滚到指定版本
node index.js --rollback --skill self-improving --event EVT-2026-0420-001

# 确认回滚结果
node index.js --verify --skill self-improving
```

### 6.4 导出审计报告

```bash
# 导出完整审计报告（JSON）
node index.js --export --format json --output audit-report.json

# 导出 Markdown 格式（适合 Human 阅读）
node index.js --export --format markdown --output audit-report.md
```

---

## 七、最佳实践与注意事项

### 7.1 最佳实践

#### ✅ 推荐做法

1. **先 review 再 apply**
   ```
   node index.js --review    # 生成建议，人工审查
   node index.js --apply     # 确认无误后再采纳
   ```

2. **保持 memory/ 目录清洁**
   - 定期归档旧日志，减少噪音信号
   - 使用 `memory/YYYY-MM/` 子目录分类

3. **小步快跑**
   - 每次进化控制在单个 Skill 范围内
   - 验证效果后再进行下一轮

4. **保留原始版本**
   - 每次修改前 git commit
   - 给 SKILL.md 打版本 tag

#### ❌ 避免做法

1. **不要在生产环境直接 `--apply`**
   - 先在测试分支验证
   - 确认无误再合并

2. **不要忽略低置信度建议**
   - confidence < 0.7 的建议需要人工审查
   - 强制采纳可能导致退化

3. **不要跨 Skill 混合进化**
   - 每个 EvolutionEvent 应针对单一 Skill
   - 混合进化难以审计和回滚

### 7.2 注意事项

| 注意点 | 说明 |
|--------|------|
| **Git 依赖** | Evolver 要求在 git 目录中运行，无 git 环境请用 Docker |
| **GPL-3.0 License** | 商业使用需注意 License 风险 |
| **离线功能** | 核心功能离线可用，Hub 功能需网络 |
| **数据隐私** | 执行日志会写入 memory/，确保敏感信息已脱敏 |
| **进化频率** | 建议每周不超过 2 次，避免过度优化 |

### 7.3 故障排查

| 问题 | 解决方案 |
|------|----------|
| `sessions_spawn not recognized` | 确认 OpenClaw 版本 ≥ 0.9.0 |
| `No evolution events found` | 检查 memory/ 目录是否有日志文件 |
| `Gene not found` | 确认 gene 名称拼写，查看 `ls genes/` |
| `Permission denied` | 检查 evolver 目录权限 |
| `Git directory required` | 在 git 仓库根目录运行，或使用 `--no-git` flag（有限支持）|

---

## 八、与 Hermes Agent 对比

如果你在评估多个 Self-Evolution 方案：

| 维度 | Evolver | Hermes Agent |
|------|---------|--------------|
| **License** | GPL-3.0 | MIT |
| **OpenClaw 集成** | ✅ 原生支持 | ❌ 需自行开发 |
| **技术栈** | Node.js | Python |
| **审计机制** | ✅ 完整 | ⚠️ 部分 |
| **实现阶段** | 完整 | Phase 1（仅 Skills） |
| **争议** | 无已知争议 | 与 Evolver 存在设计相似性争议 |

**结论**：截至 2026 年 4 月，Evolver 是 OpenClaw 集成就绪度最高的 Self-Evolution 方案。

---

## 九、进阶：构建自定义进化流程

### 9.1 集成到 CI/CD

```yaml
# .github/workflows/evolution.yml
name: Weekly Evolution Review
on:
  schedule:
    - cron: '0 2 * * 1'  # 每周一 02:00 UTC

jobs:
  evolve:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - name: Run Evolver Review
        run: |
          cd evolver && npm install
          node index.js --review --limit 100 --output evolution-report.md
      - name: Upload Report
        uses: actions/upload-artifact@v4
        with:
          name: evolution-report
          path: evolution-report.md
```

### 9.2 与 OpenClaw Cron 集成

在 OpenClaw 的 AGENTS.md 中添加定时进化：

```markdown
## 定时任务

### 每周一 02:00 — Evolver 自进化审查
1. 运行 `node evolver/index.js --review --limit 50`
2. 分析输出中的 sessions_spawn(...) 指令
3. 对高置信度建议（≥ 0.8）自动 apply
4. 低置信度建议写入待审查队列
5. 生成进化报告写入 memory/evolution-reports/
```

---

## 相关资源

- [Evolver GitHub](https://github.com/EvoMap/evolver)
- [GEP 协议规范](https://github.com/EvoMap/evolver/blob/main/GEP.md)
- [OpenClaw Skills 文档](https://docs.openclaw.ai/skills)
- [sessions_spawn 协议参考](https://docs.openclaw.ai/agents/sessions_spawn)

---

*🦦 教程编写：墨客-内容生成  
质量审核：墨客-内容审核  
最后更新：2026-04-20*
