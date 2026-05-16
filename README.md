# external-cli-subagents

[中文文档](README.zh-CN.md)

Agent skills for exposing manually authenticated external coding CLIs as controlled subagents.

The main skill, `external-cli-subagents`, is a vendor-neutral entrypoint. Vendor-specific adapters live beside it and share the same handoff, logging, concurrency, and verification contract. The first implemented adapter is `gemini-cli-subagent` for the local `gemini` command.

Beyond cost control, this package supports capability-complement routing: assign bounded work to the CLI/model that fits it best. For example, a main Codex session may keep backend architecture and integration decisions while delegating frontend polish or visual review to Gemini CLI, then independently verify the result.

## Skills

| Skill | Purpose |
| --- | --- |
| `external-cli-subagents` | Shared workflow for selecting a CLI adapter, preparing isolated work, handing off a bounded task, and triaging results |
| `gemini-cli-subagent` | Gemini CLI adapter with manual readiness checks, command templates, concurrency notes, and failure handling |

Each adapter includes a `Strength Profile / 擅长任务描述` section so the main agent can compare child agents before assigning work.

## Use Cases

- Reduce high-end main-agent context use for large, independent, verifiable tasks.
- Let a lower-cost main agent delegate difficult coding work to stronger external CLIs.
- Route work by strength profile and capability fit, such as keeping backend architecture with Codex while using Gemini CLI for frontend/UI-oriented implementation and aesthetic review.

## Install

Install the shared entrypoint plus the adapter you need:

```bash
npx skills add lechangbin/external-cli-subagents --skill external-cli-subagents
npx skills add lechangbin/external-cli-subagents --skill gemini-cli-subagent
```

If your agent environment does not support `npx skills`, copy the two skill folders into the active skills directory, for example `.agents/skills/` for Codex project-local use.

## Design Boundary

These skills do not decide when a task should be delegated. The current main agent, or its existing subagent/planning skills, should decide that first. This package only exposes external CLI coding agents after delegation has already been selected.

These skills also do not automate authentication. Users should install and log in to each vendor CLI manually through the vendor's normal flow.

## Safety Model

- Do not automate browser login, OAuth approval, token extraction, credential export, or API-key discovery.
- Prefer disposable worktrees, scratch directories, or clearly isolated branches for file-changing tasks.
- Keep write scopes explicit and non-overlapping, especially for concurrent child runs.
- Do not merge or accept child work based only on the child agent's success message.
- The main agent must inspect results and run fresh verification before accepting work.
- Do not let capability-fit assumptions replace project tests, screenshots, diffs, or code review.

## Gemini CLI Notes

Gemini CLI can run multiple bounded headless tasks concurrently when each task has its own workdir or non-overlapping write scope and its own log file.

For file-writing tasks, strict default headless mode may lack tools such as `write_file` and `run_shell_command`. Prefer narrow Policy Engine rules or minimal task-specific grants. Keep `--yolo` as an explicit fallback only for disposable, isolated workdirs when narrow grants cannot complete the task and the user accepts the broader tool-approval risk.

## Repository Metadata

Suggested GitHub description:

> Agent skills for delegating bounded coding tasks to manually authenticated external CLI subagents, starting with Gemini CLI.

Suggested Chinese description:

> 将已手动鉴权的外部 CLI 编码智能体暴露为受控子代理的 Agent Skills，当前支持 Gemini CLI。

Suggested topics:

```text
agent-skills
codex
gemini-cli
subagents
cli-agents
external-tools
agent-orchestration
gemini
```

Suggested repository social/about summary:

```text
External CLI subagent skills for bounded, manually authenticated coding delegation. Vendor-neutral entrypoint plus Gemini CLI adapter, with capability-fit routing, isolation, concurrency, logging, and main-agent verification rules.
```

## Validation

Both bundled skills were validated with the Codex skill validator:

```bash
python quick_validate.py external-cli-subagents
python quick_validate.py gemini-cli-subagent
```
