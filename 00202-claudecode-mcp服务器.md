# Claude Code - MCP 服务器详解

> MCP (Model Context Protocol) 是 Claude Code 的插件协议，允许扩展 AI 的能力，连接外部工具和服务。

## 📋 目录

- [1. MCP 简介](#1-mcp-简介)
- [2. 常用 MCP 服务器](#2-常用-mcp-服务器)
- [3. 管理 MCP 服务器](#3-管理-mcp-服务器)
- [4. 开发自定义 MCP](#4-开发自定义-mcp)

---

## 1. MCP 简介

### 1.1 什么是 MCP？

**MCP (Model Context Protocol)** 是 Claude Code 的标准化插件协议，让 AI 能够访问外部工具和服务。

### 1.2 核心特性

| 特性 | 说明 |
|------|------|
| 🔌 **标准化协议** | 统一的插件接口标准 |
| 🛠️ **能力扩展** | 为 AI 添加文件系统、网络、数据库等能力 |
| 🌐 **服务隔离** | MCP 服务器独立运行，安全隔离 |
| 🔀 **双向通信** | 支持 SSE (Server-Sent Events) 实时通信 |
| 📦 **易于开发** | 提供 Python、Node.js、TypeScript SDK |

### 1.3 MCP vs 其他扩展方式

| 对比项 | MCP | Slash Commands | Skills |
|--------|-----|---------------|---------|
| **本质** | 外部工具 | 内置命令 | AI 工作流 |
| **能力** | 扩展 AI 能力 | 快速操作 | 复杂任务 |
| **通信** | 独立进程 | 直接执行 | 直接执行 |
| **示例** | 文件系统、数据库 | /commit | 代码审查 |

---

## 2. 常用 MCP 服务器

### 2.1 文件系统 MCP (mcp-filesystem)

#### 功能
访问和操作本地文件系统，支持批量文件操作。

#### 安装配置

编辑 `~/.claude/settings.json`:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/path/to/allowed/directory"
      ]
    }
  }
}
```

#### 使用场景
- 读取多个项目文件
- 批量文件操作
- 跨文件引用和搜索

#### 实际例子

```bash
# AI 可以读取指定目录下的所有文件
> 分析 src/ 目录下所有 Python 文件的结构

# 批量操作
> 将所有 .md 文件的标题层级提升一级
```

### 2.2 Git MCP (mcp-git)

#### 功能
Git 仓库操作和历史分析。

#### 安装配置

```json
{
  "mcpServers": {
    "git": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-git", "--repository", "."]
    }
  }
}
```

#### 使用场景
- 分析 Git 历史
- 查看提交差异
- 理解代码演变
- 追踪问题引入

#### 实际例子

```bash
# 分析提交历史
> 哪个提交引入了 authentication bug？

# 查看代码演变
> 展示 UserService 从创建到现在的所有改动
```

### 2.3 Brave Search MCP (mcp-brave-search)

#### 功能
网络搜索和实时信息获取。

#### 安装配置

```json
{
  "mcpServers": {
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "your-brave-api-key"
      }
    }
  }
}
```

#### 获取 API Key

访问: https://brave.com/search/api/

免费额度：每月 2,000 次调用

#### 使用场景
- 查询最新技术信息
- 搜索文档和教程
- 获取实时数据
- 验证过时信息

#### 实际例子

```bash
# 搜索最新文档
> 搜索 FastAPI 0.104 的新特性

# 查找解决方案
> 如何解决 "SSL: CERTIFICATE_VERIFY_FAILED" 错误？
```

### 2.4 GitHub MCP (mcp-github)

#### 功能
GitHub API 操作，管理仓库、PR、Issue。

#### 安装配置

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "your-github-token"
      }
    }
  }
}
```

#### 获取 GitHub Token

1. 访问 GitHub Settings → Developer settings → Personal access tokens
2. 生成新 token，选择所需权限：
   - `repo` (完整仓库访问)
   - `pull_requests` (PR 管理)
   - `issues` (Issue 管理)

#### 使用场景
- 创建和管理 Pull Requests
- 查询仓库信息
- 自动化 GitHub 工作流
- Issue 跟踪和分析

#### 实际例子

```bash
# PR 管理
> 创建 PR: 从 feature/auth 合并到 main
> 查看 PR #123 的评论

# 仓库操作
> 列出所有 open issues
> 分析仓库的活跃度
```

