# upgrade-audit

升级审计：让 Agent 定期把对话里的知识沉淀进文档体系。

## 这是什么 / 解决什么问题

人与 Agent 协作时，很多真正有价值的知识会散落在对话里：偏好、踩坑、架构选择、可复用流程、反复出现的纠正。与此同时，已有文档会变旧、变长、互相重复，最后反过来拖累 Agent。upgrade-audit 提供一套可执行审计流程，帮助你的 Agent 定期扫描对话和文档，把知识放到正确层级。

这个包包含四件套：分层记忆模板、审计流程、文档工程参考、全球顶尖从业者经验附录。

## 核心功能/亮点

- 按完整审计或定向审查两种模式工作。
- 把每条建议归位到 always-read、on-demand 或 task-specific。
- 输出覆盖清单、归位判断、新 skill 候选和可执行修改方案。
- 提供六段式记忆主文档模板，方便从零建立分层文档体系。
- 附带 Agent 文档工程参考和公开经验附录，可作为审计判断材料。
- 支持本地水位、无人值守报告、定时自动化和多机 git 同步的通用设计思路。

## 安装

Claude Code：

```bash
git clone https://github.com/ruodou233/upgrade-audit.git ~/.claude/skills/upgrade-audit
```

Codex：

```bash
git clone https://github.com/ruodou233/upgrade-audit.git ~/.agents/skills/upgrade-audit
```

其他支持 `SKILL.md` 的平台：放入其 skills 目录即可。

## 使用示例

- "做一次每日审计，看看最近对话里有什么该沉淀。"
  - Agent 会读取本地配置和水位，清点可审计材料，生成覆盖清单与审计报告。
- "检查这个 skill 是否过时，给我修改方案。"
  - Agent 会做定向审查，重点找触发边界、重复内容、过时规则和缺失验证。
- "扫描最近几次对话，看看有没有新 skill 候选。"
  - Agent 会提取稳定触发词、可复用步骤、输入输出和置信度。

## 首次使用：环境自适应

首次使用时，让 Agent 先只读探测已安装的平台和对话记录路径，例如 `~/.claude/projects`、`~/.codex/sessions` 或其他导出目录。写配置、扫描对话记录、接入全局记忆入口、安装定时任务前，Agent 必须说明将做什么、写到哪，并获得明确同意。

本地配置优先写入 `~/.config/agentops-skills/upgrade-audit/local-config.md`；无法写入时可退回 skill 目录内 `local-config.md`。格式见 `local-config.example.md`。

## Changelog

| 时间 | 变更 |
|---|---|
| 2026-07 | 首次开源发布 |

## 反馈与作者

这个 skill 我长期维护。如果你有修改方案、发现问题、或者改出了更好的版本，欢迎通过以下任一渠道找到我：

- GitHub：本仓库提 issue 或 PR
- 小红书：错误乱码
- 微信公众号：能工智人错误乱码
- B站：若逗道人

## 相关 Skill 推荐

<!-- 本表由维护脚本生成，勿手工编辑 -->
- [agent-orchestration](https://github.com/ruodou233/agent-orchestration)：长任务治理：主代理指挥、子代理干活、状态落盘、断点续跑
- [cross-review](https://github.com/ruodou233/cross-review)：跨厂商双审：让另一家公司的最强模型独立审你的方案
- [claude-cache-keepalive](https://github.com/ruodou233/claude-cache-keepalive)：Claude 缓存保温：实测 TTL、按环境设计保温节拍，控制冷读成本

完整目录见 [GitHub 主页](https://github.com/ruodou233)。
