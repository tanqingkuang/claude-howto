<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="../resources/logos/claude-howto-logo.svg">
</picture>

# MCP（Model Context Protocol）

这个目录包含 Claude Code 中 MCP 服务器配置与使用的完整文档和示例。

## 概览

MCP（Model Context Protocol）是 Claude 访问外部工具、API 和实时数据源的标准化方式。与 Memory 不同，MCP 提供的是对变化中数据的实时访问。

主要特性：
- 访问外部服务的实时数据
- 实时数据同步
- 可扩展架构
- 安全认证
- 基于工具的交互

## MCP 架构

```mermaid
graph TB
    A["Claude"]
    B["MCP Server"]
    C["外部服务"]

    A -->|请求：list_issues| B
    B -->|查询| C
    C -->|数据| B
    B -->|响应| A

    A -->|请求：create_issue| B
    B -->|操作| C
    C -->|结果| B
    B -->|响应| A

    style A fill:#e1f5fe,stroke:#333,color:#333
    style B fill:#f3e5f5,stroke:#333,color:#333
    style C fill:#e8f5e9,stroke:#333,color:#333
```

## MCP 生态

```mermaid
graph TB
    A["Claude"] -->|MCP| B["Filesystem<br/>MCP Server"]
    A -->|MCP| C["GitHub<br/>MCP Server"]
    A -->|MCP| D["Database<br/>MCP Server"]
    A -->|MCP| E["Slack<br/>MCP Server"]
    A -->|MCP| F["Google Docs<br/>MCP Server"]

    B -->|File I/O| G["Local Files"]
    C -->|API| H["GitHub Repos"]
    D -->|Query| I["PostgreSQL/MySQL"]
    E -->|Messages| J["Slack Workspace"]
    F -->|Docs| K["Google Drive"]

    style A fill:#e1f5fe,stroke:#333,color:#333
    style B fill:#f3e5f5,stroke:#333,color:#333
    style C fill:#f3e5f5,stroke:#333,color:#333
    style D fill:#f3e5f5,stroke:#333,color:#333
    style E fill:#f3e5f5,stroke:#333,color:#333
    style F fill:#f3e5f5,stroke:#333,color:#333
    style G fill:#e8f5e9,stroke:#333,color:#333
    style H fill:#e8f5e9,stroke:#333,color:#333
    style I fill:#e8f5e9,stroke:#333,color:#333
    style J fill:#e8f5e9,stroke:#333,color:#333
    style K fill:#e8f5e9,stroke:#333,color:#333
```

## MCP 安装方式

Claude Code 支持多种连接 MCP 服务器的传输协议：

### HTTP 传输（推荐）

```bash
# Basic HTTP connection
claude mcp add --transport http notion https://mcp.notion.com/mcp

# HTTP with authentication header
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer your-token"
```

### Stdio 传输（本地）

适用于本地运行的 MCP 服务器：

```bash
# Local Node.js server
claude mcp add --transport stdio myserver -- npx @myorg/mcp-server

# With environment variables
claude mcp add --transport stdio myserver --env KEY=value -- npx server
```

### SSE 传输（已弃用）

Server-Sent Events 传输已弃用，建议改用 `http`，但仍然支持：

```bash
claude mcp add --transport sse legacy-server https://example.com/sse
```

### WebSocket 传输

用于持续双向连接的 WebSocket 传输：

```bash
claude mcp add --transport ws realtime-server wss://example.com/mcp
```

### Windows 专用说明

在原生 Windows（非 WSL）上，对 npx 命令请使用 `cmd /c`：

```bash
claude mcp add --transport stdio my-server -- cmd /c npx -y @some/package
```

### OAuth 2.0 认证

Claude Code 支持需要 OAuth 的 MCP 服务器。连接到启用 OAuth 的服务器时，Claude Code 会处理整个认证流程：

```bash
# Connect to an OAuth-enabled MCP server (interactive flow)
claude mcp add --transport http my-service https://my-service.example.com/mcp

# Pre-configure OAuth credentials for non-interactive setup
claude mcp add --transport http my-service https://my-service.example.com/mcp \
  --client-id "your-client-id" \
  --client-secret "your-client-secret" \
  --callback-port 8080
```

