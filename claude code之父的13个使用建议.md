# Claude Code 之父首次公开:我每天是这样用 AI 写代码的

作者：Alex Hu  
发布时间：2026-01-03 18:43:00

很多人问我是怎么使用 Claude Code 的，所以今天我想展示一下我的工作方式。

说实话，我的设置可能出人意料地"原生"——Claude Code 开箱即用效果就很好，所以我个人并没有做太多定制。

使用 Claude Code 没有唯一正确的方式。我们特意将它构建成可以让你按照自己喜欢的方式使用、定制和修改的工具。Claude Code 团队的每个人使用方式都非常不同。

下面是我的 13 条使用心得，希望对你有帮助。

## 1. 在终端并行运行多个 Claude
我在终端中**并行运行 5 个 Claude**，把标签页编号为 1-5，并使用系统通知来了解何时某个 Claude 需要我的输入。

这样做的好处是：当一个 Claude 在思考或执行任务时，我可以切换到另一个 Claude 继续工作，大大提高了效率。

配置系统通知的方法可以参考官方文档的 iTerm2 配置部分。

```bash
# 示例操作日志
Fix Forked Skill Progress
31 2 32 3 33 4 34 5 x5
Fix Forked Skill Progress (node) . fswatch
Added 1 line,removed1 line
12import type{z}from 'zod/v4'
aude-cli-native-darwin-arm64/package.json
13 import type {inputschema }from "./SkilLTool.js
14 import type{ Command}from'../commands.js'
aude-cli-native/package.json build-ant-native/anthropic-ai/ct
15 -import type { ProgressMessage,Message } from'../.types/message.js
15+import type{ ProgressMessage} from'../../types/message.js
16import type{ Tool}from'../../../Tool.js'
18 type Input = z.infertypeof inputSchema>
Build completed successfully!
claude-cli$0 *Math Prblen(-zh)
Now let me run typecheck to verify the changes:
Bash(bun run typecheck 2>81 | head -100)
$ bun run build:agent-sdk-typings
$ tsc
$./scripts/build-ant-sdk-typings.sh
Generating Agent SDK typings...
Generating Agent SDK tool typings ...
The typecheck passed.Now Let me run the Linter on the modified files:
Bash(bun run lint:file--src/tools/SkillTool/SkillTool.ts src/tools/SkillTool/UI.tsx src/Tool.ts2>61)
timeout:1m 0s
$ eslint--fix--no-error-on-unmatched-pattern -cache
(5s)
ctrl+b to run in background
Running typecheck and tests to verify changes..(esc to interrupt·2m 22s)
accept edits on (shift+tab to cycle)
```

## 2. 多端协作：终端 + 网页 + 手机
除了本地终端，我还会在 claude.ai/code 上**同时运行 5-10 个 Claude**，与本地 Claude 并行工作。

我的日常工作流是这样的：
- 在终端编码时，经常把本地会话**移交到网页端**（使用 `&` 命令）
- 有时在 Chrome 中手动启动会话
- 用 `--teleport` 在本地和网页端之间来回切换
- 每天早上和一整天都会从手机上（通过 Claude iOS 应用）启动几个会话，之后再回来查看进度

这种多端协作的方式，让我可以随时随地推进工作，不会被单一设备限制。

> 示例会话列表（claude.ai/code）：
> - Verify guide agent URLs are correct（claude-cli-internal · 10:42 am）
> - Deprecate statsync and migrate to read/write（claude-cli-internal · 10:41 am）
> - Add /fuzz command with parallel agent research（claude-cli-internal · 10:42 am）
> - Fix background bash freezing issue（claude-cli-internal · 10:40 am）

## 3. 模型选择：始终使用 Opus 4.5 + 思考模式
我对所有任务都使用**带有思考模式的 Opus 4.5**。这是我用过的最好的编程模型。尽管它比 Sonnet 更大更慢，但因为你需要**更少的引导**，而且它的**工具使用能力更强**，最终几乎总是比使用更小的模型更快。

很多人担心大模型会慢，但实际上：
- 小模型需要更多的来回沟通
- 小模型更容易出错，需要修正
- 算下来，大模型反而更省时间

建议：如果你还在用 Sonnet，不妨试试 Opus 4.5，体验一下"少操心"的感觉。

## 4. 团队协作：共享 CLAUDE.md
我们团队在 Claude Code 仓库中**共享一个 CLAUDE.md 文件**，已提交到 git，整个团队每周都会贡献多次。

核心做法：每当我们发现 Claude 做错了什么，就会把它添加到 CLAUDE.md 中，这样 Claude 下次就知道不要这样做了。

这就像是给 Claude 建立了一个"错题本"——团队积累的经验会自动传递给每一次新的对话。

其他团队也维护他们自己的 CLAUDE.md，每个团队都有责任保持文件是最新的。

