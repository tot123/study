# Docker 安装 Ollama 教程

> 本教程介绍如何在 Docker 环境中安装和配置 Ollama，包括 NVIDIA GPU 支持和国内镜像加速配置。

## 📋 目录

- [1. 安装 NVIDIA Container Toolkit](#1-安装-nvidia-container-toolkit)
- [2. 安装 Ollama](#2-安装-ollama)
- [3. 测试 Ollama](#3-测试-ollama)
- [4. 配置国内加速](#4-配置国内加速)
- [5. 启动模型](#5-启动模型)
- [6. 指定显卡启动模型](#6-指定显卡启动模型)

---

## 1. 安装 NVIDIA Container Toolkit

### 1.1 配置安装源

```shell
curl -s -L https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo | \
sudo tee /etc/yum.repos.d/nvidia-container-toolkit.repo
```

### 1.2 安装工具包

```shell
sudo yum install -y nvidia-container-toolkit
```

### 1.3 配置 Docker 运行时

```shell
# 配置 Docker 使用 NVIDIA 运行时
sudo nvidia-ctk runtime configure --runtime=docker

# 重启 Docker 服务
sudo systemctl restart docker
```

### 1.4 验证安装

```shell
docker run --rm --gpus all nvidia/cuda nvidia-smi
```

> 💡 **提示**: 如果看到 GPU 信息输出，说明安装成功。

---

## 2. 安装 Ollama

### 2.1 创建数据目录

```shell
mkdir -p ~/docker/ollama
```

### 2.2 启动 Ollama 容器

```shell
docker run -d \
  --gpus=all \
  -v ~/docker/ollama:/root \
  -p 11434:11434 \
  --name ollama \
  swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/ollama/ollama:latest
```

**参数说明**:
- `--gpus=all` - 使用所有可用的 GPU
- `-v ~/docker/ollama:/root` - 持久化数据目录
- `-p 11434:11434` - 端口映射
- `--name ollama` - 容器名称

---

## 3. 测试 Ollama

### 3.1 进入容器

```shell
docker exec -it ollama bash
```

### 3.2 验证 GPU 支持

```shell
# 查看 GPU 状态
nvidia-smi

# 检查 NVIDIA 设备
ls -la /dev | grep nvidia

# 检查 CUDA 库
ldconfig -p | grep cuda
```

---

## 4. 配置国内加速

> ⚠️ **注意**: 使用国内镜像可以大幅提升模型下载速度。

### 4.1 可用镜像源

| 镜像源 | 地址 |
|--------|------|
| 阿里云 | `https://registry.ollama.ai` |
| DeepSeek 官方 | `https://ollama.deepseek.com` |
| 浙江大学 | `https://ollama.zju.edu.cn` |
| 魔搭社区 | `https://ollama.modelscope.cn` |

### 4.2 配置镜像加速

```shell
# 创建配置目录
mkdir -p ~/.ollama

# 写入配置文件
cat << EOF > ~/.ollama/config.json
{
    "registry": {
        "mirrors": {
            "registry.ollama.ai": "https://registry.ollama.ai"
        }
    }
}
EOF

# 验证配置
cat ~/.ollama/config.json
```

> 💡 **提示**: 可将 `https://registry.ollama.ai` 替换为上述其他镜像源地址。

---

## 5. 启动模型

### 5.1 运行默认模型

```shell
ollama run qwen2.5:7b
```

### 5.2 常用模型

| 模型 | 命令 |
|------|------|
| Qwen2.5 7B | `ollama run qwen2.5:7b` |
| Llama 3.1 | `ollama run llama3.1` |
| DeepSeek | `ollama run deepseek-r1` |

---

## 6. 指定显卡启动模型

### 6.1 单显卡指定

```shell
# 使用 GPU 0
CUDA_VISIBLE_DEVICES=0 ollama run qwen2.5:7b

# 使用 GPU 1
CUDA_VISIBLE_DEVICES=1 ollama run qwen2.5:7b
```

### 6.2 多显卡指定

```shell
# 使用 GPU 0 和 1
CUDA_VISIBLE_DEVICES=0,1 ollama run qwen2.5:7b

# 使用所有 GPU（默认）
CUDA_VISIBLE_DEVICES=all ollama run qwen2.5:7b
```

---

## 7. Ollama 是什么？

**Ollama** 是一个开源的大语言模型运行工具，旨在让用户能够**在本地运行、管理和使用各种 LLM（大语言模型）**。

### 7.1 核心特性

| 特性 | 说明 |
|------|------|
| 🔄 **模型管理** | 支持下载、更新、删除多个开源模型 |
| 💻 **本地运行** | 所有模型在本地运行，数据隐私有保障 |
| 🎯 **简单易用** | 提供 CLI 和 API，一行命令即可运行模型 |
| 🐳 **Docker 支持** | 官方提供 Docker 镜像，便于部署 |
| 🌐 **跨平台** | 支持 Windows、macOS、Linux |
| 🔌 **API 兼容** | 提供 OpenAI 兼容的 API 接口 |

### 7.2 为什么选择 Ollama？

1. **隐私安全**：所有数据在本地处理，不会上传到云端
2. **成本可控**：无需付费 API 调用，一次下载无限使用
3. **离线可用**：模型下载后可完全离线使用
4. **灵活集成**：可通过 API 集成到各种应用中
5. **多模型支持**：支持 Llama、Qwen、Mistral、DeepSeek 等主流开源模型

---

## 8. 常用模型列表

### 8.1 推荐模型

| 模型名称 | 参数量 | 特点 | 适用场景 |
|---------|-------|------|---------|
| **qwen2.5** | 7B/14B/32B | 中文能力强，代码生成优秀 | 通用对话、编程、文档处理 |
| **llama3.1** | 8B/70B | Meta 官方，推理能力强 | 英文对话、逻辑推理 |
| **deepseek-r1** | 14B/32B | DeepSeek 推理模型 | 复杂推理、数学问题 |
| **mistral** | 7B | 轻量高效，响应速度快 | 资源受限环境 |
| **gemma2** | 9B/27B | Google 开源，多语言支持 | 多语言对话、翻译 |

### 8.2 查看所有可用模型

```shell
# 访问官方模型库
# https://ollama.com/library

# 或在容器内搜索
ollama list
```

---

## 9. Ollama 常用命令

### 9.1 模型管理

```shell
# 拉取模型（首次运行会自动下载）
ollama pull qwen2.5:7b

# 查看已下载的模型
ollama list

# 删除模型
ollama rm qwen2.5:7b

# 查看模型信息
ollama show qwen2.5:7b
```

### 9.2 运行模型

```shell
# 交互式对话
ollama run qwen2.5:7b

# 带提示语运行
ollama run qwen2.5:7b "介绍一下 Python"

# 从文件读取提示
ollama run qwen2.5:7b "$(cat prompt.txt)"
```

### 9.3 API 调用

Ollama 提供 REST API，默认端口 `11434`：

```bash
# 聊天接口
curl http://localhost:11434/api/chat -d '{
  "model": "qwen2.5:7b",
  "messages": [
    { "role": "user", "content": "你好" }
  ]
}'

# 生成接口
curl http://localhost:11434/api/generate -d '{
  "model": "qwen2.5:7b",
  "prompt": "写一首诗"
}'
```

### 9.4 容器管理

```shell
# 启动容器
docker start ollama

# 停止容器
docker stop ollama

# 重启容器
docker restart ollama

# 查看日志
docker logs -f ollama

# 进入容器
docker exec -it ollama bash
```

---

## 10. 进阶配置

### 10.1 自定义模型参数

```shell
# 调整温度（0-1，越高越随机）
ollama run qwen2.5:7b --temperature 0.7

# 调整最大输出长度
ollama run qwen2.5:7b --num_ctx 4096

# 调整 Top-K 采样
ollama run qwen2.5:7b --top_k 40

# 调整 Top-P 采样
ollama run qwen2.5:7b --top_p 0.9
```

### 10.2 持久化配置

创建 `Modelfile` 自定义模型：

```dockerfile
FROM qwen2.5:7b

# 设置系统提示
SYSTEM You are a helpful AI assistant specialized in programming.

# 调整参数
PARAMETER temperature 0.7
PARAMETER num_ctx 4096
```

构建自定义模型：

```shell
ollama create mycoder -f Modelfile
ollama run mycoder
```

### 10.3 性能优化

```shell
# 使用 GPU 指定（多显卡环境）
CUDA_VISIBLE_DEVICES=0 ollama run qwen2.5:7b

# 设置运行时线程数
OLLAMA_NUM_THREAD=8 ollama run qwen2.5:7b

# 调整 GPU 内存使用
OLLAMA_GPU_OVERHEAD=0 ollama run qwen2.5:7b
```

---

## 11. 实际应用场景

### 11.1 代码助手

```shell
# 运行编程专用模型
ollama run qwen2.5-coder:7b

# 交互式编程
> 请用 Python 写一个快速排序算法
> 解释这段代码的作用
> 如何优化这段代码？
```

### 11.2 文档问答

```shell
# 结合文档进行问答
ollama run qwen2.5:7b "根据以下文档回答问题：$(cat document.md)"
```

### 11.3 API 集成示例

Python 集成：

```python
import requests
import json

def chat_with_ollama(prompt, model="qwen2.5:7b"):
    url = "http://localhost:11434/api/chat"

    payload = {
        "model": model,
        "messages": [
            {"role": "user", "content": prompt}
        ],
        "stream": False
    }

    response = requests.post(url, json=payload)
    return response.json()['message']['content']

# 使用
result = chat_with_ollama("解释什么是机器学习")
print(result)
```

### 11.4 与 Claude Code 集成

通过 CCR (Claude Code Router) 将 Ollama 作为本地推理后端：

```json
{
  "Providers": [
    {
      "name": "ollama",
      "api_base_url": "http://localhost:11434/v1/chat/completions",
      "api_key": "ollama",
      "models": ["qwen2.5-coder:latest"]
    }
  ],
  "Router": {
    "background": "ollama,qwen2.5-coder:latest"
  }
}
```

---

## 12. 常见问题

### Q1: 模型下载速度慢怎么办？

**A**: 使用国内镜像加速配置（见第 4 节），或手动下载模型文件。

### Q2: 如何清理模型缓存？

```shell
# 查看模型占用空间
du -sh ~/docker/ollama/.ollama/models/*

# 删除不需要的模型
ollama rm <model-name>

# 清理所有缓存
rm -rf ~/docker/ollama/.ollama/models/*
```

### Q3: GPU 内存不足怎么办？

**A**:
- 选择更小的模型（如 7B 参数）
- 调整 `num_ctx` 减小上下文长度
- 使用量化版本模型（如 `:q4_0` 后缀）

### Q4: 如何设置开机自启？

```shell
# 设置 Docker 容器自启动
docker update --restart=always ollama

# 或使用 systemd 管理
sudo systemctl enable docker
```

### Q5: Ollama 和官方 API 有什么区别？

| 对比项 | Ollama | 官方 API |
|--------|--------|----------|
| **成本** | 免费（本地运行） | 按 token 计费 |
| **隐私** | 完全本地 | 数据上传云端 |
| **性能** | 依赖本地硬件 | 云端高性能 |
| **模型** | 开源模型 | Claude、GPT 等 |
| **网络** | 可离线使用 | 需要网络连接 |

---

## 13. 参考资源

- 📚 [Ollama 官方文档](https://github.com/ollama/ollama)
- 🤖 [Ollama 模型库](https://ollama.com/library)
- 💻 [Ollama API 文档](https://github.com/ollama/ollama/blob/main/docs/api.md)
- 🐳 [Docker Hub - Ollama](https://hub.docker.com/r/ollama/ollama)

---

**文档版本**: v1.0
**最后更新**: 2026-01-26
**作者**: Your Name