| 功能 | 说明 |
|---------|-------------|
| **交互式 OAuth** | 使用 `/mcp` 触发基于浏览器的 OAuth 流程 |
| **预配置 OAuth 客户端** | 内置 OAuth 客户端，适用于 Notion、Stripe 等常见服务（v2.1.30+） |
| **预配置凭据** | `--client-id`、`--client-secret`、`--callback-port` 标志用于自动化设置 |
| **Token 存储** | 令牌会安全地存储在系统 keychain 中 |
| **Step-up 认证** | 支持高权限操作的 step-up 认证 |
| **发现缓存** | OAuth discovery 元数据会缓存，以便更快重连 |
| **元数据覆盖** | 在 `.mcp.json` 中使用 `oauth.authServerMetadataUrl` 覆盖默认 OAuth 元数据发现 |

#### 覆盖 OAuth 元数据发现

如果你的 MCP 服务器在标准 OAuth 元数据端点（`/.well-known/oauth-authorization-server`）返回错误，但提供了可用的 OIDC 端点，你可以告诉 Claude Code 从特定 URL 获取 OAuth 元数据。在服务器配置的 `oauth` 对象中设置 `authServerMetadataUrl`：

```json
{
  "mcpServers": {
    "my-server": {
      "type": "http",
      "url": "https://mcp.example.com/mcp",
      "oauth": {
        "authServerMetadataUrl": "https://auth.example.com/.well-known/openid-configuration"
      }
    }
  }
}
```

该 URL 必须使用 `https://`。此选项需要 Claude Code v2.1.64 或更高版本。

### Claude.ai MCP 连接器

在你的 Claude.ai 账户里配置的 MCP 服务器会自动在 Claude Code 中可用。这意味着你通过 Claude.ai 网页界面设置的任何 MCP 连接，都无需额外配置即可访问。

Claude.ai MCP 连接器也可在 `--print` 模式中使用（v2.1.83+），支持非交互式和脚本化用法。

如果要在 Claude Code 中禁用 Claude.ai MCP 服务器，将 `ENABLE_CLAUDEAI_MCP_SERVERS` 环境变量设为 `false`：

```bash
ENABLE_CLAUDEAI_MCP_SERVERS=false claude
```

> **注意：** 此功能仅适用于使用 Claude.ai 账户登录的用户。

## MCP 设置流程

```mermaid
sequenceDiagram
    participant User
    participant Claude as Claude Code
    participant Config as Config File
    participant Service as External Service

    User->>Claude: 输入 /mcp
    Claude->>Claude: 列出可用的 MCP 服务器
    Claude->>User: 展示选项
    User->>Claude: 选择 GitHub MCP
    Claude->>Config: 更新配置
    Config->>Claude: 激活连接
    Claude->>Service: 测试连接
    Service-->>Claude: 认证成功
    Claude->>User: ✅ MCP 已连接！
```

## MCP 工具搜索

当 MCP 工具描述超过上下文窗口的 10% 时，Claude Code 会自动启用工具搜索，以便在不淹没模型上下文的情况下高效选择合适工具。

| 设置 | 值 | 说明 |
|---------|-------|-------------|
| `ENABLE_TOOL_SEARCH` | `auto`（默认） | 当工具描述超过上下文的 10% 时自动启用 |
| `ENABLE_TOOL_SEARCH` | `auto:<N>` | 在自定义阈值 `N` 个工具时自动启用 |
| `ENABLE_TOOL_SEARCH` | `true` | 不管工具数量多少，始终启用 |
| `ENABLE_TOOL_SEARCH` | `false` | 禁用；会完整发送所有工具描述 |

> **注意：** 工具搜索需要 Sonnet 4 或更高版本，或者 Opus 4 或更高版本。Haiku 模型不支持工具搜索。

## 动态工具更新

Claude Code 支持 MCP `list_changed` 通知。当 MCP 服务器动态添加、移除或修改可用工具时，Claude Code 会接收更新并自动调整工具列表，无需重新连接或重启。

## MCP 询问信息（Elicitation）

MCP 服务器可以通过交互式对话向用户请求结构化输入（v2.1.49+）。这使得 MCP 服务器可以在工作流中途请求更多信息，例如确认某项操作、从选项列表中选择，或者填写必填字段，从而增强交互性。

## 工具描述和指令上限

自 v2.1.84 起，Claude Code 对每个 MCP 服务器的工具描述和指令实施 **2 KB 上限**。这样可以防止单个服务器用过于冗长的工具定义占用过多上下文，从而减少上下文膨胀并保持交互高效。

## 将 MCP Prompts 作为斜杠命令

MCP 服务器可以暴露 prompts，并将它们显示为 Claude Code 中的斜杠命令。其命名约定如下：

```text
/mcp__<server>__<prompt>
```

例如，如果名为 `github` 的服务器暴露了一个叫 `review` 的 prompt，你可以这样调用：`/mcp__github__review`。

## Server 去重

