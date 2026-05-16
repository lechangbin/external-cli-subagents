# Adapter Contract

Use this contract when adding a new external CLI vendor adapter skill.

## Required Skill Shape

Adapter skill name: `<vendor>-cli-subagent`

Frontmatter description must say when to use the adapter and must include:

- The local CLI command name.
- Capability-fit signals for routing tasks to this adapter.
- Manual authentication requirement.
- That API calls and credential automation are out of scope.

## Required Sections

1. Overview
2. Capability Fit
3. Manual Readiness
4. Command Templates
5. Handoff Prompt
6. Concurrency and Isolation
7. Output Contract
8. Monitoring and Logs
9. Failure Handling
10. Extension Notes

## Capability Fit

Each adapter must declare where it tends to be useful and where it should not be trusted without extra care.

Include:

| Field | Meaning |
| --- | --- |
| `best_fit` | Task shapes this CLI/model is especially useful for |
| `weak_fit` | Task shapes that should stay with the main agent or another adapter |
| `validation_signals` | Tests, screenshots, diffs, benchmarks, or review checks needed before accepting work |
| `routing_notes` | Comparative notes such as "stronger for frontend polish" or "better for backend architecture" |

These notes help the main agent route by capability complementarity, but they do not decide routing on their own. The main agent or existing delegation workflow still chooses the child.

## Unified Input

Adapters accept this conceptual input from `external-cli-subagents`:

| Field | Meaning |
| --- | --- |
| `workdir` | Absolute directory where the child CLI runs |
| `task` | Bounded work item chosen by another workflow |
| `allowed_paths` | Files or folders the child may edit |
| `forbidden_paths` | Files or folders the child must not edit |
| `verification` | Commands or manual checks the child should attempt |
| `output_log` | Path where output should be saved when practical |
| `concurrency` | Whether this child may run in parallel and what isolation is required |
| `capability_fit` | Why this adapter is suitable for the selected task, if relevant |

## Handoff Template Requirements

Every adapter's child prompt template must include a concurrency section unless the adapter is strictly read-only and does not launch long-running work. If omitted for that reason, explain the omission in the adapter's Concurrency and Isolation section.

```text
Concurrency:
- This child owns only this workdir/log: <path>
- Do not use shared writable state outside the allowed scope.
- If another child may touch the same files, stop and report a conflict.
```

## Concurrency and Isolation

Each adapter must state one of:

- Parallel supported: describe required workdir, log, session, branch, or tool-policy isolation.
- Parallel limited: describe safe read-only or single-writer modes.
- Parallel unsupported: explain the limitation and the required sequential fallback.

Do not treat successful process startup as proof of safe concurrency. A safe parallel run requires non-overlapping writes, separate logs, independent sessions when the CLI supports sessions, and main-agent verification after completion.

## Unified Output

Adapters must ask the child CLI to return:

- Status: `DONE`, `DONE_WITH_CONCERNS`, `NEEDS_CONTEXT`, or `BLOCKED`.
- Summary.
- Files changed.
- Verification attempted.
- Risks and open questions.

The main agent still verifies independently.

## Safety Requirements

- Do not automate browser login, OAuth approval, token extraction, credential export, or key discovery.
- Do not add flags that bypass permissions unless the user explicitly asks and understands the risk.
- Do not run in the main worktree when a disposable worktree or branch is available.
- Do not run concurrent file-editing tasks in the same writable scope.
- Do not merge child work based only on a success message.
- Do not let capability-fit notes override evidence from tests, screenshots, diffs, or project conventions.

## Adding a Vendor

1. Create a sibling skill folder.
2. Follow the required sections above.
3. Add the adapter to `external-cli-subagents` under Available Adapters.
4. Validate both skill folders.
