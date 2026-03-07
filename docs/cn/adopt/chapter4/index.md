# 第四章 Skills 技能系统

OpenClaw 的技能系统就像手机的 App Store，让你可以为龙虾安装各种能力扩展。想让它查天气？安装天气技能。想让它管理 Gmail？安装邮件技能。技能系统让 OpenClaw 从一个通用助手变成专业工具，大幅提升工作效率。

## 1. 什么是技能

技能（Skill）本质上是一组工具定义和使用说明的组合。每个技能包含三个核心部分：

**工具定义**：告诉 OpenClaw 这个技能能做什么。比如天气技能提供 `get_weather` 工具，可以查询指定城市的天气。

**配置模板**：定义技能需要哪些配置信息。比如 Gmail 技能需要 API 密钥和邮箱地址。

**提示词指令**：教 OpenClaw 如何更好地使用这些工具。比如"查询天气时优先使用用户所在城市"。

安装技能后，OpenClaw 会自动学会使用这些工具。你不需要每次都详细解释怎么做，只需要说"帮我查天气"，OpenClaw 就知道该调用哪个工具。

## 2. 浏览和搜索技能

OpenClaw 社区维护了一个技能市场，包含数百个社区贡献的技能。查看可用技能：

```bash
openclaw skill search
```

会显示所有可用技能的列表，包括名称、描述、作者、下载量等信息。

搜索特定类型的技能：

```bash
openclaw skill search weather
openclaw skill search email
openclaw skill search database
```

查看技能的详细信息：

```bash
openclaw skill info weather
```

会显示技能的完整说明、需要的配置项、提供的工具列表、使用示例等。

## 3. 安装技能

安装技能非常简单：

```bash
openclaw skill install weather
```

OpenClaw 会自动下载技能文件，解析配置需求，然后引导你完成配置。比如安装天气技能时，会询问：

```
请输入天气 API 密钥（可在 openweathermap.org 获取）：
请输入默认城市（留空使用 IP 定位）：
```

填写完成后，技能就安装好了。你可以立即测试：

```
帮我查一下明天的天气
```

OpenClaw 会自动调用天气技能，返回详细的天气预报。

### 3.1 批量安装

如果你要在多台机器上配置相同的技能，可以导出配置：

```bash
openclaw skill export > skills.yaml
```

在新机器上批量安装：

```bash
openclaw skill import skills.yaml
```

这样就不需要每次都手动配置了。

## 4. 常用技能推荐

### 4.1 效率工具类

**weather**：查询天气预报，支持全球主要城市。可以查询未来 7 天的天气，包括温度、湿度、降水概率等。

**calendar**：管理日历事件，支持 Google Calendar、Outlook 等。可以创建、查询、修改日程，设置提醒。

**translator**：多语言翻译，支持 100+ 种语言。不仅能翻译文本，还能翻译文件、网页。

**currency**：实时汇率查询和货币转换。支持所有主流货币，数据每小时更新。

### 4.2 开发工具类

**github**：管理 GitHub 仓库，创建 issue、PR，查看代码统计。可以自动化很多 Git 操作。

**docker**：管理 Docker 容器和镜像。可以启动、停止容器，查看日志，清理无用镜像。

**database**：连接和查询数据库，支持 MySQL、PostgreSQL、MongoDB 等。可以执行 SQL 查询，导出数据。

**api-test**：测试 API 接口，支持 REST、GraphQL。可以自动生成测试报告。

### 4.3 内容创作类

**image-gen**：AI 图片生成，支持 DALL-E、Stable Diffusion 等模型。

**video-edit**：视频剪辑，支持裁剪、合并、添加字幕等基础操作。

**markdown**：Markdown 文档处理，可以转换格式、生成目录、美化排版。

**ocr**：图片文字识别，支持中英文及多种语言。

### 4.4 数据分析类

**excel**：处理 Excel 文件，支持数据清洗、统计分析、图表生成。

**csv**：CSV 文件处理，可以合并、筛选、转换格式。

**json**：JSON 数据处理，支持格式化、验证、转换。

**web-scraper**：网页数据抓取，可以提取结构化数据。