当同一个 MCP 服务器在多个作用域（本地、项目、用户）中都定义时，本地配置优先。这让你可以通过本地自定义覆盖项目级或用户级的 MCP 设置，而不会产生冲突。

## 通过 `@` 提及访问 MCP 资源

你可以在提示词中直接使用 `@` 提及语法来引用 MCP 资源：

```text
@server-name:protocol://resource/path
```

例如，要引用某个数据库资源：

```text
@database:postgres://mydb/users
```

这样 Claude 就能把 MCP 资源内容作为会话上下文的一部分直接抓取并内联使用。

## MCP 作用域

MCP 配置可以存放在不同作用域中，并拥有不同的共享级别：

| 作用域 | 位置 | 说明 | 共享对象 | 是否需要审批 |
|-------|----------|-------------|-------------|------------------|
| **Local**（默认） | `~/.claude.json`（位于项目路径下） | 仅当前用户、当前项目可见（旧版本里叫 `project`） | 只有你自己 | 否 |
| **Project** | `.mcp.json` | 提交到 git 仓库 | 团队成员 | 是（首次使用） |
| **User** | `~/.claude.json` | 在所有项目中可用（旧版本里叫 `global`） | 只有你自己 | 否 |

### 使用 Project 作用域

把项目特定的 MCP 配置存储在 `.mcp.json` 中：

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.github.com/mcp"
    }
  }
}
```

团队成员首次使用项目 MCP 时会看到审批提示。

## MCP 配置管理

### 添加 MCP 服务器

```bash
# 添加基于 HTTP 的服务器
claude mcp add --transport http github https://api.github.com/mcp

# 添加本地 stdio 服务器
claude mcp add --transport stdio database -- npx @company/db-server

# 列出所有 MCP 服务器
claude mcp list

# 获取某个服务器的详情
claude mcp get github

# 删除一个 MCP 服务器
claude mcp remove github

# 重置项目级审批选择
claude mcp reset-project-choices

# 从 Claude Desktop 导入
claude mcp add-from-claude-desktop
```

## 可用 MCP 服务器表

| MCP 服务器 | 用途 | 常见工具 | 认证 | 实时 |
|------------|---------|--------------|------|-----------|
| **Filesystem** | 文件操作 | read, write, delete | OS 权限 | ✅ 是 |
| **GitHub** | 仓库管理 | list_prs, create_issue, push | OAuth | ✅ 是 |
| **Slack** | 团队沟通 | send_message, list_channels | Token | ✅ 是 |
| **Database** | SQL 查询 | query, insert, update | 凭据 | ✅ 是 |
| **Google Docs** | 文档访问 | read, write, share | OAuth | ✅ 是 |
| **Asana** | 项目管理 | create_task, update_status | API Key | ✅ 是 |
| **Stripe** | 支付数据 | list_charges, create_invoice | API Key | ✅ 是 |
| **Memory** | 持久 memory | store, retrieve, delete | 本地 | ❌ 否 |

## 实用示例

### 示例 1：GitHub MCP 配置

**文件：** `.mcp.json`（项目根目录）

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

**可用的 GitHub MCP 工具：**

#### Pull Request 管理
- `list_prs` - 列出仓库中的所有 PR
- `get_pr` - 获取包含 diff 在内的 PR 详情
- `create_pr` - 创建新 PR
- `update_pr` - 更新 PR 描述/标题
- `merge_pr` - 将 PR 合并到 main 分支
- `review_pr` - 添加 review comments

**示例请求：**
```markdown
/mcp__github__get_pr 456

# 返回：
标题：添加深色模式支持
作者：@alice
描述：使用 CSS variables 实现深色主题
状态：OPEN
审阅者：@bob, @charlie
```

#### Issue 管理
- `list_issues` - 列出所有 issues
- `get_issue` - 获取 issue 详情
- `create_issue` - 创建新 issue
- `close_issue` - 关闭 issue
- `add_comment` - 给 issue 添加评论

#### 仓库信息
- `get_repo_info` - 仓库详情
- `list_files` - 文件树结构
- `get_file_content` - 读取文件内容
- `search_code` - 在代码库中搜索

#### 提交操作
- `list_commits` - 提交历史
- `get_commit` - 指定提交详情
- `create_commit` - 创建新提交

**设置：**
```bash
export GITHUB_TOKEN="your_github_token"
# 或者直接用 CLI 添加：
claude mcp add --transport stdio github -- npx @modelcontextprotocol/server-github
```

### 配置中的环境变量展开

MCP 配置支持带默认值回退的环境变量展开。`${VAR}` 和 `${VAR:-default}` 语法可用于以下字段：`command`、`args`、`env`、`url` 和 `headers`。

```json
{
  "mcpServers": {
    "api-server": {
      "type": "http",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}",
        "X-Custom-Header": "${CUSTOM_HEADER:-default-value}"
      }
    },
    "local-server": {
      "command": "${MCP_BIN_PATH:-npx}",
      "args": ["${MCP_PACKAGE:-@company/mcp-server}"],
      "env": {
        "DB_URL": "${DATABASE_URL:-postgresql://localhost/dev}"
      }
    }
  }
}
```

变量会在运行时展开：
- `${VAR}` - 使用环境变量；如果未设置则报错
- `${VAR:-default}` - 使用环境变量；如果未设置则回退到默认值

### 示例 2：Database MCP 设置

**配置：**

```json
{
  "mcpServers": {
    "database": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-database"],
      "env": {
        "DATABASE_URL": "postgresql://user:pass@localhost/mydb"
      }
    }
  }
}
```

**示例用法：**

```markdown
User: 查找所有订单数超过 10 的用户

