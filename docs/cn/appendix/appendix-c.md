# 附录 C：技能开发模板

OpenClaw 的技能（Skill）系统是其核心扩展机制。本附录提供基于官方信息的技能开发指南。

> 注意：OpenClaw 官方文档中技能开发相关内容正在完善中。本附录基于社区资源和官方 CLI 帮助信息整理。

## 技能开发概览

### 什么是 Skill？

根据 OpenClaw 官方文档，Skills 是扩展 OpenClaw 能力的模块化组件：
- 添加新的工具（Tools）供 Agent 调用
- 集成第三方 API 和服务
- 自定义 Agent 的行为和响应
- 实现特定的业务逻辑

### Skills vs Tools

根据官方说明：
- **Tools（工具）**：决定 OpenClaw 能不能做某类动作（"手脚"和"权限开关"）
- **Skills（技能）**：给 OpenClaw 增加特殊能力的"小程序"或"插件"

### 技能类型

OpenClaw 支持三种类型的技能：

1. **Bundled Skills（捆绑技能）**：随 OpenClaw 安装的内置技能（优先级最低）
2. **Managed Skills（托管技能）**：安装在 `~/.openclaw/skills` 的技能
3. **Workspace Skills（工作空间技能）**：工作空间目录下 `<workspace>/skills/` 的技能（优先级最高）

---

## 一、技能管理 CLI

### 查看技能

```bash
# 列出所有可用技能
openclaw skills list

# 仅显示就绪的技能
openclaw skills list --eligible

# 显示详细信息（包括缺失的需求）
openclaw skills list -v

# JSON 格式输出
openclaw skills list --json
```

### 查看技能详情

```bash
openclaw skills info <skill-name>
```

### 检查技能状态

```bash
openclaw skills check
```

### 通过 ClawHub 管理技能

```bash
# 搜索、安装和同步技能
npx clawhub
```

---

## 二、技能配置

### 在配置中启用技能

编辑 `~/.openclaw/openclaw.json`：

```json5
{
  skills: {
    entries: {
      "skill-name": {
        enabled: true,
        // 技能特定的配置
        apiKey: {
          source: "env",
          provider: "default",
          id: "SKILL_API_KEY"
        }
      }
    }
  }
}
```

### 使用 SecretRef 配置凭证

```json5
{
  skills: {
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: {
          source: "file",
          provider: "filemain",
          id: "/skills/entries/nano-banana-pro/apiKey"
        }
      }
    }
  }
}
```

**支持的 source 类型：**
- `env`：从环境变量读取
- `file`：从文件读取
- `exec`：从命令执行输出读取

---

## 三、工作空间技能开发

### 工作空间技能目录

工作空间技能位于 Agent 的工作空间目录中：

```
~/.openclaw/workspace/
└── skills/
    ├── my-skill/
    │   ├── SKILL.md       # 技能定义文件（YAML frontmatter + Markdown 指令）
    │   └── ...             # 其他辅助文件
```

### SKILL.md 结构

每个技能是一个包含 `SKILL.md` 文件的目录。`SKILL.md` 使用 YAML frontmatter 声明元数据，后面跟 Markdown 格式的指令：

````markdown
---
name: my-skill
description: 技能描述（这是最重要的字段，决定 AI 何时调用此技能）
user-invocable: true
disable-model-invocation: false
metadata:
  openclaw:
    requires:
      env:
        - MY_API_KEY
      bins:
        - curl
---

# 技能指令

这里写 Markdown 格式的技能指令，告诉 AI 如何使用这个技能。

可以用 `{baseDir}` 引用技能所在目录。
````

**YAML Frontmatter 字段说明：**

| 字段 | 必填 | 说明 |
|------|------|------|
| `name` | 是 | 技能标识符 |
| `description` | 是 | 技能描述，AI 根据此决定何时调用 |
| `user-invocable` | 否 | 是否可通过斜杠命令调用（默认 true） |
| `disable-model-invocation` | 否 | 是否从模型提示中排除（默认 false） |
| `metadata.openclaw.requires.env` | 否 | 需要的环境变量 |
| `metadata.openclaw.requires.bins` | 否 | 需要的系统命令 |
| `metadata.openclaw.requires.config` | 否 | 需要的配置路径 |

### 技能开发步骤

1. **创建工作空间技能目录**
   ```bash
   mkdir -p ~/.openclaw/workspace/skills/my-skill
   cd ~/.openclaw/workspace/skills/my-skill
   ```

