# RTK（Rust Token Killer）完整安装使用文档

## 1. 工具介绍

RTK（Rust Token Killer）是一款面向 AI 对话（尤其 Claude / Claude Code）的终端指令优化工具，通过自动压缩命令、精简输出、过滤冗余信息，大幅降低 Token 消耗，提升交互速度，长期使用可显著节省 API 额度。

## 2. 适用系统

- Linux x86_64（Debian / Ubuntu / Deepin 等主流发行版）
- 已安装 Rust 环境（无则先安装）

## 3. 前置：安装 Rust 环境

如果未安装 Rust，执行以下命令：

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```

## 4. 安装 RTK

### 4.1 国外网络（官方脚本）

```bash
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/master/install.sh | sh
```

### 4.2 国内网络（代理加速）

```bash
curl -fsSL https://gh-proxy.com/https://raw.githubusercontent.com/rtk-ai/rtk/master/install.sh | sh
```

### 4.3 国内无代理（源码编译，最稳定）

```bash
git clone https://gitee.com/rtk-ai-mirror/rtk.git
cd rtk
cargo install --path .
```

## 5. 初始化（必执行）

```bash
rtk init --global --auto-patch
```

作用：全局挂载钩子，自动优化所有终端指令与 AI 交互内容。

## 6. 常用命令

```bash
# 查看 Token 节省统计
rtk gain

# 查看详细统计
rtk stats

# 手动优化读取文件
rtk read 文件名

# 卸载钩子
rtk init --uninstall

# 查看帮助
rtk --help
```

## 7. 效果示例

```
RTK Token Savings (Global Scope)
Total commands:    19
Input tokens:      1.7K
Output tokens:     1.2K
Tokens saved:      410 (24.8%)
```

## 8. 常见问题

- `cargo: command not found`：未安装 Rust，执行第 3 步
- GitHub 下载超时：使用国内源码安装方式
- `rtk` 命令不存在：执行 `source $HOME/.cargo/env`
- 不生效：重新执行 `rtk init --global --auto-patch` 并重启终端

---

## 极简一键安装脚本（国内直接复制运行）

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source $HOME/.cargo/env
cargo uninstall rtk 2>/dev/null
git clone https://gitee.com/rtk-ai-mirror/rtk.git
cd rtk && cargo install --path .
rtk init --global --auto-patch
rtk gain
```