Claude: 我会查询你的数据库来查找这条信息。

# 使用 MCP database 工具：
SELECT u.*, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id
HAVING COUNT(o.id) > 10
ORDER BY order_count DESC;

# 结果：
- Alice：15 笔订单
- Bob：12 笔订单
- Charlie：11 笔订单
```

**设置：**
```bash
export DATABASE_URL="postgresql://user:pass@localhost/mydb"
# 或者直接用 CLI 添加：
claude mcp add --transport stdio database -- npx @modelcontextprotocol/server-database
```

### 示例 3：多 MCP 工作流

**场景：每日报告生成**

```markdown
# 使用多个 MCP 的每日报告工作流

## 设置
1. GitHub MCP - 获取 PR 指标
2. Database MCP - 查询销售数据
3. Slack MCP - 发布报告
4. Filesystem MCP - 保存报告

## 工作流

### 第 1 步：获取 GitHub 数据
/mcp__github__list_prs completed:true last:7days

输出：
- PR 总数：42
- 平均合并时间：2.3 小时
- Review 周转时间：1.1 小时

### 第 2 步：查询数据库
SELECT COUNT(*) as sales, SUM(amount) as revenue
FROM orders
WHERE created_at > NOW() - INTERVAL '1 day'

输出：
- 销售：247
- 收入：$12,450

### 第 3 步：生成报告
把数据合并成 HTML 报告

### 第 4 步：保存到文件系统
将 report.html 写入 /reports/

### 第 5 步：发布到 Slack
把摘要发送到 #daily-reports 频道

最终输出：
✅ 报告已生成并发布
📊 本周合并了 47 个 PR
💰 每日销售额 $12,450
```

**设置：**
```bash
export GITHUB_TOKEN="your_github_token"
export DATABASE_URL="postgresql://user:pass@localhost/mydb"
export SLACK_TOKEN="your_slack_token"
# 通过 CLI 添加每个 MCP server，或者在 .mcp.json 中配置它们
```

### 示例 4：Filesystem MCP 操作

**配置：**

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-filesystem", "/home/user/projects"]
    }
  }
}
```

**可用操作：**

| 操作 | 命令 | 作用 |
|-----------|---------|---------|
| 列出文件 | `ls ~/projects` | 显示目录内容 |
| 读取文件 | `cat src/main.ts` | 读取文件内容 |
| 写入文件 | `create docs/api.md` | 创建新文件 |
| 编辑文件 | `edit src/app.ts` | 修改文件 |
| 搜索 | `grep "async function"` | 在文件中搜索 |
| 删除 | `rm old-file.js` | 删除文件 |

**设置：**
```bash
# 直接使用 CLI 添加：
claude mcp add --transport stdio filesystem -- npx @modelcontextprotocol/server-filesystem /home/user/projects
```

## MCP vs Memory：决策矩阵

```mermaid
graph TD
    A["需要外部数据吗？"]
    A -->|否| B["使用 Memory"]
    A -->|是| C["它变化频繁吗？"]
    C -->|否/很少| B
    C -->|是/经常| D["使用 MCP"]

    B -->|存储| E["偏好<br/>上下文<br/>历史"]
    D -->|访问| F["实时 API<br/>数据库<br/>服务"]

    style A fill:#fff3e0,stroke:#333,color:#333
    style B fill:#e1f5fe,stroke:#333,color:#333
    style C fill:#fff3e0,stroke:#333,color:#333
    style D fill:#f3e5f5,stroke:#333,color:#333
    style E fill:#e8f5e9,stroke:#333,color:#333
    style F fill:#e8f5e9,stroke:#333,color:#333
```

