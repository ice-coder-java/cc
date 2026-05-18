本文档用于记录 Claude Code 相关开发工具、全局规则文件 MCP 服务以及 skills 的安装与配置方式。整体目标是让 Claude Code 在代码生成、模型切换、技术文档查询、网页搜索与网页内容抓取等场景中具备更好的辅助能力。

## 1. Claude Code

Claude Code 是 Anthropic 提供的命令行 AI 编程工具，可以在终端中读取项目文件、执行命令、修改代码，并结合 MCP、Skills、Plugins、Hooks 等机制扩展能力。

本次暂不记录 Claude Code 的安装过程，后续可单独补充。

## 2. CC Switch

### 2.1 项目地址

- GitHub：https://github.com/farion1231/cc-switch
- 官方网站：https://ccswitch.io

### 2.2 工具作用

CC Switch 是一个面向 Claude Code、Codex、Gemini CLI、OpenCode、OpenClaw、Hermes Agent 等 AI 编程工具的统一管理器。它可以帮助用户集中管理不同 Provider、模型配置、API 地址、密钥、MCP、Prompts、Skills 和会话记录。

在 Claude Code 使用第三方模型或多个模型服务时，CC Switch 可以减少手动修改环境变量和配置文件的成本，适合用于多模型切换和统一配置管理。

### 2.3 安装方式

根据操作系统选择安装方式。

#### Windows

进入 Releases 页面下载最新版安装包：

```text
CC-Switch-v{version}-Windows.msi
````

## 3. CLAUDE.md 规则文件

### 3.1 参考项目

* GitHub：[https://github.com/multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills)

### 3.2 文件作用

`CLAUDE.md` 是 Claude Code 的规则说明文件，用于告诉 Claude Code 在当前项目或全局环境中应该遵守哪些开发习惯和行为约束。

它的作用不是替代 README，而是给 Claude Code 看的工作规则，例如：

* 不要在没有确认的情况下做大范围重构。
* 修改代码时尽量保持最小改动。
* 先澄清需求，再进行实现。
* 不要随意引入新依赖。
* 遇到不确定内容时主动说明。
* 修改完成后说明变更内容和验证方式。

### 3.3 安装位置

#### 全局规则

适合所有项目通用的个人开发习惯：

```bash
~/.claude/CLAUDE.md
```

#### 项目级规则

适合某一个项目专属的技术栈、启动命令、目录结构和开发规范：

```bash
项目根目录/CLAUDE.md
```

或：

```bash
项目根目录/.claude/CLAUDE.md
```

## 4. Context7 MCP

### 4.1 项目与官网

* GitHub：[https://github.com/upstash/context7](https://github.com/upstash/context7)
* 官网：[https://context7.com/dashboard](https://context7.com/dashboard)

### 4.2 工具作用

Context7 是一个面向 AI 编程工具的技术文档 MCP。它可以让 Claude Code 在需要框架、库、API、依赖配置或版本差异信息时，直接查询最新文档和代码示例，减少模型依赖旧训练数据导致的错误。

适合使用 Context7 的场景包括：

* 查询 Spring Boot、Vue、React、Next.js 等框架用法。
* 查询第三方库 API。
* 生成涉及外部依赖的代码。
* 配置 SDK、插件、中间件。
* 排查由于版本差异导致的问题。

### 4.3 Claude Code 本地服务器连接

使用本地 `npx` 启动 Context7 MCP Server：

```bash
claude mcp add --scope user context7 -- npx -y @upstash/context7-mcp --api-key YOUR_API_KEY
```

将 `YOUR_API_KEY` 替换为 Context7 控制台中的 API Key。

### 4.4 Claude Code 远程服务器连接

使用 Context7 远程 MCP 服务：

```bash
claude mcp add --scope user --header "CONTEXT7_API_KEY: YOUR_API_KEY" --transport http context7 https://mcp.context7.com/mcp
```

同样需要将 `YOUR_API_KEY` 替换为自己的 Context7 API Key。

### 4.5 CLAUDE.md 中的 Context7 调用规则

为了让 Claude Code 在涉及框架、依赖库、API 文档、代码生成、安装和配置步骤时主动调用 Context7，可以在全局或项目级 `CLAUDE.md` 文件中加入以下规则：

```md
Always use the Context7 MCP server when needing library/API documentation, code generation, setup or configuration steps. Do not wait for explicit requests — do it proactively.
```

该规则来自 Context7 官方文档的最佳实践。

## 5. Firecrawl MCP

### 5.1 项目与官网

* GitHub：[https://github.com/firecrawl/firecrawl](https://github.com/firecrawl/firecrawl)
* 官网：[https://www.firecrawl.dev/app](https://www.firecrawl.dev/app)
* Firecrawl MCP Server：[https://github.com/firecrawl/firecrawl-mcp-server](https://github.com/firecrawl/firecrawl-mcp-server)

### 5.2 工具作用

Firecrawl 是一个面向 AI Agent 的网页搜索、抓取、清洗和结构化提取工具。它可以把网页内容转成更适合大模型处理的 Markdown 或结构化数据。

在 Claude Code 中接入 Firecrawl MCP 后，可以让 Claude Code 具备以下能力：

* 搜索网页内容。
* 抓取指定 URL 的正文。
* 将网页清洗为 Markdown。
* 爬取网站页面。
* 提取结构化数据。
* 辅助整理官方文档、技术教程和网页资料。

它和 Context7 的定位不同：

| 工具        | 主要用途                    |
| --------- | ----------------------- |
| Context7  | 查询框架、库、API 的最新技术文档      |
| Firecrawl | 搜索网页、抓取网页正文、清洗网页内容、爬取网站 |

### 5.3 Claude Code 远程托管方式

Firecrawl 官方推荐使用 Remote hosted URL：

```bash
claude mcp add firecrawl --url https://mcp.firecrawl.dev/your-api-key/v2/mcp
```

将 `your-api-key` 替换为 Firecrawl 控制台中的 API Key。

### 5.4 Claude Code 本地 npx 方式

也可以通过本地 `npx` 启动 Firecrawl MCP Server：

```bash
claude mcp add firecrawl -e FIRECRAWL_API_KEY=your-api-key -- npx -y firecrawl-mcp
```

### 5.5 CLAUDE.md 中的 Firecrawl 调用规则

为了让 Claude Code 在网页搜索、网页抓取、网站爬取和内容提取任务中优先使用 Firecrawl，可以在 `CLAUDE.md` 文件中加入以下规则：

```md
Always use the Firecrawl MCP server for web search, scraping, crawling, and content extraction tasks. Do not use the built-in WebFetch/WebSearch tools when Firecrawl is available — it provides better results.
```

该规则用于明确 Firecrawl 的使用边界：当任务需要搜索网页、抓取指定 URL、清洗网页正文、爬取网站内容或提取结构化信息时，优先调用 Firecrawl MCP，而不是直接使用 Claude Code 内置的网页获取工具。

不过这条规则不是 Firecrawl 官方原文，而是根据使用场景整理的提示词规则。
