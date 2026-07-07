# upgrade-audit local-config.example.md

> 复制为 `~/.config/agentops-skills/upgrade-audit/local-config.md` 或 skill 目录内 `local-config.md` 后按你的环境填写。
> 写配置、读取/扫描对话记录、修改全局记忆入口、安装定时任务前，Agent 必须先获得你的明确同意。

## 文档体系

- docs_root: `~/Documents/AgentOps`
- memory_doc: `~/Documents/AgentOps/memory.md`
- skills_dir: `~/Documents/AgentOps/skills`
- references_dir: `~/Documents/AgentOps/references`
- templates_dir: `~/Documents/AgentOps/templates`

## 对话记录

- conversation_paths:
  - `~/.claude/projects`
  - `~/.codex/sessions`
- include_exports_dir:
  - `~/Documents/AgentOps/conversation-exports`

## 水位与报告

- watermark_file: `~/Documents/AgentOps/state/upgrade-audit-watermark.txt`
- report_dir: `~/Documents/AgentOps/reports/upgrade-audit`
- advance_watermark_only_after_success: true

## 运行模式

- default_mode: `report-only`
- unattended_mode_changes_docs: false
- require_confirmation_before_doc_edits: true
- require_confirmation_before_scanning_usage_data: true

## 多机同步

- git_sync_enabled: false
- pull_before_audit: true
- push_after_approved_changes: true
- repo_remote: `<your-git-remote>`

## 开源维护可选项

- scan_open_source_issues_prs: false
- repositories:
  - `<owner>/<repo>`

## 定时自动化

- schedule_enabled: false
- scheduler: `<launchd|cron|systemd|windows-task-scheduler|platform-routine>`
- schedule_expression: `<fill-after-confirmation>`
- idempotency_window: `1 day`
