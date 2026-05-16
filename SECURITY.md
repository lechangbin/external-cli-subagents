# Security Policy

[中文安全说明](SECURITY.zh-CN.md)

This repository intentionally avoids automatic credential handling.

## Supported Boundary

The skills may instruct an agent to:

- Check that a local CLI exists.
- Run a bounded, user-approved CLI task.
- Ask the user to complete vendor authentication manually.
- Use isolated workdirs, logs, branches, or worktrees.

The skills must not instruct an agent to:

- Automate browser login or OAuth approval.
- Scrape tokens, cookies, cloud credentials, or CLI config secrets.
- Export provider API keys.
- Use broad tool approval by default.
- Merge child-agent work without independent verification.

## Reporting Concerns

Open a GitHub issue for unsafe instructions, overly broad defaults, or adapter behavior that breaks the manual-authentication boundary.
