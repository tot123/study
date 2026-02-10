# Claude Code - Skills 系统详解

> Skills 是 Claude Code 的自动化能力包，用于处理复杂、多步骤的任务。与 Slash 命令不同，Skills 可以自动发现适用场景并包含完整的验证流程。

## 📋 目录

- [1. Skills 简介](#1-skills-简介)
- [2. 常用官方 Skills](#2-常用官方-skills)
- [3. 安装和管理 Skills](#3-安装和管理-skills)
- [4. 开发自定义 Skill](#4-开发自定义-skill)

---

## 1. Skills 简介

### 1.1 什么是 Skills？

**Skills** 是 Claude Code 的自动化能力包，用于处理复杂、多步骤的任务。每个 Skill 包含：

- 📝 **完整文档**: 使用说明和最佳实践
- 🔄 **工作流程**: 多步骤执行流程
- ✅ **验证机制**: 确保输出质量
- 🎯 **自动发现**: 根据上下文自动触发

### 1.2 Skills vs 其他扩展方式

| 对比项 | Slash Commands | Skills | MCP |
|--------|---------------|---------|-----|
| **用途** | 简单模板 | 复杂工作流 | 扩展 AI 能力 |
| **触发** | 手动调用 | 自动发现 | 工具调用 |
| **复杂度** | 低 | 高 | 中 |
| **包含内容** | 提示词 | 脚本+校验+文档 | 工具实现 |
| **文件位置** | `.claude/commands/` | `~/.claude/skills/` | 外部进程 |
| **示例** | `/commit` | `code-review` | `filesystem` |

### 1.3 Skills 的优势

1. **自动发现**: AI 根据任务自动选择合适的 Skill
2. **完整流程**: 包含准备、执行、验证完整步骤
3. **最佳实践**: 内置行业标准和最佳实践
4. **可复用性**: 跨项目共享和复用
5. **质量保证**: 包含验证和测试步骤

---

## 2. 常用官方 Skills

### 2.1 Code Review Skill

#### 功能
自动化代码审查，检查代码质量、安全性、性能问题。

#### 使用方式

```bash
# Claude Code 会自动发现代码审查场景
> 请审查这次提交的代码
> 审查 src/auth.py 的安全性
```

#### 包含内容

| 检查项 | 说明 |
|--------|------|
| **代码风格** | PEP 8、命名规范、格式化 |
| **安全漏洞** | SQL 注入、XSS、认证问题 |
| **性能问题** | N+1 查询、内存泄漏、算法复杂度 |
| **最佳实践** | 设计模式、SOLID 原则 |
| **测试覆盖** | 单元测试、边界条件 |
| **文档完整性** | Docstrings、注释、README |

#### 审查报告示例

```markdown
## 代码审查报告

### 🟢 良好实践
- 使用了类型提示
- 错误处理完善
- 日志记录适当

### 🟡 需要改进
- 建议：函数过长，建议拆分
- 建议：添加更多的单元测试
- 性能：考虑使用缓存优化重复查询

### 🔴 严重问题
- 安全：存在 SQL 注入风险（第 45 行）
- 安全：密码不应记录到日志（第 67 行）

### 📋 改进建议
1. 使用参数化查询防止 SQL 注入
2. 添加输入验证
3. 补充边界条件测试
```

### 2.2 Backend Development Skill

#### 功能
后端开发最佳实践，支持多种框架。

#### 支持的框架

| 框架 | 语言 | 特性 |
|------|------|------|
| **FastAPI** | Python | 异步、自动文档、类型验证 |
| **Django** | Python | 全栈、ORM、Admin |
| **NestJS** | TypeScript | 模块化、依赖注入 |
| **Express** | TypeScript/JavaScript | 轻量、灵活 |
| **Spring Boot** | Java | 企业级、生态丰富 |

#### 使用方式

```bash
# 创建项目结构
> 创建一个 FastAPI 用户认证 API

# 添加功能
> 添加 JWT 认证中间件
> 实现用户注册和登录接口

# 数据库操作
> 创建 SQLAlchemy User 模型
> 添加数据库迁移

# 测试
> 为认证 API 编写单元测试
```

#### 自动生成的项目结构

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI 应用入口
│   ├── api/
│   │   ├── __init__.py
│   │   └── v1/
│   │       ├── __init__.py
│   │       └── endpoints/
│   │           ├── __init__.py
│   │           ├── auth.py  # 认证端点
│   │           └── users.py # 用户端点
│   ├── core/
│   │   ├── __init__.py
│   │   ├── config.py        # 配置
│   │   └── security.py      # JWT、密码哈希
│   ├── models/
│   │   ├── __init__.py
│   │   └── user.py          # 用户模型
│   ├── schemas/
│   │   ├── __init__.py
│   │   └── user.py          # Pydantic schemas
│   ├── services/
│   │   ├── __init__.py
│   │   └── user_service.py  # 业务逻辑
│   └── utils/
│       ├── __init__.py
│       └── dependencies.py  # 依赖注入
├── tests/
│   ├── __init__.py
│   ├── conftest.py          # pytest 配置
│   └── test_auth.py         # 认证测试
├── alembic.ini              # 数据库迁移
├── .env.example
├── requirements.txt
├── Dockerfile
└── README.md
```

### 2.3 Database Design Skill

#### 功能
数据库设计和优化建议。

#### 使用方式

```bash
# 设计数据库
> 设计一个电商订单系统的数据库
> 为博客系统设计数据库 Schema

# 优化查询
> 优化这个慢查询
> 为 users 表添加合适的索引

# 数据迁移
> 生成数据迁移 SQL
> 从 MySQL 迁移到 PostgreSQL
```

#### 设计流程

1. **需求分析**
   - 识别实体和关系
   - 确定数据量和访问模式
   - 定义业务规则

2. **概念设计**
   - ER 图绘制
   - 关系定义（1:1, 1:N, N:M）
   - 约束和规则

3. **逻辑设计**
   - 表结构定义
   - 字段类型和约束
   - 索引策略

4. **物理设计**
   - 分区策略
   - 索引优化
   - 查询性能优化

#### 输出示例

```sql
-- 用户表
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 索引
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);

-- 评论：添加触发器自动更新 updated_at
```

### 2.4 Testing Skill

#### 功能
测试用例生成和测试策略。

#### 使用方式

```bash
# 生成单元测试
> 为这个函数写单元测试
> 为 UserService 类生成完整测试

# 测试策略
> 制定 API 测试策略
> 添加集成测试

# Mock 数据
> 生成测试用的 Mock 数据
```

#### 测试类型

| 类型 | 框架 | 说明 |
|------|------|------|
| **单元测试** | pytest, unittest | 测试单个函数/类 |
| **集成测试** | pytest-integration | 测试模块间交互 |
| **API 测试** | pytest-httpx | 测试 HTTP 端点 |
| **性能测试** | pytest-benchmark | 测试性能指标 |
| **覆盖率** | pytest-cov | 代码覆盖率分析 |

#### 生成的测试示例

```python
# tests/test_user_service.py
import pytest
from app.services.user_service import UserService
from app.models.user import User

class TestUserService:
    """UserService 单元测试"""

    @pytest.fixture
    def user_service(self, db_session):
        """创建 UserService 实例"""
        return UserService(db_session)

    @pytest.fixture
    def sample_user(self):
        """创建测试用户数据"""
        return {
            "username": "testuser",
            "email": "test@example.com",
            "password": "securepassword123"
        }

    async def test_create_user_success(self, user_service, sample_user):
        """测试成功创建用户"""
        user = await user_service.create_user(**sample_user)

        assert user.id is not None
        assert user.username == sample_user["username"]
        assert user.email == sample_user["email"]
        assert user.password_hash != sample_user["password"]

    async def test_create_duplicate_email(self, user_service, sample_user):
        """测试重复邮箱应抛出异常"""
        await user_service.create_user(**sample_user)

        with pytest.raises(ValueError, match="Email already exists"):
            await user_service.create_user(**sample_user)

    async def test_authenticate_success(self, user_service, sample_user):
        """测试成功认证"""
        await user_service.create_user(**sample_user)

        user = await user_service.authenticate(
            sample_user["email"],
            sample_user["password"]
        )

        assert user is not None
        assert user.email == sample_user["email"]
```

### 2.5 API Design Skill

#### 功能
REST 和 GraphQL API 设计和文档。

#### 使用方式

```bash
# 设计 API
> 设计一个图书管理 REST API
> 创建用户认证 GraphQL schema

# API 最佳实践
> 按照 REST 最佳实践重构这个接口
> 添加 API 版本控制

# 文档生成
> 生成 OpenAPI 文档
```

#### REST API 设计原则

| 原则 | 说明 | 示例 |
|------|------|------|
| **名词复数** | 资源使用复数形式 | `/users` 而非 `/user` |
| **HTTP 方法** | 正确使用 HTTP 动词 | GET、POST、PUT、DELETE |
| **状态码** | 返回正确的 HTTP 状态码 | 200、201、400、404、500 |
| **版本控制** | API 版本管理 | `/api/v1/users` |
| **过滤排序** | 支持查询参数 | `/users?role=admin&sort=name` |
| **分页** | 大数据集分页 | `/users?page=1&size=20` |

#### API 响应格式

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "id": 123,
    "username": "john_doe",
    "email": "john@example.com"
  },
  "meta": {
    "page": 1,
    "size": 20,
    "total": 100
  }
}
```

---

## 3. 安装和管理 Skills

### 3.1 Skills 位置

| 类型 | 位置 | 作用域 |
|------|------|--------|
| **全局 Skills** | `~/.claude/skills/` | 所有项目可用 |
| **项目 Skills** | `.claude/skills/` | 仅当前项目可用 |

### 3.2 安装官方 Skills

#### 方法 1: Git 克隆

```bash
# 克隆到全局 skills 目录
cd ~/.claude/skills/
git clone https://github.com/anthropics/claude-code-skills.git

# 或克隆到项目目录
cd /path/to/project/.claude/skills/
git clone https://github.com/anthropics/claude-code-skills.git

# 或npx安装
npx skills add wshobson/agents


```

#### 方法 2: 使用 CC-Switch

```bash
# 打开 CC-Switch
cc-switch

# 点击 "Skills" 按钮
# 点击 "发现 Skills"
# 选择仓库 → "安装"
```

#### 方法 3: 手动下载

1. 访问 Skills 仓库
2. 下载单个 `.md` 文件
3. 放到 `~/.claude/skills/` 目录
4. 重启 Claude Code

### 3.3 查看已安装 Skills

```bash
# 列出所有 skills
ls ~/.claude/skills/

# 查看特定 skill
cat ~/.claude/skills/code-review.md
```

### 3.4 更新 Skills

```bash
# Git 方式安装的
cd ~/.claude/skills/claude-code-skills
git pull

# 或重新下载最新版本
```

### 3.5 删除 Skills

```bash
# 删除单个 skill
rm ~/.claude/skills/unwanted-skill.md

# 删除整个仓库
rm -rf ~/.claude/skills/claude-code-skills
```

---

## 4. 开发自定义 Skill

### 4.1 Skill 文件结构

```markdown
---
description: 技能的简短描述
argument-hint: [可选参数说明]
model: haiku  # 可选：指定使用的模型
allowed-tools: ["Read", "Write", "Bash"]  # 可选：允许的工具
---

# Skill 名称

## 技能描述

详细描述这个技能的功能和用途。

## 何时使用

- 场景 1
- 场景 2
- 场景 3

## 执行步骤

1. **步骤 1**: 说明
2. **步骤 2**: 说明
3. **步骤 3**: 说明

## 使用示例

### 示例 1: 场景描述

输入: ...
输出: ...

### 示例 2: 场景描述

输入: ...
输出: ...

## 注意事项

- 注意 1
- 注意 2

## 最佳实践

- 实践 1
- 实践 2
```

### 4.2 实战示例：日志添加 Skill

创建 `.claude/skills/add-logging.md`:

```markdown
---
description: Add comprehensive logging to Python functions
argument-hint: [file path]
allowed-tools: ["Read", "Write", "Edit"]
---

# Add Logging

为 Python 函数添加结构化日志记录。

## 何时使用

- 为现有函数添加日志
- 创建新函数时包含日志
- 调试生产问题
- 审计和监控

## 执行步骤

1. **读取文件**: 分析目标文件中的函数
2. **识别函数**: 找出缺少日志的函数
3. **添加导入**: 如需要，添加 logging 导入
4. **添加日志**: 在关键点添加日志语句
   - 函数入口：记录参数
   - 错误条件：记录错误
   - 函数退出：记录结果

## 日志格式

使用 Python 的 logging 模块：

```python
import logging

logger = logging.getLogger(__name__)

def my_function(param1, param2):
    logger.info("my_function called with param1=%s, param2=%s", param1, param2)
    try:
        # 业务逻辑
        result = do_something(param1, param2)
        logger.info("my_function completed successfully, result=%s", result)
        return result
    except ValueError as e:
        logger.warning("my_function failed with ValueError: %s", str(e))
        raise
    except Exception as e:
        logger.error("my_function failed unexpectedly: %s", str(e), exc_info=True)
        raise
```

## 使用示例

### 示例 1: 添加日志到单个函数

**输入文件** (`utils.py`):

```python
def calculate_discount(price, discount_percent):
    return price * (1 - discount_percent / 100)
```

**输出**:

```python
import logging

logger = logging.getLogger(__name__)

def calculate_discount(price, discount_percent):
    logger.info("calculate_discount called with price=%s, discount_percent=%s",
                price, discount_percent)

    if price < 0:
        logger.error("Invalid price: %s (must be non-negative)", price)
        raise ValueError("Price must be non-negative")

    if discount_percent < 0 or discount_percent > 100:
        logger.warning("Unusual discount_percent: %s (expected 0-100)", discount_percent)

    result = price * (1 - discount_percent / 100)
    logger.info("calculate_discount returning: %s", result)
    return result
```

### 示例 2: 为整个文件添加日志

**输入**:

```python
# user_service.py
class UserService:
    def create_user(self, username, email):
        # 创建用户逻辑
        pass

    def authenticate(self, email, password):
        # 认证逻辑
        pass
```

**输出**:

```python
# user_service.py
import logging

logger = logging.getLogger(__name__)

class UserService:
    def create_user(self, username, email):
        logger.info("Creating user: username=%s, email=%s", username, email)

        try:
            # 创建用户逻辑
            user = self._create_user_in_db(username, email)
            logger.info("User created successfully: id=%s", user.id)
            return user
        except Exception as e:
            logger.error("Failed to create user: %s", str(e), exc_info=True)
            raise

    def authenticate(self, email, password):
        logger.info("Authentication attempt: email=%s", email)

        user = self._find_user_by_email(email)
        if not user:
            logger.warning("Authentication failed: user not found (email=%s)", email)
            return None

        if self._verify_password(user, password):
            logger.info("Authentication successful: user_id=%s", user.id)
            return user
        else:
            logger.warning("Authentication failed: invalid password (email=%s)", email)
            return None
```

## 日志级别指南

| 级别 | 使用场景 | 示例 |
|------|----------|------|
| **DEBUG** | 详细诊断信息 | `logger.debug("Variable value: %s", value)` |
| **INFO** | 正常操作流程 | `logger.info("User created: id=%s", user.id)` |
| **WARNING** | 警告但可继续 | `logger.warning("Cache miss for key: %s", key)` |
| **ERROR** | 错误但不致命 | `logger.error("Failed to send email: %s", error)` |
| **CRITICAL** | 严重错误 | `logger.critical("Database connection lost!")` |

## 最佳实践

1. **使用适当的日志级别**
   - DEBUG: 开发调试信息
   - INFO: 正常业务流程
   - WARNING: 异常但可恢复
   - ERROR: 错误需要关注
   - CRITICAL: 严重问题

2. **包含上下文信息**
   ```python
   # 好
   logger.info("User %s performed action on resource %s", user_id, resource_id)

   # 不好
   logger.info("Action performed")
   ```

3. **不要记录敏感信息**
   ```python
   # 避免
   logger.info("User logged in with password: %s", password)

   # 正确
   logger.info("User logged in: email=%s", email)
   ```

4. **使用 exc_info=True 记录异常**
   ```python
   try:
       risky_operation()
   except Exception as e:
       logger.error("Operation failed: %s", str(e), exc_info=True)
   ```

5. **性能考虑**
   - 避免在循环中频繁日志
   - 使用 lazy 格式化：`logger.info("Value: %s", value)` 而非 `logger.info(f"Value: {value}")`

## 注意事项

- 确保在使用前已配置 logging
- 避免记录敏感数据（密码、token、PII）
- 生产环境考虑日志轮转和存储
- 结构化日志（JSON 格式）更易于解析
```

### 4.3 项目特定 Skills

创建 `.claude/skills/project-standards.md`:

```markdown
---
description: Enforce project-specific coding standards
---

# Project Coding Standards

本项目遵循以下编码规范。

## 命名规范

### 文件命名
- Python: `snake_case.py`
- JavaScript/TypeScript: `camelCase.js` / `PascalCase.ts`
- CSS: `kebab-case.css`

### 代码命名

| 类型 | 规范 | 示例 |
|------|------|------|
| **类名** | PascalCase | `class UserService:` |
| **函数名** | snake_case | `def get_user():` |
| **变量名** | snake_case | `user_id = 123` |
| **常量** | UPPER_SNAKE_CASE | `MAX_RETRIES = 3` |

## 代码组织

### 导入顺序
1. 标准库导入
2. 第三方库导入
3. 本地应用导入

```python
# 正确示例
import os
import sys

from fastapi import FastAPI
from sqlalchemy.orm import Session

from app.models.user import User
from app.services.auth import authenticate
```

### 文档字符串
使用 Google 风格：

```python
def create_user(username: str, email: str) -> User:
    """Create a new user.

    Args:
        username: The desired username (must be unique).
        email: The user's email address (must be unique).

    Returns:
        The created User object.

    Raises:
        ValueError: If username or email already exists.
    """
    pass
```

## 代码审查检查点

审查代码时检查：

### 类型提示
- ✅ 所有函数都有类型提示
- ✅ 使用 Pydantic 模型进行数据验证

### 错误处理
- ✅ 所有外部调用都有 try-except
- ✅ 错误消息清晰明确

### 日志记录
- ✅ 关键操作有日志记录
- ✅ 不记录敏感信息

### 安全性
- ✅ 无硬编码密钥
- ✅ 输入验证和清理
- ✅ SQL 注入防护（参数化查询）

## 测试要求

- 单元测试覆盖率 > 80%
- 所有公共 API 都有测试
- 包含边界条件测试
- Mock 外部依赖

## 提交规范

Commit message 格式：

```
<type>(<scope>): <subject>

<body>

<footer>
```

类型（type）：
- `feat`: 新功能
- `fix`: Bug 修复
- `docs`: 文档更新
- `style`: 代码格式（不影响功能）
- `refactor`: 重构
- `test`: 添加测试
- `chore`: 构建/工具链更新

示例：

```
feat(auth): add JWT token refresh

Implement token refresh mechanism to allow users to
get new access tokens without re-authenticating.

- Add /refresh endpoint
- Implement refresh token rotation
- Update auth middleware

Closes #123
```
```

### 4.4 Skill 开发技巧

#### 1. 清晰的使用场景

```markdown
## 何时使用

✅ 适合场景：
- 场景 1
- 场景 2

❌ 不适合场景：
- 场景 A（应使用其他方法）
- 场景 B
```

#### 2. 具体的示例

```markdown
## 示例

### 输入
```python
# 代码示例
```

### 输出
```python
# 期望结果
```

### 解释
逐步说明发生了什么
```

#### 3. 验证清单

```markdown
## 验证清单

执行后应确认：

- [ ] 检查项 1
- [ ] 检查项 2
- [ ] 检查项 3
```

#### 4. 故障排除

```markdown
## 常见问题

### 问题 1
**症状**: ...
**原因**: ...
**解决**: ...
```

---

## 5. 相关资源

- **官方 Skills**: https://github.com/anthropics/claude-code-skills
- **下一节**: [00204-claudecode-第三方模型.md](./00204-claudecode-第三方模型.md) - 第三方模型集成

