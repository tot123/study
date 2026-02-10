# Claude Code 完整使用指南

> 本指南是 Claude Code 的完整教程合集，涵盖从入门到精通的所有内容。每部分都有独立的详细文档，方便查阅。

## 📚 文档导航

### 🚀 快速开始

如果你是第一次使用 Claude Code，建议按顺序阅读：

1. **[00200-claude-code-基础.md](./00200-claude-code-基础.md)** - 快速入门
   - Claude Code 是什么
   - 安装与配置
   - 基础使用
   - 第一个项目

**预计阅读时间**: 15 分钟

### 📖 核心功能

掌握 Claude Code 的核心功能：

2. **[00201-claudecode-slash-命令.md](./00201-claudecode-slash-命令.md)** - Slash 命令系统
   - 30+ 内置命令详解
   - 自定义命令开发
   - 实战案例

3. **[00202-claudecode-mcp服务器.md](./00202-claudecode-mcp服务器.md)** - MCP 服务器
   - 8+ 常用 MCP 配置
   - CLI 快速安装方法
   - 自定义 MCP 开发

4. **[00203-claudecode-skills系统.md](./00203-claudecode-skills系统.md)** - Skills 系统
   - 官方 Skills 使用
   - 自定义 Skill 开发
   - 项目级 Skills

**预计阅读时间**: 每篇 20-30 分钟

### ⚙️ 高级配置

优化你的 Claude Code 体验：

5. **[00204-claudecode-第三方模型.md](./00204-claudecode-第三方模型.md)** - 第三方模型集成
   - DeepSeek 配置
   - 智谱 GLM 集成
   - Ollama 本地模型
   - 成本对比

6. **[00205-claudecode-智能路由.md](./00205-claudecode-智能路由.md)** - 智能路由系统
   - CCR 配置与使用
   - CC-Switch 可视化管理
   - 路由策略实战

**预计阅读时间**: 每篇 25 分钟

### 🎯 最佳实践

提升使用效率：

7. **[00206-claudecode-最佳实践.md](./00206-claudecode-最佳实践.md)** - 最佳实践指南
   - 成本优化策略
   - 安全建议
   - 工作流程
   - 常见问题解决

**预计阅读时间**: 30 分钟

---

## 🚀 快速安装

### 一键安装

```bash
# NPM 安装（推荐）
npm install -g @anthropic-ai/claude-code

# 验证安装
claude --version

# 启动
claude
```

### 快速配置 MCP

使用 CLI 命令快速安装常用 MCP：

```bash
# GitHub（全局）
claude mcp add github --scope user -- npx -y @modelcontextprotocol/server-github

# Git（全局）
claude mcp add git --scope user -- npx -y @modelcontextprotocol/server-git --repository .

# 文件系统（全局）
claude mcp add filesystem --scope user -- npx -y @modelcontextprotocol/server-filesystem ~/projects

# 查看已安装的 MCP
claude mcp list
```

### 接入低成本模型

**DeepSeek（推荐）**:

```bash
# Bash/Zsh
export ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
export ANTHROPIC_AUTH_TOKEN=your-deepseek-api-key
export API_TIMEOUT_MS=600000
export ANTHROPIC_MODEL=deepseek-chat

# 启动 Claude Code
claude
```

---

## 📋 学习路径

### 初学者路径

```
第1天: 基础入门
├── 阅读: 00200-claude-code-基础.md
├── 实践: 安装并完成第一个项目
└── 时间: 1-2 小时

第2天: 核心功能
├── 阅读: 00201-claudecode-slash-命令.md
├── 实践: 创建 3 个自定义命令
└── 时间: 2-3 小时

第3天: 扩展能力
├── 阅读: 00202-claudecode-mcp服务器.md
├── 实践: 安装 3 个常用 MCP
└── 时间: 2-3 小时

第4天: 高级技巧
├── 阅读: 00203-claudecode-skills系统.md
├── 实践: 开发 1 个自定义 Skill
└── 时间: 3-4 小时
```

### 进阶路径

```
第5天: 成本优化
├── 阅读: 00204-claudecode-第三方模型.md
├── 实践: 配置 DeepSeek + Ollama
└── 时间: 2-3 小时

第6天: 智能路由
├── 阅读: 00205-claudecode-智能路由.md
├── 实践: 配置 CCR 路由策略
└── 时间: 3-4 小时

第7天: 最佳实践
├── 阅读: 00206-claudecode-最佳实践.md
├── 实践: 优化你的工作流程
└── 时间: 2-3 小时
```

---

## 🎯 按场景查找

### 我想...

| 场景 | 推荐文档 | 章节 |
|------|----------|------|
| **快速上手** | [00200-claude-code-基础.md](./00200-claude-code-基础.md) | 全文 |
| **自定义命令** | [00201-claudecode-slash-命令.md](./00201-claudecode-slash-命令.md) | 第 3 节 |
| **文件系统操作** | [00202-claudecode-mcp服务器.md](./00202-claudecode-mcp服务器.md) | 第 2.1 节 |
| **GitHub 集成** | [00202-claudecode-mcp服务器.md](./00202-claudecode-mcp服务器.md) | 第 2.4 节 |
| **网络搜索** | [00202-claudecode-mcp服务器.md](./00202-claudecode-mcp服务器.md) | 第 2.3 节 |
| **降低成本** | [00204-claudecode-第三方模型.md](./00204-claudecode-第三方模型.md) | 全文 |
| **使用本地模型** | [00204-claudecode-第三方模型.md](./00204-claudecode-第三方模型.md) | 第 4 节 |
| **自动模型切换** | [00205-claudecode-智能路由.md](./00205-claudecode-智能路由.md) | 第 4 节 |
| **代码审查** | [00203-claudecode-skills系统.md](./00203-claudecode-skills系统.md) | 第 2.1 节 |
| **数据库操作** | [00202-claudecode-mcp服务器.md](./00202-claudecode-mcp服务器.md) | 第 2.5、2.7 节 |
| **浏览器自动化** | [00202-claudecode-mcp服务器.md](./00202-claudecode-mcp服务器.md) | 第 2.6 节 |
| **安全最佳实践** | [00206-claudecode-最佳实践.md](./00206-claudecode-最佳实践.md) | 第 2 节 |
| **性能优化** | [00206-claudecode-最佳实践.md](./00206-claudecode-最佳实践.md) | 第 4 节 |
| **故障排除** | [00206-claudecode-最佳实践.md](./00206-claudecode-最佳实践.md) | 第 5 节 |

