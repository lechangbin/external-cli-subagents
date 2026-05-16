# Repository Information / 仓库信息

Use this file as the source of truth when configuring GitHub repository metadata.

配置 GitHub 仓库信息时，可以直接复制本文件中的 description、topics 和 summary。

## Description / 仓库描述

Agent skills for delegating bounded coding tasks to manually authenticated external CLI subagents, starting with Gemini CLI.

中文：

将已手动鉴权的外部 CLI 编码智能体暴露为受控子代理的 Agent Skills，当前支持 Gemini CLI。

## Summary / 概要提示

English:

External CLI subagent skills for bounded, manually authenticated coding delegation. Vendor-neutral entrypoint plus Gemini CLI adapter, with strength profiles, capability-fit routing, isolation, concurrency, logging, and main-agent verification rules.

中文：

用于外派受限编码任务的外部 CLI 子代理 Skills。提供厂商无关主入口和 Gemini CLI 适配器，包含擅长任务描述，支持按能力适配路由任务，并明确隔离、并发、日志、手动鉴权与主代理验收边界。

## Topics / 主题

- agent-skills
- codex
- gemini-cli
- gemini
- subagents
- cli-agents
- external-tools
- agent-orchestration
- capability-routing

## Skill UI Metadata / Skill 展示信息

`external-cli-subagents`

- Display name: 外部 CLI 子代理
- Short description: 将外部编码 CLI 暴露为手动鉴权的受控子代理。
- Default prompt: 使用 $external-cli-subagents 运行一个手动鉴权的外部 CLI 子代理流程。

`gemini-cli-subagent`

- Display name: Gemini CLI 子代理
- Short description: 通过本地 Gemini CLI 外派受限编码任务。
- Default prompt: 使用 $gemini-cli-subagent 将一个边界清晰的任务外派给 Gemini CLI。

## Homepage / 主页

No homepage is required.

暂无需设置 homepage。

## Default Branch / 默认分支

`main`