## 请求/响应模式

```mermaid
sequenceDiagram
    participant App as Claude
    participant MCP as MCP Server
    participant DB as Database

    App->>MCP: 请求："SELECT * FROM users WHERE id=1"
    MCP->>DB: 执行查询
    DB-->>MCP: 结果集
    MCP-->>App: 返回解析后的数据
    App->>App: 处理结果
    App->>App: 继续任务

    Note over MCP,DB: 实时访问<br/>无缓存
```

## 环境变量

把敏感凭据存放在环境变量中：

```bash
# ~/.bashrc or ~/.zshrc
export GITHUB_TOKEN="ghp_xxxxxxxxxxxxx"
export DATABASE_URL="postgresql://user:pass@localhost/mydb"
export SLACK_TOKEN="xoxb-xxxxxxxxxxxxx"
```

然后在 MCP 配置中引用它们：

```json
{
  "env": {
    "GITHUB_TOKEN": "${GITHUB_TOKEN}"
  }
}
```

## 让 Claude 充当 MCP 服务器（`claude mcp serve`）

Claude Code 本身也可以作为其他应用的 MCP 服务器。这让外部工具、编辑器和自动化系统可以通过标准 MCP 协议利用 Claude 的能力。

```bash
# Start Claude Code as an MCP server on stdio
claude mcp serve
```

其他应用随后可以像连接任意基于 stdio 的 MCP 服务器一样连接到这个服务器。例如，要在另一个 Claude Code 实例中把 Claude Code 作为 MCP 服务器添加进去：

```bash
claude mcp add --transport stdio claude-agent -- claude mcp serve
```

这对于构建多代理工作流非常有用，其中一个 Claude 实例可以编排另一个实例。

## 受管 MCP 配置（企业版）

对于企业部署，IT 管理员可以通过 `managed-mcp.json` 配置文件强制执行 MCP 服务器策略。这个文件可以在组织范围内独占控制允许或阻止哪些 MCP 服务器。

**位置：**
- macOS：`/Library/Application Support/ClaudeCode/managed-mcp.json`
- Linux：`~/.config/ClaudeCode/managed-mcp.json`
- Windows：`%APPDATA%\ClaudeCode\managed-mcp.json`

**功能：**
- `allowedMcpServers` -- 允许的服务器白名单
- `deniedMcpServers` -- 禁止的服务器黑名单
- 支持按服务器名称、命令和 URL 模式匹配
- 在用户配置之前先执行组织级 MCP 策略
- 防止未授权的服务器连接

**示例配置：**

```json
{
  "allowedMcpServers": [
    {
      "serverName": "github",
      "serverUrl": "https://api.github.com/mcp"
    },
    {
      "serverName": "company-internal",
      "serverCommand": "company-mcp-server"
    }
  ],
  "deniedMcpServers": [
    {
      "serverName": "untrusted-*"
    },
    {
      "serverUrl": "http://*"
    }
  ]
}
```

> **注意：** 当 `allowedMcpServers` 和 `deniedMcpServers` 同时匹配某个服务器时，拒绝规则优先。

## 插件提供的 MCP 服务器

插件可以捆绑自己的 MCP 服务器，在插件安装后自动可用。插件提供的 MCP 服务器可以通过两种方式定义：

1. **独立 `.mcp.json`** -- 在插件根目录放一个 `.mcp.json` 文件
2. **直接写在 `plugin.json` 中** -- 在插件清单里直接定义 MCP 服务器

使用 `${CLAUDE_PLUGIN_ROOT}` 变量引用相对于插件安装目录的路径：

```json
{
  "mcpServers": {
    "plugin-tools": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/dist/mcp-server.js"],
      "env": {
        "CONFIG_PATH": "${CLAUDE_PLUGIN_ROOT}/config.json"
      }
    }
  }
}
```

## 子代理作用域的 MCP

MCP 服务器可以通过 `mcpServers:` key 直接写在 agent frontmatter 中，将其作用域限制在特定子代理，而不是整个项目。这在某个 agent 需要访问某个特定 MCP 服务器、而工作流中的其他 agent 不需要时尤其有用。

```yaml
---
mcpServers:
  my-tool:
    type: http
    url: https://my-tool.example.com/mcp
---

You are an agent with access to my-tool for specialized operations.
```

子代理作用域的 MCP 服务器只在该 agent 的执行上下文中可用，不会共享给父 agent 或兄弟 agent。

## MCP 输出限制

Claude Code 会对 MCP 工具输出实施限制，以防止上下文溢出：

