# Claude Code（智谱国内版）安装手册
## 参考文档
- Claude Code 官方文档：https://code.claude.com/docs/zh-CN/sub-agents
- 智谱 Coding Plan 快速开始：https://docs.bigmodel.cn/cn/coding-plan/quick-start

## 1. 核心安装
```bash
# 全局安装 Claude Code 核心包
npm install -g @anthropic-ai/claude-code
```

## 2.1 配置文件设置（二选一执行）
```bash
# 编辑配置文件
vim ~/.claude/setting.json
```
配置内容（替换`your_zhipu_api_key`为实际智谱API Key）：
```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "your_zhipu_api_key",
    "ANTHROPIC_BASE_URL": "https://open.bigmodel.cn/api/anthropic",
    "API_TIMEOUT_MS": "3000000",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": 1,
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "glm-4.5-air",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "glm-4.7",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "glm-4.7"
  }
}
```

## 2.2 图形化界面进行配置 配置文件设置（二选一执行）
```bash
# 启动国内版本交互界面（所有插件命令需在此界面执行）
npx @z_ai/coding-helper
```

## 3. 启动claude code
```shell
# 启动
claude

# 检查url是否切换为https://open.bigmodel.cn/api/anthropic
/status
```

## 4. 前置依赖（插件运行必备）
> 注：以下命令可在本地终端执行，无需进入Claude交互界面
```bash
npm install -g typescript-language-server pyright
```

## 5. 官方插件安装
> 注：以下所有命令需在**已启动的Claude Code交互界面**中执行
```
/plugin marketplace add anthropics/claude-code

/plugin install commit-commands@anthropics-claude-code;
/plugin install pr-review-toolkit@anthropics-claude-code;
/plugin install feature-dev@anthropics-claude-code;
/plugin install code-review@anthropics-claude-code;
/plugin install security-guidance@anthropics-claude-code;
/plugin install ralph-loop@claude-plugins-official;
/plugin install typescript-lsp@anthropics-claude-code;
/plugin install pyright-lsp@anthropics-claude-code
```

## 6. 第三方插件安装
> 注：需在Claude Code交互界面执行
```
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace

/plugin marketplace add jarrodwatts/claude-hud
/plugin install claude-hud
```

## 7. Skills 安装
> 注：需在Claude Code交互界面执行
```
/plugin marketplace add anthropics/skills
/plugin install document-skills@anthropic-agent-skills
/plugin install example-skills@anthropic-agent-skills
/plugin install plugins/frontend-design
```

## 8. 权限跳过（可选）
> 注：可在本地终端执行，也可在Claude交互界面执行
```bash
claude --dangerously-skip-permissions
```

## 9. MCP 服务器配置
> 注：可在本地终端执行，也可在Claude交互界面执行
```bash
# 仅当前项目添加 MCP 服务器
claude mcp add --transport http docs-langchain https://docs.langchain.com/mcp

# 全局添加（所有项目可用）
claude mcp add --transport http docs-langchain --scope user https://docs.langchain.com/mcp
```

## 10. 版本更新
> 注：在本地终端执行
```bash
# npm 方式更新
npm update -g @anthropic-ai/claude-code

# brew 方式更新（若通过brew安装）
brew upgrade claude-code
```

### 11. 补充扩展插件
```shell
# 自动切换不同模型
npm install -g @musistudio/claude-code-router
#  监控token消耗
npm install -g ccusage

# 实时显示部分资源消耗
/plugin marketplace add jarrodwatts/claude-hud
/plugin install claude-hud
```

### 总结
1. 执行所有插件相关命令的前提是**先启动Claude Code交互界面**（`npx @z_ai/coding-helper`）；
2. 配置`setting.json`时需替换真实智谱API Key，前置依赖需在本地终端提前安装；
3. 命令执行环境区分：插件安装命令需在Claude交互界面执行，依赖安装/版本更新在本地终端执行。