## 5. 管理已安装的技能

查看已安装的技能：

```bash
openclaw skill list
```

会显示所有已安装技能的状态、版本、最后使用时间等信息。

更新技能到最新版本：

```bash
openclaw skill update weather
```

或者更新所有技能：

```bash
openclaw skill update --all
```

卸载不需要的技能：

```bash
openclaw skill uninstall weather
```

禁用技能（保留配置但不加载）：

```bash
openclaw skill disable weather
```

重新启用：

```bash
openclaw skill enable weather
```

## 6. 配置技能

安装后可以随时修改技能配置：

```bash
openclaw skill config weather
```

会打开交互式配置界面，可以修改 API 密钥、默认参数等。

也可以直接编辑配置文件 `~/.openclaw/skills/weather/config.yaml`：

```yaml
api_key: "your_api_key"
default_city: "Beijing"
units: "metric"  # 使用摄氏度
language: "zh_CN"
```

修改后重启 OpenClaw 生效。

## 7. 创建自定义技能

如果市场上没有你需要的技能，可以自己创建。创建一个简单的技能只需要几分钟。

### 7.1 技能结构

一个技能包含以下文件：

```
my-skill/
├── SKILL.md          # 技能说明和配置
├── tools.yaml        # 工具定义
└── config.yaml       # 配置模板
```

### 7.2 编写 SKILL.md

```markdown
---
name: my-skill
version: 1.0.0
author: Your Name
description: 简短描述这个技能的功能
---

# My Skill

这个技能可以做什么，如何使用。

## 配置

需要以下配置项：
- API_KEY: 你的 API 密钥
- BASE_URL: API 基础地址
```

### 7.3 定义工具

在 `tools.yaml` 中定义技能提供的工具：

```yaml
tools:
  - name: my_tool
    description: 这个工具的功能描述
    parameters:
      - name: query
        type: string
        required: true
        description: 查询参数
    command: |
      curl -X GET "${BASE_URL}/api?q=${query}" \
        -H "Authorization: Bearer ${API_KEY}"
```

### 7.4 安装自定义技能

```bash
openclaw skill install ./my-skill
```

OpenClaw 会验证技能格式，然后安装到本地。

## 8. 实战案例

### 8.1 邮件自动分类

安装 Gmail 技能后，设置自动分类规则：

```
每天早上 9 点检查收件箱，
把来自客户的邮件标记为"重要"，
把营销邮件归档，
把需要回复的邮件生成待办清单发给我
```

OpenClaw 会学习你的邮件处理习惯，越用越智能。

### 8.2 代码审查助手

结合 GitHub 技能和代码分析技能：

```
当有新的 PR 时，
自动检查代码规范、潜在 bug、性能问题，
生成审查报告并评论到 PR 上
```

这比手动审查快得多，而且不会遗漏常见问题。

### 8.3 数据报告生成

使用 Excel 和图表技能：

```
每周一早上从数据库提取上周的销售数据，
生成包含趋势图、TOP 10 产品、同比增长的 Excel 报告，
发送到管理层邮箱
```

完全自动化，不需要人工干预。

### 8.4 多平台内容发布

组合多个社交媒体技能：

```
把这篇文章同时发布到微信公众号、知乎、小红书，
根据平台特点调整格式和配图，
记录发布结果和阅读数据
```

一次编写，多平台分发，大幅提升内容运营效率。

## 9. 最佳实践

**按需安装**：不要一次性安装太多技能，只安装真正需要的。技能越多，OpenClaw 的响应速度可能越慢。

**定期更新**：技能作者会持续改进功能和修复 bug，建议每月更新一次所有技能。

**保护敏感信息**：技能配置中的 API 密钥等敏感信息会加密存储，但仍要注意不要在公共场合分享配置文件。

**测试后再用**：新安装的技能先在测试环境试用，确认没问题后再用于生产任务。

**贡献社区**：如果你创建了有用的技能，可以提交到技能市场，帮助其他用户。

---

**下一步**：[第五章 让龙虾参与社区派对](/cn/adopt/chapter5/)