| 限制 | 阈值 | 行为 |
|-------|-----------|----------|
| **警告** | 10,000 tokens | 显示输出过大的警告 |
| **默认最大值** | 25,000 tokens | 超过此限制的输出会被截断 |
| **磁盘持久化** | 50,000 字符 | 超过 5 万字符的工具结果会持久化到磁盘 |

最大输出限制可以通过 `MAX_MCP_OUTPUT_TOKENS` 环境变量配置：

```bash
# Increase the max output to 50,000 tokens
export MAX_MCP_OUTPUT_TOKENS=50000
```

## 用代码执行解决上下文膨胀

随着 MCP 的采用规模扩大，连接几十个服务器、上百甚至上千个工具会带来一个重大挑战：**上下文膨胀**。这可以说是 MCP 在大规模场景下最大的难题，而 Anthropic 的工程团队提出了一个优雅的方案：用代码执行代替直接工具调用。

> **来源**：[Code Execution with MCP: Building More Efficient Agents](https://www.anthropic.com/engineering/code-execution-with-mcp) — Anthropic Engineering Blog

### 问题：两个 token 浪费源

**1. 工具定义会塞满上下文窗口**

大多数 MCP 客户端会预先加载所有工具定义。连接到成千上万个工具时，模型在阅读用户请求之前就必须处理几十万 token。

**2. 中间结果会消耗额外 token**

每个中间工具结果都会经过模型上下文。比如把会议记录从 Google Drive 转到 Salesforce，完整 transcript 会通过上下文 **两次**：一次读取，一次写入目标系统。一个 2 小时的会议 transcript 可能意味着 50,000+ 额外 token。

```mermaid
graph LR
    A["模型"] -->|"工具调用：getDocument"| B["MCP Server"]
    B -->|"完整 transcript（50K tokens）"| A
    A -->|"工具调用：updateRecord<br/>(重新发送完整 transcript)"| B
    B -->|"确认"| A

    style A fill:#ffcdd2,stroke:#333,color:#333
    style B fill:#f3e5f5,stroke:#333,color:#333
```

### 解决方案：把 MCP 工具当作代码 API

不要把工具定义和结果穿过上下文窗口，而是让 agent **编写代码**，把 MCP 工具当作 API 来调用。代码运行在沙盒执行环境里，只有最终结果返回给模型。

```mermaid
graph LR
    A["模型"] -->|"编写代码"| B["代码执行<br/>环境"]
    B -->|"直接调用工具"| C["MCP Servers"]
    C -->|"数据留在<br/>执行环境中"| B
    B -->|"只返回最终结果<br/>(最少 token)"| A

    style A fill:#c8e6c9,stroke:#333,color:#333
    style B fill:#e1f5fe,stroke:#333,color:#333
    style C fill:#f3e5f5,stroke:#333,color:#333
```

#### 它是如何工作的

MCP 工具会以类型化函数的文件树形式呈现：

```text
servers/
├── google-drive/
│   ├── getDocument.ts
│   └── index.ts
├── salesforce/
│   ├── updateRecord.ts
│   └── index.ts
└── ...
```

每个工具文件都包含一个类型化封装：

```typescript
// ./servers/google-drive/getDocument.ts
import { callMCPTool } from "../../../client.js";

interface GetDocumentInput {
  documentId: string;
}

interface GetDocumentResponse {
  content: string;
}

export async function getDocument(
  input: GetDocumentInput
): Promise<GetDocumentResponse> {
  return callMCPTool<GetDocumentResponse>(
    'google_drive__get_document', input
  );
}
```

然后 agent 再编写代码来编排这些工具：

```typescript
import * as gdrive from './servers/google-drive';
import * as salesforce from './servers/salesforce';

// 数据直接在工具之间流动，绝不会经过模型
const transcript = (
  await gdrive.getDocument({ documentId: 'abc123' })
).content;

await salesforce.updateRecord({
  objectType: 'SalesMeeting',
  recordId: '00Q5f000001abcXYZ',
  data: { Notes: transcript }
});
```

**结果：token 用量从约 150,000 降到约 2,000，减少了 98.7%。**

### 关键收益

| 收益 | 说明 |
|---------|-------------|
| **渐进式披露** | Agent 先浏览文件系统，只加载当前需要的工具定义，而不是一开始加载全部工具 |
| **上下文高效的结果** | 数据在执行环境中先被过滤/转换，再返回给模型 |
| **强大的控制流** | 循环、条件和错误处理都在代码中执行，不必在模型和工具之间来回传递 |
| **隐私保护** | 中间数据（PII、敏感记录）留在执行环境中，不会进入模型上下文 |
| **状态持久化** | Agent 可以把中间结果保存到文件里，构建可复用的 skill 函数 |

#### 示例：过滤大数据集

```typescript
// 不使用代码执行 - 10,000 行全部流过上下文
// TOOL CALL: gdrive.getSheet(sheetId: 'abc123')
//   -> 在上下文中返回 10,000 行

// 使用代码执行 - 在执行环境中先过滤
const allRows = await gdrive.getSheet({ sheetId: 'abc123' });
const pendingOrders = allRows.filter(
  row => row["Status"] === 'pending'
);
console.log(`Found ${pendingOrders.length} pending orders`);
console.log(pendingOrders.slice(0, 5)); // 只有 5 行到达模型
```

#### 示例：不经过 round-trip 的循环

```typescript
// 轮询部署通知 - 完全在代码中运行
let found = false;
while (!found) {
  const messages = await slack.getChannelHistory({
    channel: 'C123456'
  });
  found = messages.some(
    m => m.text.includes('deployment complete')
  );
  if (!found) await new Promise(r => setTimeout(r, 5000));
}
console.log('Deployment notification received');
```

### 需要考虑的权衡

代码执行会带来它自己的复杂度。运行 agent 生成的代码需要：

- 一个**安全的沙盒执行环境**，并具备合适的资源限制
- 对执行代码进行**监控和日志记录**
- 相比直接工具调用，带来额外的**基础设施开销**

减少 token 成本、降低延迟、提升工具组合能力等收益，应该与这些实现成本一起权衡。对于只接少量 MCP 服务器的 agent，直接工具调用可能更简单；对于大规模 agent（几十个服务器、几百个工具），代码执行会带来显著改善。

### MCPorter：MCP 工具组合的运行时

[MCPorter](https://github.com/steipete/mcporter) 是一个 TypeScript 运行时和 CLI 工具包，让调用 MCP 服务器变得实用而无需样板代码，同时通过选择性工具暴露和类型化封装来减少上下文膨胀。

**它解决的问题：** 不再需要一开始把所有 MCP 服务器的所有工具定义都加载进来，MCPorter 允许你按需发现、查看并调用具体工具，从而保持上下文轻量。

**关键功能：**

| 功能 | 说明 |
|---------|-------------|
| **零配置发现** | 自动发现来自 Cursor、Claude、Codex 或本地配置中的 MCP 服务器 |
| **类型化工具客户端** | `mcporter emit-ts` 生成 `.d.ts` 接口和可直接运行的封装 |
| **可组合 API** | `createServerProxy()` 把工具暴露为 camelCase 方法，并带有 `.text()`、`.json()`、`.markdown()` 辅助方法 |
| **CLI 生成** | `mcporter generate-cli` 可把任意 MCP 服务器转成独立 CLI，并支持 `--include-tools` / `--exclude-tools` 过滤 |
| **参数隐藏** | 可选参数默认隐藏，减少 schema 文字冗余 |

**安装：**

```bash
npx mcporter list          # 无需安装 - 立即发现服务器
pnpm add mcporter          # 添加到项目中
brew install steipete/tap/mcporter  # 通过 Homebrew 安装到 macOS
```

**示例 - 使用 TypeScript 组合工具：**

```typescript
import { createRuntime, createServerProxy } from "mcporter";

const runtime = await createRuntime();
const gdrive = createServerProxy(runtime, "google-drive");
const salesforce = createServerProxy(runtime, "salesforce");

// 数据在工具之间流动，不会经过模型上下文
const doc = await gdrive.getDocument({ documentId: "abc123" });
await salesforce.updateRecord({
  objectType: "SalesMeeting",
  recordId: "00Q5f000001abcXYZ",
  data: { Notes: doc.text() }
});
```

**示例 - CLI 调用工具：**

```bash
# 直接调用某个工具
npx mcporter call linear.create_comment issueId:ENG-123 body:'Looks good!'

# 列出可用的服务器和工具
npx mcporter list
```

MCPorter 通过提供以类型化 API 方式调用 MCP 工具的运行时基础设施，与上面描述的代码执行方案互补，使得把中间数据留在模型上下文之外变得更容易。

## 最佳实践

### 安全注意事项

#### 应该做的 ✅
- 所有凭据都使用环境变量
- 定期轮换 token 和 API key（建议每月）
- 尽可能使用只读 token
- 将 MCP 服务器的访问范围限制到最小
- 监控 MCP 服务器的使用和访问日志
- 外部服务可用时使用 OAuth
- 对 MCP 请求实施 rate limiting
- 在生产使用前先测试 MCP 连接
- 记录所有活跃的 MCP 连接
- 保持 MCP 服务器包更新

#### 不应该做的 ❌
- 不要在配置文件里硬编码凭据
- 不要把 token 或 secret 提交到 git
- 不要在团队聊天或邮件里分享 token
- 不要在团队项目中使用个人 token
- 不要授予不必要的权限
- 不要忽略认证错误
- 不要把 MCP 端点公开暴露
- 不要以 root/admin 权限运行 MCP server
- 不要把敏感数据缓存到日志里
- 不要禁用认证机制

### 配置最佳实践

1. **版本控制**：把 `.mcp.json` 放进 git，但 secret 用环境变量
2. **最小权限**：给每个 MCP server 只授予必要的最少权限
3. **隔离**：尽量把不同的 MCP 服务器运行在独立进程中
4. **监控**：记录所有 MCP 请求和错误，用于审计
5. **测试**：在部署到生产前测试所有 MCP 配置

### 性能建议

- 在应用层缓存高频访问的数据
- 使用更具体的 MCP 查询以减少数据传输
- 监控 MCP 操作的响应时间
- 面对外部 API 时考虑 rate limiting
- 在执行多项操作时使用 batching

## 安装说明

### 前置条件
- 已安装 Node.js 和 npm
- 已安装 Claude Code CLI
- 已有外部服务的 API tokens/凭据

### 分步设置

1. **通过 CLI 添加第一个 MCP 服务器**（示例：GitHub）：
```bash
claude mcp add --transport stdio github -- npx @modelcontextprotocol/server-github
```

   或者在项目根目录创建一个 `.mcp.json` 文件：
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

2. **设置环境变量：**
```bash
export GITHUB_TOKEN="your_github_personal_access_token"
```

3. **测试连接：**
```bash
claude /mcp
```

4. **使用 MCP tools：**
```bash
/mcp__github__list_prs
/mcp__github__create_issue "Title" "Description"
```

### 特定服务的安装

**GitHub MCP：**
```bash
npm install -g @modelcontextprotocol/server-github
```

**Database MCP：**
```bash
npm install -g @modelcontextprotocol/server-database
```

**Filesystem MCP：**
```bash
npm install -g @modelcontextprotocol/server-filesystem
```

**Slack MCP：**
```bash
npm install -g @modelcontextprotocol/server-slack
```

## 故障排查

### 找不到 MCP 服务器
```bash
# 验证 MCP server 是否已安装
npm list -g @modelcontextprotocol/server-github

# 如果缺失则安装
npm install -g @modelcontextprotocol/server-github
```

### 认证失败
```bash
# 验证环境变量已设置
echo $GITHUB_TOKEN

# 如有需要重新导出
export GITHUB_TOKEN="your_token"

# 验证 token 是否拥有正确权限
# 查看 GitHub token scopes：https://github.com/settings/tokens
```

### 连接超时
- 检查网络连通性：`ping api.github.com`
- 验证 API 端点可访问
- 检查 API 的 rate limit
- 尝试在配置中增加超时时间
- 检查防火墙或代理问题

### MCP 服务器崩溃
- 查看 MCP server 日志：`~/.claude/logs/`
- 验证所有环境变量都已设置
- 确保文件权限正确
- 尝试重新安装 MCP server 包
- 检查同一端口上是否有冲突进程

## 相关概念

### Memory vs MCP
- **Memory**：存储持久、不变的数据（偏好、上下文、历史）
- **MCP**：访问实时、变化的数据（API、数据库、实时服务）

### 什么时候用哪个
- **使用 Memory** 的场景：用户偏好、对话历史、已学习的上下文
- **使用 MCP** 的场景：当前 GitHub issues、实时数据库查询、实时数据

### 与其他 Claude 功能的集成
- 将 MCP 与 Memory 结合，获得更丰富的上下文
- 在提示词中使用 MCP 工具，提升推理质量
- 组合多个 MCP 来构建复杂工作流

## 额外资源

- [官方 MCP 文档](https://code.claude.com/docs/en/mcp)
- [MCP Protocol 规范](https://modelcontextprotocol.io/specification)
- [MCP GitHub 仓库](https://github.com/modelcontextprotocol/servers)
- [可用的 MCP 服务器](https://github.com/modelcontextprotocol/servers)
- [MCPorter](https://github.com/steipete/mcporter) — 用于调用 MCP 服务器的 TypeScript 运行时与 CLI，免去样板代码
- [Code Execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp) — Anthropic 关于解决上下文膨胀的工程博客
- [Claude Code CLI 参考](https://code.claude.com/docs/en/cli-reference)
- [Claude API 文档](https://docs.anthropic.com)
