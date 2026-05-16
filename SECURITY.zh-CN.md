# 安全策略

[English](SECURITY.md)

本仓库刻意避免任何自动化凭据处理。

## 支持的边界

这些 skills 可以指导 agent：

- 检查本地 CLI 是否存在。
- 运行用户批准的、边界清晰的 CLI 任务。
- 在 CLI 需要鉴权时，请用户手动完成厂商正常登录流程。
- 使用隔离工作目录、日志、分支或 worktree。

这些 skills 不应指导 agent：

- 自动化浏览器登录或 OAuth 授权。
- 抓取 token、cookie、云凭据或 CLI 配置秘密。
- 导出厂商 API key。
- 默认使用宽泛工具审批。
- 在没有主代理独立验证的情况下合并子代理结果。

## 报告问题

如果发现不安全指令、过宽默认值、或 adapter 行为破坏了“手动鉴权”边界，请打开 GitHub issue。
