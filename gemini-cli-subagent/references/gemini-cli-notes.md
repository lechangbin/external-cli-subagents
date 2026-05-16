# Gemini CLI Notes

Use this reference only when the main `SKILL.md` is not enough.

## Manual Authentication Boundary

Allowed:

- Check `gemini --version`.
- Check `gemini --help`.
- Run the user-approved `gemini` task command.
- Use narrow, task-specific file edit or command permissions inside an isolated workdir when the user has requested the CLI to complete and verify a bounded task.
- Ask the user to authenticate manually if the CLI requires it.

Not allowed:

- Open browser login flows automatically.
- Scrape tokens from Gemini, browser, cloud, or shell config files.
- Export provider API keys.
- Convert this workflow into direct Gemini API calls.
- Use `--yolo` as the default way to make headless tasks succeed.

## YOLO Mode

Retain `--yolo` as an explicit fallback, not the standard path.

Use it only when all are true:

- The task is bounded and low risk.
- The workdir is disposable or otherwise isolated.
- The allowed write scope is non-overlapping.
- Narrow Policy Engine or minimal tool grants cannot complete the task headlessly.
- The user accepts broader automatic tool approval.

Do not choose `--yolo` just to save main-agent tokens. In a three-task local comparison, minimal non-yolo grants completed all tasks with less log volume than `--yolo`.

## Prompt Sizing

Prefer targeted context over whole-repository prompts:

- Include exact files or paths relevant to the delegated task.
- Give acceptance checks instead of broad intent.
- Ask for `NEEDS_CONTEXT` when the child cannot proceed safely.

## Good Gemini Tasks

- Large-context code reading with a narrow question.
- Independent bug fixes in a disposable worktree.
- Second-opinion review of a diff or plan.
- Boilerplate or mechanical edits with precise acceptance checks.

## Concurrency Notes

Observed on Windows with Gemini CLI `0.42.0`:

- Three separate `gemini --skip-trust --approval-mode auto_edit --allowed-tools write_file --allowed-tools run_shell_command -p ...` invocations completed concurrently in separate directories.
- Default headless mode started but lacked `write_file` and `run_shell_command`, so it could not reliably complete file-writing tasks.
- PowerShell `Start-Job` caused `node-pty AttachConsole failed`; separate shell invocations worked.
- Concurrent runs may trigger model-capacity retry messages before eventually completing.

## Poor Gemini Tasks

- Architecture choices the main agent has not made.
- Tasks requiring hidden credentials or account setup.
- Live production changes.
- Broad refactors without a written scope.
