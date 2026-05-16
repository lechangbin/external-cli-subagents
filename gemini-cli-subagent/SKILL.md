---
name: gemini-cli-subagent
description: Use when external-cli-subagents has selected the local Gemini CLI as an external coding subagent for a bounded implementation, analysis, review, or bug-fix task. Use for manual readiness checks, Gemini command construction, logging, output contract, and result handoff. Do not use for Gemini API calls, automated login, credential extraction, or deciding whether delegation is appropriate.
---

# Gemini CLI Subagent

## Overview

Run a manually authenticated local `gemini` CLI as an external child agent under the `external-cli-subagents` adapter contract. The main agent remains responsible for task selection, independent verification, and merge or discard decisions.

## Capability Fit

Use Gemini CLI when the selected task benefits from a complementary external perspective, especially:

- Frontend implementation, UI refinement, copy, and interaction polish.
- Visual or aesthetic review when paired with screenshots or concrete design constraints.
- Broad code reading, second-opinion review, and bounded bug fixes.
- Mechanical implementation where the acceptance checks are clear.

Prefer the main agent or another adapter for:

- Backend architecture decisions the main agent has not already framed.
- Cross-cutting refactors with unclear ownership.
- Security-sensitive code, credential handling, or production changes.
- Tasks where verification requires project-specific judgment Gemini cannot see.

Validation signals should match the task: unit tests for logic, build/lint for integration, screenshots or browser checks for frontend work, and main-agent diff review for every file-changing task.

## Manual Readiness

Before running Gemini as a child, check readiness without attempting to authenticate for the user:

```powershell
gemini --version
gemini --help
```

If Gemini reports missing authentication, stop and tell the user to run `gemini` interactively and complete the vendor's normal login flow. Do not automate browser auth, OAuth approval, token extraction, config scraping, or environment variable creation.

## Command Templates

Use the installed Gemini CLI's help output as the source of truth for exact flags. Prefer a non-interactive prompt run when the installed version supports it:

```powershell
gemini -p "<handoff prompt>"
```

If the local CLI uses positional prompts instead, use:

```powershell
gemini "<handoff prompt>"
```

For large prompts, write the prompt to a temporary text file using the host's normal editor or an approved file-write method, then pass it according to `gemini --help`. Do not embed secrets in the prompt file. Save CLI output to a log when possible:

```powershell
gemini -p "<handoff prompt>" *> ".agents/tmp/gemini-<task-id>.log"
```

Use a project-local temp/log directory only when appropriate for the repository. Avoid committing logs unless the user asks.

For headless tasks that must write files and run verification commands, Gemini CLI may need explicit, minimal tool permissions. In an isolated workdir and only for a user-approved bounded task, prefer the narrowest available policy. On Gemini CLI `0.42.0`, this worked:

```powershell
gemini --skip-trust --approval-mode auto_edit --allowed-tools write_file --allowed-tools run_shell_command -p "<handoff prompt>"
```

`--allowed-tools` is deprecated in favor of Gemini's Policy Engine, so use Policy Engine when the local installation has an equivalent narrow policy. Do not use `--yolo` by default.

Keep `--yolo` as an explicit fallback only for disposable, isolated workdirs when narrow policy/tool grants cannot make a bounded task run headlessly and the user accepts the broader tool-approval risk. In a local three-task comparison, `--yolo` completed the same tasks as minimal non-yolo grants but produced more log volume and no observable main-agent token savings.

## Handoff Prompt

Construct the Gemini prompt from the unified adapter input:

```text
You are Gemini CLI acting as an external coding subagent.

Workdir: <absolute path>

Task:
<bounded task selected by the main agent>

Allowed edits:
- <paths>

Do not edit:
- <paths>

Concurrency:
- This child owns only this workdir/log: <path>
- Do not use shared writable state outside the allowed scope.
- If another child may touch the same files, stop and report a conflict.

Acceptance checks:
- <commands or behavior>

Rules:
- Make the smallest coherent change.
- Stay inside the allowed edit scope.
- Do not perform credential, auth, or account setup.
- If blocked, stop and report what is missing.
- Run the acceptance checks if available.

Return exactly these sections:
Status: DONE | DONE_WITH_CONCERNS | NEEDS_CONTEXT | BLOCKED
Summary:
Files changed:
Verification:
Risks:
Questions:
```

## Monitoring and Intake

During the run:

- Keep the user informed when the child starts, asks for input, fails, or finishes.
- Do not kill a long-running task just because it is slow; check logs first.
- If Gemini asks for interactive auth, stop the run and hand that action to the user.
- If Gemini tries to broaden scope, terminate or redirect with a narrower prompt.

After the run:

1. Read Gemini's status and summary.
2. Inspect the filesystem and diff yourself.
3. Run the relevant tests or reproduction command.
4. Apply the `external-cli-subagents` result triage table.

## Concurrent Runs

Gemini CLI can run multiple headless tasks concurrently when each task has:

- Its own working directory or non-overlapping write scope.
- Its own log file.
- A bounded prompt and output contract.

On Windows, avoid using PowerShell `Start-Job` for Gemini tasks that invoke terminal-backed tools; Gemini CLI `0.42.0` produced `node-pty AttachConsole failed` in background jobs. Use separate shell invocations, terminal sessions, or the host agent's parallel command facility instead.

Capacity retry messages such as "quota will reset after ..." can appear under concurrent load. Treat them as signal to inspect logs and elapsed time, not as proof the task failed.

Keep this section in sync with `external-cli-subagents/references/adapter-contract.md`. Future CLI adapters must include an equivalent Concurrency and Isolation section, even when the correct answer is "parallel unsupported."

## Failure Handling

| Symptom | Response |
| --- | --- |
| `gemini` not found | Tell the user to install Gemini CLI, then retry after it is on `PATH` |
| Authentication required | Ask the user to run `gemini` interactively; do not automate |
| `write_file` or `run_shell_command` unavailable | Use a narrow user-approved policy or minimal tool grants in an isolated workdir; never default to `--yolo` |
| Minimal policy cannot run a disposable task | Consider `--yolo` only after noting the risk and confirming isolated write scope |
| `AttachConsole failed` on Windows | Avoid PowerShell background jobs; launch separate shell invocations or terminal sessions |
| Prompt too large | Reduce context, pass specific files, or ask Gemini for analysis only |
| Edits outside scope | Discard or revert child work in the isolated work area |
| Weak or wrong implementation | Do not polish blindly; either re-prompt with targeted feedback or discard |
| Ambiguous output | Ask for a structured re-summary or inspect the diff directly |

## Extension Notes

This adapter implements the `external-cli-subagents` contract for Gemini only. When adding another CLI vendor, create a new sibling skill and keep vendor-specific flags, failure modes, and readiness checks there.
