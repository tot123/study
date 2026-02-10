# Claude Code - 最佳实践指南

> 本指南汇总使用 Claude Code 的最佳实践，包括成本优化、安全建议、工作流程和常见问题解决方案。

## 📋 目录

- [1. 成本优化策略](#1-成本优化策略)
- [2. 安全最佳实践](#2-安全最佳实践)
- [3. 工作流程建议](#3-工作流程建议)
- [4. 性能优化](#4-性能优化)
- [5. 常见问题与解决](#5-常见问题与解决)
- [6. 参考资源](#6-参考资源)

---

## 1. 成本优化策略

### 1.1 任务分层策略

根据任务复杂度选择不同成本的模型：

| 任务类型 | 推荐模型 | 原因 |
|----------|----------|------|
| **简单对话** | 本地 7B 模型 | 免费，响应快 |
| **代码补全** | DeepSeek-Coder | 代码质量高，价格低 |
| **代码审查** | 本地模型 | 可离线，无成本 |
| **复杂推理** | DeepSeek-R1 | 推理能力强，价格合理 |
| **关键任务** | Claude Sonnet | 质量稳定，可信赖 |

### 1.2 上下文管理

#### 压缩会话

```bash
# 定期压缩会话上下文
/compact focus on current task

# 或在对话中
> 压缩会话，只保留认证相关的内容
```

#### 分离任务

```bash
# 不好的做法：在一个长会话中处理多个任务
> 1. 创建用户系统
> 2. 审查代码
> 3. 写测试
> ... (上下文累积，成本高)

# 好的做法：分开会话
# 会话 1: 创建用户系统
> 创建用户认证系统

# 会话 2: 审查代码
> 审查用户系统的代码

# 会话 3: 写测试
> 为用户系统写测试
```

#### 使用 /clear

```bash
# 任务完成后清空
/clear

# 开始新任务
> 新任务：订单系统
```

### 1.3 背景任务优化

使用本地模型处理后台任务：

```json
{
  "Router": {
    "background": "ollama,qwen2.5-coder:latest"
  }
}
```

**适用场景**:
- 批量代码生成
- 文档转换
- 测试数据生成
- 代码格式化

### 1.4 长上下文优化

#### 选择合适的模型

```json
{
  "Router": {
    "longContext": "anthropic,claude-sonnet-4",
    "longContextThreshold": 60000
  }
}
```

#### 分段处理

```python
# 不好的做法：一次性处理整个项目
> 分析整个项目的所有文件

# 好的做法：分模块处理
> 分析 src/auth/ 模块
> 分析 src/user/ 模块
> 分析 src/payment/ 模块
```

### 1.5 成本监控

定期检查使用情况：

```bash
# 查看当前会话成本
/cost

# 查看配额使用
/usage

# 设置预算提醒（在 settings.json 中）
{
  "budgetAlert": {
    "enabled": true,
    "dailyLimit": 10.0,  // 美元
    "alertThreshold": 0.8  // 80% 时提醒
  }
}
```

### 1.6 缓存策略

#### 项目级缓存

```markdown
# CLAUDE.md

# 项目缓存

## 重要的架构决策
- 使用 FastAPI + PostgreSQL
- JWT 认证
- SQLAlchemy ORM

## 这些决策不应该重复讨论
```

#### 技能缓存

常用操作封装成 Skill，避免重复解释：

```markdown
# .claude/skills/standard-api.md

---
description: 创建标准 REST API endpoint
---

每次使用相同的模式和结构，无需重复说明。
```

---

## 2. 安全最佳实践

### 2.1 API Key 管理

#### 环境变量（推荐）

```bash
# ~/.bashrc 或 ~/.zshrc
export ANTHROPIC_AUTH_TOKEN="$ANTHROPIC_API_KEY"
export DEEPSEEK_API_KEY="$DEEPSEEK_KEY"
export GITHUB_TOKEN="$GITHUB_TOKEN"
```

#### .env 文件

```bash
# ~/.claude/.env
ANTHROPIC_AUTH_TOKEN=sk-xxxxxx
DEEPSEEK_API_KEY=sk-xxxxxx

# 添加到 .gitignore
echo ".env" >> .gitignore
```

#### 避免硬编码

❌ **不好的做法**:

```python
# config.py
API_KEY = "sk-xxxxxxxxxxxxxxxx"  # 硬编码，不安全
```

✅ **好的做法**:

```python
# config.py
import os

API_KEY = os.getenv("ANTHROPIC_AUTH_TOKEN")
if not API_KEY:
    raise ValueError("ANTHROPIC_AUTH_TOKEN not set")
```

### 2.2 配置文件权限

```bash
# 限制配置文件权限（仅所有者可读写）
chmod 600 ~/.claude/settings.json
chmod 600 ~/.claude-code-router/config.json
chmod 600 ~/.claude/.env

# 验证权限
ls -la ~/.claude/
```

### 2.3 Git 忽略敏感文件

创建 `.gitignore`:

```gitignore
# Claude Code
.claude/settings.json
.claude/.env
*.key
*.pem

# 环境变量
.env
.env.local
.env.production

# API Keys
secrets/
credentials/
```

### 2.4 日志安全

#### 避免记录敏感信息

```python
# ❌ 不好
logger.info(f"User logged in: {username}, password: {password}")

# ✅ 好
logger.info(f"User logged in: {username}")
```

#### 脱敏处理

```python
import re

def mask_email(email):
    """隐藏邮箱中间部分"""
    if '@' in email:
        name, domain = email.split('@')
        return f"{name[0]}***@{domain}"
    return email

logger.info(f"User: {mask_email(user.email)}")
```

### 2.5 代码审查安全检查

使用 Skills 进行安全审查：

```bash
# 使用 Security Review Skill
> 审查这段代码的安全性

# 检查项目：
- SQL 注入
- XSS 漏洞
- CSRF 保护
- 认证授权
- 敏感数据泄露
```

---

## 3. 工作流程建议

### 3.1 项目初始化流程

```bash
# 1. 创建项目目录
mkdir my-project && cd my-project

# 2. 初始化 Git
git init

# 3. 启动 Claude Code
claude

# 4. 初始化项目
> /init

# 5. Claude Code 会创建 CLAUDE.md
# 6. 根据项目需要编辑 CLAUDE.md
```

### 3.2 日常开发流程

#### 开始工作

```bash
# 1. 启动 Claude Code
claude

# 2. 压缩之前的会话（如果有）
/compact

# 3. 说明今天的工作
> 今天的工作：
> 1. 完成用户注册 API
> 2. 添加单元测试
> 3. 代码审查
```

#### 编码阶段

```bash
# 使用编程专用模型
> 为用户注册创建 FastAPI endpoint

# Claude Code 会：
# 1. 生成代码
# 2. 自动创建文件
# 3. 遵循项目规范（从 CLAUDE.md）
```

#### 测试阶段

```bash
# 使用测试 Skill
> 为注册 API 写单元测试

# 或使用自定义命令
/test tests/test_registration.py
```

#### 审查阶段

```bash
# 使用审查 Skill
> 审查这次提交的代码

# Claude Code 会：
# 1. 检查代码质量
# 2. 识别安全问题
# 3. 提供改进建议
```

#### 提交流程

```bash
# 使用自定义 commit 命令
/commit "feat: add user registration API"

# 推送到远程
> 推送到 GitHub
```

### 3.3 团队协作流程

#### 共享 CLAUDE.md

```markdown
# CLAUDE.md (提交到 Git)

# Team Coding Standards

本项目遵循以下规范...

## 团队约定
- 使用 FastAPI 框架
- 所有 API 返回统一格式
- ...
```

#### 共享自定义命令

```bash
# .claude/commands/ (提交到 Git)

# team-review.md
---
description: Team code review process
---

执行团队的代码审查流程...

## 审查清单
- [ ] 代码风格
- [ ] 测试覆盖
- [ ] 文档完整
- [ ] 安全检查
```

#### 共享 Skills

```bash
# .claude/skills/ (提交到 Git)

# project-workflow.md
团队特定的开发工作流...
```

### 3.4 项目阶段切换

#### 开发阶段

```bash
# 使用快速、低成本模型
export ANTHROPIC_MODEL=deepseek-chat

# 或本地模型
export ANTHROPIC_BASE_URL=http://localhost:8080/anthropic
```

#### 生产准备阶段

```bash
# 切换到高质量模型
export ANTHROPIC_MODEL=claude-sonnet-4

# 或使用推理模型
export ANTHROPIC_MODEL=deepseek-reasoner
```

#### 代码审查阶段

```bash
# 使用最严格的模型
export ANTHROPIC_MODEL=claude-opus-4

# 使用 Security Review Skill
> /security-review
```

---

## 4. 性能优化

### 4.1 响应速度优化

#### 使用更快的模型

```json
{
  "Router": {
    "fast": "ollama,qwen2.5:7b",
    "background": "ollama,qwen2.5:7b"
  }
}
```

#### 减少上下文

```bash
# 定期压缩
/compact

# 或清空重开
/clear
```

#### 禁用不必要的功能

```bash
# settings.json
{
  "features": {
    "webSearch": false,  // 禁用联网搜索
    "mcpAutoConnect": false  // 禁用自动 MCP 连接
  }
}
```

### 4.2 并发处理

#### 使用后台任务

```bash
# 启动后台 bash
/bashes

# 查看后台任务
> 列出所有后台任务
```

#### 批量操作

```python
# 不好：逐个处理
for file in files:
    process(file)  # 每次等待响应

# 好：批量处理
process_batch(files)  # 一次性处理
```

### 4.3 本地缓存

#### 使用 CC-Switch 缓存

```bash
# CC-Switch 会缓存：
# - Provider 响应
# - MCP 连接
# - Skills 加载
```

#### 文件缓存

```markdown
# CLAUDE.md

# 缓存：常用模块

## User 模块
位置: src/models/user.py
用途: 用户数据模型
关键方法: get_by_id(), create(), update()

## Auth 模块
位置: src/services/auth.py
用途: 认证服务
关键方法: authenticate(), generate_token()
```

---

## 5. 常见问题与解决

### 5.1 安装问题

#### 问题：NPM 安装失败

```bash
# 错误：EACCES
sudo npm install -g @anthropic-ai/claude-code

# 或使用 nvm
nvm use node
npm install -g @anthropic-ai/claude-code
```

#### 问题：权限被拒绝

```bash
# macOS/Linux
sudo chown -R $USER ~/.claude

# Windows（以管理员身份运行 PowerShell）
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### 5.2 连接问题

#### 问题：API 超时

```bash
# 增加超时时间
export API_TIMEOUT_MS=600000  # 10 分钟

# 或使用更快的 API
export ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
```

#### 问题：代理设置

```bash
# 设置 HTTP 代理
export HTTP_PROXY=http://proxy.example.com:8080
export HTTPS_PROXY=http://proxy.example.com:8080

# Claude Code 特定
export CLAUDE_CODE_PROXY=http://proxy.example.com:8080
```

### 5.3 性能问题

#### 问题：响应慢

**诊断**:

```bash
# 查看上下文大小
/context

# 查看模型状态
> /status

# 压缩会话
/compact
```

**解决**:

```bash
# 1. 切换到更快的模型
/model haiku

# 2. 或压缩会话
/compact remove earlier context

# 3. 或清空重开
/clear
```

#### 问题：内存占用高

```bash
# 限制上下文窗口
export MAX_CONTEXT_SIZE=100000

# 或定期重启
/clear
```

### 5.4 模型问题

#### 问题：模型回答质量差

**诊断**:

```bash
# 查看当前使用的模型
> /model

# 查看任务类型
> /status
```

**解决**:

```bash
# 1. 切换模型
/model sonnet  # 使用更高质量的模型

# 2. 或提供更多上下文
> 这是背景信息：...（详细描述）

# 3. 或使用特定 Skill
> 使用 code-review skill 审查这段代码
```

#### 问题：模型不支持的功能

```bash
# 检查模型能力
> 这个模型支持代码执行吗？

# 切换到支持该功能的模型
/model opus
```

### 5.5 MCP 问题

#### 问题：MCP 服务器无法连接

**诊断**:

```bash
# 检查 MCP 配置
/mcp

# 测试 MCP 服务器
# 手动运行 MCP 服务器
npx @modelcontextprotocol/server-filesystem --help
```

**解决**:

```json
// ~/.claude/settings.json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path"],
      "env": {
        "NODE_OPTIONS": "--max-old-space-size=4096"
      }
    }
  }
}
```

### 5.6 Skills 问题

#### 问题：Skill 不生效

**诊断**:

```bash
# 检查 Skills 目录
ls ~/.claude/skills/

# 验证 Skill 文件
cat ~/.claude/skills/skill-name.md
```

**解决**:

```bash
# 1. 检查 frontmatter 格式
# 必须有 description 字段

# 2. 重启 Claude Code
/exit
claude

# 3. 或查看 Skill 日志
tail -f ~/.claude/logs/skills.log
```

---

## 6. 进阶技巧

### 6.1 组合使用多个工具

```bash
# 1. 使用 MCP 读取多个文件
# 2. 使用 Skill 分析代码
# 3. 使用 Slash 命令执行操作

> 使用 filesystem MCP 读取 src/ 目录下所有 Python 文件
> 然后使用 code-review skill 审查这些文件
> 最后使用 /commit 提交更改
```

### 6.2 创建工作流 Skill

```markdown
# .claude/skills/daily-workflow.md

---
description: Execute daily development workflow
---

# 每日开发工作流

## 步骤

1. **拉取最新代码**
   ```bash
   git pull origin main
   ```

2. **查看今日任务**
   - 检查 TODO
   - 查看 Issues

3. **开始开发**
   - 使用 coding skill
   - 遵循 CLAUDE.md 规范

4. **测试**
   - 运行单元测试
   - 运行集成测试

5. **提交代码**
   - 使用 commit 命令
   - 推送到远程

## 每日检查清单

- [ ] 代码风格检查
- [ ] 测试覆盖
- [ ] 文档更新
- [ ] 安全审查
```

### 6.3 自动化脚本

创建 `dev.sh`:

```bash
#!/bin/bash

# 每日开发启动脚本

echo "📅 每日开发工作流启动"

# 1. 更新代码
echo "📥 拉取最新代码..."
git pull origin main

# 2. 启动 Claude Code
echo "🤖 启动 Claude Code..."
claude

# 3. 退出后总结
echo "✅ 今日工作完成"
git log -1 --oneline
```

使用：

```bash
chmod +x dev.sh
./dev.sh
```

---

## 7. 参考资源

### 7.1 官方资源

- 📚 [Claude Code 官方文档](https://docs.anthropic.com/claude-code)
- 🔌 [MCP 官方文档](https://modelcontextprotocol.io/)
- 🎯 [Claude Agent Skills 文档](https://docs.anthropic.com/claude-code/skills)

### 7.2 社区资源

- 💻 [Claude Code Router](https://github.com/musistudio/claude-code-router)
- 🖥️ [CC-Switch](https://github.com/farion1231/cc-switch)
- 📦 [MCP Servers](https://github.com/modelcontextprotocol/servers)
- 🎯 [Claude Code Skills](https://github.com/anthropics/claude-code-skills)

### 7.3 第三方模型

- 🤖 [DeepSeek](https://platform.deepseek.com/)
- 🌐 [智谱 GLM](https://open.bigmodel.cn/)
- 🦙 [Ollama](https://ollama.com/)
- 🔀 [OpenRouter](https://openrouter.ai/)

### 7.4 系列文档

本教程的其他部分：

1. [00200-claude-code-基础.md](./00200-claude-code-基础.md) - 基础教程
2. [00201-claudecode-slash-命令.md](./00201-claudecode-slash-命令.md) - Slash 命令
3. [00202-claudecode-mcp服务器.md](./00202-claudecode-mcp服务器.md) - MCP 服务器
4. [00203-claudecode-skills系统.md](./00203-claudecode-skills系统.md) - Skills 系统
5. [00204-claudecode-第三方模型.md](./00204-claudecode-第三方模型.md) - 第三方模型
6. [00205-claudecode-智能路由.md](./00205-claudecode-智能路由.md) - 智能路由
7. [00206-claudecode-最佳实践.md](./00206-claudecode-最佳实践.md) - 本文档

### 7.5 相关教程

- [00001-docker安装ollama.md](./00001-docker安装ollama.md) - Ollama 安装教程

---

**文档版本**: v1.0
**最后更新**: 2026-01-26
**作者**: Your Name

---

## 附录：快速参考

### 常用命令速查

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

# 配置管理
/model              # 切换模型
/config             # 打开配置
/mcp                # 管理 MCP

# 开发命令
/init               # 初始化项目
/review             # 代码审查
/security-review    # 安全审查
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
export MAX_CONTEXT_SIZE=200000
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

### 快速故障排除

| 问题 | 快速解决 |
|------|----------|
| **无法启动** | `claude --version` 检查版本 |
| **API 错误** | 检查 `ANTHROPIC_AUTH_TOKEN` |
| **响应慢** | `/compact` 压缩上下文 |
| **MCP 不工作** | `/mcp` 检查配置 |
| **Skill 不生效** | 检查 frontmatter 格式 |
| **成本高** | 切换到 DeepSeek 或本地模型 |
