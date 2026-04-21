# Claude Code - 接入第三方模型

> 本指南介绍如何将 DeepSeek、智谱 GLM、Ollama 等第三方模型集成到 Claude Code，以降低使用成本并提高灵活性。

## 📋 目录

- [1. 为什么需要第三方模型](#1-为什么需要第三方模型)
- [2. DeepSeek 集成](#2-deepseek-集成)
- [3. 智谱 GLM 集成](#3-智谱-glm-集成)
- [4. 本地模型集成 (Ollama)](#4-本地模型集成-ollama)
- [5. 其他第三方模型](#5-其他第三方模型)

---

## 1. 为什么需要第三方模型

### 1.1 官方模型的限制

| 限制 | 说明 | 影响 |
|------|------|------|
| **成本高** | 按 token 计费 | 长期使用费用显著 |
| **额度限制** | 每日/每周调用限制 | 大型项目受限 |
| **灵活性差** | 仅支持 Claude 模型 | 无法使用其他模型 |
| **网络依赖** | 必须联网 | 离线环境无法使用 |

### 1.2 第三方模型的优势

| 优势 | 说明 | 示例 |
|------|------|------|
| **成本可控** | 更低的 token 价格 | DeepSeek 约 1/10 价格 |
| **无限额度** | 按需付费，无限制 | 适合大规模项目 |
| **本地运行** | 隐私保护，无网络 | Ollama + 本地 GPU |
| **模型多样** | 选择最适合的模型 | 编程用 Qwen，推理用 DeepSeek-R1 |
| **离线可用** | 完全本地部署 | 敏感数据处理 |

### 1.3 解决方案对比

| 方案 | 适用场景 | 优缺点 |
|------|----------|--------|
| **直接替换** | 单一模型使用 | ✅ 简单<br>❌ 无法切换 |
| **环境变量切换** | 多模型切换 | ✅ 灵活<br>❌ 手动切换 |
| **CCR 路由** | 智能模型选择 | ✅ 自动优化<br>❌ 配置复杂 |
| **CC-Switch** | 可视化管理 | ✅ 易用<br>❌ 仅本地 |

---

## 2. DeepSeek 集成

### 2.1 DeepSeek 简介

**DeepSeek** 是国内领先的 AI 模型提供商，提供与 Claude API 兼容的接口。

| 模型 | 参数 | 特点 | 价格 |
|------|------|------|------|
| **deepseek-chat** | - | 通用对话 | ¥1/M tokens |
| **deepseek-coder** | - | 代码生成 | ¥1/M tokens |
| **deepseek-reasoner** | - | 复杂推理 | ¥1/M tokens |

### 2.2 获取 API Key

1. 访问: https://platform.deepseek.com/
2. 注册/登录账户
3. 进入 API Keys 页面
4. 创建新的 API Key

### 2.3 Bash/Zsh 配置

添加到 `~/.bashrc` 或 `~/.zshrc`:

```bash
# DeepSeek API 配置
export ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
export ANTHROPIC_AUTH_TOKEN=sk-xxxxxxxxxxxxxxxxxxxxxxxx
export API_TIMEOUT_MS=600000
export ANTHROPIC_MODEL=deepseek-chat
export ANTHROPIC_SMALL_FAST_MODEL=deepseek-chat
export CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1

# 可选：使用推理模型
# export ANTHROPIC_SMALL_FAST_MODEL=deepseek-reasoner
```

### 2.4 Fish 配置

添加到 `~/.config/fish/config.fish`:

```fish
# DeepSeek API 配置
set -gx ANTHROPIC_BASE_URL https://api.deepseek.com/anthropic
set -gx ANTHROPIC_AUTH_TOKEN sk-xxxxxxxxxxxxxxxxxxxxxxxx
set -gx API_TIMEOUT_MS 600000
set -gx ANTHROPIC_MODEL deepseek-chat
set -gx ANTHROPIC_SMALL_FAST_MODEL deepseek-chat
set -gx CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC 1
```

### 2.5 验证配置

```bash
# 重新加载配置
source ~/.bashrc  # Bash/Zsh
# 或
source ~/.config/fish/config.fish  # Fish

# 启动 Claude Code
claude

# 在 Claude Code 中查看状态
> /status
```

### 2.6 使用说明

配置完成后，所有模型调用将自动转向 DeepSeek：

```bash
# 直接使用，无需指定模型
> 创建一个用户认证系统

# 自动使用 deepseek-chat
```

### 2.7 性能对比

| 指标 | Claude Sonnet | DeepSeek Chat |
|------|--------------|---------------|
| **响应速度** | 快 | 快 |
| **代码质量** | 优秀 | 优秀 |
| **中文理解** | 良好 | 优秀 |
| **价格** | $3/M tokens | ¥1/M tokens |
| **适用场景** | 通用 | 编程、中文 |

---

## 3. 智谱 GLM 集成

### 3.1 智谱 GLM 简介

**智谱 AI** 提供的 GLM 系列模型，支持中文和多语言任务。

| 模型 | 参数 | 特点 |
|------|------|------|
| **glm-4.5-air** | - | 快速响应，适合简单任务 |
| **glm-4.6** | - | 通用模型，平衡性能 |
| **glm-4-plus** | - | 高性能，复杂任务 |
| **glm-4-flash** | - | 极速响应 |

### 3.2 获取 API Key

1. 访问: https://open.bigmodel.cn/
2. 注册/登录账户
3. 进入 API Keys 页面
4. 创建新的 API Key

### 3.3 自动配置（推荐）

```bash
curl -fsSL "https://cdn.bigmodel.cn/install/claude_code_env.sh" | bash
```

脚本会自动修改 `~/.claude/settings.json`。

### 3.4 手动配置

#### 方式 1: settings.json

编辑 `~/.claude/settings.json`:

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "your_zhipu_api_key",
    "ANTHROPIC_BASE_URL": "https://open.bigmodel.cn/api/anthropic",
    "API_TIMEOUT_MS": "3000000",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "glm-4.5-air",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "glm-4.6",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "glm-4.6"
  }
}
```

#### 方式 2: 环境变量

```bash
export ANTHROPIC_AUTH_TOKEN=your_zhipu_api_key
export ANTHROPIC_BASE_URL=https://open.bigmodel.cn/api/anthropic
export API_TIMEOUT_MS=3000000
export ANTHROPIC_DEFAULT_HAIKU_MODEL=glm-4.5-air
export ANTHROPIC_DEFAULT_SONNET_MODEL=glm-4.6
export ANTHROPIC_DEFAULT_OPUS_MODEL=glm-4.6
```

### 3.5 国际版配置

如果使用国际版 API：

```bash
export ANTHROPIC_BASE_URL=https://api.z.ai/api/anthropic
```

### 3.6 验证配置

```bash
# 重启 Claude Code
claude

# 查看状态
> /status

# 测试对话
> 你好，请介绍一下你自己
```

---

## 4. 本地模型集成 (Ollama)

### 4.1 为什么使用本地模型？

| 优势 | 说明 |
|------|------|
| **完全免费** | 无 API 调用费用 |
| **隐私保护** | 数据不离开本地 |
| **离线可用** | 无需网络连接 |
| **可控性** | 完全控制模型版本 |

### 4.2 Ollama + CCR 集成

Ollama 本身不直接兼容 Claude API，需要通过 CCR 转换。

#### 步骤 1: 安装 Ollama

参考: [00001-docker安装ollama.md](./00001-docker安装ollama.md)

#### 步骤 2: 配置 CCR

创建 `~/.claude-code-router/config.json`:

```json
{
  "APIKEY": "your-secret-key",
  "LOG": true,
  "API_TIMEOUT_MS": 600000,
  "Providers": [
    {
      "name": "ollama",
      "api_base_url": "http://localhost:11434/v1/chat/completions",
      "api_key": "ollama",
      "models": [
        "qwen2.5-coder:latest",
        "qwen2.5:7b",
        "deepseek-r1:14b"
      ]
    }
  ],
  "Router": {
    "default": "ollama,qwen2.5-coder:latest",
    "background": "ollama,qwen2.5-coder:latest",
    "coding": "ollama,qwen2.5-coder:latest"
  }
}
```

#### 步骤 3: 启动 CCR

```bash
# 启动 CCR 服务
ccr code

# 或后台运行
nohup ccr code > /dev/null 2>&1 &
```

#### 步骤 4: 配置 Claude Code

```bash
export ANTHROPIC_BASE_URL=http://localhost:8080/anthropic
export ANTHROPIC_AUTH_TOKEN=your-secret-key
```

### 4.3 推荐的本地模型

| 模型 | 参数量 | 显存需求 | 适用场景 |
|------|--------|----------|----------|
| **qwen2.5-coder:7b** | 7B | ~8GB | 代码生成、编程 |
| **qwen2.5:14b** | 14B | ~16GB | 通用对话、中文 |
| **deepseek-r1:14b** | 14B | ~16GB | 复杂推理 |
| **codellama:13b** | 13B | ~16GB | 代码补全 |

### 4.4 性能优化

#### GPU 指定

```json
{
  "Providers": [
    {
      "name": "ollama",
      "env": {
        "CUDA_VISIBLE_DEVICES": "0"  // 使用 GPU 0
      }
    }
  ]
}
```

#### 量化模型

使用量化版本减少显存占用：

```bash
# 4-bit 量化
ollama run qwen2.5:7b-q4_0

# 8-bit 量化
ollama run qwen2.5:7b-q8_0
```

### 4.5 本地模型 vs 云端模型

| 对比项 | 本地模型 | 云端模型 |
|--------|----------|----------|
| **成本** | 免费（硬件成本） | 按 token 计费 |
| **性能** | 依赖本地硬件 | 云端高性能 |
| **隐私** | 完全本地 | 数据上传 |
| **速度** | 快（无网络延迟） | 取决于网络 |
| **质量** | 7B-14B 模型 | 可用更大模型 |
| **维护** | 需要维护 | 无需维护 |

---

## 5. 其他第三方模型

### 5.1 OpenRouter

**OpenRouter** 聚合了多个模型的访问平台。

#### 配置

```bash
export ANTHROPIC_BASE_URL=https://openrouter.ai/api/v1/chat/completions
export ANTHROPIC_AUTH_TOKEN=sk-or-xxxxxx
```

#### 可用模型

- `google/gemini-2.5-pro-preview`
- `anthropic/claude-sonnet-4`
- `meta-llama/llama-3.1-70b`
- `mistralai/mistral-large`

### 5.2 阿里云百炼

```bash
export ANTHROPIC_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
export ANTHROPIC_AUTH_TOKEN=sk-xxxxxx
export ANTHROPIC_MODEL=qwen-plus
```

### 5.3 火山引擎

```bash
export ANTHROPIC_BASE_URL=https://ark.cn-beijing.volces.com/api/v3
export ANTHROPIC_AUTH_TOKEN=xxxxxx
```

### 5.4 Together AI

```bash
export ANTHROPIC_BASE_URL=https://api.together.xyz/v1/chat/completions
export ANTHROPIC_AUTH_TOKEN=xxxxxx
```

---

## 6. 模型选择建议

### 6.1 按任务类型选择

| 任务类型 | 推荐模型 | 原因 |
|----------|----------|------|
| **代码生成** | DeepSeek-Coder / Qwen2.5-Coder | 专门训练，代码质量高 |
| **中文对话** | DeepSeek / GLM-4.6 | 中文理解优秀 |
| **复杂推理** | DeepSeek-R1 / Claude Opus | 推理能力强 |
| **快速原型** | 本地 Qwen2.5:7B | 响应快，成本低 |
| **生产环境** | Claude Sonnet 3.5 | 质量稳定 |

### 6.2 按成本选择

| 预算 | 方案 | 月成本估算 |
|------|------|------------|
| **免费** | 本地 Ollama (7B) | 0 元 |
| **低成本** | DeepSeek API | ¥50-200 |
| **中等** | GLM-4.6 API | ¥200-500 |
| **高预算** | Claude Official | $100-500 |

### 6.3 混合策略

```bash
# 简单任务用本地模型
export BACKGROUND_MODEL=ollama,qwen2.5:7b

# 复杂任务用云端模型
export THINKING_MODEL=deepseek,deepseek-reasoner

# 编程用代码专用模型
export CODING_MODEL=deepseek,deepseek-coder
```

---

## 7. 故障排除

### 7.1 常见问题

#### 问题 1: API 调用失败

**症状**: `Error: 401 Unauthorized`

**解决**:
```bash
# 检查 API Key
echo $ANTHROPIC_AUTH_TOKEN

# 验证 API Key
curl -H "x-api-key: $ANTHROPIC_AUTH_TOKEN" \
     https://api.deepseek.com/v1/models
```

#### 问题 2: 超时错误

**症状**: `Error: Request timeout`

**解决**:
```bash
# 增加超时时间
export API_TIMEOUT_MS=600000  # 10 分钟
```

#### 问题 3: 响应质量差

**原因**: 模型不适合任务

**解决**: 切换到更适合的模型

```bash
# 从通用模型切换到代码模型
export ANTHROPIC_MODEL=deepseek-coder
```

### 7.2 调试技巧

```bash
# 查看当前配置
> /status

# 查看环境变量
env | grep ANTHROPIC

# 测试 API 连接
curl https://api.deepseek.com/v1/models \
  -H "Authorization: Bearer $ANTHROPIC_AUTH_TOKEN"
```

---

## 8. 相关资源

- **DeepSeek**: https://platform.deepseek.com/
- **智谱 GLM**: https://open.bigmodel.cn/
- **Ollama**: https://ollama.com/
- **OpenRouter**: https://openrouter.ai/
- **下一节**: [00205-claudecode-智能路由.md](./00205-claudecode-智能路由.md) - 智能路由系统

