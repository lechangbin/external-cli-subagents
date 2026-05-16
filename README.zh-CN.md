# external-cli-subagents

[English](README.md)

将已手动鉴权的外部 CLI 编码智能体暴露为受控子代理的 Agent Skills。

主 skill `external-cli-subagents` 是厂商无关入口。不同厂商 CLI 通过独立 adapter skill 接入，并共享同一套任务外派、日志、并发、隔离和主代理验收契约。当前已实现的 adapter 是面向本地 `gemini` 命令的 `gemini-cli-subagent`。

除了节省主代理上下文成本，本仓库也支持“能力互补路由”：把边界清晰的任务分配给更适合的 CLI/模型。例如主 Codex 会话保留后端架构和集成判断，把前端润色、视觉审美或 UI 复核外派给 Gemini CLI，然后由主代理独立验收结果。

## Skills

| Skill | 用途 |
| --- | --- |
| `external-cli-subagents` | 共享入口：选择 CLI adapter、准备隔离工作区、外派边界清晰的任务、接收结果并决定验收/返工/丢弃 |
| `gemini-cli-subagent` | Gemini CLI adapter：手动就绪检查、命令模板、并发说明、日志约定和失败处理 |

## 使用场景

- 把大型、独立、可验证任务外派出去，减少高级主代理的上下文占用。
- 让低成本主代理把困难编码任务委托给更强的外部 CLI。
- 按能力适配路由任务，例如 Codex 负责后端架构和集成判断，Gemini CLI 负责前端/UI 实现、文案、视觉润色或审美复核。

## 安装

安装共享入口和需要的 adapter：

```bash
npx skills add lechangbin/external-cli-subagents --skill external-cli-subagents
npx skills add lechangbin/external-cli-subagents --skill gemini-cli-subagent
```

如果你的 agent 环境不支持 `npx skills`，可以把这两个 skill 目录复制到当前生效的 skills 目录。例如 Codex 项目级目录通常是 `.agents/skills/`。

## 设计边界

这些 skills **不负责判断什么时候应该外派任务**。是否调用子代理，应由当前主代理或已有的规划/子代理 skills 决定。

这些 skills 只负责在“已经决定外派”之后，把外部 CLI 编码智能体以受控方式暴露给主代理使用。

这些 skills 也**不自动化鉴权**。用户应自行安装对应 CLI，并通过厂商正常交互流程手动登录。

## 安全模型

- 不自动化浏览器登录、OAuth 授权、token 提取、凭据导出或 API key 搜索。
- 文件修改任务优先使用一次性 worktree、临时目录或明确隔离的分支。
- 写入范围必须明确，尤其是并发子代理运行时必须避免重叠写入。
- 不因为子代理自称成功就合并或接受结果。
- 主代理必须检查 diff/文件结果，并重新运行验证命令后再接受工作。
- 不能让“某模型更擅长某类任务”的经验替代测试、截图、diff 或代码审查证据。

## Gemini CLI 说明

Gemini CLI 可以并发运行多个边界清晰的 headless 任务，但每个任务必须有独立工作目录或不重叠的写入范围，并且使用独立日志。

严格默认的 headless 模式可能没有 `write_file` 和 `run_shell_command` 等工具权限。推荐使用 Gemini Policy Engine 或最小任务级授权。`--yolo` 只应作为显式 fallback：仅在一次性隔离目录中、窄授权无法运行、且用户接受更宽自动审批风险时使用。

## 仓库设置建议

GitHub description：

> 将已手动鉴权的外部 CLI 编码智能体暴露为受控子代理的 Agent Skills，当前支持 Gemini CLI。

GitHub topics：

```text
agent-skills
codex
gemini-cli
gemini
subagents
cli-agents
external-tools
agent-orchestration
```

概要提示：

```text
用于外派受限编码任务的外部 CLI 子代理 Skills。提供厂商无关主入口和 Gemini CLI 适配器，支持按能力适配路由任务，并明确隔离、并发、日志、手动鉴权与主代理验收边界。
```

## 验证

两个 skill 都已通过 Codex skill validator：

```bash
python quick_validate.py external-cli-subagents
python quick_validate.py gemini-cli-subagent
```
