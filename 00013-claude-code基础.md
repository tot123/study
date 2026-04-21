# Claude Code 基础教程

> 本教程介绍 Claude Code 是什么、如何安装配置以及基础使用方法。

## 📋 目录

- [1. Claude Code 是什么](#1-claude-code-是什么)
- [2. 安装与配置](#2-安装与配置)
- [3. 基础使用](#3-基础使用)

---

## 1. Claude Code 是什么

**Claude Code** 是 Anthropic 推出的终端 AI 编码助手，核心理念是"住在终端里"。

### 1.1 核心特性

| 特性 | 说明 |
|------|------|
| 🎯 **终端原生** | 直接在命令行中使用，无需离开工作流 |
| 🧠 **上下文理解** | 自动读取项目结构，理解代码关系 |
| 🛠️ **全功能操作** | 编辑代码、执行命令、管理 Git |
| 🔌 **IDE 集成** | 支持 VSCode、JetBrains 等 IDE |
| 🤖 **多模型支持** | 官方 Claude 模型 + 第三方模型 |
| 🔌 **插件系统** | 支持 MCP 服务器和自定义 Skills |

### 1.2 为什么选择 Claude Code？

1. **无缝集成**：直接在终端工作，无需切换窗口
2. **项目感知**：理解整个项目结构，不是单文件分析
3. **可扩展性**：支持自定义命令、Skills、MCP 插件
4. **成本优化**：可接入第三方低成本或本地模型
5. **隐私保护**：支持本地模型，数据不上传

---

## 2. 安装与配置

### 2.1 安装方式

#### NPM 安装（推荐）

```bash
npm install -g @anthropic-ai/claude-code
```

#### 原生安装脚本

**macOS / Linux / WSL:**

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**Windows PowerShell:**

```powershell
irm https://claude.ai/install.ps1 | iex
```

**Windows CMD:**

```cmd
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

#### 验证安装

```bash
claude --version
# 或直接运行
claude
```

### 2.2 登录与授权

首次运行 `claude` 会提示登录，支持两种方式：

- **个人用户**: 使用 Claude.ai 账户交互式登录
- **企业/API 用户**: 使用 Anthropic API Key 授权

凭据保存在 `~/.config/claude`，后续可通过 `/login` 命令重新登录或切换账户。

---

## 3. 基础使用

### 3.1 启动 Claude Code

```bash
# 进入项目目录
cd /path/to/your/project

# 启动 Claude Code
claude
```

### 3.2 交互式对话

```bash
# 在 Claude Code 提示符下
> create utilities/logger.py with rotating file handler
> write unit tests for date parsing edge cases
> explain this function foo in module bar
> refactor module baz to async/await
```

Claude Code 会自动读取项目上下文，理解代码结构并执行操作。

### 3.3 项目指南 (CLAUDE.md)

创建 `CLAUDE.md` 文件为项目提供上下文：

```markdown
# Project Guide

This is a FastAPI backend project with PostgreSQL database.

## Key Directories
- `app/api/` - API endpoints
- `app/models/` - Database models
- `app/services/` - Business logic

## Important Patterns
- Use async/await for all database operations
- All endpoints return {code, message, data} format
- Use Pydantic for request/response validation
```

### 3.4 常用快捷命令

```bash
# 查看帮助
/help

# 清空会话
/clear

# 查看状态
/status

# 查看上下文使用情况
/context

# 查看成本
/cost

# 退出
/exit
```

---

## 4. 相关文档

- **下一节**: [00201-claudecode-slash-命令.md](./00201-claudecode-slash-命令.md) - Slash 命令详解
- [00202-claudecode-mcp服务器.md](./00202-claudecode-mcp服务器.md) - MCP 服务器配置
- [00203-claudecode-skills系统.md](./00203-claudecode-skills系统.md) - Skills 系统使用
