npm install -g @anthropic-ai/claude-code

npx @z_ai/coding-helper
or
vim ~/.claude/setting.json
```
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "your_zhipu_api_key",
    "ANTHROPIC_BASE_URL": "https://open.bigmodel.cn/api/anthropic",
    "API_TIMEOUT_MS": "3000000",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": 1
  }
}
```


老版本
{
  "env": {
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "glm-4.5-air",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "glm-4.7",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "glm-4.7"
  }
}


安装官方插件
```
# 前置条件
npm install -g typescript-language-server pyright


# 在 Claude Code 交互界面中输入
/plugin marketplace add anthropics/claude-code

# 在交互界面中一次性输入以下安装指令


/plugin install commit-commands@anthropics-claude-code;
/plugin install pr-review-toolkit@anthropics-claude-code;
/plugin install feature-dev@anthropics-claude-code;
/plugin install code-review@anthropics-claude-code;
/plugin install security-guidance@anthropics-claude-code;
/plugin install ralph-loop@claude-plugins-official;
/plugin install typescript-lsp@anthropics-claude-code;
/plugin install pyright-lsp@anthropics-claude-code

```




更新所有插件
```
npm update -g @anthropic-ai/claude-code
```
```
brew upgrade claude-code
```




```
/plugin marketplace add obra/superpowers-marketplace

/plugin install superpowers@superpowers-marketplace
# /superpowers:brainstorm - 交互式设计细化

/plugin marketplace add jarrodwatts/claude-hud

/plugin install claude-hud
```




skill
```
/plugin marketplace add anthropics/skills
/plugin install document-skills@anthropic-agent-skills
/plugin install example-skills@anthropic-agent-skills
/plugin install plugins/frontend-design

```







claude --dangerously-skip-permissions
claude --dangerously-skip-permissions



MCP 服务器添加到当前项目/工作目录
```
claude mcp add --transport http docs-langchain https://docs.langchain.com/mcp
```

全局添加 MCP 服务器并在所有项目中访问它
```
claude mcp add --transport http docs-langchain --scope user https://docs.langchain.com/mcp
```