---
name: external-cli-subagents
description: Use when Codex needs to expose an already-installed external coding CLI, such as Gemini CLI, as a manually authenticated external subagent after another delegation workflow has selected a task. Use for vendor adapter selection, handoff execution, result intake, verification, merge, retry, discard, or adding new CLI adapter skills. Do not use for deciding whether to delegate or for API-based automation.
---

# External CLI Subagents

## Overview

Expose external coding CLIs as controlled subagents without automating credentials or replacing the main agent's judgment. This skill is the vendor-neutral entrypoint; vendor-specific commands live in separate adapter skills such as `gemini-cli-subagent`.

## Scope Boundary

This skill only handles external CLI subagent exposure after the main agent or another subagent workflow has already decided to delegate.

It must not:

- Decide whether subagents are needed.
- Replace current in-process subagent skills for task decomposition, model choice, or parallelism.
- Automate login, browser OAuth, token extraction, credential export, quota bypass, or API calls.
- Silently merge or trust an external agent's report.

## Available Adapters

Adapter skills implement the same contract and can be installed independently.

| Vendor | Skill | Status | Use for |
| --- | --- | --- | --- |
| Gemini CLI | `gemini-cli-subagent` | implemented | Bounded coding, review, analysis, and fix tasks through the local `gemini` command |

When the user asks to install or set up external CLI subagent support, show the adapter list and ask which CLI adapter skills to install. Multiple adapters may be selected. Current selection choices:

- `gemini-cli-subagent`

## Setup Prompt

When installation or setup is command-driven, do not silently install only the entrypoint. Present this selection first:

```text
External CLI subagent support has a shared entrypoint plus vendor adapters.
Select one or more CLI adapter skills to install:

[ ] gemini-cli-subagent - Gemini CLI adapter for the local `gemini` command
```

After the user selects adapters, install or copy:

- `external-cli-subagents`
- Each selected adapter skill

If this collection is later packaged in a remote skills repository, use the repository's normal `npx skills add <repo> --skill <skill-name>` command for the entrypoint and selected adapters. For local-only use, place the folders side by side under the active `.agents/skills` or user skills directory.

## Workflow

1. Confirm delegation has already been selected by another workflow. If the user is still asking whether to delegate, use the normal planning/subagent skills first.
2. Select an adapter skill by vendor. For Gemini, use `gemini-cli-subagent`.
3. Require manual CLI readiness:
   - The CLI executable is installed and on `PATH`.
   - The user has already completed any required interactive authentication.
   - The working tree or worktree is suitable for external edits.
4. Create or choose an isolated work area before file-modifying work. Prefer a git worktree or a clearly named branch. If no git repository exists, use a copied scratch directory and make the limitation explicit.
5. Build a bounded handoff prompt:
   - Goal and acceptance criteria.
   - Exact write scope and paths.
   - Concurrency boundary: assigned workdir, unique log path, and non-overlapping edits.
   - Commands the child should run.
   - Required output contract.
   - Instruction to stop and report if blocked.
6. Run the adapter's command and capture output to a log file when possible.
7. Inspect the child result yourself:
   - Read its summary.
   - Inspect changed files or generated output.
   - Run the relevant tests, build, lint, or reproduction command.
8. Decide integration:
   - Accept and merge only after fresh verification.
   - For small correctable issues, send targeted feedback to the same external CLI child or fix locally if that is safer.
   - For poor direction, broad wrong assumptions, unsafe edits, or low-quality structure, discard the child work and implement locally or delegate to a different child, preferably the same model family as the main agent.

## Handoff Prompt Shape

Use this shape for every adapter:

```text
You are an external CLI coding subagent working in: <absolute path>

Task:
<bounded task>

Scope:
- You may edit: <paths>
- Do not edit: <paths>

Concurrency:
- This child owns only this workdir/log: <path>
- Do not use shared writable state outside the allowed scope.
- If another child may touch the same files, stop and report a conflict.

Acceptance:
- <tests or behavior>

Process:
- Make the smallest coherent change.
- Run the listed verification commands if available.
- If blocked, stop and explain the blocker instead of guessing.

Return:
- Summary of changes.
- Files changed.
- Verification commands run and results.
- Remaining risks or questions.
```

## Result Triage

| Result | Main agent action |
| --- | --- |
| Correct, minimal, verified | Accept and merge through normal project workflow |
| Mostly correct with small bugs | Provide concise fix feedback or patch locally after verifying |
| Correct idea, messy implementation | Ask child for focused cleanup or reimplement locally from the learned approach |
| Wrong architecture, wrong scope, unsafe edits | Discard child changes; do not spend time polishing |
| Blocked on missing context | Provide the missing context and retry once |
| Repeatedly blocked or low quality | Re-delegate to a different adapter or use the main agent |

## Adapter Contract

Each vendor adapter skill must provide:

- Trigger conditions and non-goals.
- Manual readiness checks.
- Command templates for foreground and, when safe, background execution.
- Concurrency and isolation requirements, including whether parallel runs are supported.
- Prompt and output contract compatibility with this skill.
- Logging guidance.
- Known failure modes and recovery.
- A statement that it does not automate authentication or credential extraction.

For detailed extension rules, read `references/adapter-contract.md` before adding a new vendor adapter.
