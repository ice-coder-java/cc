# Claude Code 开发辅助个人环境配置记录

本文档用于记录个人使用 Claude Code 相关开发工具、全局规则文件、MCP 服务、Plugins 以及 Skills 的安装与配置方式。

---

## 1. Claude Code

### 1.1 工具作用

Claude Code 是 Anthropic 提供的命令行 AI 编程工具，可以在终端中读取项目文件、执行命令、修改代码，并结合 MCP、Skills、Plugins、Hooks 等机制扩展能力。

### 1.2 安装方式

使用 npm 全局安装 Claude Code：

```bash
npm install -g @anthropic-ai/claude-code
```

安装完成后，可以在终端中执行：

```bash
claude
```

进入 Claude Code 交互界面。

### 1.3 基础配置

在 Windows 环境下，Claude Code 的用户级配置文件通常位于：

```text
C:\Users\{username}\.claude.json
```

如果需要跳过初始化引导，可以在 `.claude.json` 中加入：

```json
{
  "hasCompletedOnboarding": true
}
```

---

## 2. CC Switch

### 2.1 项目地址

* GitHub：[https://github.com/farion1231/cc-switch](https://github.com/farion1231/cc-switch)
* 官方网站：[https://ccswitch.io](https://ccswitch.io)

### 2.2 工具作用

在 Claude Code 使用第三方模型或多个模型服务时，CC Switch 可以减少手动修改环境变量和配置文件的成本，适合用于多模型切换、统一配置管理和跨工具使用。

### 2.3 安装方式

根据操作系统选择对应安装方式。

#### Windows

进入 Releases 页面下载最新版安装包：

```text
CC-Switch-v{version}-Windows.msi
```

下载后按照安装向导完成安装即可。

---

## 3. CLAUDE.md 规则文件

### 3.1 参考项目

* GitHub：[https://github.com/multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills)

### 3.2 文件作用

`CLAUDE.md` 是 Claude Code 的规则说明文件，用于告诉 Claude Code 在当前项目或全局环境中应该遵守哪些开发习惯和行为约束。

它的作用不是替代 README，而是给 Claude Code 看的工作规则。README 更多是给开发者阅读，而 `CLAUDE.md` 更偏向于约束 AI 在项目中的行为。

### 3.3 文件位置

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

### 3.4 使用建议

全局 `CLAUDE.md` 适合放通用开发习惯，例如最小改动、先分析再实现、不要随意引入依赖等。

项目级 `CLAUDE.md` 适合放项目专属内容，例如：
- 技术栈；
- 启动命令；
- 测试命令；
- 目录结构；
- 代码规范；
- 接口规范；
- 数据库迁移规则；
- 前后端联调方式。
---

## 4. MCP 服务配置

MCP 是 Model Context Protocol 的缩写，可以让 Claude Code 连接外部工具和数据源。通过 MCP，Claude Code 可以获得技术文档查询、网页搜索、网页抓取、数据库访问、文件系统访问等扩展能力。

在安装 MCP Server 时，需要注意安装作用域。Claude Code 支持 `local`、`project` 和 `user` 三种作用域，用来决定 MCP 配置在哪些项目中生效。

| 作用域       | 生效范围      | 是否适合提交 Git | 适合场景                         |
| --------- | --------- | ---------- | ---------------------------- |
| `local`   | 当前项目，个人可用 | 否          | 临时测试、某个项目的个人配置               |
| `project` | 当前项目，团队共享 | 是          | 团队统一 MCP 配置                  |
| `user`    | 所有项目，个人可用 | 否          | Context7、Firecrawl 等个人常用 MCP |

---

### 4.1 Context7 MCP

#### 4.1.1 项目与官网

* GitHub：[https://github.com/upstash/context7](https://github.com/upstash/context7)
* 官网：[https://context7.com/dashboard](https://context7.com/dashboard)

#### 4.1.2 工具作用

Context7 是一个面向 AI 编程工具的技术文档 MCP。它可以让 Claude Code 在需要框架、库、API、依赖配置或版本差异信息时，直接查询最新文档和代码示例，减少模型依赖旧训练数据导致的错误。

#### 4.1.3 本地 npx 方式

使用本地 `npx` 启动 Context7 MCP Server：

```bash
claude mcp add --scope user context7 -- npx -y @upstash/context7-mcp --api-key YOUR_API_KEY
```

将 `YOUR_API_KEY` 替换为 Context7 控制台中的 API Key。

#### 4.1.4 远程 HTTP 方式

使用 Context7 远程 MCP 服务：

```bash
claude mcp add --scope user --header "CONTEXT7_API_KEY: YOUR_API_KEY" --transport http context7 https://mcp.context7.com/mcp
```

同样需要将 `YOUR_API_KEY` 替换为自己的 Context7 API Key。

#### 4.1.5 个人安装命令

我实际使用的是远程 HTTP 方式，并将作用域设置为 `user`，使 Context7 在所有 Claude Code 项目中都可以使用：

```bash
claude mcp add --scope user --header "CONTEXT7_API_KEY: YOUR_API_KEY" --transport http context7 https://mcp.context7.com/mcp
```

#### 4.1.6 CLAUDE.md 调用规则

为了让 Claude Code 在涉及框架、依赖库、API 文档、代码生成、安装和配置步骤时主动调用 Context7，可以在全局或项目级 `CLAUDE.md` 中加入以下规则：

```md
Always use the Context7 MCP server when needing library/API documentation, code generation, setup or configuration steps. Do not wait for explicit requests — do it proactively.
```

该规则来自 Context7 官方文档的最佳实践。

---

### 4.2 Firecrawl MCP

#### 4.2.1 项目与官网

* GitHub：[https://github.com/firecrawl/firecrawl](https://github.com/firecrawl/firecrawl)
* 官网：[https://www.firecrawl.dev/app](https://www.firecrawl.dev/app)
* Firecrawl MCP Server：[https://github.com/firecrawl/firecrawl-mcp-server](https://github.com/firecrawl/firecrawl-mcp-server)

#### 4.2.2 工具作用

Firecrawl 是一个面向 AI Agent 的网页搜索、抓取、清洗和结构化提取工具。它可以把网页内容转成更适合大模型处理的 Markdown 或结构化数据。

它和 Context7 的定位不同：

| 工具        | 主要用途                    |
| --------- | ----------------------- |
| Context7  | 查询框架、库、API 的最新技术文档      |
| Firecrawl | 搜索网页、抓取网页正文、清洗网页内容、爬取网站 |

#### 4.2.3 远程托管方式

Firecrawl 官方推荐使用 Remote hosted URL：

```bash
claude mcp add firecrawl --url https://mcp.firecrawl.dev/YOUR_API_KEY/v2/mcp
```

将 `YOUR_API_KEY` 替换为 Firecrawl 控制台中的 API Key。

#### 4.2.4 本地 npx 方式

也可以通过本地 `npx` 启动 Firecrawl MCP Server：

```bash
claude mcp add firecrawl -e FIRECRAWL_API_KEY=YOUR_API_KEY -- npx -y firecrawl-mcp
```

#### 4.2.5 个人安装命令

我实际使用的 Firecrawl MCP 安装命令如下：

```bash
claude mcp add --transport http firecrawl https://mcp.firecrawl.dev/YOUR_API_KEY/v2/mcp --scope user
```

#### 4.2.6 CLAUDE.md 调用规则

为了让 Claude Code 在网页搜索、网页抓取、网站爬取和内容提取任务中优先使用 Firecrawl，可以在 `CLAUDE.md` 文件中加入以下规则：

```md
Always use the Firecrawl MCP server for web search, scraping, crawling, and content extraction tasks. Do not use the built-in WebFetch/WebSearch tools when Firecrawl is available — it provides better results.
```

这条规则不是 Firecrawl 官方原文，而是根据实际使用场景整理出的提示词规则。

---

## 5. Plugins 配置

Plugins 是 Claude Code 的插件机制，可以为 Claude Code 增加额外能力，例如命令、Skills、Hooks、MCP Server、上下文注入等。

Claude Code 插件通常支持以下作用域：

| 作用域       | 生效范围       | 适合场景       |
| --------- | ---------- | ---------- |
| `user`    | 当前用户所有项目   | 个人常用插件     |
| `project` | 当前项目，团队共享  | 团队统一插件配置   |
| `local`   | 当前项目，仅个人可用 | 临时测试或单项目试用 |

安装插件（默认作用域user）：

```bash
/plugin install 插件名@插件市场名
```

禁用插件：

```bash
/plugin disable 插件名@插件市场名 --scope user
```

启用插件：

```bash
/plugin enable 插件名@插件市场名 --scope user
```

卸载插件：

```bash
/plugin uninstall 插件名@插件市场名 --scope user
```

刷新插件：

```bash
/reload-plugins
```

查看已安装插件：

```bash
/plugin list
```

查看插件详情：

```bash
/plugin details 插件名
```

### 5.1 Superpowers 插件

#### 5.1.1 项目地址

* GitHub：[https://github.com/obra/superpowers#superpowers-marketplace](https://github.com/obra/superpowers#superpowers-marketplace)

#### 5.1.2 插件作用

Superpowers 是一个面向 Claude Code 的开发流程增强插件。它不是单纯提升模型能力的工具，而是通过一组开发流程、Agents、Hooks 和命令，让 Claude Code 在处理复杂开发任务时更重视需求澄清、方案设计、计划拆解、系统化调试、测试驱动和代码审查。

它比较适合用于从 0 到 1 开发完整功能、多文件修改、复杂 Bug 排查、代码重构和需要先写计划再执行的任务。对于简单的小改动，例如修改字段名、补充注释、调整少量逻辑，不一定需要强制使用 Superpowers，否则可能会显得流程较重。

#### 5.1.3 README 中的安装方式

Superpowers README 中提供了两种安装方式。

第一种是通过 Anthropic 官方插件市场安装：

```bash
/plugin install superpowers@claude-plugins-official
````

第二种是通过 Superpowers 自己的插件市场安装。先注册 marketplace：

```bash
/plugin marketplace add obra/superpowers-marketplace
```

再从该 marketplace 安装插件：

```bash
/plugin install superpowers@superpowers-marketplace
```

#### 5.1.4 我的实际安装过程

我最开始尝试使用官方插件市场安装：

```bash
/plugin install superpowers@claude-plugins-official
```

但出现了以下报错：

```text
Failed to load all marketplaces. Errors: claude-plugins-official: Failed to load marketplace "claude-plugins-official" from source (github): Failed to clone marketplace repository: SSH authentication failed. Please ensure your SSH keys are configured for GitHub, or use an HTTPS URL instead.

Original error: Cloning into 'C:\Users\YS-PC-1134\.claude\plugins\marketplaces\anthropics-claude-plugins-official'...
ssh: connect to host github.com port 22: Connection timed out
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

由于当时是在公司网络环境下操作，所以推测可能和公司内网限制、GitHub 访问策略或终端代理配置有关。

当时我先执行了以下两条 Git 配置：

```bash
git config --global url."https://github.com/".insteadOf "git@github.com:"
git config --global url."https://github.com/".insteadOf "ssh://git@github.com/"
```

这两条命令的作用是：当 Git 遇到 GitHub 的 SSH 地址时，自动替换成 HTTPS 地址。

不过这一步是在我还不完全确定是否为公司内网限制时执行的，不一定是必需步骤。

之后我又配置了 Git 的 HTTPS 代理，端口根据自己代理软件的实际端口填写：

```bash
git config --global https.proxy http://127.0.0.1:7897
```

配置完成后，可以先用下面命令测试 Git 是否能访问 GitHub：

```bash
git ls-remote https://github.com/obra/superpowers-marketplace.git
```

如果能够正常输出远程仓库信息，说明 Git 已经可以通过代理访问 GitHub。

随后再次尝试安装：

```bash
/plugin install superpowers@claude-plugins-official
```

这时出现了新的提示：

```text
Plugin "superpowers" not found in any marketplace
```

这个问题不是网络连接失败，而是因为当前 Claude Code 中还没有正确添加官方插件市场。于是我手动添加 Anthropic 官方 marketplace：

```bash
/plugin marketplace add https://github.com/anthropics/claude-plugins-official.git
```

添加成功后显示：

```text
Successfully added marketplace: claude-plugins-official
```

接着重新执行安装命令：

```bash
/plugin install superpowers@claude-plugins-official
```

最后执行刷新插件命令：

```bash
/reload-plugins
```

#### 5.1.5 安装后的清理操作

由于上述 Git 代理和地址替换配置是在公司环境下临时设置的，为了避免影响公司内网 Git 仓库、HTTPS 拉取或其他项目配置，安装完成后我将这些临时配置恢复。

取消 HTTPS 代理：

```bash
git config --global --unset https.proxy
git config --global --unset http.proxy
```

删除 GitHub 地址替换规则：

```bash
git config --global --unset-all url.https://github.com/.insteadOf
```

最后检查全局 Git 配置：

```bash
git config --global --list
```

确认里面不再包含以下内容：

```text
https.proxy=http://127.0.0.1:7897
http.proxy=http://127.0.0.1:7897
http.https://github.com.proxy=http://127.0.0.1:7897
https.https://github.com.proxy=http://127.0.0.1:7897
url.https://github.com/.insteadof=git@github.com:
url.https://github.com/.insteadof=ssh://git@github.com/
```