2. **创建 SKILL.md**
   ```markdown
   ---
   name: hello-world
   description: 一个简单的示例技能，当用户说 hello 或 hi 时触发
   ---

   # Hello World 技能

   当用户打招呼时，回复一段友好的欢迎语。
   ```

3. **验证技能**
   ```bash
   openclaw skills check
   openclaw skills info hello-world
   ```

4. **测试技能**
   通过聊天界面发送触发关键词，测试技能是否正常工作。

---

## 四、技能最佳实践

### 1. 命名规范

- 使用小写字母和连字符：
  ✅ `my-awesome-skill`
  ❌ `MyAwesomeSkill`
  ❌ `my_awesome_skill`

### 2. 版本控制

遵循语义化版本规范（SemVer）：
- `MAJOR.MINOR.PATCH`
- 例如：`1.2.3`

### 3. 配置安全

- 使用 SecretRef 存储敏感信息
- 不要将 API 密钥硬编码在 SKILL.md 中
- 使用环境变量或文件存储凭证

### 4. 错误处理

- 提供清晰的错误信息
- 优雅处理 API 失败情况
- 记录调试信息

### 5. 文档编写

每个技能应包含：
- SKILL.md：技能定义文件（frontmatter 元数据 + Markdown 指令）
- 可选的辅助文件（脚本、配置模板等）

---

## 五、技能开发工具

### 使用内置的 skill-creator

OpenClaw 内置了 `skill-creator` 技能，可以帮助创建新技能：

```bash
# 查看内置技能
openclaw skills list --built-in

# 使用 skill-creator（通过聊天界面）
# 发送消息：帮我创建一个天气查询技能
```

### 插件开发

对于更复杂的扩展，可以开发插件：

```bash
# 列出插件
openclaw plugins list

# 安装插件
openclaw plugins install <path|.tgz|npm-spec>

# 启用/禁用插件
openclaw plugins enable <id>
openclaw plugins disable <id>

# 插件诊断
openclaw plugins doctor
```

---

## 六、技能发布

### 发布到 ClawHub

1. **准备技能包**
   ```bash
   # 确保包含所有必要文件
   my-skill/
   ├── SKILL.md
   └── ...
   ```

