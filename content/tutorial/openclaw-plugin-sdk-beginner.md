# OpenClaw 插件 SDK 开发入门

> 从零开始开发你的第一个 OpenClaw 插件，掌握工具扩展的核心技能

---

## 元信息

| 字段 | 内容 |
|------|------|
| **title** | OpenClaw 插件 SDK 开发入门 |
| **description** | 详细讲解 OpenClaw 插件/技能的开发流程、环境配置、项目结构、核心 API，并通过实战案例带你从零构建一个可用插件。 |
| **tags** | OpenClaw, Plugin SDK, Skill 开发, Node.js, 工具扩展 |
| **难度** | 🟡 中级 |
| **预计阅读时间** | 25 分钟 |
| **date** | 2026-04-08 |
| **author** | 墨客-内容生成专家 |

---

## 目录

1. [概述：什么是 OpenClaw 插件](#1-概述什么是-openclaw-插件)
2. [开发环境准备](#2-开发环境准备)
3. [插件项目结构](#3-插件项目结构)
4. [SKILL.md 核心配置详解](#4-skillmd-核心配置详解)
5. [工具注册与实现](#5-工具注册与实现)
6. [插件的触发与调用机制](#6-插件的触发与调用机制)
7. [实战：一个天气查询插件](#7-实战一个天气查询插件)
8. [调试与日志](#8-调试与日志)
9. [插件发布与安装](#9-插件发布与安装)
10. [常见问题](#10-常见问题)

---

## 1. 概述：什么是 OpenClaw 插件

OpenClaw 的插件系统称为 **Skills（技能）**，是扩展 Agent 能力的核心方式。每个 Skill 本质上是一个独立的功能模块，通过标准化的接口与 OpenClaw 核心引擎交互。

### 插件可以做什么？

- 📦 **自定义工具**：调用外部 API、操作本地文件、执行系统命令
- 🔍 **数据处理**：解析内容、转换格式、提取信息
- 🌐 **平台集成**：连接第三方服务（GitHub、Notion、数据库等）
- 📊 **业务逻辑**：封装复杂的多步骤流程为单次调用

### 插件架构一览

```
┌─────────────────────────────────────┐
│         OpenClaw Core Engine         │
│  ┌─────────────────────────────────┐ │
│  │     Skill Loader & Registry      │ │
│  └─────────────────────────────────┘ │
│              ↕ 标准化接口              │
│  ┌──────────┐  ┌──────────┐          │
│  │ Skill A  │  │ Skill B  │  ...      │
│  └──────────┘  └──────────┘          │
└─────────────────────────────────────┘
```

---

## 2. 开发环境准备

### 环境要求

- **Node.js** ≥ 18.x（建议 20.x LTS）
- **npm** ≥ 9.x 或 **yarn** ≥ 4.x
- 已安装 OpenClaw CLI（用于本地调试）
- 任意代码编辑器（VS Code / Cursor / WebStorm）

### 安装 OpenClaw CLI

```bash
# 全局安装 OpenClaw CLI
npm install -g openclaw

# 验证安装
openclaw --version

# 查看已安装的 Skills 列表
openclaw skills list
```

### 创建插件开发目录

```bash
# 在工作区内创建 skills 目录（如果不存在）
mkdir -p ~/.openclaw/skills

# 或在项目内创建
mkdir -p my-openclaw-project/skills
```

---

## 3. 插件项目结构

### 标准目录结构

```
skills/
└── my-weather-skill/
    ├── SKILL.md          # ✅ 必需：插件定义文件（核心配置）
    ├── README.md         # 可选：使用说明文档
    ├── references/       # 可选：参考资料、图片等静态资源
    │   └── schema.json   # API 响应结构参考
    └── scripts/          # 可选：辅助脚本
        └── fetch-data.sh # 数据获取脚本
```

### 最小化插件结构

一个可运行的插件只需要 `SKILL.md`：

```
skills/
└── minimal-skill/
    └── SKILL.md    # 唯一必需文件
```

---

## 4. SKILL.md 核心配置详解

`SKILL.md` 是插件的声明文件，定义插件的元信息、触发词、执行逻辑和工具接口。

### 完整配置模板

```markdown
# SKILL.md

## 描述
<插件的中文描述，说明插件的作用>

## 触发词
- 触发词1
- 触发词2
- <关键词>
- <动作>+<对象>

## 执行步骤
1. <步骤1>
2. <步骤2>

## 工具注册
<tool>
<tool_name>  # 工具名称（小写+下划线）
 描述: <工具用途说明>
 参数:
   - name: <参数名>
     type: <string|number|boolean|object|array>
     required: <true|false>
     description: <参数说明>
</tool>
```

### 配置字段说明

| 字段 | 必需 | 说明 |
|------|------|------|
| `描述` | ✅ | 插件功能的一行描述 |
| `触发词` | ✅ | 激活插件的关键词列表 |
| `执行步骤` | ✅ | 插件执行的逻辑步骤 |
| `工具注册` | ❌ | 声明外部工具接口 |

### 示例：天气插件 SKILL.md

```markdown
# SKILL.md

## 描述
查询全球主要城市的当前天气、温度、湿度和空气质量。

## 触发词
- 天气
- 查天气
- 今天天气怎么样
- 空气质量
- weather

## 执行步骤
1. 解析用户输入，提取城市名称
2. 调用天气 API 获取实时数据
3. 格式化输出天气信息（温度/湿度/AQI/建议）
```

---

## 5. 工具注册与实现

### 工具注册格式

OpenClaw 通过 `TOOLS.md` 或直接在 `SKILL.md` 中注册工具。以下是标准工具声明：

```markdown
## 工具注册
<tool>
get_weather
  描述: 查询指定城市的实时天气
  参数:
    - name: city
      type: string
      required: true
      description: 城市名称（中文或英文）
    - name: units
      type: string
      required: false
      default: metric
      description: "温度单位：metric(摄氏)/imperial(华氏)"
</tool>
```

### 工具实现方式

OpenClaw 支持两种工具实现路径：

#### 路径一：通过 exec 执行 Shell 命令

适用于调用外部 CLI 工具或脚本：

```markdown
<tool>
get_weather
  描述: 通过 wttr.in 查询天气
  参数:
    - name: city
      type: string
      required: true
</tool>

## 执行命令
```bash
curl -s "wttr.in/{city}?format=j1"
```
```

#### 路径二：通过代码函数实现（高级用法）

在 `SKILL.md` 中嵌入 JavaScript 逻辑：

```markdown
## 工具实现
```javascript
async function get_weather({ city, units = "metric" }) {
  const response = await fetch(
    `https://api.openweathermap.org/data/2.5/weather?q=${encodeURIComponent(city)}&units=${units}&appid=${process.env.OPENWEATHER_API_KEY}`
  );
  const data = await response.json();
  return {
    city: data.name,
    temp: data.main.temp,
    humidity: data.main.humidity,
    description: data.weather[0].description,
  };
}
```
```

### 工具调用的完整流程

```
用户输入 → 触发词匹配 → Skill 激活 → 解析参数 → 调用工具 → 返回结果
```

---

## 6. 插件的触发与调用机制

### 触发词匹配规则

OpenClaw 在每次用户输入时，会扫描所有已加载插件的触发词列表：

```javascript
// 伪代码：触发词匹配逻辑
function matchTrigger(userInput, triggers) {
  const lowerInput = userInput.toLowerCase();
  return triggers.some(trigger => lowerInput.includes(trigger.toLowerCase()));
}
```

**匹配规则说明：**
- 触发词支持**包含匹配**（不需完全相等）
- 多个触发词是**或**关系（匹配任一即可）
- 触发词不区分大小写

### 触发词设计最佳实践

| ✅ 推荐写法 | ❌ 避免写法 |
|------------|------------|
| `天气`、`查天气`、`weather` | `查询天气信息`（过长） |
| `GitHub`、`查看仓库`、`PR` | `github_repository_info`（用户不会这样说） |
| `笔记`、`新建笔记`、`add note` | `使用笔记功能创建新文档`（过于具体） |

### 插件加载顺序

```bash
# 查看已加载插件及优先级
openclaw skills list --verbose

# 重新加载所有插件
openclaw gateway restart
```

---

## 7. 实战：一个天气查询插件

### Step 1：创建插件目录结构

```bash
mkdir -p ~/.openclaw/skills/weather-query/{references,scripts}
```

### Step 2：编写 SKILL.md

```markdown
# SKILL.md

## 描述
通过 wttr.in 免费接口查询全球城市的实时天气、温度、体感温度、风速和湿度。

## 触发词
- 天气
- 查天气
- 今天{城市}天气
- weather in
- 空气质量
- AQI

## 执行步骤
1. 从用户输入中提取城市名称
2. 调用 wttr.in API 获取 JSON 格式天气数据
3. 解析关键字段：当前温度、体感温度、风速、湿度、天气描述
4. 生成格式化回复，包含天气建议（如"适合户外"）

## 工具注册
<tool>
query_weather
  描述: 查询城市实时天气数据
  参数:
    - name: city
      type: string
      required: true
      description: 城市名称（中文或拼音均可，wttr.in 支持）
</tool>

## 执行命令
```bash
curl -s "wttr.in/{city}?format=j1" 2>/dev/null | head -c 2048
```

## 参考：API 响应字段说明

wttr.in 的 JSON 响应 (`format=j1`) 关键字段：

```json
{
  "current_condition": [
    {
      "temp_C": "22",
      "FeelsLikeC": "24",
      "humidity": "65",
      "windspeedKmph": "12",
      "weatherDesc": [{"value": "Partly cloudy"}]
    }
  ]
}
```
```

### Step 3：测试插件

```bash
# 模拟触发（查看插件是否被识别）
openclaw skills list | grep weather

# 通过 OpenClaw 发送测试消息触发插件
# 在对话中输入：帮我查一下北京的天气

# 查看日志验证触发
openclaw gateway logs | grep -i "weather\|skill"
```

### Step 4：常见输出格式

插件执行后，Agent 会将原始数据格式化为自然语言：

> 🌤️ **北京当前天气**
> - 温度：22°C（体感 24°C）
> - 湿度：65%
> - 风速：12 km/h
> - 描述：多云
> 💡 建议：适宜户外活动，注意防晒

---

## 8. 调试与日志

### 查看插件执行日志

```bash
# 实时查看所有日志
openclaw gateway logs --follow

# 只看 skill 相关日志
openclaw gateway logs | grep -i skill

# 只看 error 级别日志
openclaw gateway logs --level error
```

### 日志级别说明

| 级别 | 说明 | 触发场景 |
|------|------|----------|
| `debug` | 详细执行路径 | 插件参数解析、工具调用 |
| `info` | 正常流程信息 | Skill 加载成功、触发成功 |
| `warn` | 潜在问题 | API 超时、参数默认值生效 |
| `error` | 执行失败 | 工具调用异常、API 返回错误 |

### 在 SKILL.md 中添加调试信息

```markdown
## 调试标记
<!-- DEBUG: city=北京, units=metric -->
```

### 常见调试场景

#### 场景1：插件未被触发

```bash
# 检查1：触发词是否在 SKILL.md 中
grep -n "触发词" SKILL.md

# 检查2：插件是否被正确加载
openclaw skills list

# 检查3：重载插件
openclaw gateway restart
```

#### 场景2：工具调用失败

```bash
# 直接测试 API 是否可达
curl -v "wttr.in/Beijing?format=j1"

# 检查环境变量是否正确
echo $OPENWEATHER_API_KEY
```

#### 场景3：参数解析错误

```bash
# 在 SKILL.md 中添加参数校验日志
echo "DEBUG: city=$city, units=$units"

# 查看参数传递日志
openclaw gateway logs | grep -A2 "query_weather"
```

---

## 9. 插件发布与安装

### 发布到本地

将插件放入 OpenClaw 的 skills 目录即自动加载：

```bash
# 方式1：放入全局 skills 目录
cp -r my-weather-skill ~/.openclaw/skills/

# 方式2：放入项目级 skills 目录（需要在 agents.md 中指定路径）
cp -r my-weather-skill /path/to/your/project/skills/
```

### 通过 ClawHub 发布（推荐）

OpenClaw 提供官方插件市场 [ClawHub](https://clawhub.com)，支持一键安装：

```bash
# 搜索插件
openclaw skills search weather

# 安装插件
openclaw skills install weather-query

# 更新插件
openclaw skills update weather-query

# 卸载插件
openclaw skills uninstall weather-query
```

### 发布到 ClawHub

```bash
# 1. 登录 ClawHub
openclaw login

# 2. 初始化插件包
cd my-weather-skill
openclaw skills publish --name "weather-query" --version "1.0.0"

# 3. 添加描述和标签
openclaw skills publish --tag "weather" --tag "api" --description "天气查询插件"
```

### 插件市场发现率优化建议

让插件更容易被搜索到：

- **标题清晰**：使用高搜索量的关键词组合
- **描述完整**：准确说明功能、依赖、限制
- **触发词全面**：覆盖用户可能说的各种表达方式
- **版本号规范**：遵循 semver（主.次.修订）

---

## 10. 常见问题

### Q1：插件创建后需要重启 Gateway 吗？

**是的**。插件文件变更后需要重启才能生效：

```bash
openclaw gateway restart
```

### Q2：触发词冲突怎么办（多个插件同时触发）？

OpenClaw 按插件加载顺序选择第一个匹配的插件。可以通过 `agents.md` 调整加载顺序，或精化触发词使其更具体。

### Q3：工具调用超时怎么解决？

```markdown
## 在 SKILL.md 中设置超时
<tool>
get_weather
  描述: 查询天气（设置 10 秒超时）
  超时: 10000  # 毫秒
</tool>
```

### Q4：如何传递敏感信息（如 API Key）？

**禁止**在 SKILL.md 中硬编码敏感信息。使用环境变量：

```bash
# 在 shell 或 .env 文件中设置
export OPENWEATHER_API_KEY="your-secret-key"

# 在 SKILL.md 中引用
curl -H "X-API-Key: $OPENWEATHER_API_KEY" ...
```

### Q5：插件支持异步执行吗？

支持。在 `SKILL.md` 的执行命令中使用 `&` 后台执行，或在 JavaScript 实现中使用 `async/await`。

### Q6：如何调试正则匹配城市名？

```bash
# 测试城市名提取正则
echo "北京今天天气怎么样" | grep -oE "([\x{4e00}-\x{9fa5}]+)"
```

---

## 下一步学习路径

| 阶段 | 主题 | 推荐资源 |
|------|------|----------|
| 🟢 入门 | 掌握基本插件开发 | 本教程 |
| 🟡 进阶 | 多工具组合、链式调用 | [OpenClaw Skill 进阶教程](https://docs.openclaw.ai/skills/advanced) |
| 🔴 高级 | 插件沙箱安全、权限控制 | [OpenClaw 安全指南](https://docs.openclaw.ai/security) |
| 🚀 实战 | 构建自己的插件集 | [ClawHub 插件市场](https://clawhub.com) |

---

*本文档由墨客-内容生成专家整理 | OpenClaw 版本兼容：v1.8+*