### 2.5 PostgreSQL MCP (mcp-postgres)

#### 功能
PostgreSQL 数据库查询和管理。

#### 安装配置

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "postgresql://user:password@localhost:5432/dbname"
      }
    }
  }
}
```

#### 连接字符串格式

```
postgresql://username:password@host:port/database
```

#### 使用场景
- 数据库查询优化
- Schema 分析和设计
- 自动生成 SQL
- 数据迁移辅助

#### 实际例子

```bash
# 查询分析
> 找出 users 表中查询最慢的 10 个操作

# Schema 理解
> 解释这个数据库的表关系

# SQL 生成
> 生成查询：找出最近 7 天注册的用户
```

### 2.6 Puppeteer MCP (mcp-puppeteer)

#### 功能
浏览器自动化和网页操作。

#### 安装配置

```json
{
  "mcpServers": {
    "puppeteer": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-puppeteer"]
    }
  }
}
```

#### 使用场景
- 网页截图
- 表单自动填写
- UI 测试自动化
- 网页数据抓取
- 生成页面 PDF

#### 实际例子

```bash
# 网页操作
> 打开 https://example.com 并截图
> 填写登录表单并提交

# 数据抓取
> 抓取新闻网站的标题列表
```

### 2.7 SQLite MCP (mcp-sqlite)

#### 功能
SQLite 数据库操作。

#### 安装配置

```json
{
  "mcpServers": {
    "sqlite": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sqlite", "--db-path", "./database.db"]
    }
  }
}
```

#### 使用场景
- 本地数据库开发
- 移动应用数据库分析
- 轻量级数据存储
- 测试数据管理

### 2.8 Fetch MCP (mcp-fetch)

#### 功能
HTTP 请求和网页内容抓取。

#### 安装配置

```json
{
  "mcpServers": {
    "fetch": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-fetch"]
    }
  }
}
```

#### 使用场景
- 调用 REST API
- 网页内容抓取
- Webhook 测试
- API 集成测试

#### 实际例子

```bash
# API 调用
> GET https://api.github.com/users/octocat

# 数据抓取
> 获取 https://example.com 的 HTML 内容
```

### 2.9 其他有用 MCP

#### Memory MCP (mcp-memory)

```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

**功能**: 持久化记忆存储，跨会话保留信息。

#### Playwright MCP

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@executeautomation/playwright-mcp-server"]
    }
  }
}
```

**功能**: 比 Puppeteer 更强大的浏览器自动化。

---

## 3. 管理 MCP 服务器

### 3.1 快速安装 MCP

Claude Code 提供了命令行工具来快速安装 MCP 服务器，无需手动编辑配置文件。

#### 使用 claude mcp add 命令

**语法**:
```bash
claude mcp add <name> --scope <scope> [options] -- <command>
```

**参数说明**:
- `name`: MCP 服务器名称
- `--scope`: 作用域（`user` 全局 或 `project` 项目）
- `--`: 命令分隔符
- `command`: 启动 MCP 服务器的命令

#### 在用户全局作用域安装

**示例 1: 安装 GitHub MCP（全局）**

```bash
claude mcp add github --scope user -- npx -y @modelcontextprotocol/server-github
```

**说明**: 这个 MCP 会在所有项目中可用。

**示例 2: 安装 Git MCP（全局）**

```bash
claude mcp add git --scope user -- npx -y @modelcontextprotocol/server-git --repository .
```

**示例 3: 安装文件系统 MCP（全局）**

```bash
claude mcp add filesystem --scope user -- npx -y @modelcontextprotocol/server-filesystem /path/to/directory
```

#### 在项目作用域安装

**示例 1: 安装 Sentry MCP（项目）**

```bash
claude mcp add sentry --scope project --transport http https://mcp.sentry.dev/mcp
```

**说明**: 这个 MCP 仅在当前项目中可用。

**示例 2: 安装项目特定的数据库 MCP**

```bash
claude mcp add postgres --scope project -- npx -y @modelcontextprotocol/server-postgres
```

#### 带 API Key 的安装

对于需要 API Key 的 MCP，安装后需要手动配置环境变量：

```bash
# 1. 安装 MCP
claude mcp add brave-search --scope user -- npx -y @modelcontextprotocol/server-brave-search