2. **提交到 ClawHub**
   - 访问 [clawhub.ai](https://clawhub.ai)
   - 按照提交指南上传技能

### 本地共享

```bash
# 打包技能
tar -czf my-skill.tgz my-skill/

# 分享给其他用户
# 其他用户安装：
openclaw plugins install ./my-skill.tgz
```

---

## 七、故障排查

### 技能无法加载

```bash
# 检查技能状态
openclaw skills check

# 查看详细信息
openclaw skills info <skill-name> -v

# 检查网关日志
openclaw logs
```

### 技能配置错误

```bash
# 验证配置
openclaw config validate

# 查看配置
openclaw config get skills.entries.<skill-name>
```

### 技能权限问题

确保技能文件有正确的权限：
```bash
chmod 755 ~/.openclaw/workspace/skills/my-skill/
```

---

## 八、参考资源

### 官方资源

- [OpenClaw 官方文档](https://docs.openclaw.ai)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [ClawHub](https://clawhub.ai)

### 社区资源

- [Awesome OpenClaw Skills](https://github.com/VoltAgent/awesome-openclaw-skills)
- [OpenClaw Discord](https://discord.gg/openclaw)

---

## 九、实战案例：tavily-search 技能解析

以 ClawHub 上的 [tavily-search](https://clawhub.ai/arun-8687/tavily-search)（v1.0.0）为例，看一个真实技能的完整结构。这是一个通过 Tavily API 为 AI Agent 提供网络搜索能力的技能。

### 目录结构

```
tavily-search-1.0.0/
├── SKILL.md           # 技能定义（frontmatter + 使用说明）
├── _meta.json         # ClawHub 发布元数据
└── scripts/
    ├── search.mjs     # 网络搜索实现
    └── extract.mjs    # 网页内容提取实现
```

### SKILL.md 完整内容

```markdown
---
name: tavily
description: AI-optimized web search via Tavily API. Returns concise, relevant results for AI agents.
homepage: https://tavily.com
metadata: {"clawdbot":{"emoji":"🔍","requires":{"bins":["node"],"env":["TAVILY_API_KEY"]},"primaryEnv":"TAVILY_API_KEY"}}
---

# Tavily Search

AI-optimized web search using Tavily API. Designed for AI agents - returns clean, relevant content.

## Search

​```bash
node {baseDir}/scripts/search.mjs "query"
node {baseDir}/scripts/search.mjs "query" -n 10
node {baseDir}/scripts/search.mjs "query" --deep
node {baseDir}/scripts/search.mjs "query" --topic news
​```

## Options

- `-n <count>`: Number of results (default: 5, max: 20)
- `--deep`: Use advanced search for deeper research (slower, more comprehensive)
- `--topic <topic>`: Search topic - `general` (default) or `news`
- `--days <n>`: For news topic, limit to last n days

## Extract content from URL

​```bash
node {baseDir}/scripts/extract.mjs "https://example.com/article"
​```

Notes:
- Needs `TAVILY_API_KEY` from https://tavily.com
- Tavily is optimized for AI - returns clean, relevant snippets
- Use `--deep` for complex research questions
- Use `--topic news` for current events
```

### 解析要点

**1. Frontmatter 设计**

| 字段 | 值 | 作用 |
|------|---|------|
| `name` | `tavily` | 技能标识符，用户通过 `clawhub install tavily-search` 安装后以此名称识别 |
| `description` | `AI-optimized web search...` | AI 根据这段描述判断何时调用此技能——写清楚"做什么"很关键 |
| `homepage` | `https://tavily.com` | 指向服务官网，方便用户注册获取 API Key |
| `metadata.requires.bins` | `["node"]` | 声明需要 Node.js 运行时，OpenClaw 会在技能加载时检查 |
| `metadata.requires.env` | `["TAVILY_API_KEY"]` | 声明需要环境变量，未设置时技能标记为"不可用"而非报错 |
| `metadata.primaryEnv` | `TAVILY_API_KEY` | 告诉 OpenClaw 配置向导该提示用户填哪个 Key |

**2. 脚本设计模式**

`search.mjs` 的核心逻辑（约 100 行）：

```javascript
// 1. 解析命令行参数
const query = args[0];
let n = 5, searchDepth = "basic", topic = "general";

// 2. 读取环境变量中的 API Key
const apiKey = (process.env.TAVILY_API_KEY ?? "").trim();
if (!apiKey) { console.error("Missing TAVILY_API_KEY"); process.exit(1); }

// 3. 调用 Tavily API
const resp = await fetch("https://api.tavily.com/search", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ api_key: apiKey, query, search_depth: searchDepth, ... }),
});

// 4. 格式化输出为 Markdown（AI 友好的格式）
if (data.answer) console.log("## Answer\n" + data.answer);
for (const r of results) {
  console.log(`- **${r.title}** (relevance: ${(r.score * 100).toFixed(0)}%)`);
  console.log(`  ${r.url}`);
}
```

**关键设计原则**：
- **输出 Markdown 格式**：AI Agent 能直接解析结构化结果
- **参数校验前置**：缺少 API Key 时 `process.exit(1)` 而非抛异常，让 OpenClaw 能捕获并提示用户
- **结果截断**：`content.slice(0, 300)` 避免输出过长影响 Agent 上下文窗口
- **`{baseDir}` 变量**：SKILL.md 中用 `{baseDir}` 引用技能目录，OpenClaw 运行时自动替换为实际路径

**3. 配置方式**

用户安装后，在 `openclaw.json` 中配置 API Key：

```json
{
  "skills": {
    "entries": {
      "tavily": {
        "enabled": true
      }
    }
  },
  "env": {
    "TAVILY_API_KEY": "tvly-你的密钥"
  }
}
```

或直接设置环境变量：`export TAVILY_API_KEY=tvly-你的密钥`

---

## 十、技能开发清单

开发技能前，确保完成以下检查：

- [ ] SKILL.md frontmatter 包含 name 和 description
- [ ] description 清晰描述触发条件（AI 靠这个决定何时调用）
- [ ] 环境变量需求在 metadata.requires.env 中声明
- [ ] 系统命令依赖在 metadata.requires.bins 中声明
- [ ] 脚本输出 Markdown 格式（对 AI 友好）
- [ ] 敏感信息使用环境变量或 SecretRef，不硬编码
- [ ] 错误处理完善（缺少 Key 时 exit(1)，API 失败时输出清晰错误）
- [ ] 在本地测试通过（`openclaw skills check` + 实际对话测试）

---

**提示**：OpenClaw 的技能系统正在快速发展中，建议关注官方文档和 GitHub 仓库获取最新信息。更多技能示例可在 [ClawHub](https://clawhub.ai) 浏览。