---

## 🔧 快速参考

### 常用命令

```bash
# 基础命令
claude              # 启动 Claude Code
/help               # 帮助
/clear              # 清空会话
/exit               # 退出

# 状态查看
/status             # 查看状态
/context            # 查看上下文
/cost               # 查看成本
/usage              # 查看配额

# MCP 管理
claude mcp list                    # 列出所有 MCP
claude mcp add github --scope user -- npx -y @modelcontextprotocol/server-github
claude mcp remove github --scope user

# 配置管理
/model              # 切换模型
/config             # 打开配置
/mcp                # 管理 MCP（在 Claude Code 中）
```

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

### MCP 快速安装

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

---

## 📊 内容概览

### 文档统计

| 文档 | 章节 | 字数 | 预计阅读时间 |
|------|------|------|--------------|
| 00200-基础 | 4 | ~3,000 | 15 分钟 |
| 00201-Slash命令 | 6 | ~8,000 | 25 分钟 |
| 00202-MCP服务器 | 5 | ~12,000 | 35 分钟 |
| 00203-Skills系统 | 5 | ~10,000 | 30 分钟 |
| 00204-第三方模型 | 7 | ~8,000 | 25 分钟 |
| 00205-智能路由 | 7 | ~9,000 | 30 分钟 |
| 00206-最佳实践 | 7 | ~10,000 | 30 分钟 |
| **总计** | **41** | **~60,000** | **3 小时** |

### 核心概念

- **Claude Code**: 终端 AI 编码助手
- **Slash 命令**: 内置和自定义命令系统
- **MCP**: Model Context Protocol，插件协议
- **Skills**: 自动化能力包
- **CCR**: Claude Code Router，智能路由工具
- **CC-Switch**: 可视化配置管理工具

### 技术栈

| 类别 | 技术 |
|------|------|
| **语言** | Python, Node.js, TypeScript |
| **AI 模型** | Claude, DeepSeek, GLM, Ollama |
| **协议** | MCP, SSE, HTTP |
| **工具** | FastAPI, Git, Docker |

---

## 🌟 特色功能

### 1. CLI 快速安装 MCP

无需手动编辑配置文件，一行命令完成安装：

```bash
claude mcp add <name> --scope <scope> -- <command>
```

详见：[00202-claudecode-mcp服务器.md - 第 3.1 节](./00202-claudecode-mcp服务器.md#31-快速安装-mcp)

### 2. 智能模型路由

根据任务类型自动选择最合适的模型：

- 简单任务 → 本地模型（免费）
- 编程任务 → DeepSeek-Coder（便宜）
- 复杂推理 → DeepSeek-R1（高质量）
- 关键任务 → Claude Opus（最佳质量）

详见：[00205-claudecode-智能路由.md - 第 4 节](./00205-claudecode-智能路由.md#4-路由策略配置)

### 3. 成本优化策略

通过任务分层和智能路由，可节省 70-90% 成本：

| 策略 | 成本节省 |
|------|----------|
| 使用本地模型 | 100% |
| 使用 DeepSeek | 80-90% |
| 智能路由组合 | 70-90% |

详见：[00206-claudecode-最佳实践.md - 第 1 节](./00206-claudecode-最佳实践.md#1-成本优化策略)

---

## 🔗 相关资源

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

### 相关教程

- [00001-docker安装ollama.md](./00001-docker安装ollama.md) - Ollama 安装教程

---

## 💡 使用建议

### 新手建议

1. **先读基础**: 从 [00200-claude-code-基础.md](./00200-claude-code-基础.md) 开始
2. **动手实践**: 每读完一章立即实践
3. **循序渐进**: 不要一次看完所有文档
4. **遇到问题**: 查看 [00206-claudecode-最佳实践.md](./00206-claudecode-最佳实践.md) 的故障排除部分

### 进阶建议

1. **自定义工作流**: 创建自己的 Slash 命令和 Skills
2. **优化成本**: 配置智能路由，使用本地模型
3. **团队协作**: 共享项目级配置和 Skills
4. **持续学习**: 关注官方文档和社区动态

---

## 📝 更新日志

### v1.0 (2026-01-26)

- ✅ 初始版本发布
- ✅ 7 个核心文档
- ✅ 添加 MCP CLI 快速安装方法
- ✅ 完整的实战案例和配置示例

---

## 📧 反馈与贡献

如果你在使用过程中遇到问题或有改进建议：

1. 查看 [00206-claudecode-最佳实践.md](./00206-claudecode-最佳实践.md) 的常见问题
2. 查阅官方文档
3. 参与社区讨论


