# Claude Code - Slash 命令详解

> Slash 命令是 Claude Code 的内置命令和自定义命令系统，提供强大的自动化和扩展能力。

## 📋 目录

- [1. 什么是 Slash 命令](#1-什么是-slash-命令)
- [2. 常用内置命令](#2-常用内置命令)
- [3. 自定义命令](#3-自定义命令)
- [4. Skills vs Slash Commands](#4-skills-vs-slash-commands)

---

## 1. 什么是 Slash 命令

**Slash 命令** 是 Claude Code 的命令系统，分为：

- **内置命令**: Claude Code 自带的命令
- **自定义命令**: 用户或项目定义的命令
- **插件命令**: 随插件分发的命令
- **MCP 命令**: 由 MCP 服务器动态发现的命令

### 命令格式

```bash
/command-name [arguments]
```

---

## 2. 常用内置命令

### 2.1 核心命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `/help` | 获取帮助列表和用法 | `/help` |
| `/clear` | 清空会话历史 | `/clear` |
| `/exit` | 退出 REPL | `/exit` |
| `/model` | 选择或切换 AI 模型 | `/model` |
| `/config` | 打开设置界面 | `/config` |

### 2.2 状态查看命令

| 命令 | 说明 |
|------|------|
| `/status` | 打开状态界面（版本、模型、账户、连通性） |
| `/context` | 以彩色网格方式可视化上下文使用情况 |
| `/cost` | 显示 token 使用统计 |
| `/usage` | 显示订阅计划使用限额和速率限制状态 |

### 2.3 会话管理命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `/compact [instructions]` | 压缩会话上下文，可带聚焦指令 | `/compact focus on authentication` |
| `/export [filename]` | 导出当前会话到文件或剪贴板 | `/export session.md` |
| `/memory` | 编辑 `CLAUDE.md` 的记忆文件 | `/memory` |
| `/resume` | 恢复会话 | `/resume session-id` |

### 2.4 功能管理命令

| 命令 | 说明 |
|------|------|
| `/mcp` | 管理 MCP 服务器连接和 OAuth 授权 |
| `/agents` | 管理自定义子代理（AI subagents） |
| `/plugin` | 管理 Claude Code 插件 |
| `/permissions` | 查看或更新权限规则 |
| `/hooks` | 管理与工具事件相关的 hook 配置 |

### 2.5 开发辅助命令

| 命令 | 说明 |
|------|------|
| `/init` | 用 `CLAUDE.md` 指南初始化项目 |
| `/review` | 请求代码审查 |
| `/security-review` | 对当前分支的待定更改执行安全审查 |
| `/todos` | 列出当前 todo 项 |

### 2.6 高级命令

| 命令 | 说明 |
|------|------|
| `/add-dir` | 添加额外的工作目录 |
| `/bashes` | 列出/管理后台 bash 任务 |
| `/vim` | 进入 vim 模式（交替插入/命令模式） |
| `/sandbox` | 启用受限的 sandboxed bash（文件系统与网络隔离） |
| `/terminal-setup` | 为 iTerm2 / VSCode 安装 Shift+Enter 换行快捷键 |
| `/rewind` | 回退会话或代码状态 |
| `/output-style [style]` | 设置输出风格 |

---

## 3. 自定义命令

### 3.1 命令类型

| 类型 | 位置 | 说明 |
|------|------|------|
| **项目命令** | `.claude/commands/` | 对团队共享 |
| **个人命令** | `~/.claude/commands/` | 在所有项目可用 |
| **插件命令** | 插件的 `commands/` 目录 | 随插件分发 |
| **MCP 命令** | 动态发现 | 格式：`/mcp__<server>__<prompt>` |

### 3.2 命令文件结构

创建 `.claude/commands/command-name.md`:

```markdown
---
description: 命令的简短描述
argument-hint: [参数提示]
allowed-tools: ["Bash", "Read", "Write"]
model: haiku  # 可选：指定使用的模型
---

# 命令标题

命令的详细说明和使用方法。

## 参数说明

- `$ARGUMENTS`: 捕获所有参数字符串
- `$1`, `$2`, ...: 按位置访问单个参数

## 使用示例

示例 1: ...
示例 2: ...
```

### 3.3 示例 1: Git 提交命令

创建 `.claude/commands/commit.md`:

```markdown
---
description: Create a git commit with staged changes
argument-hint: [commit message]
allowed-tools: ["Bash"]
---

# Git Commit

创建 Git 提交，包含所有已暂存的更改。

## 参数

- `$ARGUMENTS`: 提交消息

## 使用示例

```bash
/commit "Fix authentication bug in login flow"
/commit "Add user registration feature"
```

## 注意事项

- 自动添加 Co-Authored-By 标记
- 自动运行 git status 验证提交成功
```

### 3.4 示例 2: 测试命令

创建 `.claude/commands/test.md`:

```markdown
---
description: Run tests and show coverage
argument-hint: [test file or pattern]
allowed-tools: ["Bash"]
---

# Run Tests

运行测试套件并显示覆盖率报告。

## 参数

- `$ARGUMENTS` (可选): 特定的测试文件或模式

## 使用示例

```bash
# 运行所有测试
/test

# 运行特定测试文件
/test tests/test_auth.py

# 运行匹配模式的测试
/test test_*user*
```

## 输出

- 测试结果摘要
- 覆盖率报告
- 失败测试的详细信息
```

### 3.5 示例 3: 代码审查命令

创建 `.claude/commands/review-pr.md`:

```markdown
---
description: Review a pull request
argument-hint: [PR number or URL]
allowed-tools: ["Bash", "Read", "Grep"]
---

# Pull Request Review

审查 Pull Request 的代码更改。

## 参数

- `$1`: PR 编号或完整 URL

## 使用示例

```bash
/review-pr 123
/review-pr https://github.com/owner/repo/pull/123
```

## 审查内容

- 代码质量和风格
- 安全问题
- 性能考虑
- 测试覆盖率
- 文档完整性
```

### 3.6 参数处理技巧

#### 捕获所有参数

```markdown
---
argument-hint: [any text]
---

使用 `$ARGUMENTS` 捕获所有输入：

> This is the full command: $ARGUMENTS
```

#### 位置参数

```markdown
---
argument-hint: <file> <line-number>
---

- 第一个参数: `$1`
- 第二个参数: `$2`
- 所有参数: `$ARGUMENTS`
```

#### 可选参数与默认值

```markdown
创建文件: ${1:-default-file.txt}
操作类型: ${2:-create}
```

### 3.7 Bash 执行

在命令文件中可用 `!` 前缀执行 bash 命令：

```markdown
---
allowed-tools: ["Bash"]
---

# 项目信息

```bash
!git remote -v
!git log -1 --oneline
!pwd
```
```

### 3.8 文件引用

使用 `@path/to/file` 在命令上下文中引用文件内容：

```markdown
---
allowed-tools: ["Read"]
---

# 配置检查

检查配置文件:

@config/database.json
@.env.example
```

---

## 4. Skills vs Slash Commands

### 4.1 对比表格

| 对比项 | Slash Commands | Skills |
|--------|---------------|---------|
| **用途** | 简单、重复的提示 | 复杂、多文件、自动发现的能力 |
| **触发** | 明确手动触发 | 自动发现和调用 |
| **复杂度** | 低 | 高（包含脚本和校验流程） |
| **适用场景** | 快速模板、常用操作 | 复杂工作流、多步骤任务 |
| **文件位置** | `.claude/commands/` | `~/.claude/skills/` |
| **参数传递** | 命令行参数 | 自然语言描述 |
| **验证流程** | 无 | 包含完整验证 |

### 4.2 选择建议

**使用 Slash Commands 当**:
- ✅ 任务简单明确
- ✅ 需要快速执行
- ✅ 参数固定
- ✅ 手动触发更合适

**使用 Skills 当**:
- ✅ 任务复杂多步骤
- ✅ 需要自动发现
- ✅ 包含验证流程
- ✅ 需要复杂的逻辑处理

### 4.3 实际案例对比

#### Slash Command: 快速提交

```bash
# 简单直接
/commit "Fix bug"
```

#### Skill: 完整代码审查流程

```bash
# 自动发现并执行多步骤审查
> 请审查这次提交
```

Skill 会自动：
1. 分析代码更改
2. 检查安全问题
3. 验证测试覆盖
4. 提供改进建议
5. 生成审查报告

---

## 5. 高级技巧

### 5.1 命令链

可以组合多个命令：

```bash
/commit "Add feature" && /push && /clean
```

### 5.2 权限控制

通过 `allowed-tools` 限制命令可用的工具：

```markdown
---
allowed-tools: ["Read"]  # 只允许读取文件
---
```

### 5.3 模型选择

为特定命令指定最优模型：

```markdown
---
model: haiku  # 使用快速模型
---
```

### 5.4 禁用模型调用

阻止 AI 自动调用此命令：

```markdown
---
disable-model-invocation: true
---
```

---

## 6. 相关文档

- **上一节**: [00200-claude-code-基础.md](./00200-claude-code-基础.md) - 基础教程
- **下一节**: [00202-claudecode-mcp服务器.md](./00202-claudecode-mcp服务器.md) - MCP 服务器
- [00203-claudecode-skills系统.md](./00203-claudecode-skills系统.md) - Skills 系统





我现在需要创建一个自定义Slash 命令，功能如下
自动将当前项目未提交的项目提交到远端
请注意忽略的文件不需要提交，注册命令为 /git-push
---
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git commit:*), Bash(git diff:*), Bash(git log:*), Bash(git push:*), Bash(git branch:*)
description: 提交代码到远端代码库
---

## 上下文

- 当前git状态：!`git status`
- 当前git差异（已暂存和未暂存的更改）：!`git diff HEAD`
- 最近的提交：!`git log --oneline -10`
- 当然分支：!`git branch`

## 您的任务
执行命令'git push origin HEAD:refs/for/${当前分支}'
