# Claude Code 完整使用指南

> 本指南整合了 Claude Code 的核心功能、命令系统、扩展能力、最佳实践等内容，帮助你从零开始掌握这款强大的终端 AI 编码助手。

## 📚 目录

- [一、Claude Code 简介](#一claude-code-简介)
- [二、安装与配置](#二安装与配置)
- [三、命令系统详解](#三命令系统详解)
- [四、第三方模型集成](#四第三方模型集成)
- [五、Agent Skills 系统](#五agent-skills-系统)
- [六、Subagents 系统](#六subagents-系统)
- [七、MCP 服务器](#七mcp-服务器)
- [八、实用插件推荐](#八实用插件推荐)
- [九、最佳实践](#九最佳实践)

---

## 一、Claude Code 简介

### 1.1 什么是 Claude Code

Claude Code 是 Anthropic 推出的**终端 AI 编码助手**，核心理念是"住在终端里"。它能够：

- 📖 理解项目结构和代码上下文
- ✏️ 直接编辑代码文件
- 🔧 执行终端命令
- 🔄 管理 Git 操作
- 🎯 提供 IDE 集成支持

**核心优势**：
- 命令行驱动，适合开发者工作流
- 深度理解代码库，提供精准建议
- 支持自定义命令和扩展
- 可集成多种 AI 模型降低成本

### 1.2 核心概念

| 概念 | 说明 |
|------|------|
| **Slash 命令** | 内置和自定义命令系统，快速执行特定操作 |
| **MCP** | Model Context Protocol，插件协议，连接外部系统 |
| **Skills** | 模块化能力包，打包专业知识和工作流 |
| **Subagents** | 子代理系统，多智能体协作处理复杂任务 |
| **Hooks** | 用户提示提交钩子，拦截和处理提示词 |
| **CCR** | Claude Code Router，智能模型路由工具 |

---

## 二、安装与配置

### 2.1 安装方式

#### NPM 安装（推荐）

```bash
npm install -g @anthropic-ai/claude-code
```

#### 原生安装脚本

**macOS / Linux / WSL**:
```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**Windows PowerShell**:
```powershell
irm https://claude.ai/install.ps1 | iex
```

**Windows CMD**:
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

### 2.3 基础使用

进入项目目录并启动：

```bash
cd /path/to/your/project
claude
```

交互模式下，直接用自然语言描述需求：

```
> create utilities/logger.py with rotating file handler
> write unit tests for date parsing edge cases
> explain this function foo in module bar
> refactor module baz to async/await
```

Claude Code 会自动读取项目上下文，理解代码结构并执行操作。

---

## 三、命令系统详解

### 3.1 符号命令

| 符号 | 说明 | 示例 |
|------|------|------|
| `!` | bash 模式 | `!ls -la` |
| `/` | 命令模式 | `/help` |
| `@` | 引用文件路径 | `@src/main.ts` |
| `&` | 后台运行 | `&npm test` |

### 3.2 快捷键

| 快捷键 | 说明 |
|--------|------|
| `Esc + Esc` | 清除所有输入内容 |
| `Shift + Tab` | 自动接收编辑 |
| `Ctrl + O` | 查看详细输出 |
| `Shift + Enter` | 换行 |
| `Ctrl + _` | 撤销 |
| `Ctrl + Z` | 暂停 |
| `Cmd + V` | 粘贴图片（macOS） |
| `Ctrl + S` | 暂存提示词 |

### 3.3 常用 Slash 命令

#### 会话管理

| 命令 | 说明 |
|------|------|
| `/help` | 显示帮助和可用命令 |
| `/clear` | 清除历史会话，释放上下文 |
| `/exit` | 退出 REPL |
| `/compact [instructions]` | 清除对话历史但保留摘要，可选精简指令 |
| `/export [filename]` | 导出当前会话到文件或剪贴板 |
| `/resume` | 恢复对话 |

#### 状态查看

| 命令 | 说明 |
|------|------|
| `/status` | 查看状态（版本、模型、账户、API 连接） |
| `/context` | 将上下文占用可视化为彩色网格 |
| `/cost` | 显示当前会话的总开销 |
| `/usage` | 显示计划使用限额 |
| `/stats` | 查看使用数据和活动记录 |

#### 配置管理

| 命令 | 说明 |
|------|------|
| `/config` | 打开配置面板 |
| `/model` | 设置 AI 模型 |
| `/mcp` | 管理 MCP 服务器 |
| `/permissions` | 管理工具权限规则 |
| `/hooks` | 管理工具事件钩子配置 |

#### 功能操作

| 命令 | 说明 |
|------|------|
| `/add-dir` | 添加工作目录 |
| `/memory` | 编辑 Claude 记忆文件（CLAUDE.md） |
| `/agents` | 管理智能体配置 |
| `/skills` | 列举可用技能 |
| `/todos` | 列举当前待办项 |
| `/init` | 用 CLAUDE.md 指南初始化项目 |
| `/review` | 审查拉取请求 |
| `/security-review` | 对当前分支完成安全审核 |

#### 高级功能

| 命令 | 说明 |
|------|------|
| `/plan` | 启用规划模式或查看当前会话规划 |
| `/plugin` | 管理 Claude Code 插件 |
| `/tasks` | 列出并管理后台任务 |
| `/rewind` | 将代码和/或对话恢复至之前的节点 |
| `/fork` | 在此节点复刻当前对话的分支版本 |
| `/doctor` | 诊断并验证安装及设置情况 |

### 3.4 自定义命令

自定义命令通过 Markdown 文件定义，支持参数、bash 执行、文件引用和权限控制。

#### 命令类型

- **项目命令 (project)**: 位于 `.claude/commands/`，团队共享
- **个人命令 (user)**: 位于 `~/.claude/commands/`，所有项目可用
- **插件命令**: 随插件分发，使用 `/plugin-name:command` 命名空间
- **MCP 命令**: 由 MCP 服务器动态发现，格式 `/mcp__<server>__<prompt>`

#### 基本结构

```markdown
---
name: commit
description: 创建 git commit，遵循约定式提交规范
argument-hint: <commit-type> <description>
allowed-tools: ["bash"]
---

# Git Commit 命令

使用此命令创建规范的 git commit。

## 参数

- $ARGUMENTS: 完整的提交消息字符串

## 执行步骤

1. 验证 git 仓库状态
2. 添加所有更改的文件
3. 使用提供的消息创建提交
4. 显示提交结果

!git add -A
!git commit -m "$ARGUMENTS"
!git log -1 --stat
```

#### Frontmatter 字段

| 字段 | 必需 | 说明 |
|------|------|------|
| `name` | ✅ | 命令唯一标识符，小写字母、数字、连字符 |
| `description` | ✅ | 功能描述，Claude 决定是否使用的关键 |
| `argument-hint` | ❌ | 参数提示文本 |
| `allowed-tools` | ❌ | 限制命令可用的工具 |
| `model` | ❌ | 指定使用的模型 |
| `disable-model-invocation` | ❌ | 禁用模型调用 |

---

## 四、第三方模型集成

### 4.1 为什么需要第三方模型

直接使用官方模型存在以下问题：

- **成本高**: 按 token 计费，长期使用费用显著
- **额度限制**: 每日/每周调用限制
- **灵活性差**: 无法使用 DeepSeek、本地 LLM 等第三方模型

**解决方案**: 通过环境变量或路由工具接入第三方模型，降低成本 70-90%。

### 4.2 快速接入：兼容模型直连

#### DeepSeek 集成（推荐）

**Bash/Zsh 配置**（添加到 `~/.bashrc` 或 `~/.zshrc`）：

```bash
export ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
export ANTHROPIC_AUTH_TOKEN=your-deepseek-api-key
export API_TIMEOUT_MS=600000
export ANTHROPIC_MODEL=deepseek-chat
```

**Fish 配置**（添加到 `~/.config/fish/config.fish`）：

```fish
set -gx ANTHROPIC_BASE_URL https://api.deepseek.com/anthropic
set -gx ANTHROPIC_AUTH_TOKEN your-deepseek-api-key
set -gx API_TIMEOUT_MS 600000
set -gx ANTHROPIC_MODEL deepseek-chat
```

#### 智谱 GLM 集成

**自动配置（推荐）**：

```bash
curl -fsSL "https://cdn.bigmodel.cn/install/claude_code_env.sh" | bash
```

**手动配置**（编辑 `~/.claude/settings.json`）：

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "your_zhipu_api_key",
    "ANTHROPIC_BASE_URL": "https://open.bigmodel.cn/api/anthropic/",
    "API_TIMEOUT_MS": "3000000",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "glm-4.5-air",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "glm-4.6",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "glm-4.6"
  }
}
```

### 4.3 进阶方案：智能路由工具

#### Claude Code Router (CCR)

**CCR** 是命令行路由解决方案，提供：

- 多 Provider 统一管理
- 灵活的路由规则引擎
- 请求/响应转换器
- 交互式模型切换
- 完整的日志与统计系统

**安装**：

```bash
npm install -g @musistudio/claude-code-router
```

**配置文件示例**（`~/.claude-code-router/config.json`）：

```json
{
  "APIKEY": "your-secret-key",
  "PROXY_URL": "http://127.0.0.1:7890",
  "LOG": true,
  "API_TIMEOUT_MS": 600000,
  "Providers": [
    {
      "name": "deepseek",
      "api_base_url": "https://api.deepseek.com/chat/completions",
      "api_key": "sk-xxx",
      "models": ["deepseek-chat", "deepseek-reasoner"]
    },
    {
      "name": "ollama",
      "api_base_url": "http://localhost:11434/v1/chat/completions",
      "api_key": "ollama",
      "models": ["qwen2.5-coder:latest"]
    }
  ],
  "Router": {
    "default": "deepseek,deepseek-chat",
    "background": "ollama,qwen2.5-coder:latest",
    "think": "deepseek,deepseek-reasoner"
  }
}
```

**使用**：

```bash
# 启动 Claude Code
ccr code

# 查看配置
ccr model

# 重启服务
ccr restart
```

#### CC-Switch

**CC-Switch** 是跨平台桌面应用，提供可视化的配置管理。

**核心功能**：

- Provider 一键切换和速度测试
- MCP 服务器统一管理
- Skills 仓库扫描和安装
- Prompts 多预设管理
- 系统托盘快速切换

**安装**（macOS Homebrew）：

```bash
brew tap farion1231/ccswitch
brew install --cask cc-switch
```

---

## 五、Agent Skills 系统

### 5.1 什么是 Agent Skills

Agent Skills 是基于文件系统的**可复用能力包**，将特定操作流程和知识编码为模块化资源，能够按需加载。

**核心优势**：

- 📦 **专业化**: 为特定领域任务定制能力
- 🔄 **减少重复**: 创建一次，自动使用
- 🧩 **组合能力**: 组合多个 Skills 构建复杂工作流

### 5.2 Skills vs Prompt vs MCP

| 维度 | Prompt | Skills | MCP |
|------|--------|--------|-----|
| **作用范围** | 会话级别的一次性指令 | 持久化的可复用能力 | 外部系统连接 |
| **生命周期** | 单次对话 | 跨会话、跨项目 | 持久连接 |
| **加载方式** | 全量加载到上下文 | 渐进式按需加载 | 即时调用 |
| **适用场景** | 临时性、一次性任务 | 重复性、专业化任务 | 访问外部系统 |

**协同关系**：
- **MCP** 提供基础设施（连接外部系统）
- **Skills** 提供专业知识（如何使用这些连接）
- **Prompt** 提供具体任务（实际要做什么）

### 5.3 渐进式加载机制

Skills 采用**三层内容加载模型**：

#### Level 1: 元数据（始终加载）

**加载时机**: Claude 启动时
**内容类型**: 轻量级的发现信息

```markdown
---
name: react-component-dev
description: 开发 React 组件，包含 TypeScript 类型定义、Hooks 使用、性能优化、测试等。当创建或审查 React 组件时使用。
---
```

每个 Skill 只占用约 50 tokens，可以安装数十个而无上下文惩罚。

#### Level 2: 主要指令（触发时加载）

**加载时机**: 用户请求匹配 Skill 描述时
**内容类型**: 程序性知识、工作流、最佳实践

```markdown
# React 组件开发技能

## 快速开始

使用函数组件和 TypeScript：

\`\`\`typescript
import React from 'react'

interface Props {
  title: string
}

export const MyComponent: React.FC<Props> = ({ title }) => {
  return <div>{title}</div>
}
\`\`\`
```

#### Level 3: 资源和代码（按需加载）

**加载时机**: 被 SKILL.md 引用时
**内容类型**: 详细文档、可执行脚本、参考材料

**目录结构**：

```
react-component-dev/
├── SKILL.md              # 主要指令
├── HOOKS_GUIDE.md        # Hooks 使用指南
├── TESTING_GUIDE.md      # 测试指南
└── scripts/
    └── lint.sh           # 代码检查脚本
```

**关键特性**：
- Claude 仅在被引用时访问这些文件
- 脚本执行时，代码本身不进入上下文
- 未使用的文件消耗零 token

### 5.4 创建自定义 Skill

#### 基本结构

```markdown
---
name: my-skill-name
description: 清晰描述这个技能的功能和使用场景（最大 1024 字符）
allowed-tools: ["bash", "read"]
---

# 技能名称

[添加 Claude 激活此技能时应遵循的指令]

## 快速开始

[核心工作流]

## 示例

- 示例用法 1
- 示例用法 2

## 指导原则

- 原则 1
- 原则 2
```

#### 最佳实践

1. **保持简洁**: SKILL.md 主文件保持在 5,000 字以内
2. **渐进式披露**: 在 SKILL.md 中提供概览，详细内容外部化
3. **利用脚本**: 将确定性操作实现为脚本，节省 token
4. **清晰描述**: description 字段应包含功能、技术栈、触发关键词
5. **模块化设计**: 每个 Skill 专注于单一职责

### 5.5 四种部署方式

| 部署方式 | 共享范围 | 适用场景 | 管理方式 |
|---------|----------|----------|----------|
| **Claude API** | 组织级 | 生产环境 | 集中管理 |
| **Claude Code** | 本地 | 开发测试 | 文件系统 |
| **Agent SDK** | 项目级 | 应用集成 | 配置文件 |
| **Claude.ai** | 用户级 | 个人使用 | Web 界面 |

---

## 六、Subagents 系统

### 6.1 什么是 Subagents

**Subagents（子代理）** 是 Claude Code 的多智能体协作系统，允许你创建专门的 AI 代理来处理特定类型的任务。每个 subagent 可以：

- 🎯 拥有独立的技能配置
- 📦 访问特定的工具和资源
- 🔄 并行或串行执行任务
- 📊 跟踪自己的进度和状态

**核心优势**：
- **任务专业化**: 不同 subagent 专注不同领域
- **并行处理**: 多个 subagent 同时工作
- **状态隔离**: 每个 subagent 独立跟踪状态
- **灵活组合**: 可以创建复杂的工作流

### 6.2 Subagents vs Skills

| 维度 | Skills | Subagents |
|------|--------|-----------|
| **本质** | 知识和能力包 | 独立的执行代理 |
| **状态** | 无状态 | 有状态（跟踪进度） |
| **执行** | 主代理调用 | 独立执行任务 |
| **协作** | 组合使用 | 可以互相调用 |
| **适用场景** | 打包专业知识 | 复杂多步骤任务 |

**协同使用**：
- **Skills** 提供 subagent 需要的专业知识
- **Subagents** 使用这些知识独立完成复杂任务

### 6.3 使用场景

#### 1. 并行任务处理

```
主代理
  ├─ Subagent A: 处理前端代码
  ├─ Subagent B: 处理后端 API
  └─ Subagent C: 编写测试用例
```

**示例**: 全栈功能开发

```
> 创建一个用户管理功能，包含前端界面和后端 API

# 主代理会创建 3 个 subagents:
- frontend-dev: 创建 React 组件和表单
- backend-dev: 创建 Go API 和数据库模型
- test-writer: 编写单元测试和集成测试
```

#### 2. 专业化任务分工

```
主代理
  ├─ Code Reviewer: 审查代码质量
  ├─ Security Expert: 检查安全问题
  └─ Test Specialist: 验证测试覆盖
```

**示例**: 全面代码审查

```
> 审查当前分支的所有更改

# 主代理会创建专业化的 subagents:
- code-reviewer: 检查代码风格和最佳实践
- security-auditor: 扫描安全漏洞
- coverage-analyzer: 验证测试覆盖率
```

#### 3. 长期运行任务

```
主代理
  └─ Research Agent: 持续监控和收集信息
```

**示例**: 代码库分析和优化建议

```
> 分析整个代码库的性能瓶颈

# 创建一个长期运行的 subagent:
- performance-analyzer: 扫描代码库，识别性能问题，生成报告
```

### 6.4 创建和管理 Subagents

#### 使用 `/agents` 命令

```bash
# 列出所有可用的 subagents
/agents

# 创建新的 subagent
/agents create

# 配置 subagent 属性
/agents config <agent-name>

# 删除 subagent
/agents delete <agent-name>
```

#### Subagent 配置

创建 subagent 时可以配置：

| 配置项 | 说明 | 示例 |
|--------|------|------|
| **name** | Subagent 名称 | `frontend-dev` |
| **description** | 功能描述 | "专门处理前端开发任务" |
| **skills** | 使用的技能 | `["react-dev", "typescript"]` |
| **tools** | 可用工具 | `["read", "write", "bash"]` |
| **model** | 使用的模型 | `deepseek-chat` |
| **permissions** | 权限级别 | `default` / `bypass` |

#### 配置文件示例

Subagent 配置通常存储在 `.claude/agents/` 目录：

```yaml
# .claude/agents/frontend-dev.yaml
name: frontend-dev
description: 专门处理 React/TypeScript 前端开发
model: deepseek-chat
skills:
  - react-component-dev
  - typescript-best-practices
  - testing-library
tools:
  - read
  - write
  - bash
permissions: default
```

### 6.5 Subagent 工作流程

#### 1. 自动创建

Claude Code 会根据任务自动创建合适的 subagents：

```
> 重构用户认证系统

# Claude 自动分析任务，创建:
- auth-expert: 处理认证逻辑
- db-migrator: 处理数据库迁移
- test-writer: 编写认证测试
```

#### 2. 手动创建

使用 `/agents` 命令手动创建专用 subagent：

```
> /agents create
What type of agent? code-reviewer
What skills should it use? security, performance
> 完成
```

#### 3. Subagent 协作

Subagents 可以相互协作：

```
主代理
  ↓ 分配任务
Subagent A (分析)
  ↓ 传递结果
Subagent B (实现)
  ↓ 传递结果
Subagent C (测试)
  ↓ 汇总结果
主代理 (整合)
```

### 6.6 最佳实践

#### 1. 合理划分职责

✅ **好的做法**：
```yaml
# 每个 subagent 专注一个领域
- frontend-dev: 前端开发
- backend-dev: 后端开发
- devops-engineer: DevOps 任务
```

❌ **避免**：
```yaml
# subagent 职责过于宽泛
- fullstack-dev: 做所有事情
```

#### 2. 明确定义接口

确保 subagents 之间的交互清晰：

```
# Subagent A 输出 → Subagent B 输入
code-analyzer (输出问题列表)
  ↓
refactoring-agent (接收问题，执行重构)
```

#### 3. 使用合适的模型

为不同任务配置不同模型：

| 任务类型 | 推荐模型 | 原因 |
|---------|---------|------|
| 简单代码生成 | Haiku / DeepSeek | 快速、便宜 |
| 复杂重构 | Sonnet / DeepSeek-R1 | 平衡质量 |
| 安全审查 | Opus / Claude | 最高质量 |

#### 4. 监控和调试

使用命令监控 subagent 状态：

```bash
# 查看所有 subagents 状态
/agents

# 查看特定 subagent 详情
/agents status <agent-name>

# 查看 subagent 日志
/agents logs <agent-name>
```

### 6.7 实际案例

#### 案例 1: 微服务开发

**场景**: 开发一个新的用户服务

```
> 创建一个用户微服务，包含认证、Profile 管理和通知功能

# Claude 自动创建 subagents:
1. service-architect: 设计服务架构
2. api-developer: 实现 RESTful API
3. database-designer: 设计数据模型
4. test-engineer: 编写测试用例
5. doc-writer: 生成 API 文档

# 工作流程:
architect → developer → database → test → doc → (验证) → 完成
```

#### 案例 2: 代码库迁移

**场景**: 将代码库从 JavaScript 迁移到 TypeScript

```
> 将这个项目从 JS 迁移到 TypeScript

# Claude 创建专业化 subagents:
1. type-analyzer: 分析代码并推断类型
2. migration-specialist: 执行迁移
3. test-validator: 验证迁移后功能正确
4. doc-updater: 更新类型文档

# 并行执行:
analyzer (扫描整个代码库)
  ↓
migration-specialist (逐文件迁移)
  ↓
test-validator (运行测试)
  ↓
doc-updater (更新文档)
```

#### 案例 3: 性能优化

**场景**: 优化应用性能

```
> 分析并优化这个应用的性能瓶颈

# 创建专门的 subagents:
1. profiler: 性能分析和瓶颈识别
2. optimizer: 实施优化方案
3. benchmark: 性能基准测试
4. validator: 验证优化效果

# 迭代流程:
profiler (识别问题) → optimizer (实施优化) → benchmark (测试) → validator (验证)
  ↑                                                                    ↓
  └─────────────────────── 未达标，继续优化 ←──────────────────────────┘
```

### 6.8 高级技巧

#### 1. 条件性 Subagent 创建

根据任务复杂度决定是否创建 subagent：

```
# 简单任务: 主代理直接处理
> 修复这个拼写错误
# 主代理直接修复，不创建 subagent

# 复杂任务: 创建 subagents
> 重构整个认证系统
# 创建多个专业化 subagents 处理
```

#### 2. Subagent 池

维护常用 subagent 配置：

```yaml
# .claude/agents/pool.yaml
available-agents:
  - name: frontend-dev
    skills: [react, vue, angular]
    ready: true

  - name: backend-dev
    skills: [go, python, java]
    ready: true

  - name: security-auditor
    skills: [security, owasp]
    ready: true
```

#### 3. Subagent 链式调用

创建 subagent 处理流水线：

```
input → parser → processor → validator → output
         (SA1)     (SA2)       (SA3)
```

每个 subagent 处理数据后传递给下一个。

---

## 七、MCP 服务器

### 7.1 什么是 MCP

**MCP (Model Context Protocol)** 是一个开放标准，提供 AI 访问外部系统的标准化接口，类似于"USB-C for LLMs"。

**核心特性**：
- Client-Server 架构
- 标准化连接方式
- 支持文件系统、GitHub、数据库等外部资源

### 7.2 CLI 快速安装 MCP

使用 `claude mcp` 命令快速安装常用 MCP：

```bash
# GitHub（全局）
claude mcp add github --scope user -- npx -y @modelcontextprotocol/server-github

# Git（全局）
claude mcp add git --scope user -- npx -y @modelcontextprotocol/server-git --repository .

# 文件系统（全局）
claude mcp add filesystem --scope user -- npx -y @modelcontextprotocol/server-filesystem ~/projects

# Brave Search（需要 API Key）
claude mcp add brave-search --scope user -- npx -y @modelcontextprotocol/server-brave-search

# PostgreSQL（项目级）
claude mcp add postgres --scope project -- npx -y @modelcontextprotocol/server-postgres

# SQLite（项目级）
claude mcp add sqlite --scope project -- npx -y @modelcontextprotocol/server-sqlite --db-path ./database.db

# Puppeteer（全局）
claude mcp add puppeteer --scope user -- npx -y @modelcontextprotocol/server-puppeteer

# Fetch（全局）
claude mcp add fetch --scope user -- npx -y @modelcontextprotocol/server-fetch

# Memory（全局）
claude mcp add memory --scope user -- npx -y @modelcontextprotocol/server-memory

# 查看已安装的 MCP
claude mcp list

# 移除 MCP
claude mcp remove github --scope user
```

### 7.3 常用 MCP 服务器

#### GitHub MCP

提供访问 GitHub API 的能力：

- 读取仓库内容
- 查看提交历史
- 管理 Pull Request
- 查看和创建 Issues

#### Git MCP

提供 Git 版本控制操作：

- 查看状态和日志
- 创建和切换分支
- 提交和推送更改

#### Filesystem MCP

提供文件系统访问：

- 读取和编辑文件
- 目录遍历
- 文件搜索

#### Database MCP（PostgreSQL/SQLite）

提供数据库操作：

- 执行 SQL 查询
- 读取表结构
- 数据导入导出

#### Puppeteer MCP

提供浏览器自动化：

- 网页截图
- 表单填写
- 点击和导航
- 数据抓取

---

## 八、实用插件推荐

### 8.1 Prompt Improver ⭐ 强烈推荐

**功能**: 自动评估和改进模糊提示词，减少 token 浪费和迭代疲劳。

**工作流程**：
1. 拦截你的提示词
2. 评估清晰度
3. 对于模糊提示词，研究代码库
4. 生成 1-6 个针对性问题
5. 基于答案执行任务

**安装**：

```bash
# 添加 marketplace
claude plugin marketplace add severity1/severity1-marketplace

# 安装插件
claude plugin install prompt-improver@severity1-marketplace

# 重启 Claude Code
```

**使用**：

```
> fix the bug           # Hook 评估，可能问问题
> add tests             # Hook 评估，可能问问题
> * add dark mode       # * = 跳过评估
> /help                 # / = 斜杠命令绕过
```

**适合场景**：
- ✅ 经常写模糊提示词
- ✅ 关注 token 消耗
- ✅ 在大型代码库工作
- ✅ 想要更高的一次成功率

**GitHub**: https://github.com/severity1/claude-code-prompt-improver

### 8.2 其他实用插件

| 插件 | 功能 | GitHub |
|------|------|--------|
| claude-code-auto-memory | 自动维护 CLAUDE.md 文件 | [severity1](https://github.com/severity1/claude-code-auto-memory) |
| terraform-cloud-mcp | 连接 AI 与 Terraform Cloud | [severity1](https://github.com/severity1/terraform-cloud-mcp) |
| argocd-mcp | ArgoCD 基础设施管理 | [severity1](https://github.com/severity1/argocd-mcp) |

---

## 九、最佳实践

### 9.1 成本优化策略

通过任务分层和智能路由，可节省 70-90% 成本：

| 策略 | 成本节省 | 说明 |
|------|----------|------|
| 使用本地模型 | 100% | Ollama 等本地 LLM |
| 使用 DeepSeek | 80-90% | 兼容 API，成本极低 |
| 智能路由组合 | 70-90% | CCR 根据任务类型自动选择 |

**路由策略示例**：

- 简单任务 → 本地模型（免费）
- 编程任务 → DeepSeek-Coder（便宜）
- 复杂推理 → DeepSeek-R1（高质量）
- 关键任务 → Claude Opus（最佳质量）

### 9.2 提示词优化

1. **使用 Prompt Improver**: 自动改进模糊提示词
2. **提供上下文**: 明确文件路径、函数名、行号
3. **描述意图**: 说明想要达到什么效果
4. **分步执行**: 复杂任务分解为多个步骤

### 9.3 安全建议

1. **API Key 管理**: 使用环境变量，避免明文存储
2. **权限控制**: 合理设置 `/permissions` 规则
3. **代码审查**: 对生成的代码进行审查
4. **备份重要文件**: 使用 Git 追踪更改

### 9.4 工作流程建议

#### 新项目启动

```bash
# 1. 初始化项目
cd /path/to/project
claude

# 2. 初始化 CLAUDE.md
/init

# 3. 配置 MCP
/mcp

# 4. 安装必要插件
/plugin
```

#### 日常开发流程

1. **明确需求**: 使用具体提示词或依赖 Prompt Improver
2. **执行任务**: 让 Claude Code 自动完成
3. **审查结果**: 检查生成的代码和更改
4. **迭代优化**: 根据反馈调整
5. **提交代码**: 使用自定义 commit 命令

#### 团队协作

1. **共享项目命令**: 在 `.claude/commands/` 创建团队命令
2. **共享 Skills**: 开发符合团队规范的 Skills
3. **统一 MCP 配置**: 在项目级配置必需的 MCP
4. **代码审查流程**: 使用 `/review` 命令标准化审查

### 9.5 故障排除

#### 常见问题

| 问题 | 解决方案 |
|------|----------|
| 安装失败 | 使用 `/doctor` 诊断环境 |
| 模型调用失败 | 检查 API Key 和网络连接 |
| 上下文不足 | 使用 `/compact` 压缩会话 |
| 权限错误 | 检查 `/permissions` 设置 |
| MCP 不工作 | 使用 `/mcp` 检查连接状态 |

#### 调试技巧

```bash
# 查看详细状态
/status

# 查看上下文使用
/context

# 查看成本统计
/cost

# 查看使用统计
/stats

# 诊断环境
/doctor
```

---

## 九、参考资源

### 官方资源

- 📚 [Claude Code 官方文档](https://docs.anthropic.com/claude-code)
- 🔌 [MCP 官方文档](https://modelcontextprotocol.io/)
- 🎯 [Claude Agent Skills 文档](https://docs.anthropic.com/claude-code/skills)

### 社区资源

- 💻 [Claude Code Router](https://github.com/musistudio/claude-code-router)
- 🖥️ [CC-Switch](https://github.com/farion1231/cc-switch)
- 📦 [MCP Servers](https://github.com/modelcontextprotocol/servers)
- 🎯 [Claude Code Skills](https://github.com/anthropics/claude-code-skills)

### 第三方模型

- 🤖 [DeepSeek](https://platform.deepseek.com/)
- 🌐 [智谱 GLM](https://open.bigmodel.cn/)
- 🦙 [Ollama](https://ollama.com/)
- 🔀 [OpenRouter](https://openrouter.ai/)

---

## 附录：快速参考

### 环境变量速查

```bash
# API 配置
export ANTHROPIC_AUTH_TOKEN=sk-xxxxx
export ANTHROPIC_BASE_URL=https://api.anthropic.com
export API_TIMEOUT_MS=600000

# 模型配置
export ANTHROPIC_MODEL=claude-sonnet-4
export ANTHROPIC_SMALL_FAST_MODEL=claude-haiku-4

# 功能配置
export CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1
```

### 配置文件位置

```bash
~/.claude/
├── settings.json           # Claude Code 配置
├── commands/               # 自定义命令
│   └── *.md
├── skills/                 # Skills
│   └── *.md
└── .env                    # 环境变量（不提交）

~/.claude-code-router/
├── config.json             # CCR 配置
└── logs/                   # CCR 日志
```




## 配置优化：让 Claude 记住你的习惯

###  技巧 1：CLAUDE.md - 让 Claude 记住你的项目
**问题：**每次打开 Claude Code，都要重复说明项目结构、常用命令、代码风格。  
**解决方案：**在项目根目录创建 CLAUDE.md 文件，Claude 启动时会自动加载。  

**创建方式：**进入一个项目文件夹后，执行 /init 命令，它就会扫描整个项目文件夹，生成一个 CLAUDE.md 文件。

### 技巧 2：权限自动批准 - 告别重复点击
**问题：**每次 Claude 读文件、写代码、运行命令，都要点「允许」。

**解决方案：**启动时加参数：claude --dangerously-skip-permissions

更方便的方式 - 在命令行里设置 alias：
```shell
alias cc="claude --dangerously-skip-permissions"
```
以后直接输入 cc 启动。如果不知道上面的代码如何配置，直接启动 cc，让它帮你配置就行。

使用建议：

* ✅ 个人项目放心用
* ✅ 熟悉的工作流放心用
* ❌ 不熟悉的代码谨慎用

### 技巧 3：自定义 Slash 命令 - 一键触发工作流
**问题：**每次部署，都要重复说一遍流程。

**步骤：**

1. 创建 .claude/commands/ 文件夹
2. 创建 部署.md，写入流程：
```markdown
---
name: 部署到生产环境
description: 自动化 GitHub 和 Vercel 部署流程
---


## 部署步骤
- [ ] Step 1: 运行 `pnpm run build` 检查构建
- [ ] Step 2: 创建 GitHub 私有仓库（可选，已有仓库则跳过创建）
- [ ] Step 3: Push 代码到 GitHub
- [ ] Step 4: 部署到 Vercel
- [ ] Step 5: 验证部署结果


**重要**：只有 build 成功后才能继续
使用：输入 /部署，自动执行
效果：用这个命令部署了十几个网站，基本不会有问题。
```