```markdown
# CLAUDE.md 示例内容
# Development Workflow
**Always use "bun",not "npm"**
```sh
# 1.Make changes
# 2.Typecheck(fast)
bun run typecheck
# 3.Run tests
bun run test---t"test name" # Single suite
bun run test:file -"glob" #Specific files
# 4.Lint before committing
bun run lint:file -"file1.ts" # Specific files
bun run lint #All files
# 5. Before creating PR
bun run Lint:claude
bun run test
```
```

## 5. 代码审查中使用 @.claude
在代码审查过程中，我经常会**在同事的 PR 上标记 @.claude**，作为 PR 的一部分向 CLAUDE.md 添加内容。

我们使用 Claude Code GitHub Action（通过 `/install-github-action` 安装）来实现这一点。

这其实是一种 "复利工程" 的实践——每次代码审查都是一次让 Claude 变得更聪明的机会。长期下来，这些小改进会累积成巨大的效率提升。

> 示例 PR 互动：
> - bcherny 3 days ago: nit: use a string literal, not ts enum
> - @claude add to CLAUDE.md to never use enums, always prefer literal unions
> - claude bot 3 days ago·edited: Claude finished @bcherny's task in 39s--View job
>   - Adding enum guidance to CLAUDE.md
>   - √ Read current CLAUDE.md to understand existing guidance
>   - Update CLAUDE.md to strengthen "no enums"guidance
>   - Commit the change
>   - Updated CLAUDE.md line 36 from:
>     - Prefer 'type' over 'interface'; avoid 'enum(use string unions)
>     - to:
>     - Prefer 'type' over 'interface;**never use 'enum**(use string literal unions instead)
>   - Committed in 156a458.

## 6. Plan 模式是我的标配
大多数会话都从 Plan 模式开始（按两次 `Shift+Tab`）。

我的典型工作流是这样的：
- 规划阶段：使用 Plan 模式，与 Claude 反复讨论直到我满意它的计划
- 执行阶段：切换到自动接受编辑模式（按 `Shift+Tab`），Claude 通常可以一次性完成

一个好的计划真的很重要！很多人直接让 Claude 开始写代码，结果发现方向错了要推翻重来。先花几分钟确认计划，能省下大量返工时间。

> 示例指令：i want to improve progress notification rendering for skills.can you make it Look and feel a bit more like subagent progress?
> （plan mode on (shift+tab to cycle)）

## 7. 自定义 Slash 命令：让高频操作一键完成
我对每天要做很多次的 "内循环"工作流都使用 Slash 命令。

这样做的好处：
- 避免重复输入相同的提示
- Claude 也能使用这些工作流
- 命令已提交到 git，存放在 `.claude/commands/` 目录下

举个例子：我每天都会使用 `/commit-push-pr` 命令几十次。这个命令使用内联 bash 预先计算 git status 和其他信息，使命令快速运行，避免与模型来回交互。

大多数 Slash 命令都只有一行，用内联 bash 让编写尽可能高效。

> 常用命令：/commit-push-pr（功能：Commit,push,and open a PR）

## 8. 子代理：自动化常见工作流
我经常使用几个**子代理（Subagents）**：

| 子代理 | 功能 |
|--------|------|
| code-simplifier | 在 Claude 完成工作后简化代码 |
| verify-app | 包含端到端测试 Claude Code 的详细指令 |

与 Slash 命令类似，我把子代理看作是**自动化我在大多数 PR 中执行的最常见工作流**。这就像是给 Claude 配备了专门的"助手"，每个助手负责特定的任务，让整个工作流更加流畅。

> 子代理配置文件示例：
> .claude/agents/
> - build-validator.md
> - code-architect.md
> - code-simplifier.md
> - oncall-guide.md
> - verify-app.md

## 9. 使用 Hook 自动格式化代码
我们使用 PostToolUse hook 来格式化 Claude 的代码。

Claude 通常会生成格式良好的代码，hook 只是处理**最后 10% 的格式问题**，以避免之后在 CI 中出现格式错误。

这是一个很小的优化，但能省去很多手动调整格式的麻烦。

```json
{
  "PostToolUse": {
    "matcher": "Write|Edit",
    "hooks": [
      {
        "type": "command",
        "command": "bun run format",
        "always": true
      }
    ]
  }
}
```

## 10. 权限配置：精细控制而非一刀切
我不使用 `--dangerously-skip-permissions`。

相反，我使用 `/permissions` 预先允许我知道在我的环境中是安全的常用 bash 命令，以避免不必要的权限提示。

大部分配置都已提交到 `.claude/settings.json` 并与团队共享。

为什么不跳过所有权限？因为我想知道 Claude 是否做了什么意外的事情。完全跳过权限虽然方便，但可能会错过一些需要人工判断的操作。精细控制是更安全的选择。