# 2. 配置 API Key
export BRAVE_API_KEY="your-api-key-here"

# 3. 重启 Claude Code
claude
```

#### 查看已安装的 MCP

```bash
# 列出所有 MCP
claude mcp list

# 查看 MCP 详情
claude mcp show github
```

#### 删除 MCP

```bash
# 删除用户全局 MCP
claude mcp remove github --scope user

# 删除项目 MCP
claude mcp remove sentry --scope project
```

#### 命令对比

| 操作 | 手动配置 | CLI 命令 |
|------|----------|----------|
| **安装** | 编辑 settings.json | `claude mcp add` |
| **查看** | `cat ~/.claude/settings.json` | `claude mcp list` |
| **删除** | 手动删除配置块 | `claude mcp remove` |
| **验证** | 重启 Claude Code | 自动验证配置 |

#### CLI 命令优势

✅ **快速**: 无需编辑配置文件
✅ **验证**: 自动检查命令有效性
✅ **隔离**: 自动区分全局和项目配置
✅ **安全**: 自动设置正确的文件权限

#### 常用 MCP 快速安装命令

```bash
# GitHub
claude mcp add github --scope user -- npx -y @modelcontextprotocol/server-github

# Git
claude mcp add git --scope user -- npx -y @modelcontextprotocol/server-git --repository .

# 文件系统
claude mcp add filesystem --scope user -- npx -y @modelcontextprotocol/server-filesystem ~/projects

# Brave Search（需要 API Key）
claude mcp add brave-search --scope user -- npx -y @modelcontextprotocol/server-brave-search

# PostgreSQL
claude mcp add postgres --scope project -- npx -y @modelcontextprotocol/server-postgres

# SQLite
claude mcp add sqlite --scope project -- npx -y @modelcontextprotocol/server-sqlite --db-path ./database.db

# Puppeteer
claude mcp add puppeteer --scope user -- npx -y @modelcontextprotocol/server-puppeteer

# Fetch
claude mcp add fetch --scope user -- npx -y @modelcontextprotocol/server-fetch

# Memory
claude mcp add memory --scope user -- npx -y @modelcontextprotocol/server-memory
```

### 3.2 查看已安装的 MCP

```bash
# 在 Claude Code 中
/mcp

# 或查看配置文件
cat ~/.claude/settings.json
```

### 3.3 启用/禁用 MCP 服务器

编辑 `~/.claude/settings.json`:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path"]
      // "enabled": false  // 添加这行禁用
    }
    // 删除整个配置块 = 完全移除
  }
}
```

### 3.4 使用 CC-Switch 管理

```bash
# 打开 CC-Switch
cc-switch

# 点击 "MCP" 按钮
# 可视化添加、删除、启用/禁用 MCP 服务器
```

**CC-Switch 优势**:
- 可视化界面
- 一键安装常用 MCP
- 配置验证
- 快速启用/禁用

### 3.5 MCP 环境变量管理

#### 方式 1: 在配置文件中

```json
{
  "mcpServers": {
    "github": {
      "env": {
        "GITHUB_TOKEN": "ghp_xxx"
      }
    }
  }
}
```

#### 方式 2: 使用系统环境变量

```bash
# ~/.bashrc 或 ~/.zshrc
export BRAVE_API_KEY="xxx"
export GITHUB_TOKEN="xxx"

# Claude Code 会自动继承环境变量
```

#### 方式 3: 使用 .env 文件

```bash
# ~/.claude/.env
BRAVE_API_KEY=xxx
GITHUB_TOKEN=xxx
```

### 3.6 MCP 调试

#### 查看 MCP 日志

```bash
# Claude Code 日志
tail -f ~/.claude/logs/claude-code.log

# MCP 服务器日志
journalctl -u mcp-* -f  # Linux
```

#### 测试 MCP 连接

```bash
# 在 Claude Code 中
> 测试 filesystem MCP 是否工作

# 或手动测试
npx @modelcontextprotocol/server-filesystem --help
```

---

## 4. 开发自定义 MCP

### 4.1 Python MCP 服务器示例

创建 `my_mcp_server.py`:

```python
#!/usr/bin/env python3
"""
自定义 MCP 服务器示例
"""

from mcp.server import Server
from mcp.types import Tool, TextContent
import asyncio

app = Server("my-custom-mcp")

@app.list_tools()
async def list_tools() -> list[Tool]:
    """列出可用的工具"""
    return [
        Tool(
            name="greet",
            description="向指定的人问好",
            inputSchema={
                "type": "object",
                "properties": {
                    "name": {
                        "type": "string",
                        "description": "要问好的人的名字"
                    }
                },
                "required": ["name"]
            }
        ),
        Tool(
            name="calculate",
            description="执行简单的数学计算",
            inputSchema={
                "type": "object",
                "properties": {
                    "expression": {
                        "type": "string",
                        "description": "数学表达式，如 '2 + 2'"
                    }
                },
                "required": ["expression"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    """处理工具调用"""
    if name == "greet":
        message = f"你好，{arguments['name']}！很高兴见到你。"
        return [TextContent(type="text", text=message)]

    elif name == "calculate":
        try:
            # 注意：实际应用中应使用更安全的方式
            result = eval(arguments['expression'])
            message = f"计算结果：{arguments['expression']} = {result}"
            return [TextContent(type="text", text=message)]
        except Exception as e:
            return [TextContent(type="text", text=f"计算错误：{str(e)}")]

    raise ValueError(f"未知的工具：{name}")

async def main():
    """启动 MCP 服务器"""
    from mcp.server.stdio import stdio_server

    async with stdio_server() as (read_stream, write_stream):
        await app.run(
            read_stream,
            write_stream,
            app.create_initialization_options()
        )

if __name__ == "__main__":
    asyncio.run(main())
```

### 4.2 配置自定义 MCP

```json
{
  "mcpServers": {
    "my-custom": {
      "command": "python",
      "args": ["/absolute/path/to/my_mcp_server.py"]
    }
  }
}
```

### 4.3 Node.js MCP 服务器示例

创建 `my-mcp-server.js`:

```javascript
#!/usr/bin/env node

import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
  {
    name: "my-custom-mcp",
    version: "1.0.0",
  },
  {
    capabilities: {
      tools: {},
    },
  }
);

// 列出可用工具
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: "get_time",
        description: "获取当前时间",
        inputSchema: {
          type: "object",
          properties: {
            timezone: {
              type: "string",
              description: "时区，如 Asia/Shanghai",
            },
          },
        },
      },
      {
        name: "reverse_text",
        description: "反转文本",
        inputSchema: {
          type: "object",
          properties: {
            text: {
              type: "string",
              description: "要反转的文本",
            },
          },
          required: ["text"],
        },
      },
    ],
  };
});

// 处理工具调用
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  if (name === "get_time") {
    const timezone = args.timezone || "UTC";
    const time = new Date().toLocaleString("zh-CN", { timeZone: timezone });
    return {
      content: [
        {
          type: "text",
          text: `当前时间（${timezone}）：${time}`,
        },
      ],
    };
  }

  if (name === "reverse_text") {
    const reversed = args.text.split("").reverse().join("");
    return {
      content: [
        {
          type: "text",
          text: `反转结果：${reversed}`,
        },
      ],
    };
  }

  throw new Error(`未知工具：${name}`);
});

// 启动服务器
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
}

main().catch(console.error);
```

配置：

```json
{
  "mcpServers": {
    "my-custom": {
      "command": "node",
      "args": ["/absolute/path/to/my-mcp-server.js"]
    }
  }
}
```

### 4.4 MCP 开发最佳实践

#### 错误处理

```python
try:
    result = await some_operation()
    return [TextContent(type="text", text=str(result))]
except Exception as e:
    return [TextContent(type="text", text=f"错误：{str(e)}")]
```

#### 输入验证

```python
def validate_args(args, required_fields):
    """验证参数"""
    missing = [f for f in required_fields if f not in args]
    if missing:
        raise ValueError(f"缺少必需参数：{', '.join(missing)}")
```

#### 资源清理

```python
async with resource_manager():
    # 使用资源
    pass
# 自动清理
```

---

## 5. 相关资源

- **MCP 官方文档**: https://modelcontextprotocol.io/
- **MCP SDK 仓库**: https://github.com/modelcontextprotocol
- **MCP Servers 仓库**: https://github.com/modelcontextprotocol/servers
- **下一节**: [00203-claudecode-skills系统.md](./00203-claudecode-skills系统.md) - Skills 系统