> 权限配置示例：
> /permissions
> Permissions: Allow Ask Deny Workspace(←/>)
> Claude Code won't ask before using allowed tools.
> 12. Bash(bq query:*)
> 13. Bash(bun run build:*)
> 14. Bash(bun run Lint:file:*)
> 15. Bash(bun run test:x)
> 16. Bash(bun run test:file:x)
> 17. Bash(bun run typecheck:x)
> 18. Bash(bun test:*)
> 19. Bash(cc:x)
> 20. Bash(comm:*)
> 21. Bash(find:*)

## 11. MCP 集成：让 Claude 使用所有工具
Claude Code 为我使用所有的工具。它经常：
- 通过 MCP 服务器搜索和发布到 Slack
- 使用 bq CLI 运行 BigQuery 查询来回答分析问题
- 从 Sentry 获取错误日志

Slack MCP 配置已提交到我们的 `.mcp.json` 并与团队共享。

这意味着 Claude 不只是一个编程助手，它还是一个**全能的工作助手**——能帮你查数据、发消息、看日志，几乎什么都能干。

```json
// .mcp.json 示例
{
  "mcpervers": {
    "slack": {
      "type": "http",
      "url": "//slack.mcp.anthropic.com/mcp"
    }
  }
}
```

## 12. 长时间任务的处理方式
对于**非常长时间运行的任务**，我有几种处理方式：
- 后台代理验证：提示 Claude 在完成后用后台代理验证其工作
- Stop hook：使用 Stop hook 更确定性地做到这一点
- ralph-wiggum 插件：使用这个插件（最初由 @GeoffreyHuntley 构想）自动管理长任务

我还会在沙盒中使用 `--permission-mode=dontAsk` 或 `--dangerously-skip-permissions` 来避免会话中的权限提示，这样 Claude 可以不被我阻塞地持续工作。

这就是为什么我能让 Claude 连续运行几个小时甚至更长时间——它不需要等我确认每一步操作。

> 长任务运行示例：*Reticulating...(1d 2h 47m·↓2.4m tokens·thinking)*

## 13. 最重要的一条：建立验证循环
这可能是从 Claude Code 获得出色结果最重要的事情——给 Claude 一种验证其工作的方式。

如果 Claude 有这个反馈循环，**最终结果的质量将提高 2-3 倍**。

我是这样做的：Claude 使用 Claude Chrome 扩展测试我提交到 claude.ai/code 的每一个更改。它会打开浏览器，测试 UI，并迭代直到代码能工作且用户体验感觉良好。

验证在每个领域看起来都不同：
- 可能简单到运行一个 bash 命令
- 或运行测试套件
- 或在浏览器或手机模拟器中测试应用

确保投入精力使验证循环非常可靠。这是投入产出比最高的优化。

---

## 总结：核心要点速查表

| 序号 | 技巧 | 要点 |
|------|------|------|
| 1 | 并行运行 | 终端 5 个 + 网页端 5-10 个 + 手机端 |
| 2 | 多端协作 | 用 & 和 --teleport 在各端之间切换 |
| 3 | 模型选择 | 始终使用 Opus 4.5 + thinking 模式 |
| 4 | 团队协作 | 共享 CLAUDE.md，记录错误，每周更新 |
| 5 | 代码审查 | 用 @.claude 在 PR 中添加改进 |
| 6 | Plan 模式 | 几乎所有会话都从 Plan 模式开始 |
| 7 | Slash 命令 | 高频操作命令化，如 /commit-push-pr |
| 8 | 子代理 | code-simplifier、verify-app 等 |
| 9 | Hook | PostToolUse hook 自动格式化代码 |
| 10 | 权限管理 | /permissions 精细配置，不一刀切 |
| 11 | MCP 集成 | Slack、BigQuery、Sentry 等工具 |
| 12 | 长任务 | Stop hook 或 ralph-wiggum 插件 |
| 13 | 验证循环 | 最重要！质量提升 2-3 倍 |

## 快捷键速查

| 快捷键 | 功能 |
|--------|------|
| Shift+Tab × 2 | 进入 Plan 模式 |
| Shift+Tab | 切换自动接受编辑模式 |
| & | 将本地会话移交到网页端 |
| --teleport | 在本地和网页端之间切换 |
| /permissions | 配置权限 |
| /install-github-action | 安装 GitHub Action |

---

## 写在最后
希望这些分享对你有帮助！

记住：使用 Claude Code 没有唯一正确的方式。我分享的只是我的方法，你完全可以根据自己的工作习惯来调整。

最重要的是那条验证循环——无论你怎么用 Claude Code，一定要给它一种验证工作的方式，这会让结果质量提升 2-3 倍。

你有什么使用 Claude Code 的技巧？欢迎在评论区分享！

## 参考资料
- 原文推文：https://x.com/bcherny/status/2007179832300581177
- Boris Cherny X 账号：@bcherny
- Claude Code 官方文档：https://code.claude.com/docs